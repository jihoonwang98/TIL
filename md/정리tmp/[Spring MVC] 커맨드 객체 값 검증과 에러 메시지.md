# [Spring MVC] 커맨드 객체 값 검증과 에러 메시지



> 참고 자료: 
>
> 웹 개발자를 위한 Spring 4.0 프로그래밍 by 최범균,
>
> [Spring documentation](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/validation/Errors.html)





## 1. Validator와 Errors/BindingResult를 이용한 객체 검증

`org.springframework.validation.Validator` 인터페이스를 사용하면,

스프링에 제공하는 객체 검증 및 에러 메시지 지원 등의 기능을 사용할 수 있다.

컨트롤러에서 커맨드 객체의 값을 검증할 때 특히 `Validator`를 유용하게 사용할 수 있다.





### `Validator` 인터페이스

`Validator` 인터페이스는 다음과 같이 두 개의 메서드를 정의하고 있다.

```java
package org.springframework.validation;

public interface Validator {
    boolean supports(Class<?> aClass);
    void validate(Object target, Errors errors);
}
```



- `boolean supports(Class<?>)`: Validator가 해당 타입의 객체를 지원하는지 여부를 리턴한다.
- `void validate(Object, Errors)`: Object 값을 검증하며, 값이 올바르지 않을 경우 그 내용을 Errors 객체에 저장한다.







#### `Validator` 인터페이스 구현 예시



**MemberRegisterRequest 클래스**

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class MemberRegisterRequest {
    
    private String email;
    private String name;
    private String password;
    private String confirmPassword;
    
    private Address address;
}
```



**MemberRegisterValidator 클래스**

```java
public class MemberRegisterValidator implements Validator {


    @Override
    public boolean supports(Class<?> aClass) {
        return MemberRegisterRequest.class.isAssignableFrom(aClass);
    }

    @Override
    public void validate(Object target, Errors errors) {
        MemberRegisterRequest request = (MemberRegisterRequest) target;

        // email이 null이거나 blank 값("   " 같은거)인 경우 reject
        if (request.getEmail() == null || request.getEmail().trim().isEmpty()) {
            errors.rejectValue("email", "required");
        }

        ValidationUtils.rejectIfEmptyOrWhitespace(errors, "name", "required");
        ValidationUtils.rejectIfEmptyOrWhitespace(errors, "password", "required");
        ValidationUtils.rejectIfEmptyOrWhitespace(errors, "confirmPassword", "required");

        if (request.getPassword().length() < 5) {
            errors.rejectValue("password", "ShortPassword");
        } else if (!request.getPassword().equals(request.getConfirmPassword())) {
            errors.rejectValue("confirmPassword", "confirmPasswordNotSame");
        }


        Address address = request.getAddress();

        if(address == null) {
            errors.rejectValue("address", "required");
        } else {
            errors.pushNestedPath("address");

            try {
                ValidationUtils.rejectIfEmptyOrWhitespace(errors, "address1", "required");
                ValidationUtils.rejectIfEmptyOrWhitespace(errors, "address2", "required");
            } finally {
                errors.popNestedPath();
            }
        }
    }
}
```



**메서드 설명**

- `Errors#rejectValue(String field, String errorCode)`: field에 대한 errorCode를 추가한다.

- `ValidationUtils.rejectIfEmptyOrWhitespace(Errors errors, String field, String errorCode)`: 

  field 값이 null인 경우 또는 길이가 0이거나 값이 공백 문자로 구성된 경우, errors에 field에 대한 errorCode를 추가한다.



> 위의 코드와 아래 코드는 같은 처리를 한다.
>
> ```java
> if (request.getEmail() == null || request.getEmail().trim().isEmpty()) {
>     errors.rejectValue("email", "required");
> }
> ```
>
> ```java
> ValidationUtils.rejectIfEmptyOrWhitespace(errors, "email", "required");
> ```





Validator를 구현했다면, 컨트롤러에서 Validator를 이용해서 커맨드 객체를 검증할 수 있다.

앞서 작성한 Validator를 이용해 값을 검증해 보자.



**RegistrationController 클래스**

