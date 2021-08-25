# [JPA - Best Practice] Step-02,03 효과적인 Validation, 예외 처리 

API를 개발하다 보면 Front에서 넘어온 값에 대한 Validation 검사를 수없이 진행하게 된다.

이러한 **validation 작업을 보다 효율적으로 처리하고 정확한 예외 메시지를 Front에 전달해주는 것**이 이번 포스팅의 목표다.



### 중요 포인트

- `@Valid`를 통한 유효성 검사
- `@ControllerAdvice`를 이용한 Exception handling
- `ErrorCode` 에러 메시지 통합





## 1. `@Valid`를 통한 유효성 검사

### (1) DTO에 유효성 검사 애너테이션 추가

**AccountDto 클래스**

```java
public class AccountDto {

    @Getter
    @NoArgsConstructor(access = AccessLevel.PROTECTED)
    public static class SignUpReq {
        @Email
        private String email;
        @NotEmpty
        private String fistName;
        @NotEmpty
        private String lastName;
        @NotEmpty
        private String password;
        @NotEmpty
        private String address1;
        @NotEmpty
        private String address2;
        @NotEmpty
        private String zip;
        @Builder
        public SignUpReq(String email, String fistName, String lastName, String password, String address1, String address2, String zip) {
            this.email = email;
            this.fistName = fistName;
            this.lastName = lastName;
            this.password = password;
            this.address1 = address1;
            this.address2 = address2;
            this.zip = zip;
        }

        public Account toEntity() {
            return Account.builder()
                    .email(this.email)
                    .fistName(this.fistName)
                    .lastName(this.lastName)
                    .password(this.password)
                    .address1(this.address1)
                    .address2(this.address2)
                    .zip(this.zip)
                    .build();
        }

    }
    
    // ...
}
```

- 회원 가입을 위한 SignUpReq 클래스에 유효성 검사를 위해 `@Email`, `@NotEmpty`를 추가했다.

- 실질적인 유효성 검사는 Controller에서 `@Valid` 애너테이션을 통해 검사된다.





### (2) Controller에서 유효성 검사

```java
@RestController
@RequestMapping("accounts")
@AllArgsConstructor
public class AccountController {

    private AccountService accountService;

    @RequestMapping(method = RequestMethod.POST)
    @ResponseStatus(value = HttpStatus.CREATED)
    public AccountDto.Res signUp(@RequestBody @Valid final AccountDto.SignUpReq dto) {
        return new AccountDto.Res(accountService.create(dto));
    }
    
    // ...
}
```

- 파라미터의 DTO 앞에 `@Valid` 애너테이션을 붙여서 유효성 검사를 진행한다.
  - 유효성 검사에 실패하면 `MethodArgumentNotValidException` 예외가 발생한다.

- Front에서 넘겨받은 값에 대한 유효성 검사는 엄청나게 반복적이며, 

  실패했을 경우 사용자에게 적절한 Response 값을 리턴해주는 것 또한 

  중요한 비즈니스 로직이 아님에도 불구하고 많은 시간을 할애하게 된다.

  - 다음 절에서 `MethodArgumentNotValidException` 발생 시 공통적으로 사용자에게 적절한 Response 값을 리턴해주는 작업을 진행한다.



## 2. `@ControllerAdvice`를 이용한 Exception handling



### (1) 스프링 자체의 Default Error Handling

**Error Response**

```json
{
  "timestamp": 1525182817519,
  "status": 400,
  "error": "Bad Request",
  "exception": "org.springframework.web.bind.MethodArgumentNotValidException",
  "errors": [
    {
      "codes": [
        "Email.signUpReq.email",
        "Email.email",
        "Email.java.lang.String",
        "Email"
      ],
      "arguments": [
        {
          "codes": [
            "signUpReq.email",
            "email"
          ],
          "arguments": null,
          "defaultMessage": "email",
          "code": "email"
        },
        [],
        {
          "arguments": null,
          "defaultMessage": ".*",
          "codes": [
            ".*"
          ]
        }
      ],
      "defaultMessage": "이메일 주소가 유효하지 않습니다.",
      "objectName": "signUpReq",
      "field": "email",
      "rejectedValue": "string",
      "bindingFailure": false,
      "code": "Email"
    }
  ],
  "message": "Validation failed for object='signUpReq'. Error count: 3",
  "path": "/accounts"
}
```

