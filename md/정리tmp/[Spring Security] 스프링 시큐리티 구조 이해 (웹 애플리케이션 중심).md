

# [Spring Security] 스프링 시큐리티 구조 이해 (웹 애플리케이션 중심)



> **[참고]** https://www.slideshare.net/madvirus/ss-36809454



### 목표

- 스프링 시큐리티의 구성 요소와 동작 방식 이해
  - 이를 통한 커스터마이징 포인트 잡기!







## 기본 구성 요소



### 보안 관련 3요소

- 접근 주체 (Principal)
  - 보호된 대상에 접근하는 사용자
- 인증(Authentication)
  - 현재 사용자가 누구인지 확인하는 과정
  - 일반적으로 아이디/패스워드를 이용해서 인증을 처리
- 권한(Authorization)
  - 현재 사용자가 특정 대상(URL, 기능 등)을 사용(접근)할 권한이 있는지 검사하는 것





### 스프링 시큐리티와 보안 3요소의 매칭

![](https://lh6.googleusercontent.com/Nkr5OdbvUYWH4h-jMoEmtK5AJyKw1xBd1pJZU-nWHhwVg0peJLEddAJ-ahC_yuehSqra0u-wXR5oZrC95ePMkIvRUs2cTRkr0OMWfNo7-W9Y3K7NzZo2BrWUbnkAC62RApjO_I5g)







### Authentication과 SecurityContext

- Authentication의 용도
  - 현재 접근 주체 정보를 담는 목적
  - 인증 요청을 할 때, 요청 정보를 담는 목적
- SecurityContext
  - Authentication을 보관
  - 스프링 시큐리티는 현재 사용자에 대한 Authentication 객체를 구할 때 SecurityContext로부터 구함



### SecurityContextHolder

- SecurityContext를 보관

  - 기본: ThreadLocal에 SecurityContext를 보관

  

- 전형적인 SecurityContext 설정 코드

```java
Authentication auth = someMethodForGettingAuth(req, resp);
try {
    SecurityContextHolder.getContext().setAuthentication(auth);
    chain.doFilter(request, response); // 이후 코드에서 동일 SecurityContext 사용
} finally {
    SecurityContextHolder.clearContext();
}
```

스프링 시큐리티가 이미 위와 유사한 필터를 제공하고 있다.





### Authentication의 주요 메서드

- Authentication의 주요 메서드
  - String getName(): 사용자의 이름
  - Object getCredential(): 증명 값 (비밀번호 등)
  - Object getPrincipal(): 인증 주체 정보
  - boolean isAuthenticated(): 인증 되었는지 여부
  - Collection\<GrantedAuthority> getAuthorities(): 현재 사용자가 가진 권한





### AuthenticationManager

**역할**: 인증을 처리함.

```java
public interface AuthenticationManager {
    Authentication authenticate(Authentication authentication) throws AuthenticationException;
}
```

- 인증에 성공하면 인증 정보를 담고 있는 Authentication 객체를 리턴
  - 스프링 시큐리티는 리턴한 Authentication 객체를 SecurityContext에 보관 및 인증 상태를 유지하기 위해 세션에 보관
- 인증 실패 시 AuthenticationException을 발생시킴.





### (Abstract) SecurityInterceptor

**역할**: 권한을 처리함.

```java
public interface AccessDecisionManaer {
	void decide(Authentication authentication, // 사용자
                Object object, // 접근 자원
                Collection<ConfigAttribute> configAttributes /* 보안 설정 */) throws AccessDeniedException, InsufficientAuthenticationException;
    
    boolean supports(ConfigAttribute attribute);
    boolean supports(Class<?> clazz);
}
```

- 웹의 경우 FilterSecurityInterceptor 구현체 사용
- AccessDecisionManager에 권한 검사 위임
- 사용자가 자원의 보안 설정 기준으로, 접근 권한이 없을 경우 Exception 발생







## SecurityFilterChain



### 웹 애플리케이션 보안 지원

![](https://lh4.googleusercontent.com/5rn6CejA35eM-5nnI8OnWO-ynXeRmE_CSoh70Mp4GRdJ1namDsPDwo91yik1a2X9dlJcA0fM6giG_U3iBhCV5xuy98GdJscmuy7qPBKIGMkXBx1K0Ow-pXLO1Rm0NAafKWXTXLQ-)





### 웹 요청을 FilterChainProxy 적용

![](https://lh5.googleusercontent.com/f1hPA5L-ayEPZSqWqCSP6Smf10qrlztl4GyKavsBeB5nIZy9mXCZ_8heBLc4xqRAWLIXGaWqKkC4A9ajgToJWVDUzCXM_emEEyoJY6qPJU5bEfS2Su7DejKYW16-inTOPMPRdOG5)





### 보안 필터 체인의 주요 구성 요소

![](https://lh6.googleusercontent.com/EyrsLEetyPz4-OxI648Tgpp_zfQHQJnUB4jo-XQ3kDVB6gddF8Adfi1TuRhQRexNCvyTQBr7sp-kIzZ_BuV8YnCRIe_0m2xq1G-jyMCJCcTpVAyN5bPAnvxC9tty78UXZb-OoC8r)





### 접근 권한 없을 때 처리, 인증 전

![](https://lh3.googleusercontent.com/MltcSu4y-Njz-iRmsMp5BgcZjHDrtWEAiWFb8biW_Ia7xi_kDfdexbNyabmb7QDaxUxcIfUK-wKaypgRtD-F7bImWEWxMKtwfwXDytN5CJpDGYgPDrREjGVMVeaFB-95UuoBcxka)





### 접근 권한 없을 때 처리, 인증 상태

![](https://lh4.googleusercontent.com/vA6CSh0et-RGyly5iO9_2PFq-zxiDH9RB6xIVFuY4TaT5A32atnnmSkonKlT1ydUuQ1-J23GpbYIFRj9_H4yi9ZI4iwukQk9DJXJAoheYMNF98tkp9qLVLY7kQ4H7NY0I0JNqQL9)





### 기본 로그인 폼 제공

![](https://lh6.googleusercontent.com/sanueP1AfAUeN1Uz9jBX-SCbyery-hl5vp5jJy7tEL3kjPd1OT984snMtarjjSPWg8OJiNFPaix8zhsDf5_I-2VBk-h_Ld3vXYdcO0fGuB7sDhA4S43wkXf3Y2fcqgDx8Ci0FfCl)





### 인증 요청 처리 과정 (성공시)

![](https://lh3.googleusercontent.com/Q_XIWBoae_th2Uokjjphvr7d5r-_F8flvNeaG832JJk8zZ8WpQ7OTsLHXveU_YCMfURUE-_29SeiIlJKwkvGrFlqv9ljha-lHWzlg2raV8dvfedb20oOgTB0Qu1sczlbG0M4Q6qO)





### 인증 요청 처리 과정 (실패시)

![](https://lh4.googleusercontent.com/MdGB5Vqli9Ygsaqa9mP06MOXhyrpVDDlyqa5bfdureHxnGOOzmDKMQTdyc6tR3BKTKe2P8bSXlZUy77A2tFEFQi6fpARU6UJR2xVoTdLsMCByTKA4xNGvVPufK8sngm5VYWglYQU)





### 로그아웃 요청 처리 과정

![](https://lh4.googleusercontent.com/-az4egbtmGOUDNH1oRBPdHeStN2D-jJgGY-29P2IBvsGBU-Nlisf8Fgjs8R7eJOUs2T7TMjNxiWns-U9yQfHlV4Qloa-vNvsiK5QQS-CQ6_TOTYlgkcuak_NCYeW1wNFFqEQlV-j)







## AuthenticationManager

![](http://i.imgur.com/nCawRsL.png)









![](https://prod-acb5.kxcdn.com/wp-content/uploads/2019/09/Spring-Security-Architecture--1024x607.png)





![](https://i.stack.imgur.com/3gLa3.jpg)





![](https://prod-acb5.kxcdn.com/wp-content/uploads/2020/07/DaoAuthenticationProvider.png)







![](https://i.stack.imgur.com/PHOBF.png)





![](https://bravenamme.github.io/files/posts/201908/spring_sec_authentication.png)





## FilterSecurityInterceptor & AccessDecisionManager





### FilterSecurityInterceptor

![](https://image.slidesharecdn.com/random-140709165445-phpapp01/95/-26-638.jpg?cb=1404925455)





### AccessDecisionManager

![](https://image.slidesharecdn.com/random-140709165445-phpapp01/95/-27-638.jpg?cb=1404925455)





