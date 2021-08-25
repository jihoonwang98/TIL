# Error Handling for REST with Spring :leopard:



>**[참고]**: Error Handling for REST with Spring
>
>https://www.baeldung.com/exception-handling-for-rest-with-spring
>
>의역과 오역에 주의...





## 1. Overview :mag_right:

이번 포스팅은 Spring에서 REST API를 설계할 때 어떻게 **<u>Exception Handling</u>**을 구현할지에 대한 방법을 설명한다.

또, 지금까지 스프링이 exception handling을 어떻게 해왔는지에 대한 historical overview를 잠깐 살펴보고 새로운 버전이 업그레이드됨에 따라 어떤 새로운 option들이 도입되었는지 살펴보자.



먼저 Spring 3.2 버전 전에는, Spring MVC 애플리케이션에서 **Exception handling**하는 두가지 main 접근법은 아래와 같았다.

- **HandlerExceptionResolver**
- **@ExceptionHandler** 어노테이션

두가지 방법 모두 명확한 한계가 존재했다.



3.2 버전 이후로, 우리는 **<u>@ControllerAdvice</u>** 어노테이션을 통해 

위의 접근법들에 대한 문제점을 처리하고 

애플리케이션 전체에 대해 통일화된 exception handling 방법을 구축할 수 있게 되었다.



현재 Spring 5 버전에서는 **<u>ResponseStatusException</u>** 클래스를 도입해 REST API의 기본적인 error handling을 훨씬 빠르고 쉽게 할 수 있게 만들었다.





위에서 언급한 접근법들은 <u>한 가지 공통점</u>을 가지고 있다.

이 접근법들은 모두 **separation of concerns(SoC)** 원칙을 매우 잘 지키고 있다는 것이다.

애플리케이션이 error를 던지면 그 에러는 독립적으로 처리된다.







## 2. Solution 1: the Controller-Level (@ExceptionHandler) :detective:

첫번째 error 처리 solution은 **@Controller** 수준에서 처리하는 방법이다.



<u>우리는 exception을 처리하는 method를 작성하고 그 위에다가 **@ExceptionHandler** 어노테이션을 붙여줄 것이다</u>.

```java
public class FooController{
    
    //...
    @ExceptionHandler({ CustomException1.class, CustomException2.class })
    public void handleException() {
        //
    }
}
```



이 접근법은 큰 결함이 있다.

바로 이 **@ExceptionHandler**가 붙은 메서드는 application 전체에 global하게 적용되는 것이 아니라, 이 controller 내에서만 작동 가능하다는 것이었다.

즉, 이 어노테이션을 사용하면 모든 controller마다 

하나하나 method 추가하고 어노테이션을 붙여줘야 한다. :sob:



물론, Base Controller class를 하나 만들어서 여기서 에러 처리를 하게 구현한 후 

모든 Controller들이 이 Base Controller class를 상속받게 하는 수가 있지만 좋은 방법은 아니다.





## 3. Solution 2: the Handler ExceptionResolver :female_detective:

두 번재 error 처리 방법은 **HandlerExceptionResolver**를 정의하는 것이다.

이 방법은 애플리케이션 내에 어떤 exception이 thrown 되던지 상관없이 다 처리할 수 있는 방법이다.

이 방법은 또한 우리의 REST API에서 **uniform**한(통일된) exception handling 메커니즘을 구현하게 해준다.



참고로, 얘는 인터페이스다.

```java
public interface HandlerExceptionResolver {
    @Nullable
    ModelAndView resolveException(HttpServletRequest var1, HttpServletResponse var2, @Nullable Object var3, Exception var4);
}
```





이 인터페이스를 이용해서 custom resolver를 만들기 전에,

이미 Spring이 만들어둔 구현체들을 살펴보자.



### 3.1 ExceptionHandlerExceptionResolver

이 resolver는 Spring 3.1 버전에 도입됐고 *DispatcherServlet*에서 default로 enable 되어 있다.

참고로, 이 resolver는 앞에서 설명했던 **@ExceptionHandler**가 에러를 처리할 때 이용되는 핵심 구성 요소다.





### 3.2 DefaultHandlerExceptionResolver

이 resolver는 Spring 3.0 버전에 도입됐고 *DispatcherServlet*에서 default로 enable 되어 있다.

얘는 Spring exception들을 그에 대응하는 HTTP Status Code와 매핑해주는 역할을 했다.



이 resolver가 **응답의 Status Code**를 적절하게 세팅을 해줬지만, 

한가지 문제점이 있었는데 그건 바로 **응답의 body**에 아무것도 세팅을 안해준다는 것이었다.



> REST API에서는 **Status Code**는 Client에게 정보를 전달할 때 충분한 정보가 되지 못한다.
>
> error에 대한 추가적인 정보를 주기 위해서는 응답에 **Body** 또한 포함해야만 한다.