```java
@RestController
public class RegistrationController {

    @PostMapping("/register")
    public Object register(@RequestBody MemberRegisterRequest registerRequest, BindingResult bindingResult) {

        // 유효성 검사
        new MemberRegisterValidator().validate(registerRequest, bindingResult);

        // errors 값 체크
        if (bindingResult.hasErrors()) {
            List<FieldError> fieldErrors = bindingResult.getFieldErrors();
            return fieldErrors;
        }

        return "Success";
    }
}
```





> **[주의]** 
>
> Errors 파라미터와 BindingResult 파라미터는 반드시 커맨드 객체 파라미터 바로 뒤에 위치해야 한다.
>
> 안그러면 스프링 MVC는 Exception을 발생시킨다.





**JSON 요청/응답 예시**

```json
// 요청
{
    "email": "jihoonwang98@naver.com",
    "name": "jihoonwang",
    "password": "password",
    "confirmPassword": "password",
    "address": {
        "address1": "addr1",
        "address2": "addr2"
    }
}

// 응답
Success
```



```json
// 요청 (빈 문자열 전달한 경우)
{
    "email": "jihoonwang98@naver.com",
    "name": "jihoonwang",
    "password": "password",
    "confirmPassword": "password",
    "address": {
        "address1": "addr1",
        "address2": ""
    }
}

// 응답
[
    {
        "codes": [
            "required.memberRegisterRequest.address.address2",
            "required.address.address2",
            "required.address2",
            "required.java.lang.String",
            "required"
        ],
        "arguments": null,
        "defaultMessage": null,
        "objectName": "memberRegisterRequest",
        "field": "address.address2",
        "rejectedValue": "",
        "bindingFailure": false,
        "code": "required"
    }
]
```



```json
// 요청 (비밀번호 다른 경우)
{
    "email": "jihoonwang98@naver.com",
    "name": "jihoonwang",
    "password": "password",
    "confirmPassword": "asdf",
    "address": {
        "address1": "addr1",
        "address2": "addr2"
    }
}

// 응답
[
    {
        "codes": [
            "confirmPasswordNotSame.memberRegisterRequest.confirmPassword",
            "confirmPasswordNotSame.confirmPassword",
            "confirmPasswordNotSame.java.lang.String",
            "confirmPasswordNotSame"
        ],
        "arguments": null,
        "defaultMessage": null,
        "objectName": "memberRegisterRequest",
        "field": "confirmPassword",
        "rejectedValue": "asdf",
        "bindingFailure": false,
        "code": "confirmPasswordNotSame"
    }
]
```



```json
// 요청 (비밀번호 짧은 경우)
{
    "email": "jihoonwang98@naver.com",
    "name": "jihoonwang",
    "password": "pas",
    "confirmPassword": "pas",
    "address": {
        "address1": "addr1",
        "address2": "addr2"
    }
}

// 응답
[
    {
        "codes": [
            "ShortPassword.memberRegisterRequest.password",
            "ShortPassword.password",
            "ShortPassword.java.lang.String",
            "ShortPassword"
        ],
        "arguments": null,
        "defaultMessage": null,
        "objectName": "memberRegisterRequest",
        "field": "password",
        "rejectedValue": "pas",
        "bindingFailure": false,
        "code": "ShortPassword"
    }
]
```





## 2. Errors와 BindingResult 인터페이스의 주요 메서드

`Validator`는 객체 데이터에 오류가 있을 경우 `Errors` 객체에 오류를 등록한다.

`org.springframework.validation.Errors` 인터페이스는 오류를 등록하기 위한 다양한 메서드를 제공하고 있으며, 주요 메서드는 다음과 같다.





### `Errors` 인터페이스의 주요 메서드 - 1 (에러 등록)

| 메서드                                                       | 설명                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| **void reject(String errorCode)**                            | 전체 객체에 대한 글로벌 에러 코드를 추가한다.                |
| void reject(String errorCode, String defaultMessage)         | 전체 객체에 대한 글로벌 에러 코드를 추가한다. 에러 코드에 메시지가 존재하지 않을 경우 defaultMessage를 사용한다. |
| void reject(String errorCode, Object[] errorArgs, String defaultMessage) | 전체 객체에 대한 글로벌 에러 코드를 추가한다. 메시지 인자로 errorArgs를 전달한다. 에러 코드에 대한 메시지가 존재하지 않을 경우 defaultMessage를 사용한다. |
| **void rejectValue(String field, String errorCode)**         | 필드에 대한 에러 코드를 추가한다.                            |
| void rejectValue(String field, String errorCode, String defaultMessage) | 필드에 대한 에러 코드를 추가한다. 에러 코드에 대한 메시지가 존재하지 않을 경우 defaultMessage를 사용한다. |
| void rejectValue(String field, String errorCode, Object[] errorArgs, String defaultMessage) | 필드에 대한 에러 코드를 추가한다. 메시지 인자로 errorArgs를 전달한다. 에러 코드에 대한 메시지가 존재하지 않을 경우 defaultMessage를 사용한다. |



