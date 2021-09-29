Http는 Stateless로 무상태 프로토콜이다.

그런데 우리가 웹 페이지를 이용할 때를 생각해보면 로그인을 한번 하고 나면 계속해서 로그인 상태가 유지된다.

이렇게 로그인 상태를 계속 유지하기 위해서 매 Http 요청마다 쿼리 파라미터를 계속 유지해서 보낼 수도 있지만, 이렇게 되면 매우 번거롭고 불필요한 반복 동작을 하게 된다.

그래서 우리는 쿠키를 사용해보자

<br><br>

# 로그인 처리하기 - 쿠키 사용
**쿠키에는 영속 쿠키와 세션 쿠키가 있다**
* 영속 쿠키 : 만료 날짜를 입력하면 해당 날짜까지만 쿠키를 유지
* 세션 쿠키 : 만료 날짜를 생략하면 브라우저 종료시까지만 유지

브라우저 종료시 로그아웃이 되길 기대하므로, 우리에게 필요한건 세션 쿠키이다.

<br>

**LoginController - login()**  
로그인 성공시 세션 쿠키를 생성하자.
```Java
@PostMapping("/login")
public String login(HttpServletResponse response) {
    
    //로그인 성공 처리
    {...}

    //쿠키에 시간 정보를 주지 않으면 세션 쿠키(브라우저 종료시 모두 종료)
    Cookie idCookie = new Cookie("memberId", "1");
    response.addCookie(idCookie);

    return "redirect:/";
}
```

<br>

**쿠키 생성 로직**
```Java
Cookie idCookie = new Cookie("memberId", "1");
response.addCookie(idCookie);
```
로그인 성공하면 쿠키를 생성하고 HttpServletResponse에 담는다. 쿠키 이름은 "memberId"이고 값은 회원의 id를 담는데 여기서는 편의상 "1"을 담았다. 그러면 이제 웹 브라우저는 종료전까지 회원의 id인 "1"의 값을 서버에 계속 보내줄 것이다.

<br><br>

로그인을 해서 쿠키를 생성했다면 이제 쿠키를 어떻게 사용하는지를 살펴보자.

**홈 - 로그인 처리**
```Java
@GetMapping("/")
public String homeLogin(@CookieValue(name = "memberId", required = false) Long memberId) {

    if (memberId == null) {
        return "home";
    }

    //로그인
    Member loginMember = memberRepository.findById(memberId);
    if (loginMember == null) {
        return "home";
    }

    return "loginHome";
}
```
* `@CookieValue`를 사용하면 편리하게 쿠키를 조회할 수 있다.
* 로그인하지 않은 사용자도 접근할 수 있으므로 `required=false`를 사용한다.
  * false값이면 쿠키에서 "memberId"라는 쿠키값이 없으면 그냥 null값을 반환한다.
  * true라면 "memberId"라는 쿠키값이 필수로 있어야 한다.

<br>

# 로그아웃 기능
이제 로그아웃 기능을 만들어 보자. 로그아웃 방법은 다음과 같다.
* 세션 쿠키이므로 웹 브라우저 종료시 쿠키 삭제
* 서버에서 해당 쿠키의 종료 날짜를 0으로 지정

**LoginController - logout 기능**
```JAVA
@PostMapping("/logout")
public String logout(HttpServletResponse response) {
    Cookie cookie = new Cookie("memberId", null);
    cookie.setMaxAge(0);
    response.addCookie(cookie);
    return "redirect:/";
}
```
쿠키 `Max-age=0` 으로 설정해 해당 쿠키를 즉시 종료시킨다.

<br>

# 쿠키의 보안 문제
**보안 문제**
- 쿠키 값은 임의로 변경할 수 있다.
    - 클라이언트가 쿠키를 강제로 변경하면 다른 사용자가 된다.
    - 실제 웹 브라우저 개발자 모드 → Application → Cookie 변경으로 확인 가능하다.
    - Cookie: memberId=1 → Cookie: memberId=2 (다른 사용자의 이름이 보임)
- 쿠키에 보관된 정보는 훔쳐갈 수 있다.
    - 만약 쿠키에 개인정보나, 신용카드 정보가 있다면?
    - 이 정보가 웹 브라우저에도 보관되고, 네트워크 요청마다 계속 클라이언트에서 서버로 전달된다.
    - 쿠키의 정보가 나의 로컬 PC가 털릴 수도 있고, 네트워크 전송 구간에서 털릴 수도 있다.
- 해커가 쿠키를 한번 훔쳐가면 평생 사용할 수 있다.
    - 해커가 쿠키를 훔쳐가서 그 쿠키로 악의적인 요청을 계속 시도할 수 있다.

<br>

**대안**
- 쿠키에 중요한 값을 노출하지 않고, 사용자 별로 예측 불가능한 임의의 토큰(랜덤 값)을 노출하고, 서버에서 토큰과 사용자 id를 매핑해서 인식한다. 그리고 서버에서 토큰을 관리한다.
- 토큰은 해커가 임의의 값을 넣어도 찾을 수 없도록 예상 불가능 해야 한다.
- 해커가 토큰을 털어가도 시간이 지나면 사용할 수 없도록 서버에서 해당 토큰의 만료시간을 짧게(예: 30분)유지한다.또는 해킹이 의심되는 경우 서버에서 해당 토큰을 강제로 제거하면 된다.


# 로그인 처리하기 - 세션 직접 만들기
앞서 쿠키의 보안 문제에 의해서 중요한 정보는 모두 서버에 저장하는것이 좋다. 그리고 클라이언트와 서버는 추정 불가능한 임의의 식별자 값으로 연결해야 한다.

