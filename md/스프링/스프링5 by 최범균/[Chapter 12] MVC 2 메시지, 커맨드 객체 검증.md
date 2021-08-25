# [Chapter 12] MVC 2: 메시지, 커맨드 객체 검증





## 1. 메시지

메시지 파일을 작성해보자.

메시지 파일은 자바의 프로퍼티 파일 형식으로 작성한다.

메시지 파일을 보관하기 위해 `src/main/resources`에 `message` 폴더를 생성하고 이 폴더에 `label.properties` 파일을 생성하자.



**label.properties**

```properties
member.register=회원가입

term=약관
term.agree=약관동의
next.btn=다음단계

member.info=회원정보
email=이메일
name=이름
password=비밀번호
password.confirm=비밀번호 확인
register.btn=가입 완료

register.done=<string>{0}님</strong>, 회원 가입을 완료했습니다.

go.main=메인으로 이동
```



다음으로 `MessageSource` 타입의 빈을 추가하자.

스프링 설정 중 한 곳에 추가하면 된다.

예제에서는 `MvcConfig` 설정 클래스에 추가해보겠다.

```java
@Configuration
@EnableWebMvc
public class MvcConfig implements WebMvcConfigurer {

    ...
        
    @Bean
    public MessageSource messageSource() {
        ResourceBundleMessageSource ms = new ResourceBundleMessageSource();
        ms.setBasenames("message.label");
        ms.setDefaultEncoding("UTF-8");
        return ms;
    }
}
```

basenames 프로퍼티 값으로 "message.label"을 주었다.

이는 message 패키지에 속한 label 프로퍼티 파일로부터 메시지를 읽어온다고 설정한 것이다.

`src/main/resources` 폴더도 classpath에 포함되고 message 폴더는 message 패키지에 대응한다.

따라서 이 설정은 앞서 작성한 label.properties 파일로부터 메시지를 읽어온다.





setBasenames() 메서드는 가변 인자이므로 아래와 같이 사용할 메시지 프로퍼티 목록을 전달할 수 있다.

```java
@Bean
public MessageSource messageSource() {
	ResourceBundleMessageSource ms = new ResourceBundleMessageSource();
    ms.setBasenames("message.label", "message.error");
    ms.setDefaultEncoding("UTF-8");
    return ms;
}
```



위 코드에서 주의할 점은 **<u>빈의 아이디를 "messageSource"로 지정해야 한다</u>**는 것이다.

**<u>다른 이름을 사용할 경우 정상적으로 동작하지 않는다.</u>**





### 메시지 처리를 위한 `MessageSource`와 \<spring:message> 태그

스프링은 Locale(지역)에 상관없이 일관된 방법으로 문자열(메시지)을 관리할 수 있는 `Messagesource` 인터페이스를 정의하고 있다.

`MessageSource` 인터페이스는 다음과 같이 정의되어 있다.

```java
public interface MessageSource {
    
    String getMessage(String code, Object[] args, String defaultMessage, Locale locale);
    
    String getMessage(String code, Object[] args, Locale locale) throws NoSuchMessageException;
    
    ...
}
```

특정 Locale에 해당하는 메시지가 필요한 코드는 `MessageSource`의 `getMessage()` 메서드를 이용해서 필요한 메시지를 가져와서 사용하는 식이다.

`getMessage()` 메서드의 `code` 파라미터는 메시지를 구분하기 위한 코드이고 `locale` 파라미터는 지역을 구분하기 위한 `Locale`이다.

같은 코드라 하더라도 지역에 따라 다른 메시지를 제공할 수 있도록 `MessageSource`를 설계했다.



`MessageSource`의 구현체로는 자바의 프로퍼티 파일로부터 메시지를 읽어오는 `ResourceBundleMessageSource` 클래스를 사용한다.

이 클래스는 메시지 코드와 일치하는 이름을 가진 프로퍼티의 값을 메시지로 제공한다.



`ResourceBundleMessageSource` 클래스는 자바의 ResourceBundle을 사용하기 때문에 해당 프로퍼티 파일이 classpath에 위치해야 한다.

앞서 예제에서도 classpath에 포함되는 `src/main/resources`에 프로퍼티 파일을 위치시켰다.