> 위에서 '**전체 객체에 대한 에러**'란 말은 
>
> **필드**에 대한 에러가 아니라 그 **객체**에 대한 에러란 뜻이다.





### 객체 자체에 Error가 있는 경우

앞 절의 예시에서는 필드에 대한 에러 코드를 추가해봤다.

그럼 **객체 자체에 에러**가 있는 경우는 어떤 경우일까?

예를 들어, 로그인을 처리하는 경우 아이디와 암호가 일치하지 않으면 로그인 폼을 다시 보여주고 에러 정보를 출력할 것이다.

이 경우 아이디 필드나 암호 필드에 에러가 있다기보다는 객체 자체에 문제가 있다고 볼 수 있다. 

이렇게 객체 자체에 에러가 있는 경우 다음과 같이 reject() 메서드를 사용해서 글로벌 에러 정보를 추가할 수 있다.



> 객체 자체에 대한 에러  ==  글로벌 에러



**Authenticator 클래스**

```java
public class Authenticator {
    
    private static String username = "user1";
    private static String password = "pwd1234";
    
    public static void authenticate(String username, String password) throws AuthenticationException {
        if(!(username.equals(Authenticator.username) && password.equals(Authenticator.password))) {
            throw new AuthenticationException("Invalid User");
        }
    }
}
```

예시를 단순화하기 위해 username이 "user1"이고, password가 "pwd1234"인 유저만 로그인할 수 있다고 하자. 



**LoginRequest 클래스**

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class LoginRequest {

    private String username;
    private String password;
}
```





**LoginController 클래스**

```java
@RestController
public class LoginController {

