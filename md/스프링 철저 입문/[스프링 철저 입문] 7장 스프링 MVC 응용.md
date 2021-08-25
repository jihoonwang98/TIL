## 7.1 HTTP 세션 이용

이번 절에서는 스프링 MVC에서 **javax.servlet.http.HttpSession** 객체(이후 HTTP)를 이용하는 방법을 설명한다.



웹 애플리케이션에서 위저드 방식으로 여러 화면에 걸쳐서 입력한 데이터나 온라인 쇼핑몰의 장바구니처럼 담아둔 상품 데이터를 취급하려면 여러 요청이 같은 데이터를 공유할 수 있어야 한다. 이처럼 여러 요청에 걸쳐 데이터를 공유하는 방법은 몇 가지 있지만 **HTTP 세션**을 이용하는 것이 가장 쉬운 방법이다. 단, **HTTP 세션**에 데이터를 저장하면 새로 열리는 창이나 탭에도 같은 **HTTP 세션**을 공유하기 때문에 탭이나 창마다 서로 다른 데이터를 필요로 하는 경우에는 사용할 수 없는 단점이 있다. 그래서 **HTTP 세션**은 아무렇게나 사용하는 것이 아니라 애플리케이션이나 시스템의 요구사항을 고려해서 사용 여부를 결정해야 한다.



스프링 MVC에서 **HTTP 세션**에 데이터를 관리할 때는 다음의 세 가지 방법을 이용할 수 있다.



| HTTP 세션 이용 방법                    | 설명                                                         |
| -------------------------------------- | ------------------------------------------------------------ |
| **세션 속성(@SessionAttributes) 이용** | 스프링 MVC의 org.springframework.ui.Model에 추가한 객체를 HttpSession API를 직접 사용하지 않고 HTTP 세션에서 관리한다. |
| **세션 스코프 빈 이용**                | HTTP 세션에 관리하고 싶은 객체를 DI 컨테이너에 세션 스코프 빈으로 등록해서 HttpSession API를 직접 사용하지 않고 HTTP 세션에서 관리한다. |
| **HttpSession API 이용**               | HttpSession API(setAttribute, getAttribute, removeAttribute 등)를 직접 사용해서 대상 객체를 HTTP 세션에서 관리한다. |



> 스프링 MVC는 컨트롤러의 핸들러 메서드 매개변수로 HttpSession 객체를 받을 수 있지만 가능한 한 HttpSession API를 직접 사용하지 않는 방법(스프링 MVC의 서블릿 API의 추상화 구조)를 이용하자. 참고로 이 책에서는 HttpSession API를 직접 사용하는 방법에 대해서는 따로 소개하지 않는다.





### 7.1.1 세션 속성(@SessionAttributes)

**@org.springframework.web.bind.annotation.SessionAttributes**는 하나의 컨트롤러에서 여러 요청 간에 데이터를 공유하는 경우에 효과적인 방법이다. 입력 화면이 여러 페이지로 나눠진 형태로 구성되거나 복잡한 프로세스를 따라 화면을 이동해야 하는 경우에는 **@SessionAttributes**로 폼 객체를 세션에서 관리하는 방법을 검토해 보자. 폼 객체를 세션에 저장하면 애플리케이션의 설계나 구현이 단순해질 수 있다. 만약 입력 화면, 확인 화면, 완료 화면을 각각 별도의 페이지로 구성되는 비교적 단순한 화면 흐름이라면 HTTP 세션을 사용하지 않고 HTML 폼의 hidden 요소에 데이터를 가지고 다니는 방법도 고려해볼 수 있다.