보통 관리 편의성을 위해 프로퍼티 파일을 한 곳에 모은다. 예제에서는 message라는 패키지에 프로퍼티 파일을 위치시켰다.



\<spring:message> 태그는 스프링 설정에 등록된 'messageSource' 빈을 이용해서 메시지를 구한다.

즉, \<spring:message> 태그를 실행하면 내부적으로 `MessageSource`의 `getMessage()` 메서드를 실행해서 필요한 메시지를 구한다.





## 2. 커맨드 객체 값 검증과 에러 메시지 처리



### 커맨드 객체 검증과 에러 코드 지정하기

스프링 MVC에서 커맨드 객체의 값이 올바른지 검사하려면 다음의 두 인터페이스를 사용한다.

- `org.springframework.validation.Validator`
- `org.springframework.validation.Errors`



객체를 검증할 때 사용하는 `Validator` 인터페이스는 다음과 같다.

```java
public interface Validator {
	boolean supports(Class<?> aClass);
	void validate(Object target, Errors errors);
}
```

위 코드에서 `supports()` 메서드는 `Validator`가 검증할 수 있는 타입인지 검사한다.

`validate()` 메서드는 첫 번째 파라미터로 전달받은 객체를 검증하고 오류 결과를 `Errors`에 담는 기능을 정의한다.



일단 `Validator` 인터페이스를 구현한 클래스를 만들어보고, 주요 코드를 보면서 구현 방법을 살펴보자.

```java
public class RegisterRequestValidator implements Validator {
    
    private static final String emailRegExp = "regexp값...(생략함)";
    private Pattern pattern;
    
    public RegisterRequestValidator() {
        pattern = Pattern.compile(emailRegExp);
    }
    
    
    @Override
    public boolean supports(Class<?> aClass) {
        return RegisterRequest.class.isAssignableFrom(aClass);
    }
    
    @Override
    public void validate(Object target, Errors errors) {
        RegisterRequest regReq = (RegisterRequest) target;
        
        if(regReq.getEmail() == null || regReq.getEmail().trim().isEmpty()) {
            errors.rejectValue("email", "required");
        } else {
            Matcher matcher = pattern.matcher(regReq.getEmail());
            if(!matcher.matches()) {
                errors.rejectValue("email", "bad");
            }
        }
        
        ValidationUtils.rejectIfEmptyOrWhitespace(errors, "name", "required");
        ValidationUtils.rejectIfEmpty(errors, "password", "required");
        ValidationUtils.rejectIfEmpty(errors, "confirmPassword", "required");
        
        if(!regReq.getPassword().isEmpty() && !regReq.isPasswordEqualToConfirmPassword()) {
        	errors.rejectValue("confirmPassowrd", "nomatch");
        }
    }
}
```

- `supports()` 메서드는 파라미터로 전달받은 `aClass` 객체가 `RegisterRequest` 클래스로 타입 변환이 가능한지 확인한다.
- `validate()` 메서드는 두 개의 파라미터를 갖는다.
  - `target` 파라미터는 검사 대상 객체이고,
  - `errors` 파라미터는 검사 결과 에러 코드를 설정하기 위한 객체이다.
- `validate()` 메서드는 보통 다음과 같이 구현한다.
  - 검사 대상 객체의 특정 프로퍼티나 상태가 올바른지 검사
  - 올바르지 않다면 `Errors`의 `rejectValue()` 메서드를 이용해서 에러 코드 저장

- `Errors`의 `rejectValue()` 메서드는 

  첫 번째 파라미터로 프로퍼티의 이름을 전달받고,

  두 번째 파라미터로 에러 코드를 전달받는다.

  - 여기서 지정한 에러 코드를 이용해서 에러 메시지를 출력할 수 있다.

- `ValidationUtils` 클래스는 객체의 값 검증을 간결하게 해준다.

  - `ValidationUtils.rejectIfEmptyOrWhitespace(errors, "name", "required");`  

    위의 코드는 검사 대상 객체의 "name" 프로퍼티가 null이거나 공백문자로만 되어 있는 경우 "name" 프로퍼티의 에러코드로 "required"를 추가한다.

    

  - ```java
    String name = regReq.getName();
    if(name == null || name.trim().isEmpty()) {
    	errors.rejectValue("name", "required");
    }
    ```

    즉, 위의 코드와 정확히 같은 코드다.







