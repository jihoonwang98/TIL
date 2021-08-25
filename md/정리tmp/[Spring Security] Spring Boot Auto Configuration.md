# [Spring Security] Spring Boot Auto Configuration

스프링 부트는 다음과 같은 일을 자동으로 해준다.



- 스프링 시큐리티의 디폴트 설정을 활성화해서 `springSecurityFilterChain`이라는 이름의 서블릿 `Filter` 빈을 생성한다.
  - 이 빈이 애플리케이션 내의 모든 보안 처리를 담당한다. (애플리케이션 URL 보호, 제출한 Username/Password 검증, 로그인 폼으로 리다이렉트 등).
- `user`라는 사용자 이름과 콘솔에도 출력되는 랜덤 생성한 비밀번호를 가지고 있는 `UserDetailsService` 빈을 만든다.
- 서블릿 컨테이너에 `springSecurityFilterChain`이란 이름의 `Filter` 빈을 등록해 모든 요청에 적용한다.





스프링 부트 설정은 간단하지만 많은 일을 해준다. 기능을 요약하면 다음과 같다.

- 애플리케이션의 모든 상호작용에 사용자 인증 요구
- 디폴트 로그인 폼 생성
- `user`라는 이름과 콘솔에 출력한 비밀번호를 사용한 폼 기반 인증 지원 
- BCrypt로 저장할 비밀번호 보호
- 사용자 로그아웃 지원
- CSRF 공격 방어
- Session Fixation 방어
- 보안 헤더 통합
  - HTTP Strict Transport Security로 요청을 보호
  - X-Content-Type-Options 통합
  - Cache Control (애플리케이션에서 특정 스태틱 리소스에 캐시를 허용하도록 재정의할 수 있다)
  - X-XSS-Protection 통합
  - X-Frame-Options 통합으로 클릭재킹 방어 지원
- 서블릿 API 메서드 통합
  - `HttpServletRequest#getRemoteUser()`
  - `HttpServletRequest.html#getUserPrincipal()`
  - `HttpServletRequest.html#isUserInRole(String)`
  - `HttpServletRequest.html#login(String, String)`
  - `HttpServletRequest.html#logout()`

