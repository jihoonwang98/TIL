## 4.1 스프링 MVC

**스프링 MVC**는 자바 기반의 웹 애플리케이션을 개발할 때 사용하는 프레임워크의 하나로서 프레임워크 아키텍처로 **MVC 패턴**을 채택했다. **MVC 패턴**을 적용한 웹 애플리케이션은 **모델(Model)**, **뷰(View)**, **컨트롤러(Controller)**와 같은 세 가지 역할의 컴포넌트로 구성되어 클라이언트의 요청을 처리한다. 각 컴포넌트에 의한 일반적인 처리 흐름은 아래와 같다.





![](https://mblogthumb-phinf.pstatic.net/MjAxOTA3MDFfMjUy/MDAxNTYxOTYwMDIyMjE3.xD4tCw4qFwgF0CTqLZCol1P5nPZLQ2sJeTis22MLlzUg.-5LKB1QigjZzVY5SWlyMHxxcw99Iz4s51Y9ePHBVILsg.PNG.sejun3278/%EC%A0%9C%EB%AA%A9_%EC%97%86%EC%9D%8C.png?type=w800)



**MVC 패턴**에서 각 컴포넌트의 역할은 다음과 같다.



**MVC 패턴에서 각 컴포넌트의 역할**

| 컴포넌트명   | 설명                                                         |
| ------------ | ------------------------------------------------------------ |
| **모델**     | 애플리케이션 상태(데이터)나 비즈니스 로직을 제공하는 컴포넌트 |
| **뷰**       | 모델이 보유한 애플리케이션 상태(데이터)를 참조하고 클라이언트에 반환할 응답 데이터를 생성하는 컴포넌트 |
| **컨트롤러** | 요청을 받아 모델과 뷰의 호출을 제어하는 컴포넌트로 컨트롤러라는 이름처럼 요청과 응답의 처리 흐름을 제어한다 |



> 스프링 MVC는 MVC 패턴을 채택한 프레임워크라고 설명했지만 정확히 말하면 **프런트 컨트롤러(Front Controller)**를 채택한 것이다. **프런트 컨트롤러 패턴**은 MVC 패턴이 가진 약점을 개선한 아키텍처 패턴으로서 많은 MVC 프레임워크에서 사용된다. 





---

### 4.1.1 웹 애플리케이션 개발의 특징

우선 스프링 MVC로 개발된 **웹 애플리케이션의 특징**을 살펴보자. 스프링 MVC는 웹 애플리케이션을 매우 편리하게 개발할 수 있는 프레임워크로서 다음과 같은 특징이 있다.



- **POJO(Plain Old Java Object) 구현**

  **컨트롤러**나 **모델** 등의 클래스는 **POJO** 형태로 구현된다. 특정 프레임워크에 종속적인 형태로 구현할 필요가 없기 때문에 단위 테스트를 하는 것이 상대적으로 수월해진다.



- **애너테이션을 이용한 정의 정보 설정**

  요청 매핑과 같은 각종 정의 정보를 설정 파일이 아닌 **애너테이션** 방식으로 설정할 수 있다. 비즈니스 로직과 그 로직을 수행하기 위한 각종 정의 정보를 자바 파일 안에서 함께 기술할 수 있기 때문에 효율적으로 웹 애플리케이션을 개발할 수 있다.



- **유연한  메서드 시그니처 정의**

  **컨트롤러** 클래스의 **메서드 매개변수**에는 처리에 필요한 것만 골라서 정의할 수 있다. 인수에 지정할 수 있는 타입도 다양한 타입이 지원되며, 프레임워크가 인수에 전달하는 값을 자동으로 담아주거나 변환하기 때문에 사양 변경이나 리팩터링에 강한 아키텍처를 가지고 있다. 반환값도 다양한 타입을 지원한다.



- **Servlet API 추상화**

  스프링 MVC는 **서블릿 API(HttpServletRequest, HttpServletResponse, HttpSession 등의 API)**를 추상화하는 기능을 제공한다. **서블릿 API**를 추상화한 구조를 이용하면 **컨트롤러** 클래스 구현에서 **서블릿 API**를 직접 사용하는 코드가 제거되기 때문에 **컨트롤러** 클래스의 테스트가 **서블릿 API**를 사용할 때보다 상대적으로 쉬워진다.



- **뷰 구현 기술의 추상화**

  **컨트롤러**는 **뷰 이름**(뷰의 논리적인 이름)을 반환하고 스프링 MVC는 뷰 이름에 해당하는 화면이 표시되게 한다. 이처럼 **컨트롤러**는 뷰 이름만 알면 되기 때문에 그 뷰가 어떤 구현 기술(JSP, 타임리프, 서블릿 API, 프리마커 등)로 만들어졌는지 구체적인 내용을 몰라도 된다.



- **스프링의 DI 컨테이너와의 연계**

  스프링 MVC는 스프링의 DI 컨테이너 상에서 동작하는 프레임워크다. 스프링의 DI 컨테이너가 제공하는 **DI**나 **AOP **같은 구조를 그대로 활용할 수 있어서 웹 애플리케이션을 효과적으로 개발할 수 있다.



> 스프링 MVC는 서블릿 API를 추상화하는 메커니즘을 제공하는 반면 서블릿 API를 직접 사용할 수도 있다. 쿠키(Cookie)에 데이터를 쓰는 것과 같은 일부 작업은 서블릿 API를 직접 사용하지 않으면 구현할 수 없는 경우도 있다.





스프링 MVC의 특징을 잘 살려서 만든 **컨트롤러** 클래스는 다음과 같은 형태가 된다.

```java
@Controller  // DI 컨테이너와의 연계 (빈 정의 + DI)
public class WelcomeController { // 프레임워크에서 제공하는 인터페이스 구현은 불필요 (= POJO) 
    @Autowired  // DI 컨테이너와의 연계 (빈 정의 + DI)
	MyService myService;
    
    @RequestMapping("/") // 애너테이션에서 정의 정보를 지정
    public String home(Model model /* 유연한 인수 정의 */) {
        Date now = myService.getCurrentDate();
        model.addAttribute("now", now); // 서블릿 API에 의존하지 않는 구현
        return "home"; // 뷰 구현 기술에 의존하지 않는 뷰 이름 지정
    }
}
```







### 4.1.2 MVC 프레임워크로서의 특징

다음은 MVC 프레임워크로서의 특징을 소개한다. 스프링 MVC는 높은 확장성과 엔터프라이즈 애플리케이션이 필요로 하는 기능을 갖춘 MVC 프레임워크로서 다음과 같은 특징이 있다.



- **풍부한 확장 포인트 제공**

  스프링 MVC에서는 **컨트롤러**나 **뷰**와 같이 각 역할별로 필요한 인터페이스를 제공한다. 그래서 기본 동작을 확장하고 싶다면 이러한 인터페이스를 자신만의 방법으로 구현하면 된다. 이러한 인터페이스는 커스터마이징을 유연하고 쉽게 만들어 주는 확장점이 된다.



- **엔터프라이즈 애플리케이션에 필요한 기능 제공**

  스프링 MVC는 단순히 MVC 패턴의 프레임워크 구현만 제공하는 것이 아니다. **메시지 관리, 세션 관리, 국제화, 파일 업로드** 같은 엔터프라이즈 애플리케이션용 웹 애플리케이션을 개발할 때 필요한 다양한 기능들도 함께 제공한다.



- **서드파티 라이브러리와의 연계 지원**

  스프링 MVC는 **서드파티 라이브러리**를 이용할 때 필요한 각종 **어댑터**를 제공하며, 다음과 같은 라이브러리를 스프링 MVC와 연계해서 사용할 수 있다.

  - Jackson (JSON / XML 처리)

  - Google Gson (JSON 처리)

  - Google Protocol Buffers (Protocol Buffers로 불리는 직렬화 형식 처리)

  - Apache Tiles (레이아웃 엔진)

  - FreeMarker (템플릿 엔진)

  - Rome (RSS / Feed 처리)

  - JasperReports (보고서 출력)

  - ApachePOI (엑셀 처리)

  - Hibernate Validator (빈 유효성 검증)

  - Joda-Time (날짜 / 시간 처리)

    

  또한 서드 파티 라이브러리 자체가 스프링 MVC와의 연계를 지원하는 형태도 있다.

  - Thymeleaf (템플릿 엔진)
  - HDIV (보안 강화)





## 4.2 첫 번째 스프링 MVC 애플리케이션

스프링 MVC를 더 자세히 설명하기 전에 간단한 애플리케이션을 먼저 만들어보고 스프링 MVC를 이용한 애플리케이션 개발을 기초부터 배워보자.



![183 page 그림]()



이 절에서는 위와 같은 입력 화면에서 입력 값을 받아 출력 화면으로 표시하는 에코 애플리케이션을 만들어볼 것이다. 





### 4.2.1 프로젝트 생성



### 4.2.2 스프링 MVC 적용

















## 4.3 스프링 MVC 아키텍처

이번 절에서는 MVC 프레임워크 자체의 아키텍처 개요와 스프링 MVC를 구성하는 주요 컴포넌트 역할을 설명하면서 프레임워크의 동작 방식을 살펴보자.





### 4.3.1 프레임워크 아키텍처

스프링 MVC는 **'프런트 컨트롤러 패턴(Front Controller Pattern)'**이라고 하는 아키텍처를 채택하고 있다. **Front Controller Pattern**은 클라이언트 요청을 **Front Controller**라는 컴포넌트가 받아 요청 내용에 따라 수행하는 **핸들러(handler)**를 선택하는 아키텍처다.



![](http://iloveulhj.github.io/images/dispatcherservlet/dispatcherservlet2.png)



**Front Controller Pattern**은 공통적인 처리를 **Front Controller**에 통합할 수 있어서 **핸들러(컨트롤러)**에서 처리하는 내용을 줄일 수 있다. 스프링 MVC 아키텍처에서는 다음과 같은 기능들을 **Front Controller**가 대행해주고 있다.

- **클라이언트의 요청 접수**
- **요청 데이터를 자바 객체로 변환**
- **입력값 검사 실행(Bean Validation)**
- **핸들러 호출**
- **뷰 선택**
- **클라이언트에 요청 결과 응답**
- **예외 처리**





### 4.3.2 프런트 컨트롤러 아키텍처

여기서는 스프링 MVC의 **Front Controller**가 어떤 **아키텍처**로 되어 있는지 하나씩 설명하겠다.



먼저 스프링 MVC에서 **Front Controller**의 **처리 흐름**을 살펴보자. 스프링 MVC의 **Front Controller**는`org.springframework.web.servlet.DispatcherServlet` 클래스(서블릿)로 구현돼 있으며, 다음과 같은 흐름으로 처리를 수행한다.



![](https://changrea.io/static/b1c7cdd1c8cefbdfa437ec5e36d70ea4/fcda8/spring-mvc-frontArchitecture.png)



**1)** **DispatcherServlet** 클래스는 클라이언트의 요청을 받는다.

**2)** **DispatcherServlet** 클래스는 **HandlerMapping** 인터페이스의 **getHandler** 메서드를 호출해서 요청 처리를 하는 **Handler** 객체(컨트롤러)를 가져온다.

**3)** **DispatcherServlet** 클래스는 **HandlerAdapter** 인터페이스의 **handle** 메서드를 호출해서 **Handler** 객체의 메서드 호출을 의뢰한다.