`ValidationUtils` 클래스는 검사 대상 객체인 `target`을 파라미터로 전달하지 않았는데 어떻게 `target` 객체의 "name" 프로퍼티의 값을 검사할까?



비밀은 `Errors` 객체에 있다.

스프링 MVC에서 `Validator`를 사용하는 코드는 요청 매핑 애너테이션 적용 메서드에 `Errors` 타입 파라미터를 전달받고,

이 `Errors` 객체를 `Validator`의 `validate()` 메서드의 두 번째 파라미터로 전달한다.

```java
@Controller
public class RegisterController {
	private MemberRegisterService memberRegisterService;
    
    ...
    
    @PostMapping("/register/step3")
    public String handle(RegisterRequest regReq, Errors errors) {
        new RegisterRequestValidator().validate(regReq, errors);
        
        if(errors.hasErrors()) return "register/step2";
        
        try {
            memberRegisterService.register(regReq);
            return "register/step3";
        } catch (DuplicateMemberException ex) {
            errors.rejectValue("email", "duplicate");
            return "register/step2";
        }
    }
}
```



요청 매핑 애너테이션 적용 메서드의 커맨드 객체 파라미터 뒤에 `Errors ` 타입 파라미터가 위치하면,

스프링 MVC는 `handle` 메서드를 호출할 때 커맨드 객체와 연결된 `Errors` 객체를 생성해서 파라미터로 전달한다.

이 `Errors` 객체는 커맨드 객체의 특정 프로퍼티 값을 구할 수 있는 `getFieldValue()` 메서드를 제공한다.

따라서 `ValidationUtils.rejectIfEmptyOrWhitespace()` 메서드는 커맨드 객체를 전달받지 않아도 `Errors` 객체를 이용해서 지정한 값을 구할 수 있다.

```java
// errors 객체의 getFieldValue("name") 메서드를 실행해서 커맨드 객체의 name 프로퍼티를 구함.
// 따라서 커맨드 객체를 직접 전달하지 않아도 값 검증을 할 수 있음
ValidationUtils.rejectIfEmptyOrWhitespace(erros, "name", "required");
```



위의 코드를 보면 앞에서 작성한 `RegisterRequestValidator` 객체를 생성하고 `validate()` 메서드를 실행한다.

이를 통해 `RegisterRequest` 커맨드 객체의 값이 올바른지 검사하고 그 결과를 `Errors` 객체에 담는다.







커맨드 객체의 특정 프로퍼티가 아닌 **커맨드 객체 자체**가 잘못되었을 수도 있다.

이런 경우에는 `rejectValue()` 메서드 대신에 `reject()` 메서드를 사용한다.

예를 들어 로그인 아이디와 비밀번호를 잘못 입력한 경우 아이디와 비밀번호가 불일치한다는 메시지를 보여줘야 한다.

이 경우 특정 프로퍼티에 에러를 추가하기 보다는 커맨드 객체 자체에 에러를 추가해야 하는데, 이때 `reject()` 메서드를 사용한다.

```java
try {
    ... // 인증 처리 코드
} catch (WrongIdPasswordException ex) {
    // 특정 프로퍼티가 아닌 커맨드 객체 자체에 에러 코드 추가
    errors.reject("notMatchingIdPassword");
    return "login/loginForm";
}
```

`reject()` 메서드는 개별 프로퍼티가 아닌 객체 자체에 에러 코드를 추가하므로 이 에러를 **글로벌 에러**라고 부른다.





요청 매핑 애너테이션을 붙인 메서드에 `Errors` 타입의 파라미터를 추가할 때 주의할 점은 

`Errors` 타입 파라미터는 반드시 커맨드 객체를 위한 파라미터 다음에 위치해야 한다는 점이다.

그렇지 않으면 Exception이 발생한다.







### `Errors`와 `ValidationUtils` 클래스의 주요 메서드

`Errors` 인터페이스가 제공하는 에러 코드 추가 메서드는 다음과 같다.

- `reject(String errorCode)`
- `reject(String errorCode, String defaultMessage)`
- `reject(String errorCode, Object[] errorArgs, String defaultMessage)`
- `rejectValue(String field, String errorCode)`
- `rejectValue(String field, String errorCode, String defaultMessage)`
- `rejectValue(String field, String errorCode, Object[] errorArgs, String defaultMessage)`



