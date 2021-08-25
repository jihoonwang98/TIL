# [Spring Security / 아키텍처 1편] SecurityContextHolder & Authentication



> **[참고] **https://ncucu.me/103



![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbBi5d5%2Fbtqyeom0ZK5%2FP2NWuKEPH89c3YbzbM2oj1%2Fimg.png)





## SecurityContextHolder

- Security Context 제공, 기본적으로 ThreadLocal을 사용한다.
- 기본 전략은 ThreadLocal 기반으로 동작한다.
- SecurityContextHolder를 통해 SecurityContext에 접근할 수 있다.





## SecurityContext

- Authentication을 제공한다.





## Authentication

- Spring Security에서 **인증을 거치고 난 뒤의 사용자의 인증 정보**를 Principal이라고 한다.
- 이러한 Principal을 감싸고 있는 객체이다.
- Principal 외에도 GrantedAuthorities (사용자 권한 정보) 등 다양한 정보를 가지고 있다.
- SecurityContext를 통해 Authentication 객체를 받아올 수 있다.





## Principal

- UserDetailsService에서 리턴한 객체이다.
- UserDetails 타입
- 인증에 성공한 사용자 정보에 해당한다.





## GrantedAuthorities

- ROLE_USER, ROLE_ADMIN 등 Principal이 가지고 있는 권한을 나타낸다.
- 인증 이후 권한을 확인할 때 참조한다.





## ThreadLocal

- ThreadLocal은 같은 Thread 내에서 공유하는 저장소이다.
- Parameter를 넘기지 않아도, Thread 내에서 데이터를 공유 및 전달이 가능하다.





**사용 예제**

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FclxZbo%2FbtqyeHNoS1I%2FkhtUtb19Zh2PRg57Wq1UO0%2Fimg.png)