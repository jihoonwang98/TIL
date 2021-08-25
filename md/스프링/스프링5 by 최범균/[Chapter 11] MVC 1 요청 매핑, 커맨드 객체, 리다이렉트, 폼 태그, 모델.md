# [Chapter 11] MVC 1: 요청 매핑, 커맨드 객체, 리다이렉트, 폼 태그, 모델





## 1. 요청 매핑 애너테이션을 이용한 경로 매핑

웹 애플리케이션을 개발하는 것은 다음 코드를 작성하는 것이다.

- 특정 요청 URL을 처리할 코드 -> `@Controller`
- 처리 결과를 HTML과 같은 형식으로 응답하는 코드 -> `View`



컨트롤러 클래스는 요청 매핑 애너테이션을 사용해서 메서드가 처리할 요청 경로를 지정한다.

`@RequestMapping`, `@GetMapping`, `@PostMapping`, ...





## 2. 요청 파라미터(RequestParam) 접근



### `HttpServletRequest` 이용

컨트롤러 메서드에서 요청 파라미터를 사용하는 첫 번째 방법은 `HttpServletRequest`를 직접 이용하는 것이다.

예를 들면 다음과 같이 컨트롤러 처리 메서드의 파라미터로 `HttpServletRequest` 타입을 사용하고,

`HttpServletRequest`의 `getParameter()` 메서드를 이용해서 파라미터의 값을 구하면 된다.

```java
@Controller
public class RegisterController {

	@RequestMapping("/register") 
    public String handle(HttpServletRequest request) {
        String agreeParam = request.getParameter("agree");
        if(agreeParam == null || !agreeParam.equals("true")) {
            return "register/step1";
        }
        return "register/step2";
    }
}
```





### `@RequestParam` 이용

요청 파라미터에 접근하는 두 번째 방법은 `@RequestParam` 애너테이션을 이용하는 것이다.

요청 파라미터 개수가 몇 개 안 되면 이 애너테이션을 사용해서 간단하게 요청 파라미터의 값을 구할 수 있다.

다음은 위 코드를 `@RequestParam` 애너테이션을 사용해서 구현한 코드이다.

```java
@Controller
public class RegisterController {
    
    @RequestMapping("/register") 
    public String handle(@RequestParam(value="agree", defaultValue="false") Boolean agreeVal) {
        
        if(!agreeVal) {
            return "register/step1";
        }
        return "register/step2";
    }
}
```



`@RequestParam` 애너테이션은 아래와 같은 속성을 제공한다.

| 속성         | 타입    | 설명                                                         |
| ------------ | ------- | ------------------------------------------------------------ |
| value        | String  | HTTP 요청 파라미터의 이름을 지정한다.                        |
| required     | boolean | 필수 여부를 지정한다. 이 값이 true이면서 해당 요청 파라미터에 값이 없으면 Exception이 발생한다. 기본값은 true이다. |
| defaultValue | String  | 요청 파라미터가 값이 없을 때 사용할 문자열 값을 지정한다. 기본값은 없다. |



`@RequestParam` 애너테이션을 사용한 코드를 보면 다음과 같이 `agreeVal` 파라미터의 타입이 `Boolean`이다.

```java
public String handle(@RequestParam(value="agree", defaultValue="false") Boolean agreeVal) 
```



스프링 MVC는 파라미터 타입에 맞게 `String` 값을 변환해준다.

위 코드는 `agree` 요청 파라미터의 값을 읽어와 `Boolean` 타입으로 변환해서 `agreeVal` 파라미터에 전달한다.

`Boolean` 타입 외에 `int`, `long`, `Integer`, `Long` 등 기본 데이터 타입과 Wrapper 타입에 대한 변환을 지원한다.







## 3. 리다이렉트 처리

컨트롤러에서 특정 페이지로 리다이렉트시키는 방법은 간단하다.

**`redirect:경로`를 뷰 이름으로 리턴하면 된다**.

```java
@Controller 
public class RegisterController {

	@GetMapping("/register/step2") 
	public String handle() {
		return "redirect:/register/step1";
	}
}
```



