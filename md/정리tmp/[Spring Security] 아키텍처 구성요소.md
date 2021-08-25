# [Spring Security] 아키텍처 구성요소



### Architecture Components

- **SecurityContextHolder**
  - 스프링 시큐리티에서 인증한 대상에 대한 상세 정보는 `SecurityContextHolder`에 저장한다.



- **SecurityContext**
  - `SecurityContextHolder`로 접근할 수 있으며, 현재 인증한 사용자의 `Authentication`을 가지고 있다.



- **Authentication**
  - 사용자가 제공한 인증용 credential이나 `SecurityContext`에 있는 현재 사용자의 credential을 제공하며, `AuthenticationManager`의 입력으로 사용한다.



- **GrantedAuthority**
  - `Authentication`에서 접근 주체(principal)에 부여한 권한



- **AuthenticationManager**
  - 스프링 시큐리티의 필터가 인증을 어떻게 수행할지를 정의하는 API



- **ProviderManager**
  - 가장 많이 사용하는 `AuthenticationManager` 구현체



- **AuthenticationProvider**
  - `ProviderManager`가 특정 인증 유형을 수행할 때 사용한다.



- **AuthenticationEntryPoint**

  - 클라이언트에 credential을 요청할 때 사용한다. 

    (i.e. 로그인 페이지로 리다이렉트하거나 `WWW-Authenticate` 헤더를 전송하는 등)



- **AbstractAuthenticationProcessingFilter**
  - 인증에 사용할 `Filter`의 베이스. 필터를 잘 이해하면 여러 컴포넌트를 조합해서 심도 있는 인증 플로우를 구성할 수 있다.





## 1. SecurityContextHolder

스프링 시큐리티의 Authentication 모델 중심에는 `SecurityContextHolder`가 있다.

이 홀더는 `SecurityContext`를 가지고 있다.



