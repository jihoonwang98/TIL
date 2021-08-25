# [Chapter 13] MVC 3: 세션, 인터셉터, 쿠키



## 1. 로그인 처리를 위한 코드 준비

이 장에서는 세션, 인터셉터, 쿠키에 관한 내용을 설명한다.

로그인 기능을 이용해서 이 내용을 설명할 것이므로 로그인과 관련된 몇 가지 필요한 코드를 만들어보자.

먼저 로그인 성공 후 인증 상태 정보를 세션에 보관할 때 사용할 `AuthInfo` 클래스를 다음과 같이 작성한다.

```java
@AllArgsConstructor
@Getter
public class AuthInfo {
    private Long id;
    private String email;
    private String name;
}
```



암호 일치 여부를 확인하기 위한 `matchPassword()` 메서드를 `Member` 클래스에 추가한다.

```java
public class Member {
	private Long id;
    private String email;
    private String password;
    private String name;
    private LocalDateTime registerDateTime;
    ... 생략
        
    public boolean matchPassword(String password) {
        return this.password.equals(password);
    }
}
```



이메일과 비밀번호가 일치하는지 확인해서 `AuthInfo` 객체를 생성하는 `AuthService` 클래스는 아래와 같이 작성한다.

```java
public class AuthService {

	private MemberDao memberDao;
    
    public void setMemberDao(MemberDao memberDao) {
        this.memberDao = memberDao;
    }
    
    public AuthInfo authenticate(String email, String password) {
        Member member = memberDao.selectByEmail(email);
        
        if(member == null) {
            throw new WrongIdPasswordException();
        }
        if(!member.matchPassword(password)) { 
            throw new WrongIdPasswordException();
        }
        
        return new AuthInfo(member.getId(), member.getEmail(), member.getName());
    }
}
```



이제 `AuthService`를 이용해서 로그인 요청을 처리하는 `LoginController` 클래스를 작성하자.

폼에 입력한 값을 전달받기 위한 `LoginCommand` 클래스와 폼에 입력된 값이 올바른지 검사하기 위한 `LoginCommandValidator` 클래스를 각각 아래와 같이 작성한다.

```java
@Getter @Setter
public class LoginCommand {
	private String email;
    private String password;
    private boolean rememberEmail;
}
```



```java
public class LoginCommandValidator implements Validator {
    
    @Override
    public boolean supports(Class<?> aClass) {
        return LoginCommand.class.isAssignableFrom(aClass);
    }
    
    @Override
    public void validate(Object target, Errors errors) {
        ValidationUtils.rejectIfEmptyOrWhitespace(errors, "email", "required");
        ValidationUtils.rejectIfEmptyOrWhitespace(errors, "password", "required");
    }
}
```



로그인 요청을 처리하는 `LoginController` 클래스는 아래와 같이 작성한다.

```java
@Controller
@RequestMapping("/login")
@RequiredArgsConstructor
public class LoginController {
    
    private final AuthService authService;
    
    @GetMapping
    public String form(LoginCommand loginCommand) {
        return "login/loginForm";
    }
    
    @PostMapping
    public String submit(LoginCommand loginCommand, Errors errors) {
        new LoginCommandValidator().validate(loginCommand, errors);
        if(errors.hasErrors()) return "login/loginForm";
        
        try {
            AuthInfo authInfo = 
                authService.authenticate(loginCommand.getEmail(), loginCommand.getPassword());
            
            // TODO 세션에 authInfo 저장해야 함.
            
            return "login/loginSuccess";
        } catch (WrongIdPasswordException ex) {
            errors.reject("idPasswordNotMatching");
            return "login/loginForm";
        }
    }
}
```



`LoginController` 클래스는 로그인 폼을 보여주기 위해 "login/loginForm" 뷰를 사용하고 

로그인 성공 결과를 보여주기 위해 "loginSuccess" 뷰를 사용한다.

두 뷰를 위한 JSP 코드를 각각 아래와 같이 작성한다.



**loginForm.jsp**

```jsp
...
<form:form modelAttribute="loginCommand">
<form:errors />
<p>
    <label><spring:message code="email"/>: <br/>
    	<form:input path="email" />
        <form:errors path="email" />
    </label>
</p>
    
<p>
    <label><spring:message code="password"/>: <br/>
    	<form:password path="password" />
        <form:errors path="password" />
    </label>
</p>
<input type="submit" value="<spring:message code="login.btn"/>">
</form:form>
...
```



**loginSuccess.jsp**

```jsp
...
<p> 
	<spring:message code="login.done" />
</p>

<p>
    <a href="<c:url value='/main'/>">
    	[<spring:message code="go.main"/>]
    </a>
</p>
```



뷰에서 사용할 메시지를 `label.properties` 파일에 추가한다.

```properties
...
login.title=로그인
login.btn=로그인하기
idPasswordNotMatching=아이디와 비밀번호가 일치하지 않습니다.
login.done=로그인에 성공했습니다.
```



이제 남은 작업은 컨트롤러와 서비스를 스프링 빈으로 등록하는 것이다.

등록하고 로그인해보면 잘 될 것이다.





## 2. 컨트롤러에서 `HttpSession` 사용하기

로그인 기능을 구현했는데 한 가지 빠진 것이 있다.

그것은 바로 로그인 상태를 유지하는 것이다.



로그인 상태를 유지하는 방법은 크게 **`HttpSession`을 이용하는 방법**과 **쿠키를 이용하는 방법**이 있다.

외부 DB에 세션 데이터를 보관하는 방법도 사용하는데 큰 틀에서 보면 `HttpSession`과 쿠키의 두 가지 방법으로 나뉜다.