`redirect:` 뒤의 문자열이 "/"로 시작하면 웹 애플리케이션을 기준으로 이동 경로를 생성한다.

예를 들어 위 코드에서는 뷰 값으로 `redirect:/register/step1`을 사용했는데, 

이 경우 이동 경로가 "/"로 시작하므로 실제 리다이렉트할 경로는 웹 애플리케이션 경로인 `/sp5-chap11`과 `/register/step1`을 연결한 `/sp5-chap11/register/step1`이 된다.



"/"로 시작하지 않으면 현재 경로를 기준으로 상대 경로를 사용한다.

예를 들어 `redirect:step1`을 리턴했으면 현재 요청 경로인 `http://localhost:8080/sp5-chap11/register/step2`를 기준으로

상대 경로인 `http://localhost:8080/sp5-chap11/register/step1`을 리다이렉트 경로로 사용한다.



`redirect:http://localhost:8080/sp5-chap11/register/step1`과 같이 완전한 URL을 사용하면 해당 경로로 리다이렉트한다.







## 4. 커맨드 객체를 이용해서 요청 파라미터 사용하기

다음 파라미터를 이용해서 정보를 서버에 전송한다고 해보자.

- email
- name
- password
- confirmPassword



폼 전송 요청을 처리하는 컨트롤러 코드는 각 파라미터의 값을 구하기 위해 다음과 같은 코드를 사용할 수 있다.

```java
@PostMapping("/register/step3")
public String handle(HttpServletRequest request) {
	String email = request.getParameter("email");
    String name = request.getParameter("name");
    String password = request.getParameter("password");
    String confirmPassword = request.getParameter("confirmPassword");
    
    RegisterRequest regReq = new RegisterRequest();
    regReq.setEmail(email);
    regReq.setName(name);
    regReq.setPassword(password);
    regReq.setConfirmPassword(confirmPassword);
    ...
}
```

보시다싶이 굉장히 귀찮다..

요청 파라미터가 4개라 그나마 할만 하지만, 10개만 돼도 끔찍하다.





스프링은 이런 불편함을 줄이기 위해 요청 파라미터의 값을 **커맨드(command) 객체**에 담아주는 기능을 제공한다.

예를 들어, 이름이 `name`인 요청 파라미터의 값을 커맨드 객체의 `setName()` 메서드를 사용해서 커맨드 객체에 전달하는 기능을 제공한다.

커맨드 객체라고 해서 특별한 코드를 작성해야 하는 것은 아니다.

요청 파라미터의 값을 전달받을 수 있는 <u>세터 메서드를 포함하는</u> 객체를 커맨드 객체로 사용하면 된다.





커맨드 객체는 다음과 같이 요청 매핑 애너테이션이 적용된 메서드의 파라미터에 위치한다.

```java
@PostMapping("/register/step3")
public String handle(/* 커맨드 객체 */ RegisterRequest regReq) {
	...
}
```

 

`RegisterRequest` 클래스에는 `setEmail()`, `setName()`, `setPassword()`, `setConfirmPassword()` 메서드가 있다.

스프링은 이들 메서드를 사용해서 `email`, `name`, `password`, `confirmPassword` 요청 파라미터의 값을 커맨드 객체에 복사한 뒤 `regReq` 파라미터로 전달한다.



**즉, 스프링 MVC가 `handle` 메서드에 전달할 `RegisterRequest` 객체를 생성하고** 

**그 객체의 세터 메서드를 이용해서 일치하는 요청 파라미터의 값을 전달한다.**







## 5. 뷰에서 커맨드 객체 사용하기

```java
@PostMapping("/register/step3")
public String handle(RegisterRequest regReq) {
	...
}
```



```jsp
...
<p><th:block th:text="${registerRequest.name} + '님 회원 가입을 완료했습니다.'" /></p>
...
```

여기서 `registerRequest`가 커맨드 객체에 접근할 때 사용한 속성 이름이다.



