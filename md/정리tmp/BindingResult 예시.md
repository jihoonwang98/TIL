# BindingResult 예시

```java
@Test
public void 빈_request() {
    SignupRequest request = SignupRequest.builder()
            .username("")
            .password("")
            .name("")
            .tel("")
            .address(new Address("", ""))
            .build();

    BindingResult bindingResult = validate(request);

    bindingResult.reject("IDAndPWDNotMatch", "ID랑 패스워드 안 맞음");

    System.out.println("bindingResult.getAllErrors()");
    List<ObjectError> allErrors = bindingResult.getAllErrors();
    for (ObjectError error : allErrors) {
        System.out.println("error.getObjectName() = " + error.getObjectName());
        System.out.println("error.getCode() = " + error.getCode());
        System.out.println("error.getDefaultMessage() = " + error.getDefaultMessage());

        String[] codes = error.getCodes();
        System.out.println("error.getCodes() = ");
        for (String code : codes) {
            System.out.println(code);
        }
        System.out.println();
    }
    System.out.println("-----------------------------------------");
    System.out.println();


    System.out.println("bindingResult.getFieldError()");
    FieldError fieldError = bindingResult.getFieldError();
    System.out.println("fieldError = " + fieldError);
    System.out.println("-----------------------------------------");
    System.out.println();


    System.out.println("bindingResult.getFieldErrors()");
    List<FieldError> fieldErrors = bindingResult.getFieldErrors();
    for (FieldError error : fieldErrors) {
        System.out.println("error.getObjectName() = " + error.getObjectName());
        System.out.println("error.getCode() = " + error.getCode());
        System.out.println("error.getDefaultMessage() = " + error.getDefaultMessage());

        System.out.println("error.getField() = " + error.getField());
        System.out.println("error.getRejectedValue() = " + error.getRejectedValue());

        String[] codes = error.getCodes();
        System.out.println("error.getCodes() = ");
        for (String code : codes) {
            System.out.println(code);
        }
    }
    System.out.println("-----------------------------------------");
    System.out.println();


    System.out.println("bindingResult.getGlobalErrors()");
    List<ObjectError> globalErrors = bindingResult.getGlobalErrors();
    for (ObjectError error : globalErrors) {
        System.out.println("error.getObjectName() = " + error.getObjectName());
        System.out.println("error.getCode() = " + error.getCode());
        System.out.println("error.getDefaultMessage() = " + error.getDefaultMessage());

        String[] codes = error.getCodes();
        System.out.println("error.getCodes() = ");
        for (String code : codes) {
            System.out.println(code);
        }
    }
    System.out.println("-----------------------------------------");
    System.out.println();

}

private BindingResult validate(SignupRequest request) {
    BeanPropertyBindingResult bindingResult = new BeanPropertyBindingResult(request, "signupRequest");
    validator.validate(request, bindingResult);
    return bindingResult;
}
```



```
bindingResult.getAllErrors()
error.getObjectName() = signupRequest
error.getCode() = Size
error.getDefaultMessage() = ID는 6자 이상 15자 이하여야 합니다. 들어온 ID: 
error.getCodes() = 
Size.signupRequest.username
Size.username
Size.java.lang.String
Size

error.getObjectName() = signupRequest
error.getCode() = Pattern
error.getDefaultMessage() = "^\d{3}\d{3,4}\d{4}$"와 일치해야 합니다
error.getCodes() = 
Pattern.signupRequest.tel
Pattern.tel
Pattern.java.lang.String
Pattern

error.getObjectName() = signupRequest
error.getCode() = Pattern
error.getDefaultMessage() = 비밀번호는 최소 8자, 최소 하나의 문자, 하나의 숫자 및 하나의 특수 문자로 이루어져야 합니다.
error.getCodes() = 
Pattern.signupRequest.password
Pattern.password
Pattern.java.lang.String
Pattern

error.getObjectName() = signupRequest
error.getCode() = Size
error.getDefaultMessage() = 이름은 최소 2자, 최대 5자까지만 가능합니다.
error.getCodes() = 
Size.signupRequest.name
Size.name
Size.java.lang.String
Size

error.getObjectName() = signupRequest
error.getCode() = IDAndPWDNotMatch
error.getDefaultMessage() = ID랑 패스워드 안 맞음
error.getCodes() = 
IDAndPWDNotMatch.signupRequest
IDAndPWDNotMatch

-----------------------------------------

bindingResult.getFieldError()
fieldError = Field error in object 'signupRequest' on field 'username': rejected value []; codes [Size.signupRequest.username,Size.username,Size.java.lang.String,Size]; arguments [org.springframework.context.support.DefaultMessageSourceResolvable: codes [signupRequest.username,username]; arguments []; default message [username],15,6]; default message [ID는 6자 이상 15자 이하여야 합니다. 들어온 ID: ]
-----------------------------------------

bindingResult.getFieldErrors()
error.getObjectName() = signupRequest
error.getCode() = Size
error.getDefaultMessage() = ID는 6자 이상 15자 이하여야 합니다. 들어온 ID: 
error.getField() = username
error.getRejectedValue() = 
error.getCodes() = 
Size.signupRequest.username
Size.username
Size.java.lang.String
Size
error.getObjectName() = signupRequest
error.getCode() = Pattern
error.getDefaultMessage() = "^\d{3}\d{3,4}\d{4}$"와 일치해야 합니다
error.getField() = tel
error.getRejectedValue() = 
error.getCodes() = 
Pattern.signupRequest.tel
Pattern.tel
Pattern.java.lang.String
Pattern
error.getObjectName() = signupRequest
error.getCode() = Pattern
error.getDefaultMessage() = 비밀번호는 최소 8자, 최소 하나의 문자, 하나의 숫자 및 하나의 특수 문자로 이루어져야 합니다.
error.getField() = password
error.getRejectedValue() = 
error.getCodes() = 
Pattern.signupRequest.password
Pattern.password
Pattern.java.lang.String
Pattern
error.getObjectName() = signupRequest
error.getCode() = Size
error.getDefaultMessage() = 이름은 최소 2자, 최대 5자까지만 가능합니다.
error.getField() = name
error.getRejectedValue() = 
error.getCodes() = 
Size.signupRequest.name
Size.name
Size.java.lang.String
Size
-----------------------------------------

bindingResult.getGlobalErrors()
error.getObjectName() = signupRequest
error.getCode() = IDAndPWDNotMatch
error.getDefaultMessage() = ID랑 패스워드 안 맞음
error.getCodes() = 
IDAndPWDNotMatch.signupRequest
IDAndPWDNotMatch
-----------------------------------------
```

