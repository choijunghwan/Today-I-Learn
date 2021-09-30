CharlieZip 프로젝트를 진행하면서, 로그인 기능이 구현하고 나니 많은 부분에 로그인여부를 체크해야하는 일이 발생하였습니다.

그로 인해, 많은 코드 중복이 발생하게 되었고, 추후 추가나 변경이 있을시 일일히 찾아가며 수정을 해야하기 때문에 **유지보수성**이 매우 떨어질 수 있습니다. 

그래서 로그인 여부 로직을 한 곳으로 분리하여 관리하도록 리팩토링을 진행하였습니다.

<br>

# AOP VS Interceptor
애플리케이션 여러로직에서 공통으로 관심이 있는것을 공통 관심사(cross-cutting concern)라고 합니다.

이러한 공통 관심사는 스프링의 AOP로도 해결할 수 있지만, 저의 공통 관심사는 로그인 여부이고 이는 웹과 관련된 공통 관심사입니다. 

로그인 여부를 처리할 때는 HTTP의 헤더나 URL의 정보들이 필요한데 이러한 정보를 좀 더 쉽게 사용하기 위해 저는 스프링 인터셉터를 사용하였습니다.

<br>

# 스프링 인터셉터
스프링 인터셉터는 스프링 MVC가 제공하는 기술입니다.

**스프링 인터셉터 흐름**
`HTTP 요청 -> WAS -> 필터 -> 서블릿 -> 스프링 인터셉터 -> 컨트롤러`
* 스프링 인터셉터는 디스패처 서블릿과 컨트롤러 사이에서 컨트롤러 호출 직전에 호출 됩니다.

<br>

**스프링 인터셉터 인터페이스**
스프링의 인터셉터를 사용하려면 `HandlerInterceptor` 인터페이스를 구현해야 합니다.
```Java
public interface HandlerInterceptor {

    default boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {}

    default void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, @Nullable ModelAndView modelAndView) throws Exception {}

    default void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, @Nullable Exception ex) throws Exception {}

}
```
스프링 인터셉터는 3가지로 나뉘어져 있습니다.
* `preHandle` : 컨트롤러 호출 전에 호출됩니다.
  * `preHandle`의 return 값이 true이면 다음으로 진행되고, false이면 더 진행되지 않습니다.
* `postHandle` : 컨트롤러 호출 후에 호출됩니다.
  * `preHandle`의 값이 false이면 호출되지 않습니다.
* `afterCompletion` : 뷰가 렌더링 된 이후에 호출됩니다.
  * `preHandel`의 값이 false여도 호출됩니다.
  * 주로 예외가 발생하면 `afterCompletion`에 예외정보를 포함해서 호출합니다.

<br><br>

## LoginCheckInterceptor 생성
로그인 여부를 체크해주는 인터셉터 클래스를 작성해줍니다.

<br>

```Java
package study.charlieZip.intercepter;

import lombok.extern.slf4j.Slf4j;
import org.springframework.web.servlet.HandlerInterceptor;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;

@Slf4j
public class LoginCheckInterceptor implements HandlerInterceptor {

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        String requestURI = request.getRequestURI();
        log.info("인증 체크 인터셉트 실행 {}", requestURI);

        HttpSession session = request.getSession();

        if (session == null || session.getAttribute("loginMember") == null) {
            log.info("미인증 사용자 요청");
            //로그인으로 redirect
            response.sendRedirect("/login?redirectURL=" + requestURI);
            return false;
        }

        return true;
    }
}
```
* 세션을 조회해서 로그인한 사용자의 `id` 값이 있는지 조회합니다.
* 사용자의 `id`값이 있다면 인증된 사용자 이므로 true를 반환합니다.
* 사용자의 `id`값이 없다면 미인증 사용자이므로 요청 전의 페이지로 리다이렉트 시킵니다.

<br>

## WebConfig로 스프링에 등록하기
스프링 인터셉터를 스프링에 등록하여 스프링에서 관리될 수 있도록 합니다.

<br>

```Java
package study.charlieZip.config;

import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;
import study.charlieZip.intercepter.LogInterceptor;
import study.charlieZip.intercepter.LoginCheckInterceptor;

@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new LogInterceptor())
                .order(1)
                .addPathPatterns("/**")
                .excludePathPatterns("/css/**", "/js/**", "/img/**", "/favicon/**", "/*.ico", "/error");

        registry.addInterceptor(new LoginCheckInterceptor())
                .order(2)
                .addPathPatterns("/**")
                .excludePathPatterns("/css/**", "/js/**", "/img/**", "/favicon/**", "/*.ico", "/error",
                        "/", "/login", "/logout", "/coffees", "/members/new");

    }
}
```
인터셉터는 `addInterceptors`메서드를 통해 등록합니다.

* `addInterceptor` : 등록할 인터셉터
* `order` : 인터셉터 호출 순서
  * 인터셉터는 여러개 등록할 수 있는데 여러개의 인터셉터는 체인처럼 연결되어 있습니다.
  * 여러개의 인터셉터는 `order`로 정한 순서에 맞게 순서대로 호출됩니다.
* `addPathPatterns` : 인터셉터를 적용할 URL 패턴을 지정합니다.
* `excludePathPatterns` : 인터셉터를 제외할 패턴을 지정합니다.

<br>

## 정리
인터셉터를 통해서 컨트롤러에서 일일이 로그인 여부를 체크하지 않아도
웹을 사용할때 미인증 사용자의 접근을 제어 할 수 있게 되었습니다.


