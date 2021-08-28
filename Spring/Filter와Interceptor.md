Spring으로 로그인 인증을 구현하기위해 공부하게 되면 Filter와 Interceptor를 맞닥들이게 된다.

Filter와 Interceptor의 차이점을 알아보고 어느상황에 어떤걸 써야할지 알아보자.

## 실행시점

![Filter](/Img/filter.png)

[출처](https://justforchangesake.wordpress.com/2014/05/07/spring-mvc-request-life-cycle/)

<br>

## 실행순서

WAS → 필터 → 디스패처서블릿 → 인터셉터 → 컨트롤러

<br>

## 차이점

- Filter와 Interceptor는 실행되는 시점이 다른다.
- Filter는 Web Application에 등록을 하고, Interceptor는 Spring의 Context에 등록을 한다.

둘의 가장 큰 차이점은 실행되는 시점이 다르다는것이다.

하지만 둘다 컨트롤러보다 먼저 실행되기 때문에 만약 우리가 로그인 인증을 처리한다고 가정한다면 필터를 사용하든, 인터셉터를 사용하든 문제가 없다.

실행되는 시점이 다르기 때문에 느끼기 가장 큰 차이점은 예외 처리이다.

만약 Filter에서 예외가 발생하면 WAS를 거쳐 예외처리를 해야만 한다. WAS를 거쳐서 예외처리를 하게 되면 `WAS → 필터 → 디스패처서블릿 → 인터셉터 → 컨트롤러` 이 과정을 다시 거쳐 BasicErrorController가 호출되게 된다. 오류처리에 대한 자세한 내용은 서술하지 않겠다.

즉, 우리가 생각할것은 `WAS → 필터 → 디스패처서블릿 → 인터셉터 → 컨트롤러` 이 과정이 두번 반복된다는 것이다.

하지만 Interceptor에서 예외가 발생하면 컨트롤러에서 에러가 발생하면 서블릿을 거쳐 WAS로 가기전에 `ExceptionResolver` 를 거쳐서 예외를 처리할 수 있다.

좀더 예외처리 과정이 간단해지고 효율적으로 변하는 것이다.

<br>

## Interface

Filter와 Interceptor의 인터페이스를 보며 차이점을 좀더 알아보자.

<br>

**Filter**

```java
public interface Filter {
    public default void init(FilterConfig filterConfig) throws ServletException {}
    public void doFilter(ServletRequest request, ServletResponse response,
            FilterChain chain) throws IOException, ServletException;
    public default void destroy() {}
}
```

Filter는 `init, doFilter, destroy` 로 이루어져있지만 `init,destory` 는 default로 되어 있어구현하지 않아도 된다. 실제로도 아무런 역할이 없다.

실제역할은 `doFilter` 가 담당한다.

<br>

**HandlerInterceptor**

```java
public interface HandlerInterceptor {
	default boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
			throws Exception {
	
		return true;
	}
	default void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler,
			@Nullable ModelAndView modelAndView) throws Exception {
	}
	default void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler,
			@Nullable Exception ex) throws Exception {
	}
}
```

Interceptor는 `preHandle(실행하기전)`, `postHandle(실행한 후)`, `afterCompletion(완료된후)` 로,

실행시점이 나눠어져 있다.

`postHandle`이랑 `afterCompletion`의 차이가 궁금하실거다.

`postHandle`은 정상적인 로직에서만 실행되고, 컨트롤러에서 예외가 발생하면 실행되지 않는다.

허나 `afterCompletion`은 예외와 상관없이 무조건 실행된다.

`Interceptor`는 이렇게 실행시점이 나뉘어져 있어, View를 렌더링하기전에 추가적인 `ExceptionResolver`를 거치는 추가적인 작업이 가능하다.

그리고 filter는 **서블릿**의 기술이지만, Interceptor는 **스프링**이 제공하는 기능이다.

즉, Interceptor가 스프링을 사용하기에 더 편리한 기능을 많이 제공한다는 뜻이다.

<br>

## Filter를 사용할 때

다만 Interceptor에서는 불가능하고 Filter에서만 가능한 일이 있다.

Interceptor는 HttpServeltRequest를 사용하는것을 볼 수 있다.

하지만 Filter 인터페이스에 doFilter 파라미터를 잘 보면 ServletRequest로 구현되어 있다.

> **Filter는 ServletRequest를 교체하거나 커스터마이징 할 수 있다.**

<br><br>

## 마무리
---

Filter와 Interceptor는 컨트롤러 전에 실행되기 때문에 로그인 인증을 구현하기 위해서는 둘 중 어느것을 사용해도 상관없지만, 실제로 실행되는 시점이 다르고 차이점을 정확히 파악하고 사용해야 한다.

개인적으로는 Spring을 사용하신다면 Spring이 제공하는 `Interceptor`가 `filter`보다 더 많은 기능을 제공하고 더 편리하기에 `Interceptor`를 기본적으로 사용하시다가 `Filter`의 `ServletRequest`를 커스터마이징해야 하는 요구사항이 있을 때만 `Filter`를 사용하는 것을 추천한다.