**4)** **HandlerAdapter** 인터페이스 구현 클래스는 **Handler** 객체에 구현된 메서드를 호출해서 요청 처리를 수행한다.

**5)** **DispatcherServlet** 클래스는 **ViewResolver** 인터페이스의 **resolveViewName** 메서드를 호출해서 **Handler** 객체에서 반환된 뷰 이름에 대응하는 **View** 인터페이스 객체를 가져온다.

**6)** **DispatcherServlet** 클래스는 **View** 인터페이스의 **render** 메서드를 호출해서 응답 데이터에 대한 렌더링을 요청한다. **View** 인터페이스의 구현 클래스는 **JSP**와 같은 템플릿 엔진을 이용해 렌더링할 데이터를 생성한다.

**7)** **DispatcherServlet** 클래스는 클라이언트에 응답을 반환한다.



위의 흐름도를 보면 **Front Controller**의 처리 내용 대부분이 **인터페이스**를 통해 실행되고 있음을 알 수 있다. 이는 스프링 MVC가 가진 특징 중 하나로 **인터페이스**를 통해 프레임워크의 기능을 **확장**할 수 있다는 것을 알 수 있다. 이 예에서는 처리 흐름을 제어하는 대표적인 **인터페이스**만 소개했지만 그 밖에도 프레임워크가 동작하는 데 사용되는 다양한 인터페이스가 준비돼 있다.





