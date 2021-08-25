# [Spring Security] 객체 정리



![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcrSvxl%2FbtqFyRvClqL%2F217ksrm8e4nBJMdC7qlH3K%2Fimg.png)





## AuthenticationManager 인터페이스

**역할:** `Authentication` 요청을 처리하는 인터페이스

```java
public interface AuthenticationManager {

   Authentication authenticate(Authentication authentication) throws AuthenticationException;
}
```



- 인터페이스 설명
  - 파라미터: authentication 요청 객체
  - 반환값: credentials 값들을 포함한 완전히 인증된 Authentication 객체
  - 발생가능: 인증에 실패하면 `AuthenticationException`이 발생한다.



- 파라미터로 받은 `Authentication` 객체를 인증해준다.

  인증에 성공하면, GrantedAuthority를 포함한 필드 값들을 전부 채워서 리턴해준다.



- `AuthenticationManager`는 아래와 같은 Exception들을 처리해야 할 의무가 있다.
  - 사용자 계정이 disabled된 상태인 경우 `AuthenticationManager`가 이 값을 검증하는 경우에는 `DisabledException`을 throw 해야 한다. (선택사항, 검증해도 되고 안해도됨)
  - 사용자 계정이 locked된 경우 `AuthenticationManager`가 이 값을 검증하는 경우에는`LockedException`을 throw 해야 한다. (선택사항, 검증해도 되고 안해도됨)
  - 만약 잘못된 credentials 값들이 들어오면 `BadCredentialsException`을 throw 해야 한다. (**의무 사항, must임!!**, 위의 예외들과는 다르게 `AuthenticationManager`는 언제나 credentials 값을 검증한다.)



- 예외 처리 순서는 아래와 같은 순서로 진행된다.

  - `DisabledException`

  - `LockedException`

  - `BadCredentialsException`

    

- 만약, 계정이 disabled 상태거나 locked 상태면, Authentication 요청이 즉시 거절(reject)되고, credentials 검증 과정이 생략된다.

  - 따라서 disabled 상태나 locked 상태의 계정에 대해서는 credentials 값들을 검증하는 과정이 생략된다.



- 구현체
  - `ProviderManager`





## ProviderManager 구현 클래스

**역할:** `AuthenticationProvider`들로 구성된 List를 순회하면서, `Authentication` 요청을 처리할 수 있는 `AuthenticationProvider`를 찾는다.

`AuthenticaionManager`의 구현 클래스다.

```java
public class ProviderManager implements AuthenticationManager, MessageSourceAware, InitializingBean {
	
    ...
    
	@Override
	public Authentication authenticate(Authentication authentication) throws AuthenticationException {
        
		Class<? extends Authentication> toTest = authentication.getClass();
		AuthenticationException lastException = null;
		AuthenticationException parentException = null;
		Authentication result = null;
		Authentication parentResult = null;
		int currentPosition = 0;
		int size = this.providers.size();
        
        // AuthenticationManager(ProviderManager)는 
        // AuthenticationProvider 리스트를 순회하면서 인증 처리를 수행해 줄 놈을 찾는다.
		for (AuthenticationProvider provider : getProviders()) {
			
            if (!provider.supports(toTest)) {
				continue;
			}
            
            
			try {
                // AuthenticationProvider가 해당 authentication을 supports 하면,
                // try 문 안으로 들어온다.
               
				result = provider.authenticate(authentication);  // 인증 시도    
                
				if (result != null) {
                    // result가 null이 아니면 인증에 성공했다는 뜻이다.
					copyDetails(authentication, result); // details 정보를 옮긴다.
					break;  // 반복문 탈출.
				}
                
			}
			catch (AccountStatusException | InternalAuthenticationServiceException ex) {
				prepareException(ex, authentication);
				// SEC-546: Avoid polling additional providers if auth failure is due to
				// invalid account status
				throw ex;
			}
			catch (AuthenticationException ex) {
				lastException = ex;
			}
		}
        
		if (result == null && this.parent != null) {
			// Allow the parent to try.
			try {
				parentResult = this.parent.authenticate(authentication);
				result = parentResult;
			}
			catch (ProviderNotFoundException ex) {
				// ignore as we will throw below if no other exception occurred prior to
				// calling parent and the parent
				// may throw ProviderNotFound even though a provider in the child already
				// handled the request
			}
			catch (AuthenticationException ex) {
				parentException = ex;
				lastException = ex;
			}
		}
		if (result != null) {
			if (this.eraseCredentialsAfterAuthentication && (result instanceof CredentialsContainer)) {
				// Authentication is complete. Remove credentials and other secret data
				// from authentication
				((CredentialsContainer) result).eraseCredentials();
			}
			// If the parent AuthenticationManager was attempted and successful then it
			// will publish an AuthenticationSuccessEvent
			// This check prevents a duplicate AuthenticationSuccessEvent if the parent
			// AuthenticationManager already published it
			if (parentResult == null) {
				this.eventPublisher.publishAuthenticationSuccess(result);
			}

			return result;
		}

		// Parent was null, or didn't authenticate (or throw an exception).
		if (lastException == null) {
			lastException = new ProviderNotFoundException(this.messages.getMessage("ProviderManager.providerNotFound",
					new Object[] { toTest.getName() }, "No AuthenticationProvider found for {0}"));
		}
		// If the parent AuthenticationManager was attempted and failed then it will
		// publish an AbstractAuthenticationFailureEvent
		// This check prevents a duplicate AbstractAuthenticationFailureEvent if the
		// parent AuthenticationManager already published it
		if (parentException == null) {
			prepareException(lastException, authentication);
		}
		throw lastException;
	}
}
```







