# Spring Exception 처리



**목차**

- `@ExceptionHandler`를 이용한 Exception 처리
  - 특정 컨트롤러 내에서 발생하는 예외만을 처리하는 방법 (범위가 한정적임)
- `@ControllerAdvice`를 이용한 공통 Exception 처리
- `@ResponseStatus`를 이용한 Exception 처리





## 8.1 `@ExceptionHandler`를 이용한 Exception 처리



**예제**

```java
@RestController
public class CalculationController {

    @GetMapping("/cal/divide")
    public Long divide(@RequestParam Long op1, @RequestParam Long op2) throws Exception{

        return op1 / op2;
    }

    @ExceptionHandler(RuntimeException.class)
    public String handleRuntimeException(RuntimeException ex) {
        return ex.getMessage();
    }
}
```





#### `@ExceptionHandler` 사용 시 주의사항 및 특징

- `@Controller`, `@RestController`에만 적용가능하다.

  

- `@ExceptionHandler`는 정의된 Controller 안에서만 동작한다.

  - 어떤 `@ExceptionHandler`가 NullPointerException을 Handling 할때,

    다른 Controller에서 발생한 NullPointerException은 Handling 하지 못한다.

    

- ```java
  @ExceptionHandler({MyException1.class, MyException2.class})
  ```

  이런 식으로 두 개 이상의 Exception을 한꺼번에 처리할 수도 있다.



- `@ExceptionHandler`의 **파라미터**로 가능한 것들
  - Exception 객체 (Exception, RuntimeException, MyCustomException ...)
  - Request 또는 Response 객체 (ServletRequest, HttpServletRequest, ...)
  - WebRequest 또는 NativeWebRequest
  - InputStream, OutputStream
  - Reader, Writer
  - etc



- `@ExceptionHandler`로 Exception 타입을 지정하면, 해당 타입을 포함해 **하위 타입**까지도 처리할 수 있다.



- `@ExceptionHandler` 메서드를 통해서 Exception을 처리하면 기본적으로 응답 코드가 정상 처리를 뜻하는 200이 된다.
  - 만약 500과 같은 다른 응답 코드를 전송하고 싶다면 `@ResponseStatus`를 이용하자.



- **한계점**: 다른 컨트롤러에서도 같은 예외가 발생하면, 

  또 똑같은 코드를 복사 붙여넣기 해서 예외를 처리해줘야 하므로 

  중복 코드 ↑, 유지 보수 어려워짐.



- **해결책**: `@ExceptionHandler`를 `@ControllerAdvice`와 함께 사용해 예외를 전역적으로 처리한다.







## 8.2 `@ControllerAdvice`를 이용한 공통 익셉션 처리

Controller 클래스에 `@ExceptionHandler`를 적용하면 해당 컨트롤러에서 발생한 Exception만을 처리한다.

그런데, 다수의 컨트롤러에서 동일한 타입의 Exception을 발생시킬 수도 있고, 

이때 Exception 처리 코드가 동일하다면 어떻게 해야 할까?

각 컨트롤러 클래스마다 Exception 처리 메서드를 구현하는 것은 불필요한 코드 중복이 발생된다.



이렇게 여러 컨트롤러에서 동일한 Exception을 발생시킬 경우,

`@ControllerAdvice`를 적용해서 Exception 처리 메서드 중복을 제거할 수 있다.





**예제**

```java
@ControllerAdvice("com.example.prac")
public class CommonExceptionHandler {
    
    @ExceptionHandler(RuntimeException.class)
    public String handleRuntimeException(RuntimeException ex) {
        return ex.getMessage();
    }
}
```





#### `@ControllerAdvice` 사용 시 주의사항 및 특징

- `@ControllerAdvice`의 "Advice"는 AOP에서의 Advice다.

  즉, Error 처리만을 위한 어노테이션이 아니라, 컨트롤러의 공통 기능을 정의하기 위한 어노테이션이다.

  Error 처리는 공통 기능 중 하나일 뿐.



- `@ControllerAdvice` 어노테이션의 value로 **basePackages**를 설정할 수 있다.

  - 위 코드의 경우 "com.example.prac" 패키지 및 그 하위 패키지에 속한 Controller 클래스를 위한 공통 기능을 정의한다.

  - 이 패키지 및 그 하위 패키지에 속한 Controller에서 RuntimeException이 발생하면,

    위 예제 코드의 `handleRuntimeException()` 메서드를 통해 Exception을 handle하게 된다.