이렇게 프레임워크의 내부 동작 방식에 대해 대략적인 흐름을 파악했으면 이번에는 **Front Contoller**를 구성하는 **컴포넌트**에 대해 알아보자.



#### DispatcherServlet

**DispatcherServlet** 클래스는 **Front Controller**와 연동되는 진입점 역할을 하며 기본적인 처리 흐름을 제어하는 사령탑 역할을 한다. 위의 처리 흐름도에는 표현되지 않았지만 **DispatcherServlet**은 다음 표에서 설명하는 인터페이스와도 연동되어 프레임워크의 전체 기능을 완성한다.



**프레임워크 내부 동작에 필요한 인터페이스**

| **인터페이스명**                               | 역할                                                         |
| ---------------------------------------------- | ------------------------------------------------------------ |
| **HandlerExceptionResolver**                   | 예외 처리를 하기 위한 인터페이스. 스프링 MVC가 제공하는 기본 구현 클래스가 적용돼 있다. |
| **LocaleResolver,<br />LocaleContextResolver** | 클라이언트의 locale 정보를 확인하기 위한 인터페이스. 스프링 MVC가 제공하는 기본 구현 클래스가 적용돼 있다. |
| **ThemeResolver**                              | 클라이언트의 테마(UI 스타일)를 결정하기 위한 인터페이스. 스프링 MVC가 제공하는 기본 구현 클래스가 적용돼 있다. |
| **FlashMapManager**                            | **FlashMap**이라는 객체를 관리하기 위한 인터페이스. **FlashMap**은 PRG(Post Redirect Get) 패턴의 Redirect와 Get 사이에서 모델을 공유하기 위한 **Map** 객체다. 스프링 MVC가 제공하는 기본 구현 클래스가 적용돼 있다. |
| **RequestToViewNameTranslator**                | 핸들러가 뷰 이름과 뷰를 반환하지 않은 경우에 적용되는 뷰 이름을 해결하기 위한 인터페이스. 스프링 MVC가 제공하는 기본 구현 클래스가 적용돼 있다. |
| **HandlerInterceptor**                         | 핸들러 실행 전후에 하는 공통 처리를 구현하기 위한 인터페이스. 이 인터페이스는 애플리케이션 개발자가 구현하고 스프링 MVC에 등록해서 사용할 수 있다. |
| **MultipartResolver**                          | 멀티파트 요청을 처리하기 위한 인터페이스. 스프링 MVC에서 몇 가지 구현 클래스가 제공되고 있지만 기본적으로 적용되지 않는다. |