## AuthenticationProvider 인터페이스

**역할:** 특정 `Authentication` 요청을 처리할 수 있는 인터페이스

```java
public interface AuthenticationProvider {

   Authentication authenticate(Authentication authentication) throws AuthenticationException;

   boolean supports(Class<?> authentication);

}
```



- `Authentication authenticate(Authentication)`
  - `AuthenticationManager`와 똑같은 인증 과정을 처리한다.
  - 파라미터: 인증 요청 객체
  - 반환값: credentials 값들을 포함한 완전히 인증된 객체.
    - `AuthenticationProvider`가 파라미터로 넘어온 인증 객체를 지원하지 않으면 `null`을 리턴할 수 있다.
    - 위와 같은 경우에는, 다음 `AuthenticationProvider`가 인증을 시도한다.
  - 발생가능예외: 인증에 실패하면 `AuthenticationException`이이 발생한다.



- `boolean supports(Class<?> authentication)`
  - `AuthenticationProvider`가 `authentication` 객체에 대한 인증 처리를 시도해 볼 수 있으면 `true`를 리턴한다.
    - `true`를 리턴한다는 것이 넘어온 `authentication` 인스턴스를 인증함을 보증하는 것은 아니다. 
    - `AuthenticationProvider`는 여전히 `authenticate(Authentication)` 메서드에 대한 반환 값으로 `null`을 반환할 수 있다. 이런 경우에는 다른 `AuthenticationProvider`가 인증 처리를 시도하게 한다.
  - 인증을 처리할 수 있는 `AuthenticationProvider`를 고르는 것은 `ProviderManager`에 의해 런타임에 수행된다.
  - 파라미터: authentication



- 구현체
  - `AbstractUserDetailsAuthenticationProvider`
  - `AnonymousAuthenticationProvider`
  - `RememberMeAuthenticationProvider`
  - `DaoAuthenticationProvider`
  - ...





## DaoAuthenticationProvider 구현 클래스

`AuthenticationProvider` 인터페이스의 구현체로 `DaoAuthenticationProvider` 클래스를 살펴보고 넘어가자.

**역할:** `UserDetailsService`로부터 `UserDetails`를 받아온다.



