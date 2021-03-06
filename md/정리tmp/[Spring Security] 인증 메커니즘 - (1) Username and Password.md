# [Spring Security] 인증 메커니즘 - (1) Username and Password



Username과 Password를 통한 인증 방식은 사용자를 인증할 때 가장 많이 사용하는 방법 중 하나다.

그렇기 때문에 스프링 시큐리티는 이름과 비밀번호로 인증할 수 있는 방법을 종합적으로 지원한다.



### Reading the Username & Password

스프링 시큐리티는 `HttpServletRequest`에서 이름과 비밀번호를 읽을 수 있는 다음과 같은 메커니즘을 기본으로 제공한다.

- Form Login
- Basic Authentication
- Digest Authentication





### Storage Mechanisms

Username/Password 조회 메커니즘은 지원하는 저장 메커니즘 중 어떤 것과도 조합할 수 있다.

- In-Memory Authentication과 Simple Storage
- JDBC Authentication과 관계형 DB
- UserDetailsService와 커스텀 데이터 스토어
- LDAP Authentication과 LDAP 스토리지







## 1. Form Login

스프링 시큐리티는 HTML 폼 기반 Username/Password 인증을 지원한다.

이번 섹션에서는 스프링 시큐리티의 폼 기반 인증 동작 방식을 설명한다.



스프링 시큐리티에서 폼 기반 로그인이 어떻게 동작하는지 살펴보자.

먼저 로그인 폼으로 리다이렉트하는 방법을 설명한다.