    @PostMapping("/login")
    public String login(LoginRequest loginRequest, Errors errors) {

        try {
            Authenticator.authenticate(loginRequest.getUsername(), loginRequest.getPassword());
            return "login Success...";
        } catch (AuthenticationException e) {
            errors.reject("InvalidIdOrPassword");
            return "login failed...";
        }
    }
}
```

LoginRequest 객체를 검증해서 Username과 Password가 유효하지 않다면

AuthenticationException이 발생하게 되고, 이때 catch문 내에서 

`errors.reject(..)` 메서드를 통해 해당 객체에 대한 에러 정보를 추가한다.



앞선 예시에서 `errors.rejectValue(..)` 메서드는 객체 내의 필드에 대한 에러 정보를 추가했다면,

 `errors.reject(..)` 메서드는 객체 자체에 대한 에러 정보를 추가한다.





> **[참고]**
>
> 글로벌 에러 정보나 특정 필드에 대한 에러 정보는 두 번 이상 추가될 수 있다.
>
> 예를 들어, 다음과 같이 특정 필드에 대해 에러 정보를 두 번 이상 등록할 수 있다.
>
> ```java
> errors.rejectValue("id", "invalidLength");
> errors.rejectValue("id", "invalidCharacter");
> ```







### `Errors` 인터페이스의 주요 메서드 - 2 (에러 여부 확인)

| 메서드                               | 설명                                                         |
| ------------------------------------ | ------------------------------------------------------------ |
| boolean hasErrors()                  | 에러(글로벌 에러, 필드 에러 합친 전체 에러)가 존재하는 경우 true를 리턴한다. |
| int getErrorCount()                  | 에러 개수를 리턴한다.                                        |
| boolean hasGlobalErrors()            | reject() 메서드를 이용해서 추가된 글로벌 에러가 존재할 경우 true를 리턴한다. |
| int getGlobalErrorCount()            | reject() 메서드를 이용해서 추가된 글로벌 에러 개수를 리턴한다. |
| boolean hasFieldErrors()             | rejectValue() 메서드를 이용해서 추가된 에러가 존재할 경우 true를 리턴한다. |
| int getFieldErrorCount()             | rejectValue() 메서드를 이용해서 추가된 에러 개수를 리턴한다. |
| boolean hasFieldErrors(String field) | rejectValue() 메서드를 이용해서 추가한 특정 필드의 에러가 존재할 경우 true를 리턴한다. |
| int getFieldErrorCount(String field) | rejectValue() 메서드를 이용해서 추가한 특정 필드의 에러 개수를 리턴한다. |





위 메서드들을 이용하면 에러 존재 여부에 따라 알맞은 처리를 수행할 수 있다.

예를 들어, 폼 검증 결과 에러가 존재해서 다시 폼을 보여주도록 하고 싶다면, 

다음과 같이 hasErrors() 메서드를 이용해서 검증 에러 존재 여부를 확인할 수 있다.

```java
@PostMapping
public String register(@ModelAttribute("memberInfo") MemberRegisterRequest request, BindingResult bindingResult) {
    new MemberRegisterValidator().validate(request, bindingResult);
    
    if(bindingResult.hasErrors()) {
        return MEMBER_REGISTRATION_FORM;
    }
}
```









### `BindingResult` 인터페이스의 메서드

`BindingResult` 인터페이스는 `Errors` 인터페이스를 상속 받았으며, 

에러 메시지를 구하거나 검증 대상 객체를 구하는 등의 추가 기능을 정의하고 있다.

`BindingResult` 인터페이스에 정의된 메서드 중 주요 메서드는 다음과 같다. 

(실제로 컨트롤러 구현 과정에서 BindingResult 인터페이스의 메서드를 직접 호출할 경우는 거의 없다. 참고만 하자.)

| 메서드                                                       | 설명                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| Object getTarget()                                           | 검사 대상이 되는 객체를 구한다. 컨트롤러의 경우 커맨드 객체가 된다. |
| String[] resolveMessageCodes(String errorCode)               | 에러 코드를 메시지 코드로 변환한다.                          |
| String[] resolveMessageCodes(String errorCode, String field) | 에러 코드를 field에 해당하는 메시지 코드로 변환한다.         |



참고로, 메시지 코드는 뒤에서 설명할 에러 메시지 출력에 사용된다.









### `ValidationUtils` 클래스를 이용한 값 검증

Validator 구현 클래스의 validate() 메서드에서는 프로퍼티의 값이 존재하는지 확인하기 위해 

다음과 같이 값이 null 또는 공백 문자인지 여부를 확인한다.

```java
if (request.getEmail() == null || request.getEmail().trim().isEmpty()) {
    errors.rejectValue("email", "required");
}
```



위와 같은 코드는 매번 반복되고 비교를 누락하는 실수를 유발할 수 있기 때문에 

`ValidationUtils` 클래스가 제공하는 메서드를 사용하면 코드를 간결하게 유지하면서도 불필요한 실수를 줄일 수 있다.



위의 코드는 아래의 코드로 대체할 수 있다.

```java
ValidationUtils.rejectIfEmptyOrWhitespace(errors, "email", "required");
```





`ValidationUtils` 클래스에는 아래와 같은 두 가지 기능의 메서드가 들어 있다.

- `void rejectIfEmpty(..)`: 값이 null이거나 길이가 0인 경우 에러 코드를 추가.
- `void rejectIfEmptyOrWhitespace(..)`: 값이  null이거나 길이가 0인 경우 또는 공백 문자로 구성되어 있는 경우 에러 코드를 추가.





## 3. 에러 코드와 메시지

Validator를 이용해서 오류를 확인하면, 오류 내용을 화면에 보여주고 싶을 것이다.

오류 내용을 알려줄 때 사용할 수 있는 것이 **에러 코드**다.

스프링 MVC는 에러 코드로부터 메시지를 가져오는 방법을 제공하고 있으며,

이 메시지를 응답 결과에 보여주는 방법 또한 제공하고 있다.



검증 과정에서 추가된 에러 메시지를 사용하려면 다음과 같은 코드를 작성해야 한다.

- 메시지를 읽어올 때 사용할 MessageSource를 스프링 설정에 등록한다.
- MessageSource에서 메시지를 가져올 때 사용할 프로퍼티 파일을 작성한다.
- JSP와 같은 뷰 코드에서 스프링이 제공하는 태그를 이용해 에러 메시지를 출력한다.





### `MessageSource` 등록하기

에러 메시지를 출력하려면 먼저 `MessageSource`를 등록해야 한다. 

다음은 등록 예이다.



**SpringBootApplication 클래스**

```java
@SpringBootApplication
public class MySpringPracticeApplication {

