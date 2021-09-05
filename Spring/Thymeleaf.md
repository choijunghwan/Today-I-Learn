# 타임리프란?
타임리프는 백엔드 서버에서 HTML을 동적으로 렌더링 하기 위해 사용하는 뷰 템플릿 중에 하나다.

타임리프는 이런한 장점들이 있다.
* 순수 HTML을 최대한 유지하는 특징이 있다.
* HTML을 최대한 유지하기 때문에 브라우저에서 파일을 직접 열어도 내용을 확인할 수 있고, 서버를 통해 뷰 템플릿을 거치면 변경된 결과를 확인할 수 있다.
* 스프링과 자연스럽게 통합되어 스프링의 다양한 기능을 편리하게 사용할 수 있다.

타임리프의 다양한 기능중 일부의 기능에 대해 공부해보겠다.  
더 많은 기능이 궁금하다면 공식 메뉴얼을 한번 읽어보자.  
* 기본 기능 - https://www.thymeleaf.org/doc/tutorials/3.0/usingthymeleaf.html
* 스프링 통합 - https://www.thymeleaf.org/doc/tutorials/3.0/thymeleafspring.html  

<br><br>

# 다양한 기능
## 변수표현식
타임리프에서 변수를 사용할 때는 변수 표현식을 사용합니다.

기본 변수표현식 : `${...}`

또한 스프링EL 이라는 스프링이 제공하는 표현식을 사용할 수 있습니다.

**Controller**
```JAVA
@GetMapping("/variable")
public String variable(Model model) {
    User userA = new User("userA", 10);
    User userB = new User("userB", 10);

    List<User> list = new ArrayList<>();
    list.add(userA);
    list.add(userB);

    Map<String, User> map = new HashMap<>();
    map.put("userA", userA);
    map.put("userB", userB);

    model.addAttribute("user", userA);
    model.addAttribute("users", list);
    model.addAttribute("userMap", map);

    return "basic/variable";
}

@Data
static class User {
		private String username;
		private int age;
		public User(String username, int age) {
		this.username = username;
		this.age = age;
		}
}
```

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<h1>SpringEL 표현식</h1>
<ul>Object
    <li>${user.username} = <span th:text="${user.username}"></span></li>
    <li>${user['username']} = <span th:text="${user['username']}"></span></li>
    <li>${user.getUsername()} = <span th:text="${user.getUsername()}"></span></li>
</ul>
<ul>List
    <li>${users[0].username} = <span th:text="${users[0].username}"></span></li>
    <li>${users[0]['username']} = <span th:text="${users[0]['username']}"></span></li>
    <li>${users[0].getUsername()} = <span th:text="${users[0].getUsername()}"></span></li>
</ul>
<ul>Map
    <li>${userMap['userA'].username} = <span th:text="${userMap['userA'].username}"></span></li>
    <li>${userMap['userA']['username']} = <span th:text="${userMap['userA']['username']}"></span></li>
    <li>${userMap['userA'].getUsername()} = <span th:text="${userMap['userA'].getUsername()}"></span></li>
</ul>

<h1>지역 변수 - (th:with)</h1>
<div th:with="first=${users[0]}">
    <p>처음 사람의 이름은 <span th:text="${first.username}"></span></p>
</div>
</body>
</html>
```

스프링EL은 기본적으로 프로퍼티 접근법을 사용합니다.  
- `user.username` : user의 username을 프로퍼티 접근 → `user.getUsername()`
- `user['username']` : 위와 같음
- `user.getUsername()` : user의 `getUsername()` 을 직접 호출

List, Map 자료구조도 같은 프로퍼티 접근법으로 접근 가능합니다.

**지역 변수 선언**  
`th:with` 를 사용하면 지역 변수를 선언해서 사용할 수 있습니다.

지역 변수이기 때문에 선언한 태그 안에서만 사용할 수 있습니다.

<br>

## 기본 객체들
타임리프는 기본 객체들을 제공합니다.  
기본객체라 하면
- `${#request}`
- `${#response}`
- `${#session}`
- `${#servletContext}`
- `${#locale}`

위와 같은 것들이 있습니다. 기본적으로 스프링을 사용하다보면 자주 사용하는 객체들이기 때문에 타임리프가 쉽게 갖다 쓸 수 있게 기능을 구현해 놓은 것입니다.

다만 `#request`는 `HttpServletRequest` 객체가 그대로 제공되기 때문에 데이터를 조회하려면 `request.getParameter("data")` 처럼 불편하게 접근해야 한다.

이럼 점을 해결하기 위해 편의 객체도 제공한다.

- HTTP 요청 파라미터 접근: `param`
    - 예) `${param.paramData}`