이러한 인터페이스를 이용한 처리 구조에 대해서는 다음 장 이후에 순서대로 설명하겠다.





#### Handler

**HandlerMapping** 인터페이스와 **HandlerAdapter** 인터페이스의 역할을 소개하기 전에 **핸들러(handler)**부터 살펴보자. 스프링 MVC에서 하는 일은 **Front Controller**가 받은 요청에 따라 필요한 처리를 수행하는 것이다.



> 프레임워크 관점에서는 **Handler**라 부르지만, 개발자가 작성하는 클래스의 관점에서는 **Controller**라고 한다.



그럼 개발자는 어떻게 **Controller**를 구현할까? 스프링 MVC에서는 다음 두 가지 방법으로 **Controller**를 구현할 수 있다.

- **@Controller** 애너테이션(`org.springframework.stereotype.Controller`)을 클래스에 지정하고 요청 처리를 수행하는 메서드에 **@RequestMapping**을 지정한 클래스를 작성한다.
- `org.springframework.web.servlet.mvc.Controller` 인터페이스의 구현 클래스를 작성하고 요청을 처리할 메서드(**handlerRequest**)를 구현한다.



**첫 번째**는 스프링 프레임워크 3.1에서 추가된 현대적인 구현 방법으로서 신규 애플리케이션을 개발하는 경우라면 이 방법으로 컨트롤러를 구현하는 것을 **권장**한다.



