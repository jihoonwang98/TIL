# [Chapter 10] 스프링 MVC 프레임워크 동작 방식



## 1. 스프링 MVC 핵심 구성 요소

스프링 MVC의 핵심 구성 요소와 각 요소 간의 관계는 아래와 같이 정리할 수 있다.

![](https://docs.google.com/drawings/d/e/2PACX-1vRoMPmjUHxgy6wncGh_wN3rHTKoON7M81I8sA3rGlOXrpghPbR5GCiH4hhM8IzhxpEy--HUB3XnaP8D/pub?w=1167&h=661)



회색 배경을 가진 구성 요소는 개발자가 직접 구현해야 하는 요소이다.



그림의 중앙에 위치한 `DispatcherServlet`은 모든 연결을 담당한다.

웹 브라우저로부터 요청이 들어오면 `DispatcherServlet`은 그 요청을 처리하기 위한 컨트롤러 객체를 검색한다.

이때 `DispatcherServlet`은 직접 컨트롤러를 검색하지 않고 `HandlerMapping`이라는 빈 객체에게 컨트롤러 검색을 요청한다.

`HandlerMapping`은 클라이언트의 요청 경로를 이용해서 이를 처리할 컨트롤러 빈 객체를 `DispatcherServlet`에 전달한다.



컨트롤러 객체를 `DispatcherServlet`이 전달받았다고 해서 바로 컨트롤러 객체의 메서드를 실행할 수 있는 것은 아니다.

`DispatcherServlet`은 `@Controller` 애너테이션을 이용해서 구현한 컨트롤러뿐만 아니라 

스프링 2.5까지 주로 사용됐던 `Controller` 인터페이스를 구현한 컨트롤러, 

그리고 특수 목적으로 사용되는 `HttpRequestHandler` 인터페이스를 구현한 클래스를

동일한 방식으로 실행할 수 있도록 만들어졌다.

`@Controller`, `Controller` 인터페이스, `HttpRequestHandler` 인터페이스를 동일한 방식으로 처리하기 위해 중간에 사용되는 것이 바로 `HandlerAdapter` 빈이다.



`DispatcherServlet`은 `HandlerMapping`이 찾아준 컨트롤러 객체를 처리할 수 있는 `HandlerAdapter` 빈에게 요청 처리를 위임한다.

`HandlerAdapter`는 컨트롤러의 알맞은 메서드를 호출하여 요청을 처리하고 그 결과를 `DispatcherServlet`에게 리턴한다.

이때 `HandlerAdapter`는 컨트롤러의 처리 결과를 `ModelAndView`라는 객체로 변환해서 `DispatcherServlet`에 리턴한다.



`HandlerAdapter`로부터 컨트롤러의 요청 처리 결과를 `ModelAndView`로 받으면 

`DispatcherServlet`은 결과를 보여줄 뷰를 찾기 위해 `ViewResolver` 빈 객체를 사용한다.

`ModelAndView`는 컨트롤러가 리턴한 뷰 이름을 담고 있는데 

`ViewResolver`는 이 뷰 이름에 해당하는 `View` 객체를 찾거나 생성해서 리턴한다.

응답을 생성하기 위해 JSP를 사용하는 `ViewResolver`는 매번 새로운 `View` 객체를 생성해서 `DispatcherServlet`에 리턴한다.



`DispatcherServlet`은 `ViewResolver`가 리턴한 `View` 객체에게 응답 결과 생성을 요청한다.

JSP를 사용하는 경우 `View` 객체는 JSP를 실행함으로써 웹 브라우저에 전송할 응답 결과를 생성하고 이로써 모든 과정이 끝이 난다.





### 1.1 컨트롤러와 핸들러

클라이언트의 요청을 실제로 처리하는 것은 컨트롤러이고, `DispatcherServlet`은 클라이언트의 요청을 전달받는 창구 역할을 한다.

앞서 설명했듯 `DispatcherServlet`은 클라이언트의 요청을 처리할 컨트롤러를 찾기 위해 `HandlerMapping`을 사용한다.

컨트롤러를 찾아주는 객체는 `ControllerMapping` 타입이어야 할 것 같은데 실제로는 `HandlerMapping`이다. 왜 `HandlerMapping`일까?



스프링 MVC는 웹 요청을 처리할 수 있는 범용 프레임워크다.

이 책에서는 `@Controller` 애너테이션을 붙인 클래스를 이용해서 클라이언트의 요청을 처리하지만,

원한다면 자신이 직접 만든 클래스를 이용해서 클라이언트의 요청을 처리할 수도 있다.

즉, `DispatcherServlet` 입장에서는 클라이언트의 요청을 처리하는 객체의 타입이 반드시 `@Controller`를 적용한 클래스일 필요는 없다.



이런 이유로 스프링 MVC는 웹 요청을 실제로 처리하는 객체를 **핸들러(Handler)**라고 표현하고 있으며,

`@Controller` 적용 객체나 `Controller` 인터페이스를 구현한 객체는 모두 스프링 MVC 입장에서는 핸들러가 된다.

따라서 특정 요청 경로를 처리해주는 핸들러를 찾아주는 객체를 `HandlerMapping`이라고 부른다.



`DispatcherServlet`은 핸들러 객체의 실제 타입에 상관없이 실행 결과를 `ModelAndView`라는 타입으로만 받을 수 있으면 된다.

그런데 핸들러의 실제 구현 타입에 따라 `ModelAndView`를 리턴하는 객체도 있고, 그렇지 않은 객체도 있다. 

따라서 핸들러의 처리 결과를 `ModelAndView`로 변환해주는 객체가 필요하면 `HandlerAdapter`가 이 변환을 처리해준다.



핸들러 객체의 실제 타입마다 그에 알맞은 `HandlerMapping`과 `HandlerAdapter`가 존재하기 때문에,

사용할 핸들러의 종류에 따라 해당 `HandlerMapping`과 `HandlerAdapter`를 스프링 빈으로 등록해야 한다.

물론 스프링이 제공하는 설정 기능을 사용하면 이 두 종류의 빈을 직접 등록하지 않아도 된다.







# 2. `DispatcherServlet`과 스프링 컨테이너

9장의 `web.xml` 파일을 보면 다음과 같이 `DispatcherServlet`의 `contextConfiguration` 초기화 파라미터를 이용해서 스프링 설정 클래스 목록을 전달했다.

```xml
<servlet>
	<servlet-name>dispatcher</servlet-name>
    <servlet-class>
        org.springframework.web.servlet.DispatcherServlet
    </servlet-class>
    <init-param>
    	<param-name>contextClass</param-name>
        <param-value>org.springframework.web.context.support.AnnotationConfigWebApplicationContext</param-value>
    </init-param>
    <init-param>
    	<param-name>contextConfigLocation</param-name>
        <param-value>
        	config.MvcConfig
            config.ControllerConfig
        </param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
</servlet>
```

`DispatcherServlet`은 전달받은 설정 파일을 이용해서 스프링 컨테이너를 생성하는데 앞에서 언급한

`HandlerMapping`, `HandlerAdapter`, 컨트롤러, `ViewResolver` 등의 빈은 아래 그림처럼

`DispatcherServlet`이 생성한 스프링 컨테이너에서 구한다. 

따라서 `DispatcherServlet`이 사용하는 설정 파일에 이들 빈에 대한 정의가 포함되어 있어야 한다.





![](https://docs.google.com/drawings/d/e/2PACX-1vRix_fIiW9jgqco_S_UgpqiW03LroFWGr4CaApXgtRRNFKpEKUr7_3aFTE7jsmyB3MXaF-ohzOsvmKx/pub?w=912&h=635)





## 3. `@Controller`를 위한 HandlerMapping과 HandlerAdapter

`@Controller` 적용 객체는 `DispatcherServlet` 입장에서 보면 한 종류의 핸들러 객체이다.

`DispatcherServlet`은 웹 브라우저의 요청을 처리할 핸들러 객체를 찾기 위해 `HandlerMapping`을 사용하고,

핸들러를 실행하기 위해 `HandlerAdapter`를 사용한다.

`DispatcherServlet`은 스프링 컨테이너에서 `HandlerMapping`과 `HandlerAdapter` 타입의 빈을 사용하므로

핸들러에 알맞은 `HandlerMapping` 빈과 `HandlerAdapter` 빈이 스프링 설정에 등록되어 있어야 한다.



그런데 보통 이런 빈들을 직접 등록하지 않고 `@EnableWebMvc` 애너테이션을 이용한다.

```java
@Configuration
@EnableWebMvc
public class MvcConfig {
	...
}
```

위 설정은 매우 다양한 스프링 빈 설정을 추가해준다.

이 설정을 사용하지 않고 설정 코드를 직접 작성하려면 백 여 줄에 가까운 코드를 입력해야 한다.

이 태그가 빈으로 추가해주는 클래스 중에는 `@Controller` 타입의 핸들러 객체를 처리하기 위한 다음의 두 클래스도 포함되어 있다.

- `org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping`
- `org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter`



`RequestMappingHandlerMapping` 애너테이션은 `@Controller` 애너테이션이 적용된 객체의 요청 매핑 애너테이션(`@RequestMapping`) 값을 이용해서 웹 브라우저의 요청을 처리할 컨트롤러 빈을 찾는다.



`RequestMappingHandlerAdapter` 애너테이션은 컨트롤러의 메서드를 알맞게 실행하고 그 결과를 `ModelAndView` 객체로 변환해서 `DispatcherServlet`에 리턴한다.



아래의 코드를 보자.

```java
@Controller
public class HelloController {
	
    @RequestMapping("/hello")
    public String hello(Model model, @RequestParam(value = "name", required=false) String name) {
        model.addAttribute("greeting", "안녕하세요, " + name);
        return "hello";
    }
}
```



`RequestMappingHandlerAdapter` 클래스는 "/hello" 요청 경로에 대해 `hello()` 메서드를 호출한다.

이때 `Model` 객체를 생성해서 첫 번째 파라미터로 전달한다.

비슷하게 이름이 "name"인 HTTP 요청 파라미터의 값을 두 번째 파라미터로 전달한다.



`RequestMappingHandlerAdapter`는 컨트롤러 메서드 결과 값이 String 타입이면 해당 값을 뷰 이름으로 갖는 `ModelAndView` 객체를 생성해서 `DispatcherServlet`에 리턴한다.

이때 첫 번째 파라미터로 전달한 `Model` 객체에 보관된 값도 `ModelAndView`에 함께 전달한다.







## 4. WebMvcConfigurer 인터페이스와 설정

`@EnableWebMvc` 애너테이션을 사용하면 `@Controller` 애너테이션을 붙인 컨트롤러를 위한 설정을 생성한다.

또한 `@EnableWebMvc` 애너테이션을 사용하면 `WebMvcConfigurer` 타입의 빈을 이용해서 MVC 설정을 추가로 생성한다.

```java
@Configuration
@EnableWebMvc
public class MvcConfig implements WebMvcConfigurer {
	
    @Override
    public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
        configurer.enable();
    }
    
    @Override
    public void configureViewResolvers(ViewResolverRegistry registry) {
        registry.jsp("/WEB-INF/view", ".jsp");
    }
}
```



여기서 설정 클래스는 `WebMvcConfigurer` 인터페이스를 상속하고 있다.

`@Configuration` 애너테이션을 붙인 클래스 역시 컨테이너에 빈으로 등록하므로 

`MvcConfig` 클래스는 `WebMvcConfigurer` 타입의 빈이 된다.



`@EnableWebMvc` 애너테이션을 사용하면 `WebMvcConfigurer` 타입인 빈 객체의 메서드를 호출해서 MVC 설정을 추가한다.







## 5. JSP를 위한 ViewResolver

컨트롤러 처리 결과를 JSP를 이용해서 생성하기 위해 다음 설정을 사용한다.

```java
@Configuration
@EnableWebMvc
public class MvcConfig implements WebMvcConfigurer {
	
	@Override
    public void configureViewResolvers(ViewResolverRegistry registry) {
        registry.jsp("/WEB-INF/view", ".jsp");
    }
}
```



`WebMvcConfigurer` 인터페이스에 정의된 `configureViewResolvers()` 메서드는 `ViewResolverRegistry` 타입의 `registry` 파라미터를 갖는다.

`ViewResolverRegistry#jsp()` 메서드를 사용하면 JSP를 위한 `ViewResolver`를 설정할 수 있다.



위 설정은 `o.s.w.servlet.view.InternalResourceViewResolver` 클래스를 이용해서 다음 설정과 같은 빈을 등록한다.

```java
@Bean
public ViewResolver viewResolver() {
	InternalResourceViewResolver vr = new InternalResourceViewResolver();
	vr.setPrefix("/WEB-INF/view/");
	vr.setSuffix(".jsp");
    return vr;
}
```



컨트롤러의 실행 결과를 받은 `DispatcherServlet`은 `ViewResolver`에게 뷰 이름에 해당하는 `View` 객체를 요청한다.

이때 `InternalResourceViewResolver`는 "prefix + 뷰이름 + suffix"에 해당하는 경로를 뷰 코드로 사용하는 `InternalResourceView` 타입의 `View` 객체를 리턴한다.

예를 들어 뷰 이름이 "hello"라면, "/WEB-INF/view/hello.jsp" 경로를 뷰 코드로 사용하는 `InternalResourceView` 객체를 리턴한다.

`DispatcherServlet`이 `InternalResourceView` 객체에 응답 생성을 요청하면 `InternalResourceView` 객체는 경로에 지정한 JSP 코드를 실행해서 응답 결과를 생성한다.



`DispatcherServlet`은 컨트롤러의 실행 결과를 `HandlerAdapter`를 통해서 `ModelAndView` 형태로 받는다고 했다.

`Model`에 담긴 값은 `View` 객체에 `Map` 형식으로 전달된다.

예를 들어 `HelloController` 클래스는 다음과 같이 `Model`에 "greeting" 속성을 설정했다.

```java
@Controller
public class HelloController {
	
    @RequestMapping("/hello")
    public String hello(Model model, @RequestParam(value = "name", required=false) String name) {
        model.addAttribute("greeting", "안녕하세요, " + name);
        return "hello";
    }
}
```



이 경우 `DispatcherServlet`은 `View` 객체에 응답 생성을 요청할 때 greeting 키를 갖는 `Map` 객체를 `View` 객체에 전달한다.

`View` 객체는 전달받은 `Map` 객체에 담긴 값을 이용해서 알맞은 응답 결과를 출력한다.

`InternalResourceView`는 `Map` 객체에 담겨 있는 키 값을 `request.setAttribute()`를 이용해서 `request`의 속성에 저장한다.

그런 뒤 해당 경로의 JSP를 실행한다.



결과적으로 컨트롤러에서 지정한 `Model` 속성은 `request` 객체 속성으로 JSP에 전달되기 때문에 

JSP는 다음과 같이 모델에 지정한 속성 이름을 사용해서 값을 사용할 수 있게 된다.

```java
인사말: ${greeting}
```





## 6. 디폴트 핸들러와 HandlerMapping의 우선순위

```xml
<servlet>
	<servlet-name>dispatcher</servlet-name>
    <servlet-class>
    	org.springframework.web.servlet.DispatcherServlet
    </servlet-class>
    ... 생략
</servlet>

<servlet-mapping>
	<servlet-name>dispatcher</servlet-name>
    <url-pattern>/</url-pattern>
</servlet-mapping>
```



위와 같이 매핑 경로가 '/'인 경우, 

`.jsp`로 끝나는 요청을 제외한 모든 요청을 `DispatcherServlet`이 처리한다.

즉 `/index.html`이나 `/css/bootstrap.css`와 같이 확장자가 `.jsp`가 아닌 모든 요청을 `DispatcherServlet`이 처리하게 된다.



그런데 `@EnableWebMvc` 애너테이션이 등록하는 `HandlerMapping`은 `@Controller` 애너테이션을 적용한 빈 객체가 처리할 수 있는 요청 경로만 대응할 수 있다.

예를 들어 등록된 컨트롤러가 한 개이고 그 컨트롤러가 `@GetMapping("/hello")` 설정을 사용하면,  `/hello` 경로만 처리할 수 있게 된다.

따라서 `/index.html`이나 `/css/bootstrap.css` 같은 요청을 처리할 수 있는 컨트롤러 객체를 찾지 못해 `DispatcherServlet`은 404 응답을 전송한다.



`/index.html`이나 `/css/bootstrap.css` 같은 경로를 처리하기 위해 컨트롤러를 직접 구현할 수도 있지만, 

그보다는 `WebMvcConfigurer`의 `configureDefaultServletHandling()` 메서드를 사용하는 것이 편리하다.

```java
@Configuration
@EnableWebMvc
public class MvcConfig implements WebMvcConfigurer {

    @Override
    public void configureDefaultServletHandling(DefaultServletHanlderConfigurer configurer) {
        configurer.enable();
    }
}
```



위 설정에서 `DefaultServletHandlerConfigurer#enable()` 메서드는 다음의 두 빈 객체를 추가한다.

- `DefaultServletHttpRequestHandler`
- `SimpleUrlHandlerMapping`



`DefaultServletHttpRequestHandler`는 클라이언트의 모든 요청을 WAS가 제공하는 `DefaultServlet`에 전달한다.

예를 들어 `/index.html`에 대한 처리를 `DefaultServletHttpRequestHandler`에 요청하면 이 요청을 다시 `DefaultServlet`에 전달해서 처리하도록 한다.

그리고 `SimpleUrlHandlerMapping`을 이용해서 모든 경로(/**)를 `DefaultServletHttpRequestHandler`를 이용해서 처리하도록 설정한다.



`@EnableWebMvc` 애너테이션이 등록하는 `RequestMappingHandlerMapping`의 적용 우선순위가 `DefaultServletHandlerConfigurer#enable()` 메서드가 등록하는 `SimpleUrlHandlerMapping`의 우선순위보다 높다.

때문에 브라우저의 요청이 들어오면 `DispatcherServlet`은 다음과 같은 방식으로 요청을 처리한다.

1. `RequestMappingHandlerMapping`을 사용해서 요청을 처리할 핸들러를 검색한다.
   - 존재하면 해당 컨트롤러를 이용해서 요청을 처리한다.
2. 존재하지 않으면 `SimpleUrlHandlerMapping`을 사용해서 요청을 처리할 핸들러를 검색한다.
   - `DefaultServletHandlerConfigurer#enable()` 메서드가 등록한 `SimpleUrlHandlerMapping`은 "/**" 경로에 대해 `DefaultServletHttpRequestHandler`를 리턴한다.
   - `DispatcherServlet`은 `DefaultServletHttpRequestHandler`에 처리를 요청한다.
   - `DefaultServletHttpRequestHandler`는 디폴트 서블릿에 처리를 위임한다.