    public static void main(String[] args) {
        SpringApplication.run(MySpringPracticeApplication.class, args);
    }


    @Bean
    public MessageSource messageSource() {
        ResourceBundleMessageSource messageSource = new ResourceBundleMessageSource();
        messageSource.setDefaultEncoding("UTF-8");
        
        // resources/messages/error.properties에서 불러옴.
        messageSource.setBasename("messages/error");
        return messageSource;
    }
}
```



**error.properties**

```properties
required=필수 항목입니다.
minlength={0}는 최소 {1} 글자 이상 입력해야 합니다.
maxlength={0}는 최대 {1} 글자까지만 입력해야 합니다.
unsafe.password=암호는 알파벳과 숫자를 포함해야 합니다.
```

위 메시지 파일에서 required, minlength, maxlength, unsafe.password 등은 메시지 코드가 된다.



스프링 MVC는 에러 코드로부터 생성된 메시지 코드를 이용해 에러 메시지를 출력하게 된다.





#### 글로벌 에러 코드 우선 순위

**글로벌 에러 코드**인 경우 다음의 우선 순위에 따라 메시지 코드를 생성한다.

1. 에러코드 + "." + 커맨드 객체 이름
2. 에러코드



예를 들어, Validator에서 글로벌 에러 코드를 아래와 같이 등록했다고 하자.

```java
@RestController
public class LoginController {

    @PostMapping("/login")
    public String login(LoginRequest loginRequest, Errors errors) {

        try {
            Authenticator.authenticate(loginRequest.getUsername(), loginRequest.getPassword());
            return "login Success...";
        } catch (AuthenticationException e) {
            errors.reject("invalidIdOrPassword");  // 글로벌 에러 코드
            return "login failed...";
        }
    }
}
```



위 코드에서 사용한 **글로벌 에러 코드**는 "invalidIdOrPassword"고, **커맨드 객체의 모델 이름**은 "loginCommand"다.

따라서, 메시지를 읽어올 때에는 다음의 두 메시지 코드를 사용한다.

1. invalidIdOrPassword.loginCommand
2. invalidIdOrPassword



에러 메시지를 읽어올 때, 먼저 메시지 코드가 invalidIdOrPassword.loginCommand인 메시지를 찾는다.

존재하면 그 메시지를 사용하고, 존재하지 않으면 invalidIdOrPassword를 사용한다.

따라서 메시지 프로퍼티 파일에 다음과 같이 invalidIdOrPassword 메시지만 정의되어 있으면, 

에러 메시지를 출력할 때 invalidIdOrPassword 메시지의 내용을 사용한다.

```properties
required=필수 항목입니다.
minlength={0}는 최소 {1} 글자 이상 입력해야 합니다.
maxlength={0}는 최대 {1} 글자까지만 입력해야 합니다.
invalidIdOrPassword=아이디와 암호가 일치하지 않습니다.
```





#### 필드 에러 코드 우선 순위 (필드가 단순한 경우)

Errors.rejectValue(..)를 이용해서 생성한 에러 코드는 다음의 우선순위를 사용해서 메시지 코드를 생성한다.

1. 에러코드 + "." + 커맨드 객체 이름 + "." + 필드명
2. 에러코드 + "." + 필드명
3. 에러코드 + "." + 필드타입
4. 에러코드





#### 필드 에러 코드 우선 순위 (필드가 List나 목록인 경우)

필드가 List나 목록인 경우 다음의 순서를 이용해서 메시지 코드를 생성한다.

1. 에러코드 + "." + 커맨드 객체 이름 + "." + 필드명[인덱스].중첩필드명
2. 에러코드 + "." + 커맨드 객체 이름 + "." + 필드명.중첩필드명
3. 에러코드 + "." + 필드명[인덱스].중첩필드명
4. 에러코드 + "." + 필드명.중첩필드명
5. 에러코드 + "." + 중첩필드명
6. 에러코드 + "." + 필드타입
7. 에러코드





에러 메시지 프로퍼티 파일이 아래와 같고, 

email과 password 프로퍼티에 값이 없어서 둘 다 에러 코드로 "required"를 등록했다고 하자.

```properties
required=필수 항목입니다.
required.email=이메일을 입력하세요.
```



이 경우, 메시지 코드 우선순위에 따라 

"email" 프로퍼티의 에러 메시지는 "required.email" 메시지를 이용하고,

"password" 프로퍼티의 에러 메시지는 "required" 메시지를 이용하게 된다.









## 4. `@Valid` 애너테이션과 `@InitBindier` 애너테이션을 이용한 검증 실행

앞서 살펴봤던 Validator 이용 코드를 보면 다음과 같이 Validator 객체의 validate() 메서드를 직접 호출했었다.

```java
@RestController
public class RegistrationController {

