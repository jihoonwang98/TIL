# [Spring Security] DelegatingFilterProxy & FilterChainProxy



> **[참고]**
>
> https://ncucu.me/108



![](https://godekdls.github.io/images/springsecurity/filterchainproxy.png)





## Servlet Filter

- 요청을 보내면 요청을 서블릿 컨테이너가 받게 된다.
  - 서블릿 기반 애플리케이션이기 때문이다.
- 서블릿 컨테이너는 서블릿 스펙을 지원한다.
- 서블릿 스펙에는 `Filter`라는 개념이 존재한다.
  - 어떤 요청의 전후 처리를 할 수 있는 `Interceptor` 같은 개념이다.
- 이러한 `Filter`의 구현체 중 하나인 `DelegatingFilterProxy`가 존재한다.





## DelegatingFilterProxy



![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FboXFjg%2Fbtqyldeo0G7%2FgPQscL8rJRQ3ikkxPkfAmk%2Fimg.png)



- 일반적인 서블릿 필터.
- 서블릿 필터 처리를 직접 하지않고 스프링에 들어있는 빈으로 위임하고 싶을 때 사용하는 서블릿 필터이다.
- Spring IoC 컨테이너에 존재하는 특정 빈에게 위임하게 된다.
- 위임할 대상의 이름을 명시해 주어야 한다.
- Spring Boot의 도움 없이 Spring Security를 사용할 때는 web.xml에 명시하거나, `AbstractSecurityWebApplicationInitializer`를 사용해서 자동 등록한다.
- Spring Boot를 사용한다면, 자동으로 설정이 된다. (`SecurityFilterAutoConfiguration`)





**SecurityFilterAutoConfiguration**

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fb9i6T2%2FbtqykyJ5exs%2FNIEzUEd3VJzLEh8XNws2G0%2Fimg.png)



`DEFAULT_FILTER_NAME`

- `DelegatingFilterProxy`는 위임할 대상의 이름을 지정해 주어야 한다고 했는데, 우리는 `FilterChainProxy`의 이름을 지정해 주지 않았는데도 어떻게 알 수 있는 것일까?
- `AbstractSecurityWebApplicationInitializer` 클래스의 `DEFAULT_FILTER_NAME` 상수로 정의되어있는 것을 기본값으로 사용한다.



![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FkCfld%2Fbtqyj3jftor%2F9sqtRJomVgLS4d5YRKiW40%2Fimg.png)