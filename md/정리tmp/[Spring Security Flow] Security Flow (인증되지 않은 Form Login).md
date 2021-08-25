# [Spring Security] Security Flow (인증되지 않은 Form Login)





![](https://godekdls.github.io/images/springsecurity/multi-securityfilterchain.png)







## 1. Form Login



### 인증되지 않은 요청은 Login Form으로 리다이렉트

![](https://godekdls.github.io/images/springsecurity/loginurlauthenticationentrypoint.png)



**(1): **먼저 사용자가 권한이 없는 리소스 `/private`에 인증되지 않은 요청을 보낸다.

**(2): **스프링 시큐리티의 `FilterSecurityInterceptor`에서 `AccessDeniedException`을 던져 인증되지 않은 요청을 거절했음을 알린다.

**(3): **인증되지 않은 사용자이므로 `ExceptionTranslactionFilter`에서 인증을 시작하고, 설정한 `AuthenticationEntryPoint`로 로그인 페이지로의 리다이렉트 응답을 전송한다.

`AuthenticationEntryPoint`는 대부분 `LoginUrlAuthenticationEntryPoint` 인스턴스다.

**(4): **그러면 브라우저는 리다이렉트된 로그인 페이지를 요청한다.

**(5): **애플리케이션에선 로그인 페이지를 렌더링해야 한다.





좀 더 상세히 코드로 살펴보면, 

![](https://godekdls.github.io/images/springsecurity/filterchainproxy.png)



제일 먼저 `DelegatingFilterProxy` 객체의 `doFilter` 메서드에서 흐름이 시작된다.



**DelegatingFilterProxy**

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



위의 코드를 수행하고 나면 `FilterChainProxy` 객체의 `doFilter` 메서드로 흐름이 넘어간다.



**FilterChainProxy**

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



위의 코드를 수행 후 `ExceptionTranslationFilter`의 `doFilter`로 들어간다.



**ExceptionTranslationFilter**

```java
private void doFilter(HttpServletRequest request, HttpServletResponse response, FilterChain chain) throws IOException, ServletException {
    
    
        try {
            
            // 아래 statement로 들어간다.
            chain.doFilter(request, response);
            
        } catch (IOException var7) {
            ...
        } catch (Exception var8) {
            ...
        }
    
}
```



`chian.doFilter(request,response)`문 부터 처리한 뒤 `catch`문이 실행된다.

말 그대로 Exception을 Translation하는 Filter이기 때문에 `chain.doFilter(request, response)`에서 발생한 Exception들을 아래의 `catch`문으로 처리해주는 것 같다.





**AbstractSecurityInterceptor**

```java
private void attemptAuthorization(Object object, Collection<ConfigAttribute> attributes, Authentication authenticated) {
    try {
        
        // 권한이 없으면 아래 statement에서 AccessDeniedException이 발생한다.
        this.accessDecisionManager.decide(authenticated, object, attributes);
        
        
    } catch (AccessDeniedException var5) {
        ...

        this.publishEvent(new AuthorizationFailureEvent(object, attributes, authenticated, var5));
        throw var5;
    }
}
```



인증되지 않은 요청이라면 `AccessDeniedException`이 발생해서 `catch`문으로 들어가서 `AccessDeniedException`을 `throw`한다.

그럼 이 던져진 예외를 다시 `ExceptionTranslationFilter`가 받아서 `catch` 문에서 처리해야 한다. 아래 코드를 보자.



**ExceptionTranslationFilter**

```java
private void doFilter(HttpServletRequest request, HttpServletResponse response, FilterChain chain) throws IOException, ServletException {
    
    try {
        
        // 권한이 없어서 아래 statement에서 AccessDeniedException이 발생.
        // 아래의 Exception catch문에서 잡힌다.
        chain.doFilter(request, response);
        
    } catch (IOException var7) {
        throw var7;
    } catch (Exception var8) {
        
        Throwable[] causeChain = this.throwableAnalyzer.determineCauseChain(var8);
        RuntimeException securityException = (AuthenticationException)this.throwableAnalyzer.getFirstThrowableOfType(AuthenticationException.class, causeChain);
        
        
        if (securityException == null) {
            securityException = (AccessDeniedException)this.throwableAnalyzer.getFirstThrowableOfType(AccessDeniedException.class, causeChain);
        }

        if (securityException == null) {
            this.rethrow(var8);
        }

        if (response.isCommitted()) {
            throw new ServletException("Unable to handle the Spring Security Exception because the response is already committed.", var8);
        }
        

        this.handleSpringSecurityException(request, response, chain, (RuntimeException)securityException);
    }

}



private void handleSpringSecurityException(HttpServletRequest request, HttpServletResponse response, FilterChain chain, RuntimeException exception) throws IOException, ServletException {
        if (exception instanceof AuthenticationException) {
            this.handleAuthenticationException(request, response, chain, (AuthenticationException)exception);
            
            
        } else if (exception instanceof AccessDeniedException) {
            // 우리는 지금 AccessDeniedException이 발생했기 때문에 이 코드 블럭으로 들어온다.
            this.handleAccessDeniedException(request, response, chain, (AccessDeniedException)exception);
        }

}




// 실제로 AccessDeniedException을 처리하는 부분
private void handleAccessDeniedException(HttpServletRequest request, HttpServletResponse response, FilterChain chain, AccessDeniedException exception) throws ServletException, IOException {
    
    
        Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
        boolean isAnonymous = this.authenticationTrustResolver.isAnonymous(authentication);
        if (!isAnonymous && !this.authenticationTrustResolver.isRememberMe(authentication)) {
            
            ...

            this.accessDeniedHandler.handle(request, response, exception);
        } else {
            ...
            
            
            // 아래 statement에서 클라이언트에게 Authentication을 시작하라고 요청한다.
            this.sendStartAuthentication(request, response, chain, new InsufficientAuthenticationException(this.messages.getMessage("ExceptionTranslationFilter.insufficientAuthentication", "Full authentication is required to access this resource")));
        }

}



// AuthenticationEntryPoint에게 클라이언트더러 credential를 보내라고 지시한다.
protected void sendStartAuthentication(HttpServletRequest request, HttpServletResponse response, FilterChain chain, AuthenticationException reason) throws ServletException, IOException {
        SecurityContextHolder.getContext().setAuthentication((Authentication)null);
        this.requestCache.saveRequest(request, response);
    
    	// 여기서는 LoginUrlAuthenticationEntryPoint에게 지시한다.
        this.authenticationEntryPoint.commence(request, response, reason);
}
```







**LoginUrlAuthenticationEntryPoint**

```java
public void commence(HttpServletRequest request, HttpServletResponse response, AuthenticationException authException) throws IOException, ServletException {
    
    String redirectUrl;
    if (!this.useForward) {
        redirectUrl = this.buildRedirectUrlToLoginPage(request, response, authException);
        this.redirectStrategy.sendRedirect(request, response, redirectUrl);
    } else {
        redirectUrl = null;
        if (this.forceHttps && "http".equals(request.getScheme())) {
            redirectUrl = this.buildHttpsRedirectUrlForRequest(request);
        }

        if (redirectUrl != null) {
            this.redirectStrategy.sendRedirect(request, response, redirectUrl);
        } else {
            String loginForm = this.determineUrlToUseForThisRequest(request, response, authException);
            logger.debug(LogMessage.format("Server side forward to: %s", loginForm));
            RequestDispatcher dispatcher = request.getRequestDispatcher(loginForm);
            dispatcher.forward(request, response);
        }
    }
}
```



