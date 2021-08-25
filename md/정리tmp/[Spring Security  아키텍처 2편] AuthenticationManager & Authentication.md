# [Spring Security / 아키텍처 2편] AuthenticationManager & Authentication



Spring Security에서 인증(Authentication)은 **AuthenticationManager**가 담당한다.





## AuthenticationManager

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FsQixv%2FbtqyenwP2st%2FQqQFASKcF74Itq9Ycivqa1%2Fimg.png)



인자로 받은 `Authentication` 객체가 유저에 대한 인증 정보를 담고 있다.

해당 인증 정보가 유효하다면 `UserDetailsService`에서 Return한 객체 `Principal`을 담고 있는 `Authentication` 객체를 리턴한다.



기본 구현체는 `ProviderManager`를 사용한다.

직접 `AuthenticationManager`를 구현해도 되지만 대부분의 Application에서 기본 구현체 정도에서 해결이 된다. (직접 구현하는 경우는 매우 드물다.)





## Authentication 과정

- 유저를 생성하고, 로그인 폼에서 정상적인 인증 요청을 했을 경우
- `ProviderManager`(`AuthenticationManager`의 구현체)의 authenticate 메서드로 진입한다.
  - 이때 `Authentication` 객체를 파라미터로 받는다.
  - `Authentication` 객체에는 principal에 해당하는 username 정보와 credentials에 해당하는 password가 들어오게 된다.
  - authorities 권한에 해당하는 정보는 아직 알 수 없기 때문에 빈 값이 들어온다.



- 해당 요청에 대한 인증을 처리할 수 있는 `Provider`를 찾아 이를 처리한 뒤 인증 정보가 담긴 `Authentication` 객체를 반환한다.



![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FYW0jA%2FbtqyghPJcR5%2Ffa2RGcd2WyE6SR9Da5Z241%2Fimg.png)



`ProviderManager`는 인증을 직접 처리하는 것이 아닌 다른 여러 `Provider`를 사용해서 진행한다.

- 다른 여러 `Provider`들에게 위임하는 구조
- 현재 파라미터로 들어온 `Authentication` 객체는 `UsernameAuthenticationToken` 타입이다.
- `Authentication` 객체의 타입마다 처리할 수 있는 `Provider`가 다르다.



**UsernameAuthenticationToken.java**

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FdO8j6h%2FbtqyfYv5w0V%2FqT5eUaWraDSFZ87LVlm0Yk%2Fimg.png)





현재 `ProviderManager`는 `AnonoymousAuthenticationProvider`만을 가지고 있기 때문에 `UsernamePasswordAuthenticationToken`에 대한 인증을 처리할 수 없다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FRSHSr%2FbtqyepanGGv%2FQK9cEgmOBQVRfQxehHotBK%2Fimg.png)





현재 `ProviderManager`에게 처리할 수 있는 `Provider`가 존재하지 않을 경우 `ProviderManager`의 Parent에게 위임한다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FeeHO17%2Fbtqyg9b8JYG%2FOCb9p0GGET2IGI61mIfNM1%2Fimg.png)



- `ProviderManager`의 Parent가 인증 요청을 위임받아, 처리할 수 있는 `Provider`를 찾은 뒤 인증을 처리한다.

- 이때 `ProviderManager`와 `ProviderManager`의 Parent는 다른 `ProviderManager`이며 처리 가능한 `Provider`도 다르다.

  

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbY072T%2FbtqyfsxFo5O%2F7axFwk4i5J5kmQoH2KMOM1%2Fimg.png)





> 인증 요청을 위임받은 `ProviderManager`는 `DaoAuthenticationProvider`를 사용하며, `DaoAuthenticationProvider`는 우리가 직접 구현한 `UserDetailsService`를 사용한다.



![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbByHpE%2Fbtqygug0yJY%2Fl26WPwnJizOGn3DdLqlM3K%2Fimg.png)





인증이 완료되어 `ProviderManager`에서 반환하는 객체는 이전에 살펴본 `SecurityContextHolder`를 이용해 접근할 수 있는 `Authentication` 객체와 동일하다.

인증이 완료되어 반환되는 객체가 `SecurityContext`에 저장되는 것이다.



![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fdxf0Hm%2Fbtqyg7SVG9i%2Fkkaws5KCt3zU0Q14xBTzr0%2Fimg.png)





**인증 과정에서 발생하는 예외들**

- `DisabledException`
  - 계정이 비활성화 되어 있는 경우
- `LockedException`
  - 계정이 잠겨 있는 경우
- `BadCredentialsException`
  - 비밀번호가 일치하지 않는 경우