![](https://docs.google.com/drawings/d/sjca1FhGRXX8jxHhFg_Zppg/image?parent=e/2PACX-1vSgWs9-SkKDVDplyC96i2vj5TuZvbmLHvACG_NMQ3eEa1hFtJwCGbFZVRaHriS3HOeS8O0XIQ6gqqII&rev=185&h=342&w=601&ac=1)





#### 세션에 관리할 객체를 지정하는 방법

**@SessionAttributes**에 **HTTP 세션**에서 관리할 대상 객체를 지정한다. 관리 대상 객체를 지정하는 방법에는 다음 두 가지가 있다.

- **클래스명 지정**: 관리 대상이 되는 클래스명을 **types** 속성에 지정한다.
- **속성명 지정**: 관리 대상이 되는 객체명을 **names** 속성에 지정한다.





- **세션에 관리할 객체를 클래스명으로 지정한 예**

```java
@Controller
@RequestMapping("/accounts")
@SessionAttributes(types=AccountCreateForm.class)
// HTTP 세션에 저장할 객체의 클래스를 @SessionAttributes의 types 속성에 지정한다.
// @ModelAttribute 애너테이션이 붙은 메서드나 Model의 addAttribute 메서드를 통해 Model에 추가한 객체 중에 
// types 속성에서 지정한 클래스의 객체가 있다면 그 객체를 HTTP 세션에 저장한다.
public class AccountCreateController {
    // 생략
}
```



**@SessionAttributes**의 **names** 속성을 이용해 세션에 저장할 객체의 속성명을 지정할 수도 있다. 이 방법은 같은 클래스로 만들어지는 객체 중 세션에서 관리할 것과 관리하지 않을 것이 섞인 경우에 사용할 수 있다.



- **세션에 관리할 객체를 속성명으로 지정한 예**

```java
@Controller
@RequestMapping("/accounts")
@SessionAttributes(names="password")
public class AccountCreateController {
    // 생략
}
```







#### 세션에 객체를 저장하고 저장된 객체를 이용하는 방법

**@SessionAttributes**를 이용해 특정 객체를 **HTTP 세션** 안에 관리하고 싶다면 일단 그 객체를 **Model**에 저장한다. 이후 **Model**에 저장한 객체 중에서 HTTP 세션에 관리하겠다고 지정한 객체만 **HTTP 세션**에 저장(export)되고, 반대로 **HTTP 세션**에 관리되는 객체 중에서 **HTTP 세션**에 관리하겠다고 지정한 객체가 **Model**에 저장(import)된다. 이러한 메커니즘 덕분에 핸들러 메서드나 뷰에서는 사용할 객체의 스코프를 복잡하게 생각하지 않아도 된다.



**Model**에 객체를 저장하는 방법에는 **@ModelAttribute** 메서드를 이용하는 방법과 **Model API**를 이용하는 방법이 있는데, 여기서는 **@ModelAttribute** 메서드를 이용하는 방법을 예로 들었다.



- **객체를 HTTP 세션에 저장하는 예**

```java
@Controller
@RequestMapping("/accounts")
@SessionAttributes(types=AccountCreateForm.class)
public class AccountCreateController {
    
    @ModelAttribute("accountCreateForm")
    // @ModelAttribute 메서드에서 반환한 객체가 Model에 저장된다. 이 객체는 @SessionAttributes에 지정한 types에 해당
    // 하므로 HTTP 세션에도 저장된다. 이 예에서는 스프링 MVC의 명명 규칙에 따라 AccountCreateFor의 객체를 
    // accountCreateForm이라는 이름으로 세션에 저장하고 있다.
    public AccountCreateForm setUpAccountCreateForm() {
        return new AccountCreateForm();
    }
    
    // 생략
}
```





> @ModelAttribute의 value 속성을 생략하면 @ModelAttribute 메서드에서 반환되는 객체의 이름을 만드는 처리가 @ModelAttribute 메서드가 호출될 때마다 수행된다. 이 예와 같이 인스턴스를 생성만 하는 간단한 처리라면 큰 문제가 안 될 수 있지만, 데이터베이스와 같은 외부 리소스에 접근하는 상황에서는 성능 저하로 이어질 수 있다. 이런 문제는 @ModelAttribute의 value 속성에 객체 이름을 명시적으로 지정하는 방법으로 예방할 수 있다.



**Model**에서 객체를 가져오려면 핸들러 메서드에 객체를 받기 위한 인수를 선언한다.



- **Model에서 객체를 꺼내올 때의 구현 예**

```java
@RequestMapping(path="create", method=RequestMethod.POST)
public String create(@Validated AccountCreateForm form, 
                     // Model에서 객체를 받기 위한 인수를 선언한다. 인수에는 클래스명의 첫 글자를 소문자로 바꾼 이름과
                     // 같은 이름의 객체가 설정된다. 이 예에서는 이름이 accountCreateForm인 객체가 인수에 설정된다.
                    BindingResult result,
                    @ModelAttribute("password") String password,
                    // Model에서 가져올 객체의 이름을 @ModelAttribute의 value 속성에 지정할 수 있다.
                    // 만약 @ModelAttribute를 지정한 상태에서 해당 객체가 Model(HTTP 세션)에 존재하지 않을 때는
                    // org.springframework.web.HttpSession.RequiredException이 발생한다.
                    RedirectAttributes redirectAttributes) {
    // 생략
    return "redirect:/accounts/create?complete";
}
```





#### 세션에 저장된 객체의 삭제

HTTP 세션에 저장한 객체를 삭제할 때는 컨트롤러의 핸들러 메서드에서 **org.springframework.web.bind.support.SessionStatus**의 **setComplete** 메서드를 호출한다.



- **세션에 저장된 객체를 삭제하는 예**

```java
@RequestMapping(path="create", params="complete", method=RequestMethod.GET)
public String createComplete(SessionStatus sessionStatus) {
    sessionStatus.setComplete();  // SessionStatus의 setComplete 메서드를 호출하고 세션 처리가 완료됐음을 표시한다.
    return "account/createComplete";
}
```





**SessionStatus**의 **setComplete**을 호출하면 내부적으로는 다음과 같은 동작들이 실행된다.

- **@SessionAttributes에 관리되던 객체가 모두 삭제된다.**
- **SessionStatus의 setComplete 메서드를 호출한 직후에 바로 객체가 삭제되는 것은 아니고, 단지 세션 처리가 완료됐다는 표시만 한다. 실제로는 핸들러 메서드의 처리가 완료된 후에 프레임워크가 내부적으로 HTTP 세션에서 객체를 삭제한다.**
- **핸들러 메서드가 종료된 후에 HTTP 세션에서 객체가 삭제되긴 하지만, 뷰와의 데이터 연계 영역(Model)에는 같은 객체가 남아 있다. 그래서 setComplete을 호출한 직후에 이동한 뷰에서는 HTTP 세션에서 삭제한 객체를 Model에서 참조할 수 있다.**





#### 뷰에서 접근하는 방법

뷰에서 HTTP 세션에 저장한 객체에 접근할 때는 HTTP 세션을 사용하지 않을 때와 마찬가지 방법으로 접근할 수 있다. 이것은 **Model**에 저장한 객체가 **request**(요청 스코프)에도 저장되기 때문이다. 만약 **JSP**를 뷰로 사용할 때는 **EL**에서 **${속성명}**으로 지정하기만 하면 된다.



- **뷰(JSP)에서 접근하는 예**

```jsp
이메일 주소: <c:out value="${accountCreateForm.email}" />
```





### 7.1.2 세션 스코프 빈

**세션 스코프 빈**은 **여러 컨트롤러**에 걸쳐 화면을 이동해야 할 때 컨트롤러 간에 데이터를 공유하는 매개체 역할을 한다. 다양한 유스케이스에 걸쳐 데이터를 공유해야 한다면 세션 스코프 빈을 사용할 것을 검토해보자.



한편 **세션 스코프 빈**을 통해 관리되는 객체의 라이프 사이클은 **HTTP 세션** 자체의 라이프 사이클과 똑같다.



![](https://docs.google.com/drawings/d/sffAOZhz4BpI4Yy_zy_YQAg/image?parent=e/2PACX-1vSgWs9-SkKDVDplyC96i2vj5TuZvbmLHvACG_NMQ3eEa1hFtJwCGbFZVRaHriS3HOeS8O0XIQ6gqqII&rev=461&h=398&w=601&ac=1)





#### 세션 스코프 빈 정의

**HTTP 세션**에서 관리할 객체를 DI 컨테이너에 **세션 스코프 빈**으로 등록한다. 이때 **Scoped Proxy**가 활성화되게 정의한다. **Scoped Proxy**는 스코프 수명이 긴(예: 싱글턴 스코프) 빈에 스코프 수명이 짧은(예: 세션 스코프나 요청 스코프) 빈을 인젝션하기 위한 메커니즘을 제공한다.



- **애너테이션 기반 설정 방식을 이용한 세션 스코프 빈 정의**

```java
@Component
@Scope(value="session", proxyMode=ScopedProxyMode.TARGET_CLASS)
// @Scope 애너테이션의 value 속성에 'session'을 지정해 세션 스코프 빈을 정의한다. 또한 proxyMode 속성에
// ScopedProxyMode.TARGET_CLASS를 지정해 Scoped Proxy가 활성화된다.
public class Cart implements Serializable {
	// 생략
}
```



- **자바 기반 설정 방식을 이용한 세션 스코프 빈 정의**

```java
@Bean
@Scope(value="session", proxyMode=ScopedProxyMode.TARGET_CLASS)
public Cart cart() {
	return new Cart();
}
```



- **XML 기반 설정 방식을 이용한 세션 스코프 빈 정의**

```xml
<beans:bean id="cart" class="com.example.domain.Cart" scope="session">
	<aop:scoped-proxy proxy-target-class="true" />
    <!-- <aop:scoped-proxy>의 proxy-target-class에 true를 지정해 Scoped Proxy가 활성화된다. -->
</beans:bean>
```



> **JPA**의 **Entity** 클래스를 **세션 스코프 빈**으로 정의하면 **JPA API**에서 **세션 스코프 빈**을 제대로 처리하지 못하는 경우가 있다. 제대로 처리하지 못할 때는 **JPA**의 **Entity** 클래스를 **세션 스코프 빈**으로 정의하지 말고 **Entity **클래스를 감싼 래핑 클래스를 **세션 스코프 빈**으로 정의하는 방법을 검토해보자.





#### 세션 스코프 빈 이용

세션 스코프 빈을 사용할 때는 다른 빈과 마찬가지로 인젝션받아서 쓰면 된다.



- **세션 스코프 빈의 이용**

```java
@Controller
@RequestMapping("/items")
public class ItemController {
    @Autowired
    Cart cart; 
    
    // 생략
}


@Controller
@RequestMapping("/cart")
public class CartController {
    @Autowired
    Cart cart;
    
    // 생략
}


@Controller
@RequestMapping("/orders")
public class OrderController {
    @Autowired
    Cart cart;
    
    // 생략
}


// 세션 스코프 빈을 컨트롤러에 인젝션한다. 인젝션된 세션 스코프 빈의 메서드를 호출하면 같은 객체의 메서드를 호출하는
// 효과가 있기 때문에 여러 컴포넌트에서 세션 스코프의 데이터를 공유할 수 있다.
```





#### 뷰에서 접근하는 방법

**뷰**에서 **세션 스코프 빈**에 접근할 때는 **SpEL**을 이용해 DI 컨테이너에서 **세션 스코프 빈**을 꺼내온다.



- **뷰(JSP)에서 접근하는 예**

```jsp
<spring:eval var="cart" expression="@cart" />
<!-- SpEL(<spring:eval> 요소)을 이용해 세션 스코프 빈을 참조한 다음, 페이지 스코프 변수에 저장한다. 
expression 속성에는 '@+Bean명'을 지정한다. 페이지 스코프 변수에 저장하고 나면 @SessionAttributes를 사용하는 것과 같은
방법으로 HTTP 세션에 저장한 객체에 접근할 수 있다. -->
<c:forEach var="cartItem" items="${cart.cartItems}">
	<!-- 생략 -->
</c:forEach>
```





## 7.2 파일 업로드

이번 절에서는 파일을 업로드하는 방법을 설명한다. 스프링 MVC에서 파일 업로드를 할 떄는 다음 방법 중 하나를 사용한다.

- **서블릿 표준 업로드 기능**

  서블릿 3.0에서 지원하는 파일 업로드 기능과 스프링 웹에서 제공하는 컴포넌트를 이용해 파일을 업로드한다. 이 방법은 서블릿 버전이 3.0 이상인 애플리케이션 서버를 사용할 수 있을 때 이용한다.



- **Apache Commons FileUpload 업로드 기능**

  파일 업로드용 라이브러리인 **Apache Commons FileUpload**와 스프링 웹에서 제공하는 컴포넌트를 이용해 파일을 업로드한다. 이 방법은 서블릿 버전이 3.0 미만인 경우나 서블릿  표준 파일 업로드 기능으로는 요청 파라미터나 파일명이 깨지는 경우에 이용한다.





### 7.2.1 파일 업로드 구조

먼저 스프링 MVC와 스프링 웹에서 제공하는 파일 업로드 구조를 소개한다.



![](https://docs.google.com/drawings/d/s9a9EBnB_hjxqK0wfYNLCxg/image?parent=e/2PACX-1vSgWs9-SkKDVDplyC96i2vj5TuZvbmLHvACG_NMQ3eEa1hFtJwCGbFZVRaHriS3HOeS8O0XIQ6gqqII&rev=349&h=413&w=601&ac=1)



**(1)**: 업로드할 파일을 선택하고 업로드를 실행한다.

**(2)**: **DispatcherServlet**은 **org.springframework.web.multipart.MultipartResolver** 인터페이스의 메서드를 호출해서 멀티파트 요청을 해석한다.

**(3)**: **MultipartResolver** 구현 클래스는 멀티파트 요청을 해석한 후, 업로드 데이터를 담을 **org.springframework.web.multipart.MultipartFile**을 생성한다.

**(4)**: **DispatcherServlet**은 컨트롤러의 처리 메서드를 호출한다. **(3)**에서 생성한 **MultipartFile** 객체는 컨트롤러의 인수나 폼 객체에 바인드 받는다.

**(5)**: 컨트롤러는 **MultipartFile** 객체의 메서드를 통해 업로드된 파일의 내용이나 메타 정보를 가져온다.



스프링이 제공하는 컴포넌트가 **Servlet**이나 **Apache Commons FileUpload**의 **API**를 감춰주기 때문에 내부적으로 **Servlet** 표준이나 **Apache Commons FileUpload** 중 어느 쪽을 사용해도 컨트롤러를 같은 방식으로 구현할 수 있다. 참고로 이 책에서는 서블릿 표준 업로드 기능을 이용하는 방법을 소개한다.



### 7.2.2 파일 업로드 기능 설정

서블릿 표준과 스프링이 제공하는 파일 업로드 기능을 이용하려면 파일 업로드와 관련된 설정과 스프링 MVC를 연계하기 위한 설정이 필요하다.



#### 파일 업로드 기능을 이용하기 위한 설정

서블릿 표준의 파일 업로드 기능을 이용하기 위해 **web.xml**에 **\<multipart-config>** 요소를 추가한다.



- **web.xml 설정예**

```xml
<web-app
    xmlns="http://xmlns.jcp.org/xml/ns/javaee"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
                        http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
    version="3.1">
	<!-- 생략 -->
    
    <servlet>
    	<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <multipart-config />
        <!-- 서블릿 표준의 파일 업로드 기능을 사용하는 서블릿 <servlet> 요소에 <multipart-config> 요소를 추가한다. -->
    </servlet>

    <!-- 생략 -->
</web-app>
```



서블릿 표준의 파일 업로드 기능을 별다른 설정 없이 사용하면 업로드할 수 있는 파일 크기가 무제한이 된다. 파일 크기를 제한하고 싶다면 파일 단위의 최대 크기, 업로드 요청 전체의 최대 크기, 임시 파일을 생성하는 임계치 크기와 같은 세 가지 설정을 추가해야 한다.



- **\<multipart-config> 요소를 설정한 예**

```xml
<multipart-config>
	<max-file-size>5242880</max-file-size>  
    <!-- 파일 하나의 최대 바이트 수를 지정한다. 기본값은 -1(제한 없음).-->
    <max-request-size>27262976</max-request-size>
    <!-- 멀티파트 요청 전체의 최대 바이트 수를 지정한다. 기본값은 -1(제한 없음).-->
    <file-size-threshold>1048576</file-size-threshold>
    <!-- 전송된 파일의 크기가 이 크기를 넘어서면 메모리에 있던 파일 내용을 임시 파일로 만든다. 기본값은 0(항상 파일에 출력)이다.-->
</multipart-config>
```



크기 제한을 설정한 상태에서 크기 제한을 넘으면 **org.springframework.web.multipart.MultipartException**이 발생한다. 만약 크기 제한 에러를 처리하고 싶다면 **MultipartException**을 **catch**로 잡으면 된다.





#### 스프링 MVC와 연계하기 위한 설정





