- 너무 많은 값들을 돌려보내 주고 있고, 시스템 정보에 대한 값들도 포함하고 있어서 위처럼 응답하는 것은 바람직하지 않음.
- 또 응답 결과 포맷이 항상 같아야 프론트에서 처리하기 쉽다.
- 아래에서 `ErrorResponse ` 클래스를 통해 공통적인 예외 Response를 만들자.





### (2) MethodArgumentNotValidException의 Reponse 처리

**ErrorExceptionController 클래스**

```java
@ControllerAdvice
@ResponseBody
@Slf4j
public class ErrorExceptionController {

    // ...

    @ExceptionHandler(MethodArgumentNotValidException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    protected ErrorResponse handleMethodArgumentNotValidException(MethodArgumentNotValidException e) {
        log.error(e.getMessage());
        final BindingResult bindingResult = e.getBindingResult();
        final List<FieldError> errors = bindingResult.getFieldErrors();

        return buildFieldErrors(
                ErrorCode.INPUT_VALUE_INVALID,
                errors.parallelStream()
                        .map(error -> ErrorResponse.FieldError.builder()
                                .reason(error.getDefaultMessage())
                                .field(error.getField())
                                .value((String) error.getRejectedValue())
                                .build())
                        .collect(Collectors.toList())
        );
    }
    
    private ErrorResponse buildFieldErrors(ErrorCode errorCode, List<ErrorResponse.FieldError> errors) {
        return ErrorResponse.builder()
                .code(errorCode.getCode())
                .status(errorCode.getStatus())
                .message(errorCode.getMessage())
                .errors(errors)
                .build();
    }
    
    // ...
}
```



**ErrorCode Enum 클래스**

```java
@Getter
public enum ErrorCode {

    ACCOUNT_NOT_FOUND("AC_001", "해당 회원을 찾을 수 없습니다.", 404),
    EMAIL_DUPLICATION("AC_001", "이메일이 중복되었습니다.", 400),
    INPUT_VALUE_INVALID("???", "입력값이 올바르지 않습니다.", 400);


    private final String code;
    private final String message;
    private final int status;

    ErrorCode(String code, String message, int status) {
        this.code = code;
        this.message = message;
        this.status = status;
    }

}
```



**ErrorResponse 클래스**

```java
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class ErrorResponse {

    private String message;
    private String code;
    private int status;
    private List<FieldError> errors = new ArrayList<>();

    @Builder
    public ErrorResponse(String message, String code, int status, List<FieldError> errors) {
        this.message = message;
        this.code = code;
        this.status = status;
        this.errors = initErrors(errors);
    }

    private List<FieldError> initErrors(List<FieldError> errors) {
        return (errors == null) ? new ArrayList<>() : errors;
    }

    @Getter
    @NoArgsConstructor(access = AccessLevel.PROTECTED)
    public static class FieldError {
        private String field;
        private String value;
        private String reason;

        @Builder
        public FieldError(String field, String value, String reason) {
            this.field = field;
            this.value = value;
            this.reason = reason;
        }
    }

}
```





**ErrorResponse**

```json
{
  "message": "입력값이 올바르지 않습니다.",
  "code": "???",
  "status": 400,
  "errors": [
    {
      "field": "email",
      "value": "string",
      "reason": "이메일 주소가 유효하지 않습니다."
    },
    {
      "field": "lastName",
      "value": null,
      "reason": "반드시 값이 존재하고 길이 혹은 크기가 0보다 커야 합니다."
    },
    {
      "field": "fistName",
      "value": null,
      "reason": "반드시 값이 존재하고 길이 혹은 크기가 0보다 커야 합니다."
    }
  ]
}
```

- 위처럼 항상 같은 포맷으로 `ErrorResponse`를 응답할 수 있다.