- HTTP 세션 접근 : `session`
    - 예) `${session.sessionData}`
- 스프링 빈 접근: `@`
    - 예) `${@helloBean.hello('Spring!')}`

<br><br>

## 자바8 날짜
기본 객체들 만큼이나 날짜를 사용할 일이 많이 있습니다.  
그래서 타임리프는 자바8 날짜용 유틸리티 객체 `#temporals`를 제공해 줍니다.

**BasicController**

```java
@GetMapping("/date")
public String date(Model model) {
    model.addAttribute("localDateTime", LocalDateTime.now());
    return "basic/date";
}
```

**/resources/templates/basic/date.html**

```html
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<h1>LocalDateTime</h1>
<ul>
    <li>default = <span th:text="${localDateTime}"></span></li>
    <li>yyyy-MM-dd HH:mm:ss = <span th:text="${#temporals.format(localDateTime,'yyyy-MM-dd HH:mm:ss')}"></span></li>
</ul>
<h1>LocalDateTime - Utils</h1>
<ul>
    <li>${#temporals.day(localDateTime)} = <span th:text="${#temporals.day(localDateTime)}"></span></li>
    <li>${#temporals.month(localDateTime)} = <span th:text="${#temporals.month(localDateTime)}"></span></li>
    <li>${#temporals.monthName(localDateTime)} = <span th:text="${#temporals.monthName(localDateTime)}"></span></li>
    <li>${#temporals.monthNameShort(localDateTime)} = <span th:text="${#temporals.monthNameShort(localDateTime)}"></span></li>
    <li>${#temporals.year(localDateTime)} = <span th:text="${#temporals.year(localDateTime)}"></span></li>
    <li>${#temporals.dayOfWeek(localDateTime)} = <span th:text="${#temporals.dayOfWeek(localDateTime)}"></span></li>
    <li>${#temporals.dayOfWeekName(localDateTime)} = <span th:text="${#temporals.dayOfWeekName(localDateTime)}"></span></li>
    <li>${#temporals.dayOfWeekNameShort(localDateTime)} = <span th:text="${#temporals.dayOfWeekNameShort(localDateTime)}"></span></li>
    <li>${#temporals.hour(localDateTime)} = <span th:text="${#temporals.hour(localDateTime)}"></span></li>
    <li>${#temporals.minute(localDateTime)} = <span th:text="${#temporals.minute(localDateTime)}"></span></li>
    <li>${#temporals.second(localDateTime)} = <span th:text="${#temporals.second(localDateTime)}"></span></li>
    <li>${#temporals.nanosecond(localDateTime)} = <span th:text="${#temporals.nanosecond(localDateTime)}"></span></li>
</ul>
</body>
</html>
```

간단한 사용법은 위의 코드를 참고하시면 되고, 이외에 추가적인 내용이 필요하시면 필요할 때 찾아서 사용하시기를 추천드립니다.

## URL 링크
타임리프에서 URL을 생성할 때는 `@{...}` 문법을 사용합니다.

**BasicController**

```java
@GetMapping("/link")
public String link(Model model) {
    model.addAttribute("param1", "data1");
    model.addAttribute("param2", "data2");
    return "basic/link";
}
```

**/resources/templates/basic/link.html**

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>

<h1>URL 링크</h1>

<ul>
    <li><a th:href="@{/hello}">basic url</a></li>
    <li><a th:href="@{/hello(param1=${param1}, param2=${param2})}">hello query param</a></li>
    <li><a th:href="@{/hello/{param1}/{param2}(param1=${param1}, param2=${param2})}">path variable</a></li>
    <li><a th:href="@{/hello/{param1}(param1=${param1}, param2=${param2})}">path variable + query parameter</a></li>
</ul>

</body>
</html>
```

**단순한 URL**

- `@{/hello}` → `/hello`

**쿼리 파라미터**

- `@{/hello(param1=${param1}, param2=${param2})}`
    - → `/hello?param1=data&param2=data2`
    - `()`에 있는 부분은 쿼리 파라미터로 처리된다.

**경로 변수**

- `@{/hello/{param1}/{param2}(param1=${param1}, param2=${param2})}`
    - → `/hello/data1/data2`
    - URL 경ㄹ로상에 변수가 있으면 `( )` 부분은 경로 변수로 처리된다.

**경로 변수 + 쿼리 파라미터**

- `@{/hello/{param1}(param1=${param1}, param2=${param2})}`
    - → /hello/data?param2=data2
    - 경로 변수와 쿼리 파라미터를 함께 사용할 수 있다.

<br>

## 템플릿 조각
웹 페이지를 개발할 때는 공통 영역이 많이 있습니다.  
웹을 개발할때 공통적인 로직들을 처리하기 위해 스프링 AOP 를 사용하는 것처럼 타임리프에는 템플릿 레이아웃 기능을 지원합니다.

이 부분은 코드를 보고 한번 따라해보시는게 이해가 가장 잘 됩니다.

**TempleateController**

```java
package hello.thymeleaf.basic;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
@RequestMapping("/template")
public class TemplateController {