에러 코드에 해당하는 메시지가 {0}이나 {1} 같이 인덱스 기반 변수를 포함하고 있는 경우 

`Object` 배열 타입의 `errorArgs` 파라미터를 이용해서 변수에 삽입될 값을 전달한다.

`defaultMessage` 파라미터를 가진 메서드를 사용하면, 에러 코드에 해당하는 메시지가 존재하지 않을 때 Exception을 발생시키는 대신 `defaultMessage`를 출력한다.





`ValidationUtils` 클래스는 다음의 `rejectIfEmpty()` 메서드와 `rejectIfEmptyOrWhitespace()` 메서드를 제공한다.

- `rejectIfEmpty(Errors errors, String field, String errorCode)`
- `rejectIfEmpty(Errors errors, String field, String errorCode, Object[] errorArgs)`
- `rejectIfEmptyOrWhitespace(Errors errors, String field, String errorCode)`
- `rejectIfEmtpyOrWhitespace(Errors errors, String field, String errorCode, Object[] errorArgs)`



`rejectIfEmpty()` 메서드는 field에 해당하는 프로퍼티 값이 null이거나 빈 문자열("")인 경우 에러 코드로 `errorCode`를 추가한다.

`rejectIfEmptyOrWhitespace()` 메서드는 null이거나 빈 문자열인 경우 또는 공백 문자로만 구성된 경우 에러 코드로 `errorCode`를 추가한다.



에러 코드에 해당하는 메시지가 {0}이나 {1}과 같이 인덱스 기반 placeholder를 포함하고 있으면 `errorArgs`를 이용해서 메시지의 placeholder에 삽입할 값을 전달할 수 있다.







## 3. 글로벌 범위 `Validator`와 컨트롤러 범위 `Validator` 

스프링 MVC는 모든 컨트롤러에 적용할 수 있는 글로벌 `Validator`와 단일 컨트롤러에 적용할 수 있는 `Validator`를 설정하는 방법을 제공한다. 

이를 사용하면 `@Valid` 애너테이션을 사용해서 커맨드 객체에 검증 기능을 적용할 수 있다.





### 3.1 글로벌 범위 `Validator` 설정과 `@Valid` 애너테이션

글로벌 범위 `Validator`는 모든 컨트롤러에 적용할 수 있는 `Validator`다.

글로벌 범위 `Validator`를 적용하려면 다음 두 가지를 설정하면 된다.

- 설정 클래스에서 `WebMvcConfigurer`의 `getValidator()` 메서드가 `Validator` 구현 객체를 리턴하도록 구현
- 글로벌 범위 `Validator`가 검증할 커맨드 객체에 `@Valid` 애너테이션 적용



먼저 글로벌 범위 `Validator`를 설정하자.

이를 위해 해야 할 작업은 `WebMvcConfigurer` 인터페이스에 정의된 `getValidator()` 메서드를 구현하는 것이다.

```java
@Configuration
@EnableWebMvc
public class MvcConfig implements WebMvcConfigurer {

	@Override
	public Validator getValidator() {
		return new RegisterRequestValidator();
	}
}
```

스프링 MVC는 `WebMvcConfigurer` 인터페이스의 `getValidator()` 메서드가 리턴한 객체를 글로벌 범위 `Validator`로 사용한다.

글로벌 범위 `Validator`를 지정하면 `@Valid` 애너테이션을 사용해서 `Validator`를 적용할 수 있다.



`RegisterRequestValidator`는 `RegisterRequest` 타입에 대한 검증을 지원한다.

`RegisterController` 클래스의 `handle()` 메서드가 `RegisterReuqest` 타입 커맨드 객체를 사용하므로 

아래와 같이 이 메서드의 파라미터에 `@Valid` 애너테이션을 붙여서 글로벌 범위 `Validator`를 적용할 수 있다.

```java
@Controller
public class RegisterController {
    
    ...
        
    @PostMapping("/register/step3")
    public String handle(@Valid RegisterRequest regReq, Errors errors) {
        if(errors.hasErrors()) return "register/step2";
        
        try {
            memberRegisterService.register(regReq);
            return "register/step3"; 
        } catch (DuplicateMemberException ex) {
            errors.rejectValue("email", "duplicate");
            return "register/step2";
        }
    }
}
```