이 장에서는 `HttpSession`을 이용해서 로그인 상태를 유지해보자.



컨트롤러에서 `HttpSession`을 사용하려면 다음의 두 가지 방법 중 하나를 선택한다.

- 요청 매핑 애너테이션 적용 메서드에 `HttpSession` 파라미터를 추가한다.
- 요청 매핑 애너테이션 적용 메서드에 `HttpServletRequest` 파라미터를 추가하고 `HttpServletRequest`를 이용해서 `HttpServlet`을 구한다.





다음은 첫 번째 방법을 사용한 코드 예이다.

```java
@PostMapping
public String form(LoginCommand loginCommand, Errors errors, HttpSession httpSession) {
	... // session을 사용하는 코드
}
```

요청 매핑 애너테이션 적용 메서드에 `HttpSession` 파라미터가 존재할 경우 스프링 MVC는 컨트롤러의 메서드를 호출할 때 `HttpSession` 객체를 파라미터로 전달한다. 

`HttpSession`을 생성하기 전이면 새로운 `HttpSession`을 생성하고 그렇지 않으면 기존에 존재하는 `HttpSession`을 전달한다.





두 번째 방법은 다음 코드처럼 `HttpServletRequest`의 `getSession()` 메서드를 이용하는 것이다.

```java
@PostMapping
public String submit(LoginCommand loginCommand, Errors errors, HttpServletRequest req) { 
    HttpSession session = req.getSession();
    ... // session을 사용하는 코드
}
```





`LoginController` 코드에서 인증 후에 인증 정보를 세션에 담도록 `submit()` 메서드의 코드를 아래와 같이 수정해보자.

```java
@Controller
@RequestMapping("/login")
@RequiredArgsConstructor
public class LoginController {
    
    private final AuthService authService;
    
    @GetMapping
    public String form(LoginCommand loginCommand) {
        return "login/loginForm";
    }
    
    @PostMapping
    public String submit(LoginCommand loginCommand, Errors errors, HttpSession session) {
        new LoginCommandValidator().validate(loginCommand, errors);
        if(errors.hasErrors()) return "login/loginForm";
        
        try {
            AuthInfo authInfo = 
                authService.authenticate(loginCommand.getEmail(), loginCommand.getPassword());
            
            // TODO 세션에 authInfo 저장해야 함.
            session.setAttribute("authInfo", authInfo);
            
            return "login/loginSuccess";
        } catch (WrongIdPasswordException ex) {
            errors.reject("idPasswordNotMatching");
            return "login/loginForm";
        }
    }
}
```



로그아웃을 위한 컨트롤러 클래스는 `HttpSession`을 제거하면 된다.

```java
@Controller
public class LogoutController {
    
    @RequestMapping("/logout")
    public String logout(HttpSession session) {
        session.invalidate();
        return "redirect:/main";
    }
}
```





**main.jsp**

```jsp
...
<c:if test="${empty authInfo}">
<p>환영합니다.</p>
<p>
    <a href="<c:url value="/register/step1"/>" >[회원 가입하기]</a>
    <a href="<c:url value="/login"/>" >[로그인]</a>
</p>
</c:if>

<c:if test="${!empty authInfo}">
<p>${authInfo.name}님. 환영합니다.</p>
<p>
    <a href="<c:url value="/edit/changePassword"/>" >[비밀번호 변경]</a>
    <a href="<c:url value="/logout"/>" >[로그아웃]</a>
</p>
</c:if>
```







## 3. 비밀번호 변경 기능 구현

비밀번호 변경 기능을 위한 코드는 다음과 같다.

- `ChangePwdCommand`
- `ChangePwdCommandValidator`
- `ChangePwdController`
- `changePwdForm.jsp`
- `changedPwd.jsp`
- `label.properties`에 메시지 추가
- `ControllerConfig` 설정 클래스에 빈 설정 추가





**ChangePwdCommnad**

```java
@Getter @Setter
public class ChangePwdCommand {
    private String currentPassword;
    private String newPassword;
}
```





**ChangePwdCommandValidator**

```java

public class ChangePwdCommandValidator implements Validator {
    
    @Override
    public boolean supports(Class<?> aClass) {
        return ChangePwdCommand.class.isAssignableFrom(aClass);
    }
    
    @Override
    public void validate(Object target, Errors errors) {
        ValidationUtils.rejectIfEmptyOrWhitespace(errors, "currentPassword", "required");
        ValidationUtils.rejectIfEmpty(errors, "newPassword", "required");
    }
}
```





**ChangePwdController**

```java
@Controller
@RequestMapping("/edit/changePassword")
@RequiredArgsConstructor
public class ChangePwdController {
    
    private final ChangePasswordService changePasswordService;
    
    @GetMapping
    public String form(@ModelAttribute("command") ChangePwdCommand pwdCmd) {
        return "edit/changePwdForm";
    }
    
    @PostMapping
    public String submit(@ModelAttribute("command") ChangePwdCommand pwdCmd, Errors errors, HttpSession session) {
        new ChangePwdCommandValidator().validate(pwdCmd, errors);
        
        if(errors.hasErrors()) {
            return "edit/changePwdForm";
        }
        
        AuthInfo authInfo = (AuthInfo) session.getAttribute("authInfo");
        
        try {
            changePasswordService.changePassword(authInfo.getEmail(),
                                                pwdCmd.getCurrentPassword(),
                                                pwdCmd.getNewPassword());
            return "edit/changedPwd";
        }
    }
}
```