```java
public class DaoAuthenticationProvider extends AbstractUserDetailsAuthenticationProvider {



   @Override
   protected final UserDetails retrieveUser(String username, UsernamePasswordAuthenticationToken authentication)
         throws AuthenticationException {
      prepareTimingAttackProtection();
      try {
         UserDetails loadedUser = this.getUserDetailsService().loadUserByUsername(username);
         if (loadedUser == null) {
            throw new InternalAuthenticationServiceException(
                  "UserDetailsService returned null, which is an interface contract violation");
         }
         return loadedUser;
      }
      catch (UsernameNotFoundException ex) {
         mitigateAgainstTimingAttack(authentication);
         throw ex;
      }
      catch (InternalAuthenticationServiceException ex) {
         throw ex;
      }
      catch (Exception ex) {
         throw new InternalAuthenticationServiceException(ex.getMessage(), ex);
      }
   }
    
    

   @Override
   protected Authentication createSuccessAuthentication(Object principal, Authentication authentication,
         UserDetails user) {
      boolean upgradeEncoding = this.userDetailsPasswordService != null
            && this.passwordEncoder.upgradeEncoding(user.getPassword());
      if (upgradeEncoding) {
         String presentedPassword = authentication.getCredentials().toString();
         String newPassword = this.passwordEncoder.encode(presentedPassword);
         user = this.userDetailsPasswordService.updatePassword(user, newPassword);
      }
      return super.createSuccessAuthentication(principal, authentication, user);
   }

   private void prepareTimingAttackProtection() {
      if (this.userNotFoundEncodedPassword == null) {
         this.userNotFoundEncodedPassword = this.passwordEncoder.encode(USER_NOT_FOUND_PASSWORD);
      }
   }

   private void mitigateAgainstTimingAttack(UsernamePasswordAuthenticationToken authentication) {
      if (authentication.getCredentials() != null) {
         String presentedPassword = authentication.getCredentials().toString();
         this.passwordEncoder.matches(presentedPassword, this.userNotFoundEncodedPassword);
      }
   }

   public void setPasswordEncoder(PasswordEncoder passwordEncoder) {
      Assert.notNull(passwordEncoder, "passwordEncoder cannot be null");
      this.passwordEncoder = passwordEncoder;
      this.userNotFoundEncodedPassword = null;
   }

   protected PasswordEncoder getPasswordEncoder() {
      return this.passwordEncoder;
   }

   public void setUserDetailsService(UserDetailsService userDetailsService) {
      this.userDetailsService = userDetailsService;
   }

   protected UserDetailsService getUserDetailsService() {
      return this.userDetailsService;
   }

   public void setUserDetailsPasswordService(UserDetailsPasswordService userDetailsPasswordService) {
      this.userDetailsPasswordService = userDetailsPasswordService;
   }

}
```



웁스 찾아보니 우리가 찾던 `authenticate` 메서드는 상위 클래스인 `AbstractUserDetailsAuthenticationProvider` 클래스에 정의되어 있다.