커맨드 객체에 해당하는 파라미터에 `@Valid` 애너테이션을 붙이면 글로벌 범위 `Validator`가 해당 타입을 검증할 수 있는지 확인한다.

검증 가능하면 실제 검증을 수행하고 그 결과를 `Errors`에 저장한다. 이는 요청 처리 메서드 실행 전에 적용된다.



>**글로벌 Validator의 범용성**
>
>`RegisterRequestValidator` 클래스는 `RegisterRequest` 타입의 객체만 검증할 수 있으므로 
>
>모든 컨트롤러에 적용할 수 있는 글로벌 범위 `Validator`로 적합하지 않다.
>
>스프링 MVC는 자체적으로 제공하는 글로벌 `Validator`가 존재하는데 이 `Validator`를 사용하면 
>
>Bean Validation이 제공하는 애너테이션을 이용해서 값을 검증할 수 있다.





### 3.2 `@InitBinder` 애너테이션을 이용한 컨트롤러 범위 `Validator`

`@InitBinder` 애너테이션을 이용하면 컨트롤러 범위 `Validator`를 설정할 수 있다.

아래의 예시는 `@InitBinder` 애너테이션을 사용해서 컨트롤러 범위 `Validator`를 설정한 예이다.

```java
@Controller
public class RegisterController {
    
    ...
    @PostMapping("/register/step3")
    public String handler(@Valid RegisterRequest regReq, Errors errors) {
        if(errors.hasErrors()) return "register/step2";
        
        try {
            memberRegisterService.register(regReq);
            return "register/step3";
        } catch (DuplicateMemberException ex) {
            errors.rejectValue("email", "duplicate");
            return "register/step2";
        }
    }
    
    @InitBinder
    protected void initBinder(WebDataBinder binder) {
        binder.setValidator(new RegisterRequestValidator());
    }
}
```

위의 코드는 커맨드 객체 파라미터에 `@Valid` 애너테이션을 적용하고 있다.

글로벌 범위 `Validator`를 사용할 때와 마찬가지로 `handle()` 메서드에는 `Validator` 객체의 `validate()` 메서드를 호출하는 코드가 없다.



어떤 `Validator`가 커맨드 객체를 검증할 것인지는 `initBinder()` 메서드가 결정한다.

`@InitBinder` 애너테이션을 적용한 메서드는 `WebDataBinder` 타입 파라미터를 갖는데 

`WebDataBinder#setValidator()` 메서드를 이용해서 컨트롤러 범위에 적용할 `Validator`를 설정할 수 있다.



위의 코드는 `RegisterRequest` 타입을 지원하는 `RegisterRequestValidator`를 컨트롤러 범위 `Validator`로 설정했으므로

`@Valid` 애너테이션을 붙인 `RegisterRequest`를 검증할 때 이 `Validator`를 사용한다.



참고로 `@InitBinder`가 붙은 메서드는 컨트롤러의 요청 처리 메서드를 실행하기 전에 매번 실행된다.

예를 들어 `RegisterController`의 요청 처리 메서드인 `handleStep1()`, `handleStep2()`, `handleStep3()`을 실행하기 전에 `initBinder()` 메서드를 매번 호출해서 `WebDataBinder`를 초기화한다.





### 글로벌 범위 `Validator`와 컨트롤러 범위 `Validator`의 우선 순위

`@InitBinder` 애너테이션을 붙인 메서드에 전달되는 `WebDataBinder`는 내부적으로 `Validator` 목록을 갖는다.

이 목록에는 글로벌 범위 `Validator`가 기본으로 포함된다.



`WebDataBinder#setValidator(Validator validator)` 메서드를 실행하는데 

이 메서드는 `WebDataBinder`가 갖고 있는 `Validator`를 목록에서 삭제하고 파라미터로 전달받은 `Validator`를 목록에 추가한다.

즉, `setValidator()` 메서드를 사용하면 글로벌 범위 `Validator` 대신에 컨트롤러 범위 `Validator`를 사용하게 된다.



이와는 다르게 `WebDataBinder#addValidator(Validator... validators)` 메서드는 기존 `Validator` 목록에 새로운 `Validator`를 추가한다.