    @PostMapping("/register")
    public Object register(@RequestBody MemberRegisterRequest registerRequest, BindingResult bindingResult) {

        // Custom Validator 객체를 생성후 validate() 메서드를 직접 호출하고 있다.
        new MemberRegisterValidator().validate(registerRequest, bindingResult);

        if (bindingResult.hasErrors()) {
            List<FieldError> fieldErrors = bindingResult.getFieldErrors();
            return fieldErrors;
        }

        return "Success";
    }
}
```



JSR-303 표준에 정의된 `@Valid` 애너테이션을 이용하면 커맨드 객체를 검사하는 코드를 직접 호출하지 않고

스프링 프레임워크가 호출하도록 설정할 수 있다.

JSR-303이란 Bean Validation API로서 이 표준에 정의된 `@Valid` 애너테이션을 사용해서 

커맨드 객체의 유효성을 검증한다는 것을 표시한다.





스프링 MVC는 JSR-303의 `@Valid` 애너테이션과 스프링 프레임워크의 `@InitBinder` 애너테이션을 사용해서

스프링 프레임워크가 유효성 검사 코드를 실행하도록 할 수 있다.





아래의 예시 코드는 `@Valid` 애너테이션과 `@InitBinder` 애너테이션을 사용한 코드의 작성 예시이다.

```java
@RestController
public class RegistrationController {

    @PostMapping("/register")
    public Object register(@Valid @RequestBody MemberRegisterRequest registerRequest, BindingResult bindingResult) {

        /*
        // 유효성 검사
        new MemberRegisterValidator().validate(registerRequest, bindingResult);
        */

        // errors 값 체크
        if (bindingResult.hasErrors()) {
            List<FieldError> fieldErrors = bindingResult.getFieldErrors();
            return fieldErrors;
        }

        return "Success";
    }


    @InitBinder
    protected void initBinder(WebDataBinder binder) {
        binder.setValidator(new MemberRegisterValidator());
    }

}
```

직접 validate 메서드를 호출하는 부분을 주석 처리하고 initBinder를 정의한 뒤,

커맨드 객체 `MemberRegisterRequest` 앞에 `@Valid` 어노테이션을 붙여줬다.

요청을 보내보면 똑같이 잘 동작한다.



어떤 Validator가 커맨드 객체를 검증할지의 여부는 `initBinder()` 메서드를 통해서 결정된다.

스프링은 `@InitBinder` 애너테이션이 적용된 메서드를 이용해서 

폼과 커맨드 객체 사이의 매핑을 처리해주는 `WebDataBinder`를 초기화할 수 있도록 하고 있다.

이 메서드에서 `WebDataBinder.setValidator()`를 통해 커맨드 객체의 유효성 여부를 검사할 때 사용할 Validator를 설정하게 된다.

스프링 MVC는 컨트롤러의 `register` 메서드를 실행하기 전에 `@Valid` 애너테이션이 붙은 커맨드 객체를 검증한다.







## 5. 글로벌 Validator와 컨트롤러 Validator

글로벌 Validator를 사용하면 한 개의 Validator를 이용해서 모든 커맨드 객체를 검증할 수 있다.

글로벌 Validator를 적용하는 방법은 간단하다.

다음과 같이 \<mvc:annotation-driven> 태그의 validator 속성에 글로벌 Validator로 사용할 빈을 등록해주면 된다.

```xml
<mvc:annotation-driven validator="validator" />
<bean id="validator" class="custom.CommonValidator" />
```



위 코드는 CommonValidator를 글로벌 Validator로 등록했는데, 

이 경우 CommonValidator를 이용해서 `@Valid` 애너테이션이 붙은 커맨드 객체를 검사하게 된다.

글로벌 Validator가 커맨드 객체에 대한 검증을 지원하지 않으면(즉, 글로벌 Validator의 supports() 메서드가 false를 리턴하면),

글로벌 Validator는 커맨드 객체를 검증하지 않는다.



글로벌 Validator를 사용하지 않고 다른 Validator를 사용하려면 

`@InitBinder`가 적용된 메서드에서 `WebDataBinder`의 `setValidator()` 메서드를 이용하면 된다.

예를 들어, 아래 코드는 글로벌 Validator 대신 LoginCommandValidator를 사용해서 LoginCommand 객체를 검증한다.

```java
@PostMapping
public String login(@Valid LoginCommand loginCommand, Errors errors) {
    if(errors.hasErrors()) {
        return LOGIN_FORM;
    }
   	...
}