- **@RequestMapping을 이용한 컨트롤러의 작성 예**

```java
@Controller
public class WelcomeController {
    
    @RequestMapping("/")
    public String home(Model model) {
        model.addAtrribute("now", new Date());
        return "home";
    }
}
```





**두 번째**는 스프링 프레임워크 3.0.0 이전부터 지원되는 전통적인 구현 방법이다. 예전부터 스프링을 사용해온 개발자에게는 친숙한 구현 방법이지만 신규 애플리케이션을 구축하는 경우에는 이 방법으로 컨트롤러를 구현하는 것을 **권장하지 않는다.**



- **Controller 인터페이스를 이용한 컨트롤러 작성 예**

```java
@Controller("/")
public class WelcomeController extends AbstractController{
    
    @Override
    protected ModelAndView handleRequestInternal(
        HttpServletRequest request, HttpServletResponse response) throws Exception {
        ModelAndView mav = new ModelAndView("home");
        mav.addObject("now", new Date());
        return mav;
    }
}
```







#### HandlerMapping

**HandlerMapping** 인터페이스는 요청에 대응할 핸들러를 선택하는 역할을 수행한다.



스프링 MVC는 비슷한 기능을 하는 다양한 종류의 구현 클래스를 제공하는데, 그 중에서도 현대적인 개발 방법으로 사용되는 구현 클래스는 **RequestMappingHandlerMapping**이다. **RequestMappingHandlerMapping** 클래스는 **@RequestMapping**에 정의된 설정 정보를 바탕으로 실행할 핸들러를 선택한다.



예를 들어, 다음과 같은 컨트롤러 클래스에서는 **hello** 메서드와 **goodbye** 메서드가 핸들러로 인식된다.



- **RequestMappingHandlerMapping**에 의해 핸들러로 인식되는 컨트롤러 작성 예

```java
@Controller
public class GreetingController {
    
    @RequestMapping("/hello")
    public String hello() {
        return "hello";
    }
    
    @RequestMapping("/goodbye") 
    public String goodbye() {
        return "goodbye";
    }
}
```



핸들러로 인식된 메서드(**hello** 메서드와 **goodbye** 메서드)는 **@RequestMapping**에 지정된 요청 매핑 정보와 매핑된다. 여기서는 설명을 간단하게 하기 위해 요청 경로만 사용해서 매핑하는 예를 들지만 그 밖에도 HTTP 메서드, 요청 파라미터, 요청 헤드 등을 조합해서 매핑할 수 있다.



실제로 클라이언트에서 요청이 올 경우 **RequestMappingHandlerMapping**  클래스는 **요청 내용**(요청 경로나 HTTP 메서드 등)과 **요청 매핑 정보**를 매칭해서 실행할 핸들러를 선택한다. 









#### HandlerAdapter

**HandlerAdapter** 인터페이스는 핸들러 메서드를 호출하는 역할을 한다. 스프링 MVC는 비슷한 기능을 하는 다양한 종류의 구현 클래스를 제공하지만 **RequestMappingHandlerMapping** 클래스에 의해 선택된 핸들러 메서드를 호출할 때는 **RequestMappingHandlerAdapter** 클래스를 사용한다.



**RequestMappingHandlerAdapter** 클래스에는 핸들러 메서드에 매개변수를 전달하고 메서드의 처리 결과를 반환값으로 되돌려 보내는 것과 같은 스프링 MVC에서 상당히 중요한 기능을 수행한다. 핸들러 메서드에 매개변수를 전달할 때는 요청받은 데이터를 자바 객체로 변환하고, 입력값이 올바른지 검사(Bean Validation)하는 것까지 한번에 이뤄진다.



