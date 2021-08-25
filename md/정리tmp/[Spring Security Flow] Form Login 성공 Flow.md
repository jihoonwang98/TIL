# [Spring Security Flow] Form Login 성공 Flow



![](https://godekdls.github.io/images/springsecurity/filterchainproxy.png)





### DelegatingFilterProxy

제일 먼저 `DelegatingFilterProxy` 객체의 `doFilter` 메서드에서 흐름이 시작된다.

```java
public class DelegatingFilterProxy extends GenericFilterBean {
    
    ...
    
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain filterChain) throws ServletException, IOException{
        
        
        //==  DelegatingFilterProxy가 위임할 객체를 찾는다. ==//
        Filter delegateToUse = this.delegate;
        if (delegateToUse == null) {
            synchronized(this.delegateMonitor) {
                ...
                    
                this.delegate = delegateToUse;
            }
        }
        // 위의 코드를 수행하면 this.delegate에 FilterChainProxy 객체가 들어간다.
        

        // delegate(FilterChainProxy) 객체가 filter 작업을 수행하게 한다.
        this.invokeDelegate(delegateToUse, request, response, filterChain);
        
    }
    
    
    ...
}
```



`this.invokeDelegate(...)`을 실행하면 `this.delegate` 객체의 `doFilter`가 실행된다.

여기서는 `FilterChainProxy` 클래스의 `doFilter`가 실행된다.



### FilterChainProxy

```java
public class FilterChainProxy extends GenericFilterBean {
    
    ... 
        
    // SecurityFilterChain들이 들어있는 리스트.
    // SecurityFilterChain 하나는 여러개의 Filter들을 가지고 있다.
    private List<SecurityFilterChain> filterChains;
    ...
        
        
    // 실제 Filter 작업은 doFilterInternal에서 하는 듯 하다.
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
 
        boolean clearContext = request.getAttribute(FILTER_APPLIED) == null;
        
        if (!clearContext) {
            this.doFilterInternal(request, response, chain);
        } else {
            try {
                request.setAttribute(FILTER_APPLIED, Boolean.TRUE);
                this.doFilterInternal(request, response, chain);
            } catch (RequestRejectedException var9) {
                ...
            } finally {
                ...
            }

        }
    }
    
    
    
    private void doFilterInternal(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        
        ...
            
        List<Filter> filters = this.getFilters((HttpServletRequest)firewallRequest);
        
        
        if (filters != null && filters.size() != 0) {
            ...
            
            // SecurityFilterChain의 Filter들로 구성된 가상의 FilterChain을 만든다.
            FilterChainProxy.VirtualFilterChain virtualFilterChain = new FilterChainProxy.VirtualFilterChain(firewallRequest, chain, filters);
            
            
            // 가상 FilterChain의 필터링 작업을 수행한다.
            virtualFilterChain.doFilter(firewallRequest, firewallResponse);
            
            
        } else {
            ...

                
            firewallRequest.reset();
            chain.doFilter(firewallRequest, firewallResponse);
        }
    }
    
    ...
        
        
        
    private static final class VirtualFilterChain implements FilterChain {
        private final FilterChain originalChain;
        private final List<Filter> additionalFilters;
        private final FirewalledRequest firewalledRequest;
        private final int size;
        private int currentPosition;
        
        ...
            
            
        
        public void doFilter(ServletRequest request, ServletResponse response) throws IOException, ServletException {
            if (this.currentPosition == this.size) {
                
                ...
                    
                this.originalChain.doFilter(request, response);
                
            } else {
                ++this.currentPosition;
                
                
                // 현재 SecurityFilterChain의 다음 Filter를 선택한다.
                Filter nextFilter = (Filter)this.additionalFilters.get(this.currentPosition - 1);
                
                
                ...
                    
                
                // 다음 Filter의 필터링 작업을 수행한다.
                nextFilter.doFilter(request, response, this);
            }
        }
    }
}
```



위의 `nextFilter.doFilter(request, response, this)` 명령문에서 

여러가지 Filter들의 `doFilter` 문을 실행하다가 

`UsernamePasswordAuthenticationFilter` 차례가 오면 Authentication 처리를 하러간다.





### AbstractAuthenticationProcessingFilter

구현체는 `UsernamePasswordAuthenticationFilter`이다.

```java
public abstract class AbstractAuthenticationProcessingFilter extends GenericFilterBean implements ApplicationEventPublisherAware, MessageSourceAware {
    
    ...
        
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        this.doFilter((HttpServletRequest)request, (HttpServletResponse)response, chain);
    }

    
    private void doFilter(HttpServletRequest request, HttpServletResponse response, FilterChain chain) throws IOException, ServletException {
        if (!this.requiresAuthentication(request, response)) {
            chain.doFilter(request, response);
        } else {
            try {
 
                
                // 아래 명령문에서 UsernamePasswordAuthenticationFilter의 attemptAuthentication이 실행된다.
                Authentication authenticationResult = this.attemptAuthentication(request, response);

                
                
                
        }
    }

    ...
    
}
```







### UsernamePasswordAuthenticationFilter

```java
public class UsernamePasswordAuthenticationFilter extends AbstractAuthenticationProcessingFilter {

   public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response) throws AuthenticationException {
        if (this.postOnly && !request.getMethod().equals("POST")) {
            throw new AuthenticationServiceException("Authentication method not supported: " + request.getMethod());
        } else {
            String username = this.obtainUsername(request);
            username = username != null ? username : "";
            username = username.trim();
            String password = this.obtainPassword(request);
            password = password != null ? password : "";
            
            // 여기서 UsernamePasswordAuthenticationToken이 생성됨!!!!
            UsernamePasswordAuthenticationToken authRequest = new UsernamePasswordAuthenticationToken(username, password);
            
            this.setDetails(request, authRequest);
            
            return this.getAuthenticationManager().authenticate(authRequest);
        }
    }
    
    @Nullable
    protected String obtainPassword(HttpServletRequest request) {
        // passwordParamter == "password" (디폴트 값일 때)
        return request.getParameter(this.passwordParameter);
    }

    @Nullable
    protected String obtainUsername(HttpServletRequest request) {
        // usernameParameter == "username" (디폴트 값일 때)
        return request.getParameter(this.usernameParameter);
    }
    
    
    protected void setDetails(HttpServletRequest request, UsernamePasswordAuthenticationToken authRequest) {
        authRequest.setDetails(this.authenticationDetailsSource.buildDetails(request));
    }
    
}
```



위 코드에서 막 생성한 `UsernamePasswordAuthenticationToken` 안에는 다음과 같은 값들이 들어 있다.

```java
UsernamePasswordAuthenticationToken authRequest;

authRequest.principal == String 타입 "user1" (내가 username에 입력한 값)
authRequest.credentials == String 타입 "asdf" (내가 password에 입력한 값)
authRequest.authorities == EmptyList
authRequest.details == null
authRequest.authenticated == false
```



여기서 `this.setDetails(request, authRequest)` 명령문을 수행하고 나면,

```java
authRequest.details == WebAuthenticationDetails 타입 

authRequest.details.remoteAddress == "0:0:0:0:0:0:0:1"
authRequest.details.sessionId == "15CFAB9FECFB193F37B2B1B85EC9925C"
```

위와 같은 값들이 들어 있게 된다.





그 다음에 `return this.getAuthenticationManager().authenticate(authRequest)` 문이 실행된다.

현재 `AuthenticationManager`(인터페이스)에는 `ProviderManager` 객체가 들어가 있다.



![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcrSvxl%2FbtqFyRvClqL%2F217ksrm8e4nBJMdC7qlH3K%2Fimg.png)





### ProviderManager

```java
public Authentication authenticate(Authentication authentication) throws AuthenticationException {
    ...
        
    Iterator var9 = this.getProviders().iterator();

    while(var9.hasNext()) {
        AuthenticationProvider provider = (AuthenticationProvider)var9.next();
        if (provider.supports(toTest)) {
            ...
                
            try {
                result = provider.authenticate(authentication);
                ...
            } catch(...) {
                ...
            }
            
        }
    }

    if (result == null && this.parent != null) {
        try {
            parentResult = this.parent.authenticate(authentication);
            ...
        } catch(...) {
            ...
        }
    }
    
	...
}
```



여기서 `ProviderManager`가 `DaoAuthenticationProvider`를 선택해서 `provider.authenticate(authentication)` 명령문을 실행한다.





### AbstractUserDetailsAuthenticationProvider

현재 `DaoAuthenticationProvider`가 처리 중인데 

코드를 `AbstractUserDetailsAuthenticationProvider`로부터 상속받은 거임.



```java
public Authentication authenticate(Authentication authentication) throws AuthenticationException {
    
    ...
        
    String username = this.determineUsername(authentication);
    boolean cacheWasUsed = true;
    
    UserDetails user = this.userCache.getUserFromCache(username);
    
    if (user == null) {
        cacheWasUsed = false;

        
        // UserDetails 얻어오는 try 문
        try {
            
            // this.retrieveUser의 세부 구현은 
            // 구현 클래스인 DaoAuthenticationProvider 클래스에 있다.
            // 일단 여기로 들어간다.
            user = this.retrieveUser(username, (UsernamePasswordAuthenticationToken)authentication);
            ...
                
        }
    }
        
        
    // PreAuthentication 체크(유저 계정이 Locked, Enabled, NonExpired 됐는지 체크하는 거임)
    try {
            this.preAuthenticationChecks.check(user);
            this.additionalAuthenticationChecks(user, (UsernamePasswordAuthenticationToken)authentication);
    } catch (AuthenticationException var7) {
            
        if (!cacheWasUsed) {
                throw var7;
        }

        cacheWasUsed = false;
        user = this.retrieveUser(username, (UsernamePasswordAuthenticationToken)authentication);        
        this.preAuthenticationChecks.check(user);        
        this.additionalAuthenticationChecks(user, (UsernamePasswordAuthenticationToken)authentication);
    }
    
    
    
    // PostAuthentication 체크(Credential이 NonExpired인지 체크한다.)
    this.postAuthenticationChecks.check(user);
    
    if (!cacheWasUsed) {
        this.userCache.putUserInCache(user);
    }

    
    Object principalToReturn = user;
    if (this.forcePrincipalAsString) {
        principalToReturn = user.getUsername();
    }

    return this.createSuccessAuthentication(principalToReturn, authentication, user);
}
```





### DaoAuthenticationProvider

```java
protected final UserDetails retrieveUser(String username, UsernamePasswordAuthenticationToken authentication) throws AuthenticationException {
    this.prepareTimingAttackProtection();

    try {
        // UserDetailsService에 정의된 loadUserByUsername 메서드로 UserDetails를 불러온다.
        UserDetails loadedUser = this.getUserDetailsService().loadUserByUsername(username);
        
        if (loadedUser == null) {
            throw new InternalAuthenticationServiceException(...);
        } else {
            
            // 잘 불러왔으면 여기서 return함.
            return loadedUser;
        }
    } catch (UsernameNotFoundException var4) {
        this.mitigateAgainstTimingAttack(authentication);
        throw var4;
    } catch (InternalAuthenticationServiceException var5) {
        throw var5;
    } catch (Exception var6) {
        throw new InternalAuthenticationServiceException(...);
    }
}
```







### AbstractUserDetailsAuthenticationProvider

```java
public Authentication authenticate(Authentication authentication) throws AuthenticationException {
    
		... 
		try {
            
            // 받고 다시 돌아옴.
            user = this.retrieveUser(username, (UsernamePasswordAuthenticationToken)authentication);
            
            ...
        } 
    ...
        
    
    
	// PreAuthentcation 체크 후 (User 계정 Enable, Locked, NonExpired 체크)
    try {
        this.preAuthenticationChecks.check(user);
        this.additionalAuthenticationChecks(user, (UsernamePasswordAuthenticationToken)authentication);
    } catch (AuthenticationException var7) {
        if (!cacheWasUsed) {
            throw var7;
        }

        cacheWasUsed = false;
        user = this.retrieveUser(username, (UsernamePasswordAuthenticationToken)authentication);
        this.preAuthenticationChecks.check(user);
        this.additionalAuthenticationChecks(user, (UsernamePasswordAuthenticationToken)authentication);
    }

    
    // PostAuthentication 체크한다. (User 계정 credentials NonExpired인지 체크)
    this.postAuthenticationChecks.check(user);
    
    
    if (!cacheWasUsed) {
        this.userCache.putUserInCache(user);
    }

    Object principalToReturn = user;
    if (this.forcePrincipalAsString) {
        principalToReturn = user.getUsername();
    }

    
    return this.createSuccessAuthentication(principalToReturn, authentication, user);
}



private class DefaultPreAuthenticationChecks implements UserDetailsChecker {
        private DefaultPreAuthenticationChecks() {
        }

        public void check(UserDetails user) {
            if (!user.isAccountNonLocked()) {
                throw new LockedException(...);
            } else if (!user.isEnabled()) {
                throw new DisabledException(...);
            } else if (!user.isAccountNonExpired()) {
                throw new AccountExpiredException(...);
            }
        }
    
}


private class DefaultPostAuthenticationChecks implements UserDetailsChecker {
        private DefaultPostAuthenticationChecks() {
        }

        public void check(UserDetails user) {
            if (!user.isCredentialsNonExpired()) {
                throw new CredentialsExpiredException(...);
            }
        }
}


// 여기서 마지막으로 SuccessAuthentication을 만들어서 리턴한다.
protected Authentication createSuccessAuthentication(Object principal, Authentication authentication, UserDetails user) {
    
        UsernamePasswordAuthenticationToken result = new UsernamePasswordAuthenticationToken(principal, authentication.getCredentials(), this.authoritiesMapper.mapAuthorities(user.getAuthorities()));
        result.setDetails(authentication.getDetails());
        this.logger.debug("Authenticated user");
        return result;
   
}
// 여기까지는 아직 authentication.authenticated == false임.

// 여기서 리턴해서 ProviderManager로 다시 돌아간다.
```



무튼 인증댐