스프링 MVC는 커맨드 객체의 첫 글자를 소문자로 바꾼 이름을 속성 이름으로 사용해서 커맨드 객체를 뷰에 전달한다.

커맨드 객체의 클래스 이름이 `RegisterRequest`인 경우 `registerRequest`라는 이름으로 뷰에 전달된다.





## 6. `@ModelAttribute` 애너테이션으로 커맨드 객체 속성 이름 변경

커맨드 객체에 접근할 때 사용할 속성 이름을 변경하고 싶다면 

커맨드 객체로 사용할 파라미터에 `@ModelAttribute` 애너테이션을 적용하면 된다.

```java
@PostMapping("/register/step3")
public String handle(@ModelAttribute("formData") RegisterRequest regReq) {
	...
}
```







## 7. 컨트롤러 구현 없는 경로 매핑

```java
@Controller
public class MainController {

	@RequestMapping("/main")
	public String main() {
		return "main";
	}
}
```

이 컨트롤러 코드는 요청 경로와 뷰 이름을 연결해주는 것에 불과하다.

단순 연결을 위해 특별한 로직이 없는 컨트롤러 클래스를 만드는 것은 성가시다.

`WebMvcConfigurer` 인터페이스의 `addViewControllers()` 메서드를 사용하면 이런 성가심을 없앨 수 있다.

이 메서드를 재정의하면 컨트롤러 구현 없이 다음의 간단한 코드로 요청 경로와 뷰 이름을 연결할 수 있다.



```java
@Override
public void addViewControllers(ViewControllerRegistry registry) {
	registry.addViewController("/main").setViewName("main");
}
```







## 8. 커맨드 객체: 중첩&middot;콜렉션 프로퍼티

세 개의 설문 항목과 응답자의 지역과 나이를 입력받는 설문 조사 정보를 담기 위해 아래와 같이 클래스를 작성해보자.

```java
@Getter @Setter
public class Respondent {
    
    private int age;
    private String location;
}

@Getter @Setter
public class AnsweredData {
    
    private List<String> responses;
    private Respondent res;
}
```

`Respondent` 클래스는 응답자 정보를 담는다.

`AnsweredData` 클래스는 설문 항목에 대한 답변과 응답자 정보를 함께 담는다.





스프링 MVC는 커맨드 객체가 리스트 타입의 프로퍼티를 가지거나 중첩 프로퍼티를 가진 경우에도 

요청 파라미터의 값을 알맞게 커맨드 객체에 설정해주는 기능을 제공하고 있다.

규칙은 다음과 같다.

- HTTP 요청 파라미터 이름이 "프로퍼티이름[인덱스]" 형식이면 `List` 타입 프로퍼티 값 목록으로 처리한다.
- HTTP 요청 파라미터 이름이 "프로퍼티이름.프로퍼티이름"과 같은 형식이면 중첩 프로퍼티 값을 처리한다.



예를 들어 이름이 `responses`이고 `List` 타입인 프로퍼티를 위한 요청 파라미터의 이름으로 

"responses[0]", "responses[1]"을 사용하면 각각 0번 인덱스와 1번 인덱스의 값으로 사용된다.

중첩 프로퍼티의 경우 파라미터 이름을 "res.name"으로 지정하면 다음과 유사한 방식으로 커맨드 객체에 파라미터의 값을 설정한다.

```java
commandObj.getRes().setName(request.getParameter("res.name"));
```





### 예제

```java
@Controller
@RequestMapping("/survey")
public class SurveyController {
    
    @GetMapping
    public String form() {
        return "survey/surveyForm";
    }

    
	@PostMapping
	public String submit(@ModelAttribute("ansData") AnsweredData data) {
		return "survey/submitted";
	}
}
```





**surveyForm.html**