- `@Valid` 애너테이션으로 발생하는 `MethodArgumentNotValidException`들은 모두 위에서 정의한

  handleMethodArgumentNotValidException 메서드를 통해 공통된 Response 값을 리턴한다.

- 이제부터는 `@Valid`, 해당 필드에 맞는 애너테이션을 통해 모든 유효성 검사를 진행할 수 있다.





## 3. 사용자 정의 에러 Handling

**AccountNotFoundException: 새로운 Exception 정의**

```java
public class AccountNotFoundException extends RuntimeException {
    private long id;

    public AccountNotFoundException(long id) {
        this.id = id;
    }
}

public Account findById(long id) {
    final Optional<Account> account = accountRepository.findById(id);
    account.orElseThrow(() -> new AccountNotFoundException(id));
    return account.get();
}
```





**handleAccountNotFoundException: 핸들링**

```java
@ExceptionHandler(value = {
        AccountNotFoundException.class
})
@ResponseStatus(HttpStatus.NOT_FOUND)
protected ErrorResponse handleAccountNotFoundException(AccountNotFoundException e) {
    final ErrorCode accountNotFound = ErrorCode.ACCOUNT_NOT_FOUND;
    log.error(accountNotFound.getMessage(), e.getMessage());
    return buildError(accountNotFound);
}
```





**Response**

```json
{
  "message": "해당 회원을 찾을 수 없습니다.",
  "code": "AC_001",
  "status": 404,
  "errors": []
}
```





## 4. 한계

위의 유효성 검사의 단점은 아래와 같다.

- 모든 Request DTO에 대한 반복적인 유효성 검사의 애너테이션이 필요하다.
  - 회원 가입, 회원 정보 수정 등등 지속적으로 DTO 클래스가 추가되고, 그때마다 반복적으로 애너테이션이 추가된다.
- 유효성 검사 로직이 변경되면 모든 곳에 변경이 따른다.
  - 만약 비밀번호 유효성 검사 기준에 특수문자가 추가된다고 하면 비밀번호 변경에 따른 유효성 검사를 정규 표현식의 변경을 모든 DTO마다 해줘야 한다.





# [JPA - Best Practice] step-03 : 효과적인 validate, 예외 처리 (2)



### 중요 포인트

- @Embeddable / @Embedded
- DTO 변경



## 1. @Embeddable / @Embedded

**@Embeddable / @Embedded 적용**

```java
public class Account {
    @Embedded
    private com.cheese.springjpa.Account.model.Email email;
}

@Embeddable
public class Email {

    @org.hibernate.validator.constraints.Email
    @Column(name = "email", nullable = false, unique = true)
    private String address;
}
```

- 임베디드 키워드를 통해 새로운 값 타입을 직접 정의해서 사용할 수 있다.
- Email 클래스를 새로 생성하고 거기에 Email 칼럼에 매핑했다.





## 2. DTO 변경

**AccountDto.class**

```java
public static class SignUpReq {

    // @Email 기존코드
    // private String email;
    @Valid // @Valid 반드시 필요
    private com.cheese.springjpa.Account.model.Email email;

    private String zip;
    @Builder
    public SignUpReq(com.cheese.springjpa.Account.model.Email email, String fistName, String lastName, String password, String address1, String address2, String zip) {
        this.email = email;
        ...
        this.zip = zip;
    }

    public Account toEntity() {
        return Account.builder()
                .email(this.email)
                ...
                .zip(this.zip)
                .build();
    }
}
```

- 원래는 모든 Request Dto의 email 필드에 `@Email` 애너테이션을 붙여줘야 했는데,

  새로운 Email 클래스를 바라보게 변경하면 해당 클래스의 이메일 유효성 검사를 바라보게 된다.

  - 이메일에 대한 유효성 검사는 Embeddable 타입의 Email 클래스가 관리하게 된다.





## 3. 단점

```java
{
  "address1": "string",
  "address2": "string",
  "email": {
    "address": "string"
  },
  "fistName": "string",
  "lastName": "string",
  "password": "string",
  "zip": "string"
}
```

- 위처럼 json 포맷이 변경된다.