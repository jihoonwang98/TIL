# Testing MVC Web Controllers with Spring Boot and @WebMvcTest



> https://reflectoring.io/spring-boot-web-controller-test/
>
> This article is accompanied by a working code example [on GitHub](https://github.com/thombergs/code-examples/tree/master/spring-boot/spring-boot-testing).





**예시 컨트롤러**

```java
@RestController
@RequiredArgsConstructor
class RegisterRestController {
  private final RegisterUseCase registerUseCase;

  @PostMapping("/forums/{forumId}/register")
  UserResource register(
          @PathVariable("forumId") Long forumId,
          @Valid @RequestBody UserResource userResource,
          @RequestParam("sendWelcomeMail") boolean sendWelcomeMail) {

    User user = new User(
            userResource.getName(),
            userResource.getEmail());
    Long userId = registerUseCase.registerUser(user, sendWelcomeMail);

    return new UserResource(
            userId,
            user.getName(),
            user.getEmail());
  }

}
```





## 1. Web Controller의 책임



**컨트롤러의 책임(하는일)** 

| #    | 책임                        | 설명                                                         |
| :--- | :-------------------------- | :----------------------------------------------------------- |
| 1.   | **HTTP 요청 받기**          | 컨트롤러는 특정 URL, HTTP method, content-type에 반응해야 한다. |
| 2.   | **Deserialize Input**       | 컨트롤러는 HTTP 요청을 받아서 parsing하고, URL의 Path Variable과 HTTP 요청 파라미터 그리고 요청 본문을 자바 객체로 변환해야 한다. |
| 3.   | **Validate Input**          | 컨트롤러는 input 값을 validate하는 역할을 해야 한다.         |
| 4.   | **Call the Business Logic** | 컨트롤러는 parsing된 입력값을 business logic(서비스 계층)이 기대하는 입력값 형태로 변환한 후 business logic에게 전달해주어야 한다. |
| 5.   | **Serialize the Output**    | 컨트롤러는 business logic 처리 결과를 받아서 HTTP 응답으로 serialize 해야 한다. |
| 6.   | **Translate Exceptions**    | 만약 예외가 발생하면, 컨트롤러는 유저를 위해 해당 예외를 의미있는 에러메시지와 Http Status로 변환해야 한다. |

- 우리는 컨트롤러를 테스트할때는 위에 나열된 컨트롤러의 책임 리스트 외에는 관심을 가지면 안된다.
  - 예를 들어, 직접 비즈니스 로직을 수행하고 테스트한다던가 하면 안 된다.
  - 이렇게 되면, 우리의 컨트롤러 테스트 클래스가 뚱뚱해지고 유지보수하기 어려워진다.





## 2. Unit or Integration Test?



### Unit Test

- Unit test에서는 컨트롤러 단독으로 고립시켜서 테스트한다.
- 즉, 우리는 컨트롤러 객체를 인스턴스화시키고, 나머지 비즈니스 로직은 Mocking 시켜 버린다.



그렇다면 이러한 Unit Test가 위에서 살펴본 컨트롤러의 책임 6가지를 전부 제대로 해내고 있는지 테스트할 수 있을까?

| #    | 컨트롤러의 책임             | Unit Test가 제대로 테스트하고 있는가?                        |
| :--- | :-------------------------- | :----------------------------------------------------------- |
| 1.   | **Listen to HTTP Requests** | No, because the unit test would not evaluate the `@PostMapping` annotation and similar annotations specifying the properties of a HTTP request. |
| 2.   | **Deserialize Input**       | No, because annotations like `@RequestParam` and `@PathVariable` would not be evaluated. Instead we would provide the input as Java objects, effectively skipping deserialization from an HTTP request. |
| 3.   | **Validate Input**          | Not when depending on bean validation, because the `@Valid` annotation would not be evaluated. |
| 4.   | **Call the Business Logic** | Yes, because we can verify if the mocked business logic has been called with the expected arguments. |
| 5.   | **Serialize the Output**    | No, because we can only verify the Java version of the output, and not the HTTP response that would be generated. |
| 6.   | **Translate Exceptions**    | No. We could check if a certain exception was raised, but not that it was translated to a certain JSON response or HTTP status code. |

- 요약하면, Simple Unit Test는 HTTP Layer를 커버하지 못한다는 것이다.
  - 따라서 우리는 Spring의 도움을 받아서 HTTP Layer까지 테스트해야 한다.
  - 이 말은 즉슨 우리의 컨트롤러 코드와 Spring이 HTTP를 위해 지원하는 컴포넌트 사이의 Integration Test를 작성해야 한다는 말과 같다.
  - 하지만, Spring의 Context를 전체를 다 올려버리는 것이 아니라 Web Layer를 테스트할 때 필요한 Context의 부분만 따로 떼서 올리는 기능이 있다.





## 3. Verifying Controller Responsibilities with `@WebMvcTest`

Spring Boot는 `@WebMvcTest` 애너테이션을 통해 Web Controller를 테스트할 때 필요한 빈들만 포함하는 Application Context를 제공한다.



```java
@ExtendWith(SpringExtension.class) // Spring Boot 2.1 버전 이후부터는 명시하지 않아도 된다.
@WebMvcTest(controllers = RegisterRestController.class)
class RegisterRestControllerTest {
  @Autowired
  private MockMvc mockMvc;  // HTTP Layer를 테스트하기 위한 Mock 객체

  @Autowired
  private ObjectMapper objectMapper;  // 직렬화, 역직렬화 객체

  @MockBean  // RegisterRestController가 의존하는 객체를 @MockBean으로 Mocking
  private RegisterUseCase registerUseCase;

  @Test
  void whenValidInput_thenReturns200() throws Exception {
    mockMvc.perform(...);
  }

}
```







## 4. 컨트롤러의 1번 책임 테스트 - Verifying HTTP Request Matching

컨트롤러가 특정 HTTP Request에 응답하는지 Verify하는 테스트는 매우 직관적으로 작성할 수 있다.

아래처럼 우리는 단순히 `MockMvc` 객체의 `perform()` 메서드에 테스트하고 싶은 URL을 제공하기만 하면 된다.

뿐만 아니라 아래의 테스트는 올바른 HTTP 메서드와 올바른 요청 content-type인지도 검사한다.

```java
mockMvc.perform(post("/forums/42/register")
    .contentType("application/json"))
    .andExpect(status().isOk());
```



HTTP 요청을 매칭하는 더 많은 옵션은 [MockHttpServletRequestBuilder](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/test/web/servlet/request/MockHttpServletRequestBuilder.html)에서 확인할 수 있다.





## 5. 컨트롤러의 2번 책임 테스트 - Verifying Input Serialization

HTTP 요청 값이 성공적으로 Java 객체로 serialized 되었는지 검사하기 위해, 우리는 요청 값을 Test request에 제공해야 한다.

요청 값은 Request Body(`@RequestBody`)의 JSON content가 될 수도 있고,

URL 경로 변수(`@PathVariable`)이 될 수도 있고,

혹은 HTTP 요청 파라미터(`@RequestParam`)이 될 수도 있다.

```java
@Test
void whenValidInput_thenReturns200() throws Exception {
  UserResource user = new UserResource("Zaphod", "zaphod@galaxy.net");
  
   mockMvc.perform(post("/forums/{forumId}/register", 42L)  // 경로 변수 forumId에 42 설정
        .contentType("application/json")  // Request-Body의 content-type 설정
        .param("sendWelcomeMail", "true") // 요청 파라미터 설정
        .content(objectMapper.writeValueAsString(user))) // Request-Body의 content 설정
        .andExpect(status().isOk());  // ResponseStatus 결과 assert
}
```



위의 테스트가 통과하면 컨트롤러가 입력 값들을 성공적으로 serialize 하는 것을 확인할 수 있다.







## 6. 컨트롤러의 3번 책임 테스트 - Verifying Input Validation

`UserResource` 객체가 `@NotNull` 애너테이션을 사용해서 `null` 값들을 허용하지 않는다고 해보자.

```java
@Value
public class UserResource {

  @NotNull
  private final String name;

  @NotNull
  private final String email;
  
}
```

Bean Validation은 컨트롤러의 핸들러 메서드의 파라미터 앞에 `@Valid` 애너테이션을 붙여주면 자동으로 수행된다.

따라서 앞 절의 테스트가 통과되었다면 Validation 역시 통과된 것으로 볼 수 있다.





만약 validation이 실패하는 경우도 테스트하고 싶다면,

컨트롤러에게 유효하지 않은 `UserResource` JSON 객체를 보내는 테스트 케이스를 작성해보자.

이 경우에는 컨트롤러가 400 HTTP Status를 반환할 것을 예상해볼 수 있다.

```java
@Test
void whenNullValue_thenReturns400() throws Exception {
  UserResource user = new UserResource(null, "zaphod@galaxy.net");
  
  mockMvc.perform(post("/forums/{forumId}/register", 42L)
      ...
      .content(objectMapper.writeValueAsString(user)))
      .andExpect(status().isBadRequest());
}
```







## 7. 컨트롤러의 4번 책임 테스트 - Verifying Business Logic Calls

다음으로, 우리는 비즈니스 로직이 예상한 횟수대로 호출되었는지 역시 검사하고 싶다.

우리의 예시에서는 비즈니스 로직이 `RegisterUseCase`라는 인터페이스에 의해 제공되며,

`User` 객체와 `boolean` 을 입력값으로 기대한다.

```java
interface RegisterUseCase {
  Long registerUser(User user, boolean sendWelcomeMail);
}
```





우리는 컨트롤러가 아래처럼 전달받은 `UserResource` 객체를 통해 `User` 객체를 만들고 이 `User`를 `registerUser`로 전달할 것을 기대한다.

```java
// 컨트롤러 코드의 일부...
public UserResource register(@Valid @RequestBody UserResource userResource, ...) {

    User user = new User(
            userResource.getName(),
            userResource.getEmail());
    
    Long userId = registerUseCase.registerUser(user, sendWelcomeMail);

    return new UserResource(
            userId,
            user.getName(),
            user.getEmail());
}
```



이것을 검사하기 위해, 우리는 `RegisterUseCase` Mock 객체에게 물어볼 수 있다.

```java
@Test
void whenValidInput_thenMapsToBusinessModel() throws Exception {
  UserResource user = new UserResource("Zaphod", "zaphod@galaxy.net");
  mockMvc.perform(...);

  ArgumentCaptor<User> userCaptor = ArgumentCaptor.forClass(User.class);
  verify(registerUseCase, times(1)).registerUser(userCaptor.capture(), eq(true));
  assertThat(userCaptor.getValue().getName()).isEqualTo("Zaphod");
  assertThat(userCaptor.getValue().getEmail()).isEqualTo("zaphod@galaxy.net");
}
```

컨트롤러에 대한 호출이 수행되고 난 후에, 우리는 `ArgumentCaptor`를 이용해서

`RegisterUseCase.registerUser()`에 전달되었던 `User` 객체를 capture 해서

이 `User` 객체가 기대한 값들을 들고 있는지 assert 할 수 있다.







## 8. 컨트롤러의 5번 책임 테스트 - Verifying Output Serialization

business logic이 호출된 후, 우리는 컨트롤러가 결과 값을 JSON 문자열 형태로 변환하고 HTTP 응답 본문에 그것을 포함시킬 것을 기대할 수 있다.



```java
...
return new UserResource(
            userId,
            user.getName(),
            user.getEmail());
```

우리의 예시에서는, 우리는 HTTP 응답 본문이 유효한 `UserResource` 객체를 JSON 형태로 포함하고 있음을 기대할 수 있다.





```java
@Test
void whenValidInput_thenReturnsUserResource() throws Exception {
  MvcResult mvcResult = mockMvc.perform(...)
      ...
      .andReturn();

  UserResource expectedResponseBody = ...;
  String actualResponseBody = mvcResult.getResponse().getContentAsString();
  
  assertThat(actualResponseBody).isEqualToIgnoringWhitespace(
              objectMapper.writeValueAsString(expectedResponseBody));
}
```

응답 본문에 대한 assertion을 수행하기 위해, 우리는 `MvcResult` 타입의 변수에 `MockMvc.andReturn()` 메서드를 사용하여 HTTP 상호작용의 결과를 저장해야 한다.



그렇게 하고나면 우리는 응답 본문을 JSON String 형태(`mvcResult.getResponse().getContentAsString()`)로 받아올 수 있다.

그 다음 우리는 예상되는 문자열을 `isEqualToIgnoringWhitespace()` 메서드를 사용하여 비교할 수 있다.



우리는 위의 테스트 코드를 아래에서 살펴볼 custom `ResultMatcher`를 통해 더 읽기 좋게 만들 수 있다.





## 9. 컨트롤러의 6번 책임 테스트 - Verifying Exception Handling

보통 예외가 발생하면, 컨트롤러는 적절한 HTTP Status를 반환해야 한다.

예를 들어, 400은 요청에 문제가 있을 때, 500은 예외가 발생했을 때 반환된다.



스프링은 이러한 케이스들의 대부분을 default로 처리해준다.

그러나, 만약 우리가 custom exception handling을 정의해 놓았고, 그것을 테스트해보고 싶다고 하자.

우리는 structured(구조화, 미리 약속해놓은)된 JSON 에러 응답을 반환받고 싶다고 하자.

이 JSON 에러 응답은 에러가 생긴 필드명과 각 필드에 대한 에러 메시지를 포함하고 있다.



`@ControllerAdvice`를 통해 ExceptionHandler를 아래와 같이 정의했다고 하자.

```java
@ControllerAdvice
class ControllerExceptionHandler {
  @ResponseStatus(HttpStatus.BAD_REQUEST)
  @ExceptionHandler(MethodArgumentNotValidException.class)
  @ResponseBody
  public ErrorResult handleMethodArgumentNotValidException(MethodArgumentNotValidException e) {
    ErrorResult errorResult = new ErrorResult();
    for (FieldError fieldError : e.getBindingResult().getFieldErrors()) {
      errorResult.getFieldErrors()
              .add(new FieldValidationError(fieldError.getField(), 
                  fieldError.getDefaultMessage()));
    }
    return errorResult;
  }

  @Getter
  @NoArgsConstructor
  static class ErrorResult {
    private final List<FieldValidationError> fieldErrors = new ArrayList<>();
      
    public ErrorResult(String field, String message){
      this.fieldErrors.add(new FieldValidationError(field, message));
    }
  }

  @Getter
  @AllArgsConstructor
  static class FieldValidationError {
    private String field;
    private String message;
  }
  
}
```

만약 bean validation에 실패하면, Spring은 `MethodArgumentNotValidException`을 던진다.

우리는 Spring의 `FieldError` 객체를 우리의 `ErrorResult`로 변환하므로써 이 예외를 handle 할 수 있다.



위의 exception이 잘 handle 되는지를 검사하기 위해서, 아래와 같이 테스트를 작성할 것이다.

```java
@Test
void whenNullValue_thenReturns400AndErrorResult() throws Exception {
  UserResource user = new UserResource(null, "zaphod@galaxy.net");

  MvcResult mvcResult = mockMvc.perform(...)
          .contentType("application/json")
          .param("sendWelcomeMail", "true")
          .content(objectMapper.writeValueAsString(user)))
          .andExpect(status().isBadRequest())
          .andReturn();

  ErrorResult expectedErrorResponse = new ErrorResult("name", "must not be null");
  String actualResponseBody = 
      mvcResult.getResponse().getContentAsString();
  String expectedResponseBody = 
      objectMapper.writeValueAsString(expectedErrorResponse);
  assertThat(actualResponseBody)
      .isEqualToIgnoringWhitespace(expectedResponseBody);
}
```







## 10. Creating Custom ResultMatchers

어떤 assertion들은 쓰기도 힘들고 읽기도 힘들다.

특히 우리가 HTTP 응답 본문으로 받은 JSON 문자열을 우리가 기대하는 값과 비교하려면

많은 양의 코드를 써야하고 한 눈에 무슨 내용인지 파악하기가 쉽지 않다.



운 좋게도, 우리는 `MockMvc`의 fluent한 API를 사용해서 custom한 `ResultMatcher`들을 생성할 수 있다.





### 1) Matching JSON Output

우리가 아래처럼 HTTP 응답 본문을 예상값과 비교할 수 있다면 아주 편할 것이다.

```java
@Test
void whenValidInput_thenReturnsUserResource_withFluentApi() throws Exception {
  UserResource user = ...;
  UserResource expected = ...;

  mockMvc.perform(...)
      ...
      .andExpect(responseBody().containsObjectAsJson(expected, UserResource.class));
}
```





위처럼 코드를 작성하기 위해, 우리는 custom `ResultMatcher`를 생성해야 한다.

```java
public class ResponseBodyMatchers {
  private ObjectMapper objectMapper = new ObjectMapper();

  public <T> ResultMatcher containsObjectAsJson(
      Object expectedObject, 
      Class<T> targetClass) {
    return mvcResult -> {
      String json = mvcResult.getResponse().getContentAsString();
      T actualObject = objectMapper.readValue(json, targetClass);
      assertThat(actualObject).isEqualToComparingFieldByField(expectedObject);
    };
  }
  
  static ResponseBodyMatchers responseBody(){
    return new ResponseBodyMatchers();
  }
  
}
```

static method인 `responseBody()`는 우리의 fluent한 API의 entrypoint(입구, 시작점) 역할을 한다.





### 2) Matching Expected Validation Errors

```java
  ErrorResult expectedErrorResponse = new ErrorResult("name", "must not be null");
  String actualResponseBody = 
      mvcResult.getResponse().getContentAsString();
  String expectedResponseBody = 
      objectMapper.writeValueAsString(expectedErrorResponse);
  assertThat(actualResponseBody)
      .isEqualToIgnoringWhitespace(expectedResponseBody);
```

우리는 이렇게 지저분 했던 코드를 custom `ResultMatcher`를 사용하면,



```java
.andExpect(responseBody().containsError("name", "must not be null"));
```

위처럼 한 줄로 나타낼 수 있다.





이것을 가능하게 하기 위해, 우리는 `containsError()` 메서드를 `ResponseBodyMatchers` 클래스에 추가해야 한다.

```java
public class ResponseBodyMatchers {
  private ObjectMapper objectMapper = new ObjectMapper();

  public ResultMatcher containsError(
        String expectedFieldName, 
        String expectedMessage) {
    return mvcResult -> {
      String json = mvcResult.getResponse().getContentAsString();
      ErrorResult errorResult = objectMapper.readValue(json, ErrorResult.class);
      List<FieldValidationError> fieldErrors = errorResult.getFieldErrors().stream()
              .filter(fieldError -> fieldError.getField().equals(expectedFieldName))
              .filter(fieldError -> fieldError.getMessage().equals(expectedMessage))
              .collect(Collectors.toList());

      assertThat(fieldErrors)
              .hasSize(1)
              .withFailMessage("expecting exactly 1 error message"
                         + "with field name '%s' and message '%s'",
                      expectedFieldName,
                      expectedMessage);
    };
  }

  static ResponseBodyMatchers responseBody() {
    return new ResponseBodyMatchers();
  }
}
```















