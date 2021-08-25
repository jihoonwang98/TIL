## 스프링 MVC 아키텍처 분석



스프링 MVC 아키텍처의 주요 컴포넌트는 아래의 그림과 같다.



![](https://docs.google.com/drawings/d/sTKcN5bMiQmjj1fxVIT5DPw/image?parent=e/2PACX-1vRVvbIKomKLDTKGV_Zpd2jnSflB7IGvZt3_68Ps6WLyfUO3_b16JGyGje2BnfNAFQnyCXgohLQOH9QO&rev=497&h=333&w=601&ac=1)





아래의 예제 코드가 어떤 단계를 거쳐 실행되는지 살펴보자.



**BasicMovelViewController**

```java
@Controller
public class BasicModelViewController {
    @RequestMapping("/welcome-model-view")
    public ModelAndView welcome(ModelMap model) {
        model.put("name", "XYZ");
        return new ModelAndView("welcome-model-view", model);
    }
}
```





**welcome-model-view.jsp**

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" pageEncoding="UTF-8" %>
<html>
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
    <title>Title</title>
</head>
<body>
    Welcome ${name}! This is coming from a model-view - a JSP
</body>
</html>
```



**webmvc-context.xml 중 일부**

```xml
<!-- 뷰 prefix, suffix 설정 -->
<beans:bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    <beans:property name="prefix" value="/WEB-INF/views/"/>
    <beans:property name="suffix" value=".jsp"/>
</beans:bean>
```





**1. **브라우저는 특정 URL(`http://localhost:8080/welcome-model-view`)에 요청을 보낸다.

​    **DispatcherServlet**은 모든 요청을 처리하는 *FrontController*이므로 **DispatcherServlet**이 이 요청을 받는다.



**2.** **DispatcherServlet**은 URI(`/welcome-model-view`)를 보 고 이를 처리할 올바른 *Controller*를 식별해야 한다.

​    올바른 *Controller*를 찾기 위해 **HandlerMapping**과 통신한다.



**3.** **HandlerMapping**은 요청을 처리하는 특정 *handler method*(`BasicModelViewController의 welcome 메서드`)를 반환한다.



**4.** **DispatcherServlet**은 해당 *handler method*(`public ModelAndView welcome(ModelMap model))`)를 호출한다.



**5.** *handler method*는 *Model*과 *View*를 반환한다. 예제에서는 **ModelAndView** 객체가 반환된다.



**6.** **DispatcherServlet**은 논리적 뷰 이름(ModelAndView에서 가져옴. 예제에서는 `welcome-model-view`)을 반환받고 이에 대응되는 물리적 뷰 이름을 알아 내야 한다.

   사용 가능한 **ViewResolver**가 있는지 확인하고 설정된 **ViewResolver**(`org.springframework.web.servlet.view.InternalResourceViewResolver`)를 찾는다. 

   **ViewResolver**를 호출해 논리적 뷰 이름(`welcome-model-view`)을 넘겨준다.



**7.** **ViewResolver**는 논리적 뷰 이름을 물리적 뷰 이름에 매핑하는 로직을 실행한다.

​    예에서 `welcome-model-view`는 `/WEB-INF/view/welcome-model-view.jsp`로 변환된다.



**8.** **DispatcherServlet**은 *View*를 실행한다. 또한 *View*에서 *Model*을 사용할 수 있게 한다.



**9.** *View*는 **DispatcherServlet**으로 보내질 내용을 반환한다.



**10.** **DispatcherServlet**은 응답을 브라우저로 다시 보낸다.