이렇게 서버에 중요한 정보를 보관하고 연결을 유지하는 방법을 세션이라 한다.

**클라이언트와 서버는 결국 쿠키로 연결이 되어야 한다.**

- 서버는 클라이언트에 `mySessionId` 라는 이름으로 세션ID 만 쿠키에 담아서 전달한다.
- 클라이언트는 쿠키 저장소에 `mySessionId` 쿠키를 보관한다.

<br>

세션 관리는 크게 3가지 기능을 제공하면 된다.
* 세션 생성
* 세션 조회
* 세션 만료

**SessionManager - 세션 관리**
```Java
/**
 * 세션 관리
 */
@Component
public class SessionManager {

    public static final String SESSION_COOKIE_NAME = "mySessionId";
    private Map<String, Object> sessionStore = new ConcurrentHashMap<>();

    /**
     * 세션 생성
     * * sessionId 생성 (임의의 추정 불가능한 랜덤 값)
     * * 세션 저장소에 sessionId와 보관할 값 저장
     * * sessionId로 응답 쿠키를 생성해서 클라이언트에 전달
     */
    public void createSession(Object value, HttpServletResponse response) {

        //세션 id를 생성하고, 값을 세션에 저장
        String sessionId = UUID.randomUUID().toString();
        sessionStore.put(sessionId, value);

        //쿠키 생성
        Cookie mySessionCookie = new Cookie(SESSION_COOKIE_NAME, sessionId);
        response.addCookie(mySessionCookie);
    }

    /**
     * 세션 조회
     */
    public Object getSession(HttpServletRequest request) {
        Cookie sessionCookie = findCookie(request, SESSION_COOKIE_NAME);
        if (sessionCookie == null) {
            return null;
        }
        return sessionStore.get(sessionCookie.getValue());
    }

    /**
     * 세션 만료
     */
    public void expire(HttpServletRequest request) {
        Cookie sessionCookie = findCookie(request, SESSION_COOKIE_NAME);
        if (sessionCookie != null) {
            sessionStore.remove(sessionCookie.getValue());
        }
    }

    public Cookie findCookie(HttpServletRequest request, String cookieName) {
        if (request.getCookies() == null) {
            return null;
        }

        return Arrays.stream(request.getCookies())
                .filter(cookie -> cookie.getName().equals(cookieName))
                .findAny()
                .orElse(null);
    }

}
```

<br>

# 로그인 처리하기 - 직접 만든 세션 적용
**LoginController - login** - 로그인 처리
```Java
@PostMapping("/login")
public String login( HttpServletResponse response) {

    //로그인 성공 처리

    //세션 관리자를 통해 세션을 생성하고, 회원 데이터 보관
    sessionManager.createSession("1", response);

    return "redirect:/";
}
```

<br>

**LoginController - logout** - 로그아웃 처리
```Java
@PostMapping("/logout")
public String logout(HttpServletRequest request) {
    sessionManager.expire(request);
    return "redirect:/";
}
```

<br>

**홈 로그인처리**
```Java
@GetMapping("/")
public String homeLogin(HttpServletRequest request) {

    /세션 관리자에 저장된 회원 정보 조회
    Member member = (Member) sessionManager.getSession(request);

    //로그인
    if (member == null) {
        return "home";
    }

    return "loginHome";
}
```

<br>

# 로그인 처리하기 - 서블릿 세션
서블릿은 세션을 위해 HttpSession이라는 기능을 제공해준다.
HttpSession기능을 사용해보자.

HttpSession도 우리가 만든 SessionManager와 같은 방식으로 동작한다.
다만 HttpSession이 만든 쿠키이름은 `JSESSIONID` 이다.

<br>

**LoginController - login** - 로그인 처리
```Java
@PostMapping("/login")
public String login() {

    //로그인 성공 처리

    //세션이 있으면 있는 세션 반환, 없으면 신규 세션을 생성
    HttpSession session = request.getSession();
    //세션에 로그인 회원 정보 보관
    session.setAttribute("loginMember", "2");

    return "redirect:/";
}
```
request.getSession()은 기본값으로 request.getSession(true)값을 사용한다.
* request.getSession(true) 옵션
  * 세션이 있으면 기존 세션을 반환
  * 없으면 새로운 세션을 생성해서 반환한다.
* request.getSession(false) 옵션
  * 세션이 있으면 기존 세션을 반환
  * 없으면 새로운 세션을 생성하지 않고 **null**을 반환한다.

<br><br>

**LoginController - logout** - 로그아웃 처리
```Java
@PostMapping("/logout")
public String logout(HttpServletRequest request) {
    HttpSession session = request.getSession(false);
    if (session != null) {
        session.invalidate();
    }
    return "redirect:/";
}
```
`session.invalidate` : 세션을 제거한다.

<br><br>

**홈 로그인처리**
```Java
@GetMapping("/")
public String homeLogin(HttpServletRequest request) {

    HttpSession session = request.getSession(false);
    log.info("세션 정보 session={}", session);
    if (session == null) {
        return "home";
    }

    Member loginMember = (Member) session.getAttribute("loginMember");

    //세션에 회원 데이터가 없으면
    if (loginMember == null) {
        return "home";
    }

    //세션이 유지되면 로그인으로 이동
    return "loginHome";

}
```
`session.getAttribute("loginMember");` : 로그인 시점에 세션에 보관한 회원 객체를 찾는다.


> 스프링은 세션을 더 편리하게 사용할 수 있도록 `@SessionAttribute`를 제공한다. 이미 로그인 된 사용자를 찾을 때는 다음과 같이 사용하면 된다.
`@SessionAttribute(name = "loginMember", required = false) Member loginMember`