```java
public abstract class AbstractUserDetailsAuthenticationProvider
      implements AuthenticationProvider, InitializingBean, MessageSourceAware {

   @Override
   public Authentication authenticate(Authentication authentication) throws AuthenticationException {
      Assert.isInstanceOf(UsernamePasswordAuthenticationToken.class, authentication,
            () -> this.messages.getMessage("AbstractUserDetailsAuthenticationProvider.onlySupports",
                  "Only UsernamePasswordAuthenticationToken is supported"));
       
       
      String username = determineUsername(authentication);
      boolean cacheWasUsed = true;
      UserDetails user = this.userCache.getUserFromCache(username);
       
      // Cache에 UserDetails가 없는 경우
      if (user == null) {
         cacheWasUsed = false;
         try {
            user = retrieveUser(username, (UsernamePasswordAuthenticationToken) authentication);
         }
          
         catch (UsernameNotFoundException ex) {
            this.logger.debug("Failed to find user '" + username + "'");
            if (!this.hideUserNotFoundExceptions) {
               throw ex;
            }
            throw new BadCredentialsException(this.messages.getMessage("AbstractUserDetailsAuthenticationProvider.badCredentials", "Bad credentials"));
         }
          
          
      }
       
       
      try {
         this.preAuthenticationChecks.check(user);
         additionalAuthenticationChecks(user, (UsernamePasswordAuthenticationToken) authentication);
      }
      catch (AuthenticationException ex) {
         if (!cacheWasUsed) {
            throw ex;
         }
         // There was a problem, so try again after checking
         // we're using latest data (i.e. not from the cache)
         cacheWasUsed = false;
         user = retrieveUser(username, (UsernamePasswordAuthenticationToken) authentication);
         this.preAuthenticationChecks.check(user);
         additionalAuthenticationChecks(user, (UsernamePasswordAuthenticationToken) authentication);
      }
       
       
      this.postAuthenticationChecks.check(user);
      if (!cacheWasUsed) {
         this.userCache.putUserInCache(user);
      }
      Object principalToReturn = user;
      if (this.forcePrincipalAsString) {
         principalToReturn = user.getUsername();
      }
      return createSuccessAuthentication(principalToReturn, authentication, user);
   }
    
    
    

   private String determineUsername(Authentication authentication) {
      return (authentication.getPrincipal() == null) ? "NONE_PROVIDED" : authentication.getName();
   }

   /**
    * Creates a successful {@link Authentication} object.
    * <p>
    * Protected so subclasses can override.
    * </p>
    * <p>
    * Subclasses will usually store the original credentials the user supplied (not
    * salted or encoded passwords) in the returned <code>Authentication</code> object.
    * </p>
    * @param principal that should be the principal in the returned object (defined by
    * the {@link #isForcePrincipalAsString()} method)
    * @param authentication that was presented to the provider for validation
    * @param user that was loaded by the implementation
    * @return the successful authentication token
    */
   protected Authentication createSuccessAuthentication(Object principal, Authentication authentication,
         UserDetails user) {
      // Ensure we return the original credentials the user supplied,
      // so subsequent attempts are successful even with encoded passwords.
      // Also ensure we return the original getDetails(), so that future
      // authentication events after cache expiry contain the details
      UsernamePasswordAuthenticationToken result = new UsernamePasswordAuthenticationToken(principal,
            authentication.getCredentials(), this.authoritiesMapper.mapAuthorities(user.getAuthorities()));
      result.setDetails(authentication.getDetails());
      this.logger.debug("Authenticated user");
      return result;
   }


   protected abstract UserDetails retrieveUser(String username, UsernamePasswordAuthenticationToken authentication)
         throws AuthenticationException;


   @Override
   public boolean supports(Class<?> authentication) {
      return (UsernamePasswordAuthenticationToken.class.isAssignableFrom(authentication));
   }

}
```







## UserDetails 인터페이스

**역할**

- 핵심 유저 정보를 제공하는 인터페이스
- 이 인터페이스의 구현체들은 스프링 시큐리티에 의해 직접 사용되지는 않는다. (보안의 목적)
  - UserDetails 구현체들은 `Authentication` 객체 안에 캡슐화 되어서 유저 정보를 저장한다.
  - 이러한 구현의 이점은 보안과 상관 없는 유저 정보(이메일 주소, 전화번호 등등)가 조회하기 편한 위치에 저장될 수 있다는 것이다.



```java
public interface UserDetails extends Serializable {

   // User에게 부여된 권한들을 반환한다. null을 리턴할 수 없다.
   Collection<? extends GrantedAuthority> getAuthorities();

   // User를 인증할 때 사용하는 Password를 반환한다.
   String getPassword();

   // User를 인증할 때 사용하는 Username을 반환한다. null을 리턴할 수 없다.
   String getUsername();

   // User의 계정이 만료됐는지 안됐는지를 리턴한다. 만료된 계정은 인증 처리가 안된다.
   boolean isAccountNonExpired();

   // User의 계정이 잠겼는지 안됐는지를 리턴한다. 잠긴 계정은 인증 처리가 안된다.
   boolean isAccountNonLocked();

   // User의 credential이 만료됐는지 안됐는지를 리턴한다. 만료된 credential을 가진 계정은 인증 처리가 안된다.
   boolean isCredentialsNonExpired();

   // User가 사용가능한지 안한지를 리턴한다. disabled된 User는 인증 처리가 안된다.
   boolean isEnabled();

}
```







## UserDetailsService 인터페이스

**역할**

- 유저 정보를 불러오는 핵심 인터페이스
- 프레임워크 전체에서 User DAO로 사용되며, `DaoAuthenticationProvider`에서 사용되는 Strategy다.



```java
public interface UserDetailsService {    

    UserDetails loadUserByUsername(String username) throws UsernameNotFoundException;

}
```

- 파라미터: username (데이터가 요청된 User의 식별 가능한 username)
- 반환값: User record (`null`이 될 수 없다)
- 발생가능예외: `UsernameNotFoundException`
  - user를 찾을 수 없거나, user가 `GrantedAuthority`를 가지고 있지 않은 경우 발생