```jsp
...
<form method="post">
    <p>
        1. 당신의 역할은? <br/>
        <label>
            <input type="radio" name="responses[0]" value="서버"> 서버개발자
        </label>
        
        <label>
            <input type="radio" name="responses[0]" value="프론트"> 프론트개발자
        </label>
        
        <label>
            <input type="radio" name="responses[0]" value="풀스택"> 풀스택개발자
        </label>
    </p>
    
    <p>
        2. 가장 많이 사용하는 개발도구는? <br/>
        <label>
            <input type="radio" name="responses[1]" value="Eclipse"> Eclipse
        </label>
        
        <label>
            <input type="radio" name="responses[1]" value="Intellij"> Intellij
        </label>
        
        <label>
            <input type="radio" name="responses[1]" value="Sublime"> Sublime
        </label>
    </p>
    
    <p>
        3. 하고싶은 말 <br/>
        <input type="text" name="responses[2]">
    </p>
    
    <p>
        <label> 응답자 위치: <br>
            <input type="text" name="res.location">
        </label>
    </p>
    <p>
        <label> 응답자 나이: <br>
            <input type="text" name="res.age">
        </label>
    </p>
    <input type="submit" value="전송">
</form>
```



위의 코드에서 각 \<input> 태그의 name 속성은 다음과 같이 커맨드 객체의 프로퍼티에 매핑된다.

- responses[0] -> responses 프로퍼티(List 타입)의 첫 번째 값
- responses[1] -> responses 프로퍼티(List 타입)의 두 번째 값
- responses[2] -> responses 프로퍼티(List 타입)의 세 번째 값
- res.location -> res 프로퍼티(Respondent 타입)의 location 프로퍼티
- res.age -> res 프로퍼티(Respondent 타입)의 age 프로퍼티





**submitted.html (thymeleaf)**

```html
<p>응답 내용 :</p>

<ul>
    <li th:each="response : ${ansData.responses}" >
        <th:block th:text="${responseStat.count} + '번 문항: ' + ${response}" />
    </li>
</ul>

<p> 응답자 위치: <th:block th:text="${ansData.res.location}" /> </p>
<p> 응답자 나이: <th:block th:text="${ansData.res.age}" /> </p>
```







## 9. Model을 통해 컨트롤러에서 뷰에 데이터 전달하기

컨트롤러는 뷰가 응답 화면을 구성하는데 필요한 데이터를 생성해서 전달해야 한다.

이때 사용하는 것이 `Model`이다. 



뷰에 데이터를 전달하는 컨트롤러는 메서드를 작성할 때 두 가지를 하면 된다.

- 요청 매핑 애너테이션이 적용된 메서드의 파라미터로 `Model`을 추가한다.
- `Model` 파라미터의 `addAttribute()` 메서드로 뷰에서 사용할 데이터 전달



`addAttribute()` 메서드의 첫 번째 파라미터는 속성 이름이다.

뷰 코드는 이 이름을 사용해서 데이터에 접근한다.

Thymeleaf는 다음과 같이 표현식을 사용해서 속성값에 접근한다. `${greeting}`



8장에서 작성한 예제는 **surveyForm**에 설문 항목을 하드 코딩했다.

설문 항목을 컨트롤러에서 생성해서 뷰에 전달하는 방식으로 변경해보자.



먼저 개별 설문 항목 데이터를 담기 위한 클래스를 아래와 같이 작성하자.

```java
@Getter
@AllArgsConstructor
public class Question {
    
    private String title;
    private List<String> options;
    
    public Question(String title) {
        this(title, Collections.<String>emptyList());
    }
    
    public boolean isChoice() {
        return options != null && !options.isEmpty();
    }
}
```



다음 작업은 `SurveyController`가 `Question` 객체 목록을 생성해서 뷰에 전달하도록 구현하는 것이다.

실제로는 DB 같은 곳에서 정보를 읽어와 `Question` 목록을 생성하겠지만 이 예제는 간단히 컨트롤러에서 직접 생성해 구현했다.

