# [Spring] HandlerMethodArgumentResolver





## 1. HandlerMethodArgumentResolver란?

HandlerMethodArgumentResolver는 컨트롤러 메서드에서 

특정 조건에 맞는 파라미터가 있을 때 원하는 값을 바인딩해주는 인터페이스입니다.



스프링에서는 Controller에서 `@RequestBody` 어노테이션을 사용해 Request의 Body 값을 받아올 때,

`@PathVariable` 어노테이션을 사용해 Request의 Path Parameter 값을 받아올 때 

이 HandlerMethodArgumentResolver를 사용해 값을 받아옵니다.





## 2. HandlerMethodArgument 사용하기



### 객체를 Controller 파라미터에 바인딩하기

컨트롤러에 특정한 Person이라는 객체가 파라미터로 존재할 시 원하는 값을 바인딩하는 예제를 작성합니다.



#### 1. 어노테이션 작성



