@InitBinder
protected void initBinder(WebDataBinder binder) {
    // 글로벌 Validator가 아닌 다른 Validator를 대신 사용
    binder.setValidator(new LoginCommandValidator());
}
```



또는, 글로벌 Validator를 사용하면서 컨트롤러를 위한 Validator를 추가로 사용하고 싶다면

`setValidator()` 대신 `addValidator()` 메서드를 사용해서 Validator를 등록해주면 된다.

```java
@InitBinder
protected void initBinder(WebDataBinder binder) {
	// 글로벌 Validator와 함께 사용
	binder.addValidator(new LoginCommandValidator());
}
```



`@EnableWebMvc` 애너테이션을 사용했다면, 

다음과 같이 `WebMvcConfigurerAdapter`를 상속받은 `@Configuration` 클래스에서

글로벌 Validator 객체를 생성하도록 `getValidator()` 메서드를 재정의한다.

```java
@Configuration
@EnableWebMvc
public class SampleConfig extends WebMvcConfigurerAdapter {

	@Override
	public Validator getValidator() {
		return new CommonValidator();
	}
}
```



\<mvc:annotation-driven> 태그의 validator 속성을 지정하지 않거나

`getValidator()` 메서드를 재정의하지 않으면 JSR-303 애너테이션을 지원하는 Validator를 글로벌 Validator로 등록한다.

JSR-303 지원 Validator는 JSR-303에 정의된 애너테이션을 이용해서 커맨드 객체를 검증하는 기능을 제공하는데, 

이에 대한 내용을 이어서 살펴보도록 하자.



> 이 장에서 글로벌 Validator를 언급한 이유는 뒤에서 살펴볼 JSR-303 지원 Validator를 설명하려면
>
> 글로벌 Validator에 대한 기초 지식이 필요하기 때문이다. 실제로 글로벌 Validator를 구현해서 등록하는 경우는 많지 않다.





## 6. `@Valid` 애너테이션 및 JSR-303 애너테이션을 이용한 값 검증 처리

`@Valid` 애너테이션은 Bean Validation API(JSR-303)에 정의되어 있는데,

이 API에는 `@Valid` 애너테이션 뿐만 아니라 `@NotNull`, `@Digits`, `@Size` 등의 애너테이션을 제공하고 있다.

이들 애너테이션을 사용하면 Validator 클래스 작성 없이 애너테이션만으로 커맨드 객체의 값 검증을 처리할 수 있다.



JSR-303이 제공하는 애너테이션을 이용해서 커맨드 객체의 값을 검증하는 방법은 다음과 같다.

- 커맨드 객체에 `@NotNull`, `@Digits` 등의 애너테이션을 이용해서 검증 규칙을 설정한다.
- `LocalValidatorFactoryBean` 클래스를 이용해서 JSR-303 프로바이더를 스프링의 Validator로 등록한다.
- 컨트롤러가 두 번째에서 생성한 빈을 Validator로 사용하도록 설정한다.



가장 먼저 할 작업은 JSR-303 API와 JSR-303 프로바이더를 설치하는 것이다.

예를 들어, Hibernate Validator를 프로바이더 구현체로 사용하고 싶다면, 

Hibernate Validator 관련 jar 파일을 클래스패스에 등록해주면 된다.



**예시**

```xml
<dependency>
	<groupId>javax.validation</groupId>
    <artifactId>validation-api</artifactId>
    <version>1.0.0.GA</version>