```java
@Controller
@RequestMapping("/survey")
public class SurveyController {
    
    @GetMapping
    public String form(Model model) {
        List<Question> questions = createQuestions();
        model.addAttribute("questions", questions);
        return "survey/surveyForm";
    }
    
    
    private List<Question> createQuestions() {
        Question q1 = new Question("당신의 역할은 무엇입니까?", Arrays.asList("서버", "프론트", "풀스택"));
        Question q2 = new Question("많이 사용하는 개발도구는 무엇입니까?", Arrays.asList("이클립스", "인텔리제이", "서브라임"));
        Question q3 = new Question("하고 싶은 말을 적어주세요.");
        return Arrays.asList(q1, q2, q3);
    }

    
	@PostMapping
	public String submit(@ModelAttribute("ansData") AnsweredData data) {
		return "survey/submitted";
	}
}
```





**surveyForm.html (thymeleaf)**

```html
...
<form method="post">
  	<p th:each="question: ${questions}">
    <th:block th:text="|${questionStat.count}. ${question.title}|" /> <br/>

      <th:block th:if="${question.choice}">
        <label th:each="option: ${question.options}">
          <input  type="radio" th:name="'responses[__${questionStat.index}__]'"
                  th:value="${option}">
          <th:block  th:text="${option}" />
        </label>
      </th:block>

      <th:block th:if="${!question.choice}">
        <input type="text" th:name="'responses[__${questionStat.index}__]'">
      </th:block>
  </p>
</form>
...
```







### ModelAndView를 통한 뷰 선택과 모델 전달

지금까지 구현한 컨트롤러는 두 가지 특징이 잇다.

- `Model`을 이용해서 뷰에 전달할 데이터 설정
- 결과를 보여줄 뷰 이름을 리턴



`ModelAndView`를 사용하면 이 두 가지를 한 번에 처리할 수 있다.

요청 매핑 애너테이션을 적용한 메서드는 `String` 타입 대신 `ModelAndView`를 리턴할 수 있다.

`ModelAndView`는 모델과 뷰 이름을 함께 제공한다.

```java
@GetMapping
public ModelAndView form() {
    List<Question> questions = createQuestions();
    ModelAndView mav = new ModelAndView();
    mav.addObject("questions", questions);
    mav.setViewName("survey/surveyForm");
    return mav;
}
```





### GET 방식과 POST 방식에 동일 이름 커맨드 객체 사용하기

`th:object`를 사용하려면 커맨드 객체가 반드시 존재해야 한다.

최초에 폼을 보여주는 요청에 대해 `th:object`를 사용하려면 

폼 표시 요청이 왔을 때에도 커맨드 객체를 생성해서 모델에 저장해야 한다.



이를 위해 아래 코드에서 `RegisterController` 클래스의 `handle()` 메서드는 다음과 같이 `Model`에 직접 객체를 추가했다.

```java
@PostMapping("/register/step2")
public String handle(@RequestParam(value="agree", defaultValue="false") Boolean agree, Model model) {
	if(!agree) return "register/step1";
	
	model.addAttribute("registerRequest", new RegisterRequest());
	return "register/step2";
}
```



커맨드 객체를 파라미터로 추가하면 더 간단해진다.

```java
@PostMapping("/register/step2")
public String handle(@RequestParam(value="agree", defaultValue="false") Boolean agree, 
                     RegisterRequest registerRequest) {
	if(!agree) return "register/step1";
	
	return "register/step2";
}
```



이름을 명시적으로 지정하려면 `@ModelAttribute` 애너테이션을 사용한다.



예를 들어, "/login" 요청 경로일 때 GET 방식이면 로그인 폼을 보여주고, POST 방식이면 로그인을 처리하도록 구현한 컨트롤러를 만들어야 한다고 하자.

입력 폼과 폼 전송 처리에서 사용할 커맨드 객체의 속성 이름이 클래스 이름과 다르다면 

다음과 같이 GET 요청과 POST 요청을 처리하는 메서드에 `@ModelAttribute` 애너테이션을 붙인 커맨드 객체를 파라미터로 추가하면 된다.

```java
@Controller
@RequestMapping("/login")
public class LoginController {
	
	@GetMapping
	public String form(@ModelAttribute("login") LoginCommand loginCommand) {
		return "login/loginForm";
	}
	
	@PostMapping
	public String form(@ModelAttribute("login") LoginCommand loginCommand) {
		...
	}
}
```



