이 문제는 *ModelAndView*를 통해 뷰로 에러 컨텐츠를 렌더링해서 보여줌으로써 해결할 수 있지만 좋은 방법은 아니다.





### 3.3 ResponseStatusExceptionResolver

이 resolver도 Spring 3.0 버전에 도입됐고 *DispatcherServlet*에서 default로 enable 되어 있다.

이 resolver가 하는 역할은 custom exception들에 붙일 수 있는 **@ResponseStatus** 어노테이션을 사용해서 이 custom exception들을 **HTTP status code**와 매핑해주는 역할이었다.



Custom exception(사용자 정의 예외)는 아래와 같이 생겨먹을 수 있다.

```java
@ResponseStatus(value = HttpStatus.NOT_FOUND)
public class MyResourceNotFoundException extends RuntimeException {
    public MyResourceNotFoundException() {
        super();
    }
    public MyResourceNotFoundException(String message, Throwable cause) {
        super(message, cause);
    }
    public MyResourceNotFoundException(String message) {
        super(message);
    }
    public MyResourceNotFoundException(Throwable cause) {
        super(cause);
    }
}
```



이 resolver도 *DefaultHandlerExceptionResolver*랑 같은 한계점이 존재했다.

Status Code를 mapping 해주긴 하지만, Body가 여전히 null이었다는 것이다.





### 3.4 SimpleMappingExceptionResolver and AnnoatationMethodHandlerExceptionResolver

~~이름이 너무 긴거 아니냐~~



*SimpleMappingExceptionResolver*는 오래된 Spring MVC model에서 등장했으며 **REST Service**에는 적합하지 않다. 이 resovler는 exception class name을 view name에 매핑하는데 사용한다.



*AnnotationMethodHandlerExceptionResolver*는 Spring 3.0 버전에 도입됐지만 Spring 3.2 버전의 *ExceptionHandlerExceptionResolver*에 의해 deprecated 됐다.





### 3.5 Custom HandlerExceptionResolver

*DefaultHandlerExceptionResolver*와 *ResponseStatusExceptionResolver*의 조합은 

Spring RESTful Service의 error handling mechanism으로 오랫동안 이용되어져 왔다.

이 조합의 단점은 위에서도 언급했듯이, <u>**response body**에 대한 제어권이 없다는 것</u>이다.



우리는 client가 *Accept header*를 통해 요청한 포맷에 따라 JSON 또는 XML 형식으로 결과를 만들어내고 싶다. 이럴때 우리는 **Custom Exception Resolver**를 정의해 이 문제를 해결할 수 있다.



```java
@Component
public class RestResponseStatusExceptionResolver extends AbstractHandlerExceptionResolver {

    @Override
    protected ModelAndView doResolveException(
      HttpServletRequest request, 
      HttpServletResponse response, 
      Object handler, 
      Exception ex) {
        try {
            if (ex instanceof IllegalArgumentException) {
                return handleIllegalArgument(
                  (IllegalArgumentException) ex, response, handler);
            }
            ...
        } catch (Exception handlerException) {
            logger.warn("Handling of [" + ex.getClass().getName() + "] 
              resulted in Exception", handlerException);
        }
        return null;
    }

    private ModelAndView 
      handleIllegalArgument(IllegalArgumentException ex, HttpServletResponse response) 
      throws IOException {
        response.sendError(HttpServletResponse.SC_CONFLICT);
        String accept = request.getHeader(HttpHeaders.ACCEPT);
        ...
        return new ModelAndView();
    }
}
```



이 코드에서 주목해야 할 한 가지 세부사항은 우리가 ***request***에 대한 접근을 가지고 있다는 것이다. 따라서 우리는 client에 의해 전달된 *Accept* 헤더 내용을 고려할 수 있다.

예를 들어, client가 *application/json* 형식을 요청했으면, 우리는 *application/json*으로 인코딩된 **response body**를 반환할 수 있게 된다.



다른 한가지 중요한 세부 사항은 우리가 **ModelAndView**를 반환한다는 것이다. 이 <u>**ModelAndView**가 response의 body가 된다.</u> 이 객체에 필요한 어떤 정보든 세팅할 수 있다.



이 접근 방법은 일관성 있고 Spring REST Service에 대한 error handling 설정을 쉽게 할 수 있다는 장점이 있다.



하지만 아직도 한계점이 존재한다. 

이 방법은 low-level인 *HttpServletResponse*를 직접 이용하고 

*ModelAndView*를 사용하는 구식 MVC model에나 적합한 방법이다.

따라서 아직 개선의 여지가 남아있다... :cry:





## 4. Solution 3: @ControllerAdvice