![](https://godekdls.github.io/images/springsecurity/securitycontextholder.png)



`SecurityContextHolder`에는 스프링 시큐리티로 인증한 사용자의 상세 정보를 저장한다.

스프링 시큐리티는 `SecurityContextHolder`에 어떻게 값을 넣는지는 상관하지 않는다.

값이 있다면 현재 인증한 사용자 정보로 사용한다.



사용자가 인증됐음을 나타내는 가장 쉬운 방법은 직접 `SecurityContextHolder`를 설정하는 것이다.

**[예제] 직접 `SecurityContextHolder` 세팅하기** 

```java
SecurityContext context = SecurityContextHolder.createEmptyContext();  // (1)
Authentication authentication = 
    new TestingAuthenticationToken("username", "password", "ROLE_USER"); // (2)
context.setAuthentication(authentication);

SecurityContextHoler.setContext(context); // (3)
```

**(1)**: 비어있는 `SecurityContext`를 만드는 것으로 시작한다. thread 경합을 피하려면 `SecurityContextHolder.getContext().setAuthentication(authentication)`을 사용해선 안 되며, 새 `SecurityContext` 인스턴스를 생성해야 한다.



**(2)**: 그 다음 새 `Authentication` 객체를 생성한다. `Authentication` 구현체라면 모두 `SecurityContext`에 담을 수 있다. 여기선 간단하게 `TestingAuthenticationToken`을 사용했다. 프로덕션 환경에선 `UsernamePasswordAuthenticationToken(userDetails, password, authorities)`를 주로 사용한다.



**(3)**: 마지막으로 `SecurityContextHolder`에 `SecurityContext`를 설정해 준다. 스프링 시큐리티는 이 정보를 사용해서 권한을 인가한다.









인증된 주체(principal) 정보를 얻어야 한다면 `SecurityContextHolder`에 접근하면 된다.

**[예제] 현재 인증된 사용자 정보에 접근하기**

```java
SecurityContext context = SecurityContextHolder.getContext();
Authentication authentication = context.getAuthentication();
String username = authentication.getName();
Object principal = authentication.getPrincipal();
Collection<? extends GrantedAuthority> authorities = authentication.getAuthorities();
```



기본적으로 `SecurityContextHolder`는 `ThreadLocal`을 사용해서 정보를 저장하기 때문에, 메서드에 직접 `SecurityContext`를 넘기지 않아도 동일한 thread라면 항상 `SecurityContext`에 접근할 수 있다.

기존 principal 요청을 처리한 다음에 비워주는 것만 잊지 않으면 `ThreadLocal`을 사용해도 안전하다. 

스프링 시큐리티의 FilterChainProxy는 항상 `SecurityContext`를 비워준다.





## 2. SecurityContext

`SecurityContext`는 `SecurityContextHolder`로 접근할 수 있다.

`SecurityContext`는 `Authentication` 객체를 가지고 있다.





## 3. Authentication

스프링 시큐리티에서 `Authentication`이 주로 담당하는 일은 다음과 같다.

- `AuthenticationManager`의 입력으로 사용되어, 인증에 사용할 사용자의 credential을 제공한다.

  이 상황에선 `isAuthenticated()`는 `false`를 리턴한다.

- 현재 인증된 사용자를 나타낸다. 현재 `Authentication`은 `SecurityContext`에서 가져올 수 있다.





`Authentication`은 다음과 같은 정보를 가지고 있다.

- `principal`: 사용자를 식별한다. Username/Password로 인증할 땐 보통 `UserDetails` 인스턴스다.
- `credentials`: 주로 비밀번호. 대부분은 유출되지 않도록 사용자를 인증한 다음 비운다.
- `authorities`: 사용자에게 부여한 권한은 `GrantedAuthority`로 추상화 한다. 







## 4. GrantedAuthority

사용자에게 부여한 권한은 `GrantedAuthority`로 추상화한다.

예시로 role이나 scope가 있다.



`GrantedAuthority`는 `Authentication.getAuthorities()` 메서드로 접근할 수 있다.

이 메서드는 `GrantedAuthority` 객체의 `Collection`을 리턴한다.



`GrantedAuthority`는 말 그대로 인증한 주체(principal)에게 부여된 권한이다.

권한은 보통 `ROLE_ADMIN`이나 `ROLE_HR_SUPERVISOR` 같은 "**역할(role)**"이다.

이런 역할은 이후에 웹 인가(권한), 메서드 인가, 도메인 객체 인가에서 사용한다.

스프링 시큐리티의 다른 코드에선 역할(role)을 해석하고 필요한 권한을 확인한다.

Username/Password 기반 인증을 사용한다면 보통 `UserDetailsService`가 `GrantedAuthority`를 로드한다.



`GrantedAuthority` 객체는 일반적으로 애플리케이션 전체에 걸친 권한을 의미한다.

특정 도메인 객체에 국한되지 않는다. 

따라서 `Employee` 객체 번호 54의 권한을 나타내는 `GrantedAuthority`, 이런 식으로는 잘 쓰지 않는다.

이렇게 사용하면 권한을 수천 개씩 만들어야 할 수도 있고, 메모리가 부족해질 것이다.

물론, 스프링 시큐리티는 이런 일반적인 요구사항을 처리하도록 설계했지만, 원한다면 도메인 객체 단위로도 보안을 적용할 수 있다.





## 5. AuthenticationManager

`AuthenticationManager`는 스프링 시큐리티 필터의 인증(Authentication) 수행 방식을 정의하는 API다.

매니저가 리턴한 `Authentication`을 `SecurityContextHolder`에 설정하는 건, `AuthenticationManager`를 호출한 객체 (i.e. 스프링 시큐리티의 필터)가 담당한다.

스프링 시큐리티의 `Filters`를 사용하지 않는다면, `AuthenticationManager`를 사용할 필요 없이 직접 `SecurityContextHolder`를 설정하면 된다.



`AuthenticationManager` 구현체는 어떤 것을 사용해도 좋지만, 가장 많이 사용하는 구현체는 `ProviderManager`다.





## 6. ProviderManager

`ProviderManager`는 가장 많이 쓰는 `AuthenticationManager` 구현체다.

`ProviderManager`는 동작을 `AuthenticationProvider` `List`에 위임한다.

모든 `AuthenticationProvider`는 인증을 성공시키거나, 실패시키거나, 아니면 결정을 내릴 수 없는 것으로 판단하고 다운 스트림에 있는 `AuthenticationProvider`가 결정하도록 만들 수 있다.

설정해둔 `AuthenticationProvider`가 전부 인증하지 못하면 `ProviderNotFoundException`과 함께 실패한다.

이 예외는 `AuthenticationException`의 하위 클래스로, 넘겨진 `Authentication` 유형을 지원하는 `ProviderManager`를 설정하지 않았음을 의미한다.



![](https://godekdls.github.io/images/springsecurity/providermanager.png)



보통은 `AuthenticationProvider`마다 각자가 맡은 인증을 수행하는 법을 알고 있다.

예를 들어, `AuthenticationProvider` 하나로 Username/Password를 검증할 수 있고, 다른 하나는 SAML 인증을 담당할 수 있다.

이렇게 하면 인증 유형마다 담당 `AuthenticationProvider`가 있기 때문에, `AuthenticationManager` 빈 하나만 외부로 노출하면서도 여러 인증 유형을 지원할 수 있다.





원한다면 `ProviderManager`에 인증을 수행할 수 있는 `AuthenticationProvider`가 없을 때 사용할, 부모 `AuthenticationManager`를 설정할 수도 있다. 

부모는 `AuthenticationManager`의 어떤 구현체도 될 수 있지만 보통 `ProviderManager` 인스턴스를 많이 사용한다.



![](https://godekdls.github.io/images/springsecurity/providermanager-parent.png)







사실은 여러 `ProviderManager` 인스턴스에 동일한 부모 `AuthenticationManager`를 공유하는 것도 가능하다.

각각 인증 메커니즘이 다른 (`ProviderManager` 인스턴스가 다른) `SecurityFilterChain` 여러 개가 공통 인증을 사용하는 경우에 (부모 `AuthenticationManager`를 공유) 흔히 쓰는 패턴이다.



![](https://godekdls.github.io/images/springsecurity/providermanagers-parent.png)







## 7. AuthenticationProvider

`ProviderManager`엔 `AuthenticationProvider`를 여러 개 주입할 수 있다.

`AuthenticationProvider`마다 담당하는 인증 유형이 다르다.

예를 들어, `DaoAuthenticationProvider`는 Username/Password 기반 인증을,

`JwtAuthenticationProvider`는 JWT 토큰 인증을 지원한다.







## 8. Request Credentials with `AuthenticationEntryPoint`

`AuthenticationEntryPoint`는 클라이언트의 credential을 요청하는 HTTP 응답을 보낼 때 사용한다.



클라이언트가 리소스를 요청할 때 미리 Username/Password 같은 credential을 함께 보낼 때도 있다.

이럴 때는 credential을 요청하는 HTTP 응답을 만들 필요가 없다.



하지만 어떨 땐 클라이언트가 접근 권한이 없는 리소스에 인증되지 않은 요청을 보내기도 한다.

이때는 `AuthenticationEntryPoint` 구현체가 클라이언트에 credential을 요청한다. 

`AuthenticationEntryPoint`는 로그인 페이지로 리다이렉트하거나, WWW-Authenticate 헤더로 응답하는 등의 일을 담당한다.





## 9. AbstractAuthenticationProcessingFilter

`AbstractAuthenticationProcessingFilter`는 사용자의 credential을 인증하기 위한 베이스 `Filter`다.

credential을 인증할 수 없다면, 스프링 시큐리티는 보통 `AuthenticationEntryPoint`로 credential을 요청한다.



그러고 나면 `AbstractAuthenticationProcessingFilter`는 제출한 모든 인증 요청을 처리할 수 있다.



![](https://godekdls.github.io/images/springsecurity/abstractauthenticationprocessingfilter.png)



**(1): **사용자가 credential을 제출하면 `AbstractAuthenticationProcessingFilter`는 인증할 `HttpServletRequest`로부터 `Authentication`을 만든다.

생성하는 `Authentication` 타입은 `AbstractAuthenticationProcessingFilter` 하위 클래스에 따라 다르다.

예를 들어, `UsernamePasswordAuthenticationFilter`는 `HttpServletRequest`에 있는 Username과 Password로 `UsernamePasswordAuthenticationToken`을 생성한다.



**(2):** 그 다음엔 `Authentication`을 `AuthenticationManager`로 넘겨서 인증한다.



**(3):** 인증에 실패하면 (Failure)

- `SecurityContextHolder`를 비운다.
- `RemeberMeServices.loginFail`을 실행한다. Remember me를 설정하지 않았다면 아무 동작도 하지 않는다.
- `AuthenticationFailureHandler`를 실행한다.



**(4): **인증에 성공하면 (Success)

- `SessionAuthenticationStrategy`에 새로 로그인했음을 통보한다.

- `SecurityContextHolder`에 `Authentication`을 세팅한다. 

  이후, `SecurityContextPersistenceFilter`가 `HttpSession`에 `SecurityContext`를 저장한다.

- `RememberMeServices.loginSuccess`를 실행한다. Remember me를 설정하지 않았다면 아무 동작도 하지 않는다.

- `ApplicationEventPublisher`는 `InteractiveAuthenticationSuccessEvent`를 발생시킨다.