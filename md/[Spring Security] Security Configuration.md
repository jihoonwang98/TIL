# [Spring Security] Security Configuration



**용어**

- Authorization: 권한 (내가 할 수 있는게 뭐다)
- Authentication: 인증 (내가 내다)



```java
@Configuration
@EnableWebSecurity  // 스프링 시큐리티 필터가 스프링 필터체인에 등록이 된다.
@EnableGlobalMethodSecurity(securedEnabled = true, prePostEnabled = true)  // Secured 애너테이션 활성화, PreAuthorize 애너테이션 활성화
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
    

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.csrf().disable();

        http
                .authorizeRequests()
                    .antMatchers("/user/**").authenticated()  // 해당 URL은 인증필요
                    .antMatchers("/manager/**").access("hasRole('ROLE_ADMIN') or hasRole('ROLE_MANAGER')")  // 접근하려면 ROLE_ADMIN 또는 ROLE_MANAGER 권한 필요
                    .antMatchers("/admin/**").access("hasRole('ROLE_MANAGER')")
                    .anyRequest().permitAll()
                .and()
                .formLogin()
                    .loginPage("/loginForm")   // 얘를 설정해줘야 인증이 필요한 요청이 들어오면 loginPage로 자동으로 이동된다. 그리고 loginPage는 말 그대로 로그인 페이지(loginForm) URL를 설정하는 부분이다
            		.loginProcessingUrl("/login") // 실제 로그인이 처리되는 URL를 설정하는 부분. POST /login으로 요청이 들어오면 login을 스프링 시큐리티가 처리해준다. 내가 직접 컨트롤러에서 매핑안해줘도 됨.
            		.defaultSuccessUrl("/", false); // 로그인이 성공하면 이동할 URL를 설정한다. 두 번째 파라미터를 false로 설정하면 원래 이동하려고 했던 페이지로 이동시켜준다.
        
    }
}

```