Spring 3.2 버전은 **@ControllerAdvice** 어노테이션을 동반한 **global @ExceptionHandler** 기능을 도입했다.



이 방법은 더이상 구식 MVC model을 사용하지 않고 

type 안전성과 @ExceptionHandler의 유연성과 함께 *ResponseEntity*를 사용한다.

```java
@ControllerAdvice
public class RestResponseEntityExceptionHandler 
  extends ResponseEntityExceptionHandler {

    @ExceptionHandler(value 
      = { IllegalArgumentException.class, IllegalStateException.class })
    protected ResponseEntity<Object> handleConflict(
      RuntimeException ex, WebRequest request) {
        String bodyOfResponse = "This should be application specific";
        return handleExceptionInternal(ex, bodyOfResponse, 
          new HttpHeaders(), HttpStatus.CONFLICT, request);
    }
}
```



이 **@ControllerAdvice** 어노테이션은 여러개의 흩어진 **@ExceptionHandler**들을 하나의 global한 **error handling component**로 병합해주는 역할을 한다.



이 어노테이션의 실제 메커니즘은 지극히 단순하지만 매우 유연하다. :sparkles:

- Status Code 뿐만 아니라 Response Body에 대한 완전한 제어권을 개발자에게 제공한다.
- 함께 처리되어야 하는 여러개의 exception들을 동일한 하나의 메서드에 mapping 해주는 기능을 제공한다.
- RESTful *ResponseEntity* 응답을 잘 활용한다.



한가지 기억해두어야 할 것은 

<u>**@ExceptionHandler로 선언된 exception들을**</u> 

<u>**method의 parameter로 이용되는 exception에 match 시키는 것**이다.</u>



만약 match 되지 않아도 compiler는 빨간줄을 띄우지 않고 Spring도 그냥 진행시킨다.

하지만, runtime에 실제 exception이 던져졌을 때는 exception resolving mechanism이 실패할 것이다.

아래처럼...

```java
java.lang.IllegalStateException: No suitable resolver for argument [0] [type=...]
HandlerMethod details: ...
```







## 5. Solution 4: ResponseStatusException (Spring 5 and Above)

Spring 5는 *ResponseStatusException* 클래스를 도입했다.

*ResponseStatusException*의 인스턴스를 만들때는 *HttpStatus*와 *reason*, *cause*를 전달함으로써 만들 수 있다. (*Httpstatus*만 필수,  *reason, cause*는 option)



```java
@GetMapping(value = "/{id}")
public Foo findById(@PathVariable("id") Long id, HttpServletResponse response) {
    try {
        Foo resourceById = RestPreconditions.checkFound(service.findOne(id));

        eventPublisher.publishEvent(new SingleResourceRetrievedEvent(this, response));
        return resourceById;
     }
    catch (MyResourceNotFoundException exc) {
         throw new ResponseStatusException(
           HttpStatus.NOT_FOUND, "Foo Not Found", exc);
    }
}
```



그럼 *ResponseStatusException*의 **이점&middot;Benefits**은 뭐냐?

- Prototyping에 아주 좋다
  - basic solution을 아주 빠르게 구현할 수 있다.
- 1 Type, 多 Status Codes
  - 한 exception type이 여러 개의 서로 다른 응답을 만들어 낼 수도 있다.
  - **@ExceptionHandler**와 비교했을 때 결합도를 낮출 수 있다.
- 많은 custom exception class들을 만들 필요가 없어진다.
- exception들이 programmatical하게 생성될 수 있으므로 exception handling에 대한 제어가 더 강력해진다.



그럼 **단점&middot;tradeoff**은 뭐냐?

- exception handling의 통일된 방법이 존재하지 않는다.
  - global approach를 제공하는 **@ControllerAdvice**와는 반대로 application 전체에 통일된 방법을 강제하기 어렵다
- Code 중복
  - 여러개의 controller들에서 동일한 코드를 작성해야 한다.





물론, 한 애플리케이션 내에서 위에서 언급한 다양한 접근법들을 조합하는 것도 가능하다.

예를 들면, 우리는 **@ControllerAdvice**는 global하게 구현하고 **ResponseStatusExceptions**는 local하게 구현할 수 있다.



근데 이렇게 구현하면 주의할 점이 있다. 

만약 같은 exception이 다양한 방법으로 handle 될 수 있으면, 놀라운 결과를 목격할 수 있다. :grey_question:

이러한 상황에 적용되는 하나의 관습은 <u>특정한 하나의 exception은 오직 한 가지 방법으로만 handle</u> 하게 하는 것이다.



> detail한 내용은 https://www.baeldung.com/spring-response-status-exception를 참고하자...





## 6. Handle the Access Denied in Spring Security

이후 챕터는 공부를 더하고 추가하겠음 :wink:





## 7. Spring Boot Support





