- `@ControllerAdvice` 어노테이션 속성

| 타입                            | 속성                | 설명                                               |
| ------------------------------- | ------------------- | -------------------------------------------------- |
| String[]                        | value(basePackages) | 공통 설정을 적용할 컨트롤러들이 속하는 기준 패키지 |
| Class\<? extends Annotation> [] | annotations         | 특정 어노테이션이 적용된 컨트롤러 대상             |
| Class\<?> []                    | assignableTypes     | 특정 타입 또는 그 하위 타입인 컨트롤러 대상        |





## 실무에선 어떻게?

일반적인 실무에서 어떻게 사용하는지는 사실 명확하지 않다.

필자의 경험상 에러 인터페이스 정의를 제대로 해야한다.

무슨 얘기냐면 에러메시지로 나가는 포맷이 일정해야한다는 얘기다.

만약 로그인 모듈에서 발생한 예외에 응답하는 메세지는 에러코드랑 설명을 리턴해준다고 하고, 배송 모듈에서 발생한 예외는 에러코드랑 에러가 난 배송 번호를 리턴해준다고 하자

그러면 @ControllerAdvice를 이용해서 통합으로 처리하려고 했지만 리턴 타입이 다르니까 통합해서 처리할 수 없다.

HTTP 상태코드, ErrorResponse같은 경우는 좋은 예다.

에러 인터페이스, 포맷이 다 같고 클라이언트 측에서도 이해하기 좋은 에러가 날라오는 것이다.

그래서 @ExceptionHandler와 함께 @ResponseStatus(value = HttpStatus.UNAUTHORIZED)

이런 것도 집어넣어서 HTTP상태코드를 리턴하기도 한다. (앞에서 설명하진 않았지만...)

다시 한 번 정리하지만 에러 메시지가 잘 정의되어있어야 하는게 전제 조건이다.

**→ 에러 관리하기**

```java
public enum LoginErrorCode {
        OperationNotAuthorized(6000,"Operation not authorized"),
        DuplicateIdFound(6001,"Duplicate Id"),
        //...
        UnrecognizedRole(6010,"Unrecognized Role");
        private int code;
        private String description;
        private LoginErrorCode(int code, String description) {
            this.code = code;
            this.description = description;
        }
        public int getCode() {
            return code;
        }
        public String getDescription() {
            return description;
        }
    }
```

보통 에러를 위와 같이 한 곳에 정리를 할 것이다.

저렇게 미리 정의해놓고 실제 사용할 때는 LoginErrorCode.OperationNotAuthorized.getCode(); 이런식으로 불러와서 에러객체를 만들어서 리턴할 것이다.

그러면 1차적으로 에러객체 관리는 위와 같은 방법으로 끝난다.

그 다음에 아까 배운 @ControllerAdvice, @ExeptionHandler를 이용해서 에러를 처리해본다.

만약 @ControllerAdvice, @ExceptionHandler로 처리할 때, InvalidArgumentProvided 라는 에러코드를 만들었다면, 모든 컨트롤러에서 들어오는 인자(arguments)에 대해서 한 곳에서 처리하게 되므로, 중복된 코드를 쓰지 않게 된다.

그로인해 비즈니스 로직에 더 집중할 수 있고, 코드도 간단하게 조건문에 따라 throw new XXXXException(); 하고 호출해버리면 끝나기 때문에 유지보수에 아주 큰 도움이 된다.



>  출처:  https://jeong-pro.tistory.com/195 





> https://velog.io/@hanblueblue/Spring-ExceptionHandler
>
> https://jsonobject.tistory.com/387
>
> https://galid1.tistory.com/569
>
> https://bamdule.tistory.com/92
>
> https://cheese10yun.github.io/spring-guide-exception/ ****
>
> https://homo-ware.tistory.com/116
>
> https://velog.io/@waldo/%EC%97%90%EB%9F%AC%EC%B2%98%EB%A6%AC101-Enum%EC%9C%BC%EB%A1%9C-%EC%97%90%EB%9F%AC%EC%BD%94%EB%93%9C-%EA%B4%80%EB%A6%AC%ED%95%98%EA%B8%B0
>



