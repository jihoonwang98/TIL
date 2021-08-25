# [Spring MVC] Validation 

이번 포스팅에서는 커맨드 객체의 값 검증과 에러 메시지 처리에 대해 살펴본다.





## 커맨드 객체 검증과 에러 코드 지정하기

스프링 MVC에서 커맨드 객체의 값이 올바른지 검사하려면 

다음의 두 인터페이스를 사용한다.

- `org.springframework.validation.Validator`
- `org.springframework.validation.Errors`



객체를 검증할 때 사용하는 `Validator` 인터페이스는 다음과 같다.

```java
package org.springframework.validation;

public interface Validator {
    boolean supports(Class<?> clazz);
    void validate(Object target, Errors errors);
}
```



- `supports`: Validator가 검증할 수 있는 타입인지 검사한다.
- `validate`: 첫 번째 파라미터로 전달받은 객체를 검증하고, 오류 결과를 Errors에 담는 기능을 정의한다.







### 예제

```

```





























