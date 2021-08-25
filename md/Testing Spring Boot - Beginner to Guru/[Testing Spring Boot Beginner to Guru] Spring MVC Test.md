# [Testing Spring Boot: Beginner to Guru] Spring MVC Test



### Spring MVC Test

- Spring MVC Test는 Controller들의 상호작용을 테스트하는 프레임워크다.
- Web 환경을 Mocking하기 위해 Servlet API Mock 객체들을 제공한다.
  - **`MockHttpServletRequest`**: Java의 `HttpServletRequest`의 Mock 객체
  - **`MockHttpServletResponse`**: Java의 `HttpServletResponse`의 Mock 객체
- **DispatcherServlet**: 모든 요청들은 Spring MVC의 DispatcherServlet을 통해 라우팅된다.





### Spring MVC Test Configuration Modes

- Standalone Setup
  - 매우 가볍다 - 유닛 테스트하기 좋다.
  - 한 번에 한 컨트롤러만 테스트한다.
  - Controller의 request들과 response들을 테스트하는 것을 허용한다.
- WebAppContext Setup
  - Spring Configuration의 larger context를 불러온다.
  - 한 번에 여러 개의 컨트롤러를 테스트할 수 있다 - Configuration에 따라 다름
  - Application config를 테스트하는 것을 허용한다.





### Static Imports

- Spring MVC Test는 몇몇 static import를 통해 "fluent"한 API를 사용하고 있다.
- **`MockMvcRequestBuilders.*`**: request를 Build 해준다.
- **`MockMvcResultMatchers.*`**: response에 대한 assertion을 만들어 준다.
- **`MockMvcBuilders.*`**: MockMvc를 설정하고 build해준다.





### Important Differences from Container

- Spring MVC Test는 Servlet Container를 실제로 띄우는 것은 아니다.
- 어떠한 Network Request도 만들어지지 않는다. (i.e. to port 80, or 8080)
- HTML이 generated 되지 않는다. 그러므로 template들도 executed 되지 않는다. (JSP, Thymeleaf, etc)
- You can test the view (template) requested, or redirected to
  - But cannot test expected HTML to be rendered
- Spring은 Container를 띄우고 테스트하는 것도 지원한다.

