글로벌 범위 `Validator`가 존재하는 상태에서 `addValidator()` 메서드를 실행하면 

순서상 글로벌 범위 `Validator` 뒤에 새로 추가한 컨트롤러 범위 `Validator`가 추가된다.

이 경우 글로벌 범위 `Validator`를 먼저 적용한 뒤에 컨트롤러 범위 `Validator`를 적용한다.







## 4. Bean Validation을 이용한 값 검증 처리

`@Valid` 애너테이션은 Bean Validation 스펙에 정의되어 있다.

이 스펙은 `@Valid` 뿐만 아니라 `@NotNull`, `@Digits`, `@Size` 등의 애너테이션을 정의하고 있다.

이 애너테이션을 사용하면 `Validator` 작성 없이 애너테이션만으로 커맨드 객체의 값 검증을 할 수 있다.



> **[참고]**
>
> Bean Validation 2.0 버전을 **JSR 380**이라고도 부른다.
>
> 여기서 JSR은 Java Specification Request의 약자로 자바 스펙을 기술한 문서를 의미한다.
>
> 각 스펙마다 고유한 JSR 번호를 갖는다. 
>
> 예를 들어, Bean Validation 1.0 버전은 JSR 303, 1.1 버전은 JSR 349이다.



Bean Validation이 제공하는 애너테이션을 이용하여 커맨드 객체의 값을 검증하는 방법은 다음과 같다.

- Bean Validation과 관련된 의존을 설정에 추가한다.
  - `javax.validation`의 `validation-api` -> API를 정의한 모듈
  - `org.hibernate`의 `hibernate-validator` -> API를 구현한 프로바이더
- 커맨드 객체에 `@NotNull`, `@Digits` 등의 애너테이션을 이용해서 검증 규칙을 설정한다.



커맨드 객체는 다음과 같이 Bean Validation과 프로바이더가 제공하는 애너테이션을 이용해서 값 검증 규칙을 설정할 수 있다.

```java
public class RegisterRequest {
	@NotBlank
    @Email
    private String email;
    
    @Size(min = 6)
    private String password;
    
    @NotEmpty
    private Strign confirmPassword;
    
    @NotEmpty
    private String name;
}
```



Bean Validation 애너테이션을 사용했다면 그 다음으로 할 작업은 

Bean Validation 애너테이션을 적용한 커맨드 객체를 검증할 수 있는 `OptionalValidatorFactoryBean` 클래스를 빈으로 등록하는 것이다.

참고로, `@EnableWebMvc` 애너테이션을 사용하면 `OptionalValidatorFactoryBean`을 글로벌 범위 `Validator`로 등록해준다.



남은 작업은 `@Valid` 애너테이션을 붙여서 글로벌 범위 `Validator`로 검증하는 것이다.

```java
@PostMapping("/register/step3")
public String handle(@Valid RegisterRequest regReq, Errors errors) {
    if(errors.hasErrors()) return "register/step2";
    
    ...
}
```



만약 글로벌 범위 `Validator`를 따로 설정했다면 해당 설정을 삭제하자.

아래와 같이 글로벌 범위 `Validator`를 설정하면 `OptionalValidatorFactoryBean`을 글로벌 범위 `Validator`로 사용하지 않는다.

스프링 MVC는 별도로 설정한 글로벌 범위 `Validator`가 없을 때에 `OptionalValidatorFactoryBean`를 글로벌 범위 `Validator`로 사용한다.

```java
@Configuration
@EnableWebMvc
public class MvcConfig implements WebMvcConfigurer {
    
    // 글로벌 범위 Validator를 설정하면
    // OptionalValidatorFactoryBean를 사용하지 않는다.
    @Override
    public Validator getValidator() {
        return new RegisterRequestValidator();
    }
}
```





스프링 MVC는 에러 코드에 해당하는 메시지가 존재하지 않을 때 Bean Validation 프로바이더가 제공하는 기본 에러 메시지를 출력한다.

기본 에러 메시지 대신 원하는 에러 메시지를 사용하려면 다음 규칙을 따르는 메시지 코드를 메시지 프로퍼티 파일에 추가하면 된다.

- 애너테이션이름.커맨드객체모델명.프로퍼티명
- 애너테이션이름.프로퍼티명
- 애너테이션이름