인수나 반환값에 지정할 수 있는 타입으로는 다양한 타입이 기본적으로 지원되지만 기본적인 동작 방식을 변경해야 하거나 기본적으로 지원되지 않는 타입을 지원해야 할 수도 있다. 이런 상황에 대응하기 위해 스프링 MVC는 핸들러 메서드 시그니처를 유연하게 정의할 수 있도록 다음과 같은 두 가지 인터페이스를 제공한다.



**핸들러 메서드 시그니처를 정의하는 데 사용되는 인터페이스**

| 인터페이스명                        | 역할                                                         |
| ----------------------------------- | ------------------------------------------------------------ |
| **HandlerMethodArgumentResolver**   | 핸들러 메서드 매개변수에 전달하는 값을 다루기 위한 인터페이스 |
| **HandlerMethodReturnValueHandler** | 핸들러 메서드에서 반환된 값을 처리하기 위한 인터페이스       |



이러한 인터페이스의 사용법에 대해서는 다음 장 이후에 설명하겠다.





#### ViewResolver

**ViewResolver** 인터페이스는 핸들러에서 반환한 뷰 이름을 보고, 이후에 사용할 **View** 인터페이스의 구현 클래스를 선택하는 역할을 한다.



스프링 MVC는 **ViewResolver**의 다양한 클래스를 제공하는데, 아래에 주요 구현 클래스를 몇 가지만 소개한다.



**ViewResolver 인터페이스의 주요 구현 클래스**

| 클래스명                         | 설명                                                         |
| -------------------------------- | ------------------------------------------------------------ |
| **InternalResourceViewResolver** | 뷰가 JSP일 때 사용하며, 가장 기본적인 **ViewResolver**다.    |
| **BeanNameViewResolver**         | DI 컨테이너에 등록된 빈의 형태로 뷰 객체를 가져올 때 사용한다. |



**ViewResolver**의 사용법에 대해서는 다음 장 이후에 설명하겠다.





#### View

**View** 인터페이스는 클라이언트에 반환하는 응답 데이터를 생성하는 역할을 한다. 스프링 MVC는 다양한 구현 클래스를 제공하는데 여기서는 **View** 인터페이스의 주요 구현 클래스만 소개하겠다.



**View 인터페이스의 주요 구현 클래스**

| 클래스명                 | 설명                                                   |
| ------------------------ | ------------------------------------------------------ |
| **InternalResourceView** | 템플릿 엔진으로 JSP를 이용할 때 사용하는 클래스        |
| **JstlView**             | 템플릿 엔진으로 JSP + JSTL을 이용할 때 사용하는 클래스 |



**View**에 대해서는 다음 장 이후에 설명하겠다.







뷰가 응답 데이터를 생성하는 역할에 대해 의문을 갖는 사람은 없겠지만 **응답 데이터를 생성하기 위해 필요한 데이터(폼 객체나 Entity 등의 자바 객체)는 과연 어떻게 해서 뷰에 연계되는 것일까?**



이 책에서는 4.2절 '첫 번째 스프링 MVC 애플리케이션'에서 언급했지만 `org.springframework.ui.Model`을 통해 데이터를 연계하는 구조로 돼 있다. 아래 그림은 뷰로 JSP + JSTL을 사용할 때의 데이터 연계의 방식을 보여준다.



![](https://docs.google.com/drawings/d/sWjOwxlvynIRlgd-4XcHsbQ/image?parent=e/2PACX-1vSgWs9-SkKDVDplyC96i2vj5TuZvbmLHvACG_NMQ3eEa1hFtJwCGbFZVRaHriS3HOeS8O0XIQ6gqqII&rev=430&h=354&w=601&ac=1)



**1)** **Front Controller**는 **Model**을 생성한다.

**2)** **Front Controller**는 **Model**을 인수에 지정하고 **Handler Method**를 실행한다.

**3)** **Handler**는 응답 데이터를 생성하는 데 필요한 데이터(**Form** 객체나 **Entity** 등의 자바 객체)를 **Model**에 추가한다.

**4)** **Front Controller**는 **Handler**에서 반환한 뷰 이름에 대응하는 뷰 클래스(JSP + JSTL을 사용할 때는 **JstlView**)에 대해 그리기를 의뢰한다.

**5)** **JstlView** 클래스는 **Model**에 저장된 데이터를 **JSP**에서 접근할 수 있는 영역(**HttpServletRequest**)으로 export한다.