    @GetMapping("/fragment")
    public String template() {
        return "template/fragment/fragmentMain";
    }

}
```

**/resources/templates/template/fragment/footer.html**

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<body>

<footer th:fragment="copy">
    푸터 자리 입니다.
</footer>

<footer th:fragment="copyParam (param1, param2)">
    <p>파라미터 자리 입니다.</p>
    <p th:text="${param1}"></p>
    <p th:text="${param2}"></p>
</footer>

</body>
</html>
```

**/resources/templates/template/fragment/fragmentMain.html**

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>

<h1>부분 포함</h1>

<h2>부분 포함 insert</h2>
<div th:insert="~{template/fragment/footer :: copy}"></div>

<h2>부분 포함 replace</h2>
<div th:replace="~{template/fragment/footer :: copy}"></div>

<h2>부분 포함 단순 표현식</h2>
<div th:replace="template/fragment/footer :: copy"></div>

<h1>파라미터 사용</h1>
<div th:replace="~{template/fragment/footer :: copyParam ('데이터1', '데이터2')}"></div>

</body>
</html>
```

**부분 포함 insert**

`th:insert`를 사용하면 현재 태그 (`div`) 내부에 추가한다.

**부분 포함 replace**

`th:replace` 를 사용하면 현재 태그 (`div`)를 대체한다.

**부분 포함 단순 표현식**

`~{...}` 를 사용하는 것이 원칙이지만 템플릿 조각을 사용하는 코드가 단순하면 이 부분을 생략할 수 있다.

**파라미터 사용**

`<div th:replace="~{template/fragment/footer :: copyParam ('데이터1', '데이터2')}"></div>`

다음과 같이 파라미터를 전달해서 동적으로 조각을 렌더링 할 수도 있다.

<br>

## 템플릿 레이아웃
템플릿 조각은 일부 코드 조각을 가지고 와서 사용한다면 이번에는 개념을 더 확장해서 코드 조각을 레이아웃에 넘겨서 사용하는 방법을 알아보겠습니다.

역시 먼저 코드를 보겠습니다.

**TempleateController**

```java
@GetMapping("layout")
public String layout() {
    return "template/layout/layoutMain";
}
```

**/resources/templates/template/layout/base.html**

```html
<html xmlns:th="http://www.thymeleaf.org">
<head th:fragment="common_header(title,links)">

    <title th:replace="${title}">레이아웃 타이틀</title>

    <!-- 공통 -->
    <link rel="stylesheet" type="text/css" media="all" th:href="@{/css/awesomeapp.css}">
    <link rel="shortcut icon" th:href="@{/images/favicon.ico}">
    <script type="text/javascript" th:src="@{/sh/scripts/codebase.js}"></script>

    <!-- 추가 -->
    <th:block th:replace="${links}" />

</head>
```

**/resources/templates/template/layout/layoutMain.html**

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head th:replace="template/layout/base :: common_header(~{::title},~{::link})">

    <title>메인 타이틀</title>

    <link rel="stylesheet" th:href="@{/css/bootstrap.min.css}">
    <link rel="stylesheet" th:href="@{/themes/smoothness/jquery-ui.css}">

</head>
<body>
메인 컨텐츠
</body>
</html>
```

**결과**

```html
<!DOCTYPE html>
<html>
<head>
    <title>메인 타이틀</title>

    <!-- 공통 -->
    <link rel="stylesheet" type="text/css" media="all" href="/css/awesomeapp.css">
    <link rel="shortcut icon" href="/images/favicon.ico">
    <script type="text/javascript" src="/sh/scripts/codebase.js"></script>

    <!-- 추가 -->
    <link rel="stylesheet" href="/css/bootstrap.min.css"><link rel="stylesheet" href="/themes/smoothness/jquery-ui.css">

</head>
<body>
메인 컨텐츠
</body>
</html>
```

- `common_header(~{::title}, ~{::link})` 이 부분이 핵심이다.
    - `::title` 은 현재 페이지의 title 태그들을 전달한다.
    - `::link` 는 현재 페이지의 link 태그들을 전달한다.

<head> 에 공통으로 사용하는 css, javascript 같은 정보들이 있는데, 이러한 공통 정보를 한 곳에 모아두고, 공통으로 사용하지만, 각 페이지마다 필요한 정보를 더 추가해서 사용하고 싶다면 다음과 같이 사용하면 됩니다.