![](https://godekdls.github.io/images/springsecurity/loginurlauthenticationentrypoint.png)



**(1): **먼저 사용자가 권한이 없는 리소스 `/private`에 인증되지 않은 요청을 보낸다.

**(2): **스프링 시큐리티의 `FilterSecurityInterceptor`에서 `AccessDeniedException`을 던져 인증되지 않은 요청을 거절했음을 알린다.

**(3): **인증되지 않은 사용자이므로 `ExceptionTranslactionFilter`에서 인증을 시작하고, 설정한 `AuthenticationEntryPoint`로 로그인 페이지로의 리다이렉트 응답을 전송한다.

`AuthenticationEntryPoint`는 대부분 `LoginUrlAuthenticationEntryPoint` 인스턴스다.

**(4): **그러면 브라우저는 리다이렉트된 로그인 페이지를 요청한다.

**(5): **애플리케이션에선 로그인 페이지를 렌더링해야 한다.





Username과 Password를 제출하면 `UsernamePasswordAuthenticationFilter`가 이 값을 인증한다.

`UsernamePasswordAuthenticationFilter`는 `AbstractAuthenticationProcessingFilter`를 상속했기 때문에 다이어그램도 비슷하다.



![](https://godekdls.github.io/images/springsecurity/usernamepasswordauthenticationfilter.png)



**(1): **사용자가 Username과 Password를 제출하면 `UsernamePasswordAuthenticationFilter`는 `HttpServletRequest`에서 이 값을 추출해 `Authentication` 유형 중 하나인 `UsernamePasswordAuthenticationToken`을 만든다.



**(2): **그 다음엔 `UsernamePasswordAuthenticationToken`을 `AuthenticationManager`로 넘겨 인증한다.

`AuthenticationManager` 상세 동작은 사용자 정보를 저장하는 방식에 따라 다르다.



**(3): **인증에 실패하면 (Failure)

- `SecurityContextHolder`를 비운다.
- `RememberMeServices.loginFail`을 실행한다. Remember me를 설정하지 않았다면 아무 동작도 하지 않는다.
- `AuthenticationFailureHandler`를 실행한다.



**(4): **인증에 성공하면 (Success)

- `SessionAuthenticationStrategy`에 새로 로그인했음을 통보한다.
- `SecurityContextHolder`에 `Authentication`을 세팅한다.
- `RememberMeServices.loginSuccess`를 실행한다. Remember me를 설정하지 않았다면 아무 동작도 하지 않는다.
- `ApplicationEventPublisher`는 `InteractiveAuthenticationSuccessEvent`를 발생시킨다.
- `AuthenticationSuccessHandler`를 실행한다. 보통 로그인 페이지로 리다이렉트할 때는 `SimpleUrlAuthenticationSuccessHandler`가 `ExceptionTranslationFilter`에 저장된 요청으로 리다이렉트한다.





스프링 시큐리티에선 폼 로그인이 디폴트로 활성화된다. 

하지만 서블릿 기반 설정을 사용한다면 폼 기반 로그인을 명시해야 한다. 

최소한 아래와 같은 설정이 있어야 한다.



**[예시] 폼 로그인 설정**

```java
protected void configure(HttpSecurity http) {
	http
		// ...
		.formLogin(withDefaults());
}
```

이 설정에선 디폴트 로그인 페이지로 렌더링한다. 

프로덕션에서 사용할 애플리케이션은 대부분 커스텀 로그인 폼이 필요하다.







커스텀 로그인 폼을 설정하는 방법은 아래와 같다.

**[예시] 커스텀 로그인 폼 설정**

```java
protected void configure(HttpSecurity http) throws Exception{
	http
		// ...
		.formLogin(form -> form
        	.loginPage("/login")
            .permitAll()
        );
}
```

스프링 시큐리티 설정에 로그인 페이지를 명시했다면 페이지 렌더링을 직접 구현해야 한다.



다음은 로그인 페이지 `/login`에서 필요한 HTML 로그인 폼을 생성하는 Thymeleaf 템플릿이다.

**[예시] Thymeleaf 로그인 폼 예시**

```html
<!-- src/main/resources/templates/login.html --> 
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml" 
      xmlns:th="https://www.thymeleaf.org">
    
    <head>
        <title>Please Log In</title>
    </head>
    
    <body>
        <h1>Please Log In</h1>
        <div th:if="${param.error}">
            Invalid username and password.</div>
        <div th:if="${param.logout}">
            You have been logged out.</div>
        <form th:action="@{/login}" method="post">
            <div>
            <input type="text" name="username" placeholder="Username"/>
            </div>
            <div>
            <input type="password" name="password" placeholder="Password"/>
            </div>
            <input type="submit" value="Log in" />
        </form>
    </body>
    
</html>
```

디폴트 HTML 폼은 몇 가지 핵심 규칙을 따른다.

- `/login`에 `POST` 요청을 보내야 한다.
- <u>CSRF 토큰</u>을 포함해야 하며, 타임리프에서는 자동으로 추가된다.
- 사용자 이름은 `username` 파라미터로 명시해야 한다.
- 비밀번호는 `password` 파라미터로 명시해야 한다.
- HTTP 파라미터 error가 있으면 사용자가 유효한 username/password를 제공하지 못했음을 나타낸다.
- HTTP 파라미터 logout이 있으면 사용자가 로그아웃에 성공한 것을 나타낸다.



스프링 MVC를 사용한다면 `GET /login` 요청을 직접 만든 로그인 템플릿으로 매핑하는 컨트롤러가 필요하다.





## 2. Basic Authentication

이번 섹션에서는 스프링 시큐리티가 어떻게 서블릿 기반 애플리케이션에서 기본 HTTP 인증을 지원하는지 다룬다.



스프링 시큐리티에서 HTTP 기본 인증이 어떻게 동작하는지 살펴보자. 

먼저 인증되지 않은 클라이언트에게 WWW-Authenticate 헤더를 다시 전송하는 것을 확인할 것이다.



![](https://godekdls.github.io/images/springsecurity/basicauthenticationentrypoint.png)



**(1): **먼저 사용자가 권한이 없는 리소스 `/private`에 인증되지 않은 요청을 보낸다.

**(2): **스프링 시큐리티의 `FilterSecurityInterceptor`에서 `AccessDeniedException`을 던져 인증되지 않은 요청을 거절했음을 알린다.

**(3): **인증되지 않은 사용자이므로 `ExceptionTranslationFilter`에서 인증을 시작한다. 설정한 `AuthenticationEntryPoint`는 `BasicAuthenticationEntryPoint` 인스턴스로, WWW-Authenticate 헤더를 전송한다. 이때는 클라이언트가 기존 요청을 다시 전송할 수 있으므로, `RequestCache`는 보통 요청을 저장하지 않는 `NullRequestCache`를 사용한다.





클라이언트는 WWW-Authenticate 헤더를 받으면 username과 password로 재시도해야 한다는 것을 알고 있다.

다음은 username과 password를 처리하는 플로우다.



![](https://godekdls.github.io/images/springsecurity/basicauthenticationfilter.png)



**(1):** 사용자가 username과 password를 제출하면 `UsernamePasswordAuthenticationFilter`는 `HttpServletRequest`에서 이 값을 추출해 `Authentication` 유형 중 하나인 `UsernamePasswordAuthenticationToken`을 만든다.



**(2):** 그 다음엔 `UsernamePasswordAuthenticationToken`을 `AuthenticationManager`로 넘겨 인증한다. `AuthenticationManager` 상세 동작은 사용자 정보를 저장한 방식에 따라 다르다.



**(3):** 인증에 실패하면 (Failure)

- `SecurityContextHolder`를 비운다.
- `RememberMeServices.loginFail`을 실행한다. Remember me를 설정하지 않았다면 아무 동작도 하지 않는다.
- `AuthenticationEntryPoint`를 실행해서 WWW-Authenticate 전송을 트리거한다.



**(4):** 인증에 성공하면 (Success)

- `SecurityContextHolder`에 `Authentication`을 세팅한다.
- `RememberMeServices.loginSuccess`을 실행한다. Remember me를 설정하지 않았다면 아무 동작도 하지 않는다.
- `BasicAuthenticationFilter`에서 `FilterChain.doFilter(request, response)`를 호출해서 나머지 애플리케이션 로직을 실행한다.





스프링 시큐리티에선 HTTP 기본 인증을 디폴트로 활성화한다. 하지만 서블릿 기반 설정을 하나라도 사용하고 있다면 HTTP Basic을 명시해야 한다.



다음 코드는 최소한의 설정이다.

**[예시] 명시적으로 HTTP Basic 설정하기**

```java
protected void configure(HttpSecurity http) {
	http
		// ...
		.httpBasic(withDefaults());
}
```





## 3. Digest Authentication

(생략)





## 4. In-Memory Authentication

스프링 시큐리티의 `InMemoryUserDetailsManager`는 메모리 기반으로 username/password를 인증하는 `UserDetailsService` 구현체다.

`InMemoryUserDetailsManager`는 `UserDetailsManager` 인터페이스도 구현했기 때문에 `UserDetails`를 관리할 수 있다.

username/password를 읽어 인증하도록 설정하면 스프링 시큐리티는 `UserDetails`를 통해 인증한다.





이 예제에선 스프링 부트 CLI로 비밀번호 `password`를 인코딩했으며, 인코딩 된 값 `{bcrypt}$2a$10$GRLdNijSQMUvl/au9ofL.eDwmoohzzS7.rmNSJZ.0FxO/BTk76klW`를 얻었다.

**[예제] InMemoryUserDetailsManager 자바 설정**

```java
@Bean
public UserDetailsService users() {
    UserDetails user = User.builder()
        .username("user")
        .password("{bcrypt}$2a$10$GRLdNijSQMUvl/au9ofL.eDwmoohzzS7.rmNSJZ.0FxO/BTk76klW")
        .roles("USER")
        .build();
    UserDetails admin = User.builder()
        .username("admin")
        .password("{bcrypt}$2a$10$GRLdNijSQMUvl/au9ofL.eDwmoohzzS7.rmNSJZ.0FxO/BTk76klW")
        .roles("USER", "ADMIN")
        .build();
    return new InMemoryUserDetailsManager(user, admin);
}
```

위 예제는 비밀번호를 안전한 포맷으로 저장하지만, 아쉬운 점이 많다.





아래의 예제는 `User.withDefaultPasswordEncoder`로 메모리에 저장할 비밀번호를 보호한다.

하지만 소스 코드를 디컴파일하면 비밀번호를 쉽게 탈취할 수 있다.

따라서 `User.withDefaultPasswordEncoder`는 스프링 시큐리티를 처음 접할 때만 사용해야 하며, 프로덕션 코드에는 사용하면 안 된다.



**[예제] InMemoryUserDetailsManager with User.withDefaultPasswordEncoder**

```java
@Bean
public UserDetailsService users() {
    // The builder will ensure the passwords are encoded before saving in memory
    UserBuilder users = User.withDefaultPasswordEncoder();
    UserDetails user = users
        .username("user")
        .password("password")
        .roles("USER")
        .build();
    UserDetails admin = users
        .username("admin")
        .password("password")
        .roles("USER", "ADMIN")
        .build();
    return new InMemoryUserDetailsManager(user, admin);
}
```





## 5. JDBC Authentication

스프링 시큐리티의 `JdbcDaoImpl`은 JDBC 기반으로 username/password를 인증하는 `UserDetailsService` 구현체다.

`JdbcDaoImpl`을 상속한 `JdbcUserDetailsManager`는 `UserDetailsManager` 인터페이스도 구현했기 때문에 `UserDetails`를 관리할 수 있다.

username/password를 읽어 인증하도록 설정하면 스프링 시큐리티는 `UserDetails`를 통해 인증한다.



아래에서는 다음과 같은 내용을 다룬다.

- 스프링 시큐리티 JDBC 인증에서 사용하는 디폴트 스키마
- 데이터 소스 설정
- JdbcUserDetailsManager 빈





### Default Schema

스프링 시큐리티는 JDBC 기반 인증을 위한 기본 쿼리를 제공한다. 

여기에선 디폴트 쿼리에서 사용되는 디폴트 스키마를 다룬다. 

쿼리나 데이터베이스 방언(dialect)을 커스텀한다면 스티마도 함께 바꿔야 한다.





#### User Schema

`JdbcDaoImpl`에서 사용자의 비밀번호, 계정 상태 (활성화/비활성화), 권한 (roles) 리스트를 로드하려면 테이블이 있어야 한다. 

필요한 디폴트 사용자 스키마는 다음과 같다.



> 디폴트 스키마는 classpath 리소스 `org/springframework/security/core/userdetails/jdbc/users.ddl`로도 접근할 수 있다.





**[예제] Default User Schema**

```sql
create table users(
    username varchar_ignorecase(50) not null primary key,
    password varchar_ignorecase(50) not null,
    enabled boolean not null
);

create table authorities (
    username varchar_ignorecase(50) not null,
    authority varchar_ignorecase(50) not null,
    constraint fk_authorities_users foreign key(username) references users(username)
);

create unique index ix_auth_username on authorities (username,authority);
```





#### Group Schema

그룹을 사용하는 애플리케이션은 그룹 스키마도 필요하다.

디폴트 그룹 스키마는 다음과 같다.



**[예제] Default Group Schema**

```sql
create table groups (
    id bigint generated by default as identity(start with 0) primary key,
    group_name varchar_ignorecase(50) not null
);

create table group_authorities (
    group_id bigint not null,
    authority varchar(50) not null,
    constraint fk_group_authorities_group foreign key(group_id) references groups(id)
);

create table group_members (
    id bigint generated by default as identity(start with 0) primary key,
    username varchar(50) not null,
    group_id bigint not null,
    constraint fk_group_members_group foreign key(group_id) references groups(id)
);
```







### Setting up a DataSource

`JdbcUserDetailsManager`를 설정하려면 먼저 `DataSoucre`가 있어야 한다. 

이 예제에서는 Default User Schema로 초기화하는 Embedded Data Source를 설정한다.



**[예제] Embedded Data Source**

```sql
@Bean
DataSource dataSource() {
    return new EmbeddedDatabaseBuilder()
        .setType(H2)
        .addScript("classpath:org/springframework/security/core/userdetails/jdbc/users.ddl")
        .build();
}
```



프로덕션 환경에선 외부 DB 커넥션을 설정해야 한다.







### JdbcUserDetailsManager Bean

이 예제에선 스프링 부트 CLI로 비밀번호 `password`를 인코딩했으며, 인코딩된 값 `{bcrypt}$2a$10$GRLdNijSQMUvl/au9ofL.eDwmoohzzS7.rmNSJZ.0FxO/BTk76klW`를 얻었다.



**[예제] JdbcUserDetailsManager**

```java
@Bean
UserDetailsManager users(DataSource dataSource) {
    UserDetails user = User.builder()
        .username("user")
        .password("{bcrypt}$2a$10$GRLdNijSQMUvl/au9ofL.eDwmoohzzS7.rmNSJZ.0FxO/BTk76klW")
        .roles("USER")
        .build();
    UserDetails admin = User.builder()
        .username("admin")
        .password("{bcrypt}$2a$10$GRLdNijSQMUvl/au9ofL.eDwmoohzzS7.rmNSJZ.0FxO/BTk76klW")
        .roles("USER", "ADMIN")
        .build();
    JdbcUserDetailsManager users = new JdbcUserDetailsManager(dataSource);
    users.createUser(user);
    users.createUser(admin);
    return users;
}
```





# 6. UserDetails

`UserDetails`는 `UserDetailsService`가 리턴하는 값이다.

`DaoAuthenticationProvider`가 `UserDetails`를 인증하고, 이 `UserDetails`를 principal로 가진 `Authentication`을 리턴한다.





# 7. UserDetailsService

`UserDetailsService`는 `DaoAuthenticationProvider`가 username/password로 인증할 때 필요한 username, password와 다른 속성을 조회할 때 사용한다.

스프링 시큐리티가 제공하는 `UserDetailsService`는 인메모리와 JDBC 기반 구현체가 있다.



커스텀 인증을 정의하려면 커스텀 `UserDetailsService`를 빈으로 만들면 된다.

예를 들어, `CustomUserDetailsService`가 `UserDetailsService`를 구현했다고 가정하고, 인증을 커스텀해보자.



> `AuthenticationManagerBuilder`를 사용하지 않고 `AuthenticationProviderBean` 빈도 정의하지 않았을 때 사용하는 방법이다.



**[예제] Custom UserDetailsService 빈**

```java
@Bean
CustomUserDetailsService customUserDetailsService() {
    return new CustomUserDetailsService();
}
```





# 8. PasswordEncoder

서블릿에서 스프링 시큐리티를 사용하면 `PasswordEncoder`를 통합해 비밀번호를 안전하게 저장할 수 있다.

스프링 시큐리티가 사용하는 `PasswordEncoder` 구현체를 커스텀하려면 `PasswordEncoder` 빈을 정의하면 된다.





# 9. DaoAuthenticationProvider

`DaoAuthenticationProvider`는 `UserDetailsService`와 `PasswordEncoder`로 username/password를 인증하는 `AuthenticationProvider` 구현체다.



스프링 시큐리티에서 `DaoAuthenticationProvider`가 동작하는 방식을 살펴보자.

다음은 Username & Password 조회를 설명할 때 다룬 이미지에서 `AuthenticationManager`가 동작하는 방식을 더 상세히 나타낸 그림이다.



![](https://godekdls.github.io/images/springsecurity/daoauthenticationprovider.png)



**(1): **Username & Password를 조회하는 인증 `Filter`에서 `UsernamePasswordAuthenticationToken`을 `AuthenticationManager`로 넘긴다.

`AuthenticationManager`는 `ProviderManager`가 구현하고 있다.



**(2):** 이 `ProviderManager`는 `DaoAuthenticationProvider`를 `AuthenticationProvider`로 사용하도록 설정돼 있다.



**(3):** `DaoAuthenticationProvider`는 `UserDetailsService`에서 `UserDetails`를 조회한다.



**(4):** 그 다음 `DaoAuthenticationProvider`는 이전 단계에서 얻은 `UserDetails`에 있는 비밀번호를 `PasswordEncoder`로 검증한다.



**(5): **인증에 성공하면 `UsernamePasswordAuthenticationToken` 타입의 `Authentication`을 반환하며, 이 객체는 `UserDetailsService`가 리턴한 `UserDetails`를 principal로 가지고 있다. 결국에 리턴한 `UsernamePasswordAuthenticationToken`은 인증 `Filter`에서 `SecurityContextHolder`로 세팅된다.







(이후 생략)ㄴ