**6)** **JstlView** 클래스는 **JSP**로 이동한다.

**7)** **JSP**는 스프링과 JSTL 태그 라이브러리와 EL을 통해 **HttpServletRequest**에 export된 데이터를 참조한다.







### 4.3.3 DI 컨테이너와의 연계

스프링 MVC는 DI 컨테이너에서 관리되는 객체를 사용해 클라이언트에서 받은 요청을 처리하는 구조로 돼 있다. 여기서는 스프링 MVC가 어떻게 DI 컨테이너와 연계하는지 설명한다.



#### ApplicationContext 구성

스프링 MVC에서는 다음 두 가지 **ApplicationContext**를 사용한다.

- **웹 애플리케이션용 ApplicationContext**
- **DispatcherServlet용 ApplicationContext**



첫 번째는 **웹 애플리케이션 전체에서 하나**, 
두 번째는 **DispatcherServlet마다 인스턴스가 생성**되어 다음(그림)과 같이 구성된다. 이번 장의 4.2절 '첫 번째 스프링 MVC 애플리케이션'에서는 이와 같이 구성했다.





![](https://docs.google.com/drawings/d/s2YSYkPTgwui3mgxiuWNRQw/image?parent=e/2PACX-1vSgWs9-SkKDVDplyC96i2vj5TuZvbmLHvACG_NMQ3eEa1hFtJwCGbFZVRaHriS3HOeS8O0XIQ6gqqII&rev=448&h=545&w=601&ac=1)



**1)** 웹 애플리케이션용 **ApplicationContext**는 웹 애플리케이션 전체에서 사용하는 **컴포넌트(Service, Repository, DataSource, ORM 등)**의 빈을 등록한다. 기본적으로 스프링 MVC용 컴포넌트는 여기에 등록하지 않는다.



**2)** **DispatcherServlet**용 **ApplicationContext**는 스프링 MVC **Front Controller**의 구성 **컴포넌트(HandlerMapping, HandlerAdpater, ViewResolver 등)**와 **컨트롤러의 빈**을 등록한다.



이 두 가지 **ApplicationContext**는 부모 자식 관계로 돼 있어서 <u>자식에서 부모의 **ApplicationContext**에 등록돼 있는 빈을 사용</u>할 수 있다.



**DispatcherServlet**을 여러 개 정의한 경우 다음과 같이 구성된다.



![](https://docs.google.com/drawings/d/smFNJbf3-ZkVR4RCWbOtE_g/image?parent=e/2PACX-1vSgWs9-SkKDVDplyC96i2vj5TuZvbmLHvACG_NMQ3eEa1hFtJwCGbFZVRaHriS3HOeS8O0XIQ6gqqII&rev=495&h=462&w=601&ac=1)

**DispatcherServlet**용 **ApplicationContext**는 각각 독립돼 있기 때문에 다른 **DispatcherServlet**용 **ApplicationContext**에 등록돼 있는 빈은 참조할 수 없다.









#### ApplicationContext LifeCycle

스프링 MVC에서 취급하는 **ApplicationContext**의 **LifeCycle**은 다음 세 가지 단계로 나뉜다.



**ApplicationContext의 LifeCycle 세 단계**

| 단계            | 설명                                                         |
| --------------- | ------------------------------------------------------------ |
| **초기화 단계** | 웹 애플리케이션용 ApplicationContext와 DispatcherServlet용 ApplicationContext를 생성하는 단계. 이 단계는 Servlet Container를 기동할 때 진행된다. |
| **사용 단계**   | ApplicationContext에서 빈을 이용하는 단계                    |
| **파기 단계**   | 웹 애플리케이션용 ApplicationContext와 DispatcherServlet용 ApplicationContext를 파기하는 단계. 이 단계는 Servlet Container를 중지할 때 진행된다. |



구체적으로는 다음과 같은 시퀀스에서 **ApplicationContext**의 **LifeCycle**이 관리된다. **ApplicationContext**의 **LifeCycle** 관리는 스프링이 대신 처리해주기 때문에 애플리케이션 개발자가 직접 구현할 필요는 없다.