</dependency>
<dependency>
	<groupId>org.hibernate</groupId>
    <artifactId>hibernate-validator</artifactId>
    <version>4.3.1.Final</version>
</dependency>
```



스프링 JSR-303 프로바이더 구현체로 Hibernate Validator를 사용할 경우, 

`@Range`나 `@NotEmpty` 같은 추가적인 애너테이션을 사용할 수 있다.



커맨드 클래스는 다음과 같이 JSR-303이 제공하는 애너테이션을 이용해서 값 검증 규칙을 설정할 수 있다.

```java
public class MemberModRequest {

    @NotEmpty
    private String id;
    
    @NotEmpty
    private String name;
    
    @NotEmpty
    @Email
    private String email;
    
    private boolean allownNoti;
    
    @NotEmpty
    private String currentPassword;
    
    @Valid
    private Address address;
}
```



`@NotNull`, `@Size` 외에 JSR-303의 주요 애너테이션은 다음 절에서 살펴본다.

위 코드에서 눈여겨볼 점은 중첩된 객체를 검사하기 위해서 `address` 필드에 `@Valid` 애너테이션을 사용했다는 것이다.



JSR-303 애너테이션을 적용했으면, 그 다음으로 할 작업은 커맨드 객체의 값을 검증할 때 

JSR-303 프로바이더를 사용하도록 설정하는 것이다.

JSR-303 프로바이더를 Validator로 사용하려면 `LocalValidatorFactoryBean` 클래스를 빈으로 등록하고, 이 빈을 Validator로 사용하면 된다.



\<mvc:annotation-driver> 태그를 사용하면 기본으로 `LocalValidatorFacotryBean`이 생성한 JSR-303 Validator를 글로벌 Validator로 등록해준다.

따라서 \<mvc:annotation-driven> 태그를 설정하면 JSR-303 애너테이션을 사용하는 커맨드 객체에 대한 값 검증을 할 수 있다.



남은 작업은 JSR-303 애너테이션을 사용하는 커맨드 객체 앞에 `@Valid` 애너테이션을 붙여서,

JSR-303 지원 Validator가 값을 검증하도록 만들면 된다.







### JSR-303의 주요 애너테이션

JSR-303에서 제공하는 값 규칙 설정과 관련된 주요 애너테이션은 아래의 표와 같다.

모든 애너테이션은 `javax.validation.constraints` 패키지에 정의되어 있다.

| 애너테이션                   | 주요 속성 (괄호는 기본 값)                                   | 설명                                                         |
| ---------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| @NotNull                     |                                                              | 값이 null이면 안된다.                                        |
| @Size                        | min: 최소 크기, int (0)<br />max: 최대 크기, int (MAX_INTEGER) | 값의 크기가 min부터 max 사이에 있는지 검사한다. <br />String인 경우 문자열의 길이를 검사한다.<br />Collection인 경우 구성 요소의 개수를 검사한다.<br />배열인 경우 배열의 길이를 검사한다.<br />값이 null인 경우 유효한 것으로 판단한다. |
| @Min<br />@Max               | value: 최소 또는 최대 값, long                               | 숫자의 값이 지정한 값 이상 또는 이하여야 한다.<br />BigDecimal, BigInteger, 정수 타입(int, long 등) 및 Wrapper 타입에 적용된다.<br />double과 float는 지원하지 않는다.<br />값이 null인 경우 유효한 것으로 판단한다. |
| @DecimalMin<br />@DecimalMax | value: 최소 또는 최대 값, String (BigDecimal 형식으로 값 표현) | 숫자의 값이 지정한 값 이상 또는 이하여야 한다.<br />BigDecimal, BigInteger, 정수 타입(int, long 등) 및 Wrapper 타입에 적용된다.<br />double과 float는 지원하지 않는다.<br />값이 null인 경우 유효한 것으로 판단한다. |
| @Digits                      | integer: 정수부분 숫자 길이, int<br />fraction: 소수부분 숫자 길이, int | 숫자의 정수 부분과 소수점 부분의 길이가 범위에 있는지 검사한다. |
| @Pattern                     | regexp: 정규표현식, String                                   | 문자열이 지정한 패턴에 일치하는지 검사한다. <br />값이 null인 경우 유효한 것으로 판단한다. |
