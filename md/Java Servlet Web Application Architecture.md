# Java Servlet Web Application Architecture :happy:



이번 포스팅에서는 **Web Application** 개발을 위한 다양한 **Architecture**들을 살펴본다. :mag:



- **Model1 Architecture**
- **Model2 또는 MVC Architecture**
- **Front Controller가 있는 Model2**







## Model1 Architecture :sun_with_face:

**모델1 아키텍처**는 자바 기반 웹 애플리케이션을 개발할 때 사용되는 초기(~~구식~~) 아키텍처 스타일 중 하나다.



**특징**

- **JSP 페이지**는 브라우저의 요청을 직접 처리
- **JSP 페이지**는 간단한 자바 빈을 포함하는 **모델**을 사용
- 일부 애플리케이션에서는 **JSP**가 DB의 쿼리까지도 수행함 :dizzy_face:



다음 그림은 전형적인 **모델1 아키텍처**를 나타낸다.



![](https://docs.google.com/drawings/d/s6PBPMGHYs2BrOXquUpekiw/image?parent=e/2PACX-1vRVvbIKomKLDTKGV_Zpd2jnSflB7IGvZt3_68Ps6WLyfUO3_b16JGyGje2BnfNAFQnyCXgohLQOH9QO&rev=59&h=443&w=407&ac=1)





위 아키텍처의 **단점**

- **어려운 관점 분리**: **JSP**는 데이터를 <u>검색</u>하고 데이터를 <u>표시</u>한다. 또한, 다음에 표시할 <u>페이지(플로우)</u>를 결정하고 때로는 <u>비즈니스 로직</u>까지 책임졌다.
- **복잡한 JSP**: **JSP**가 많은 로직을 처리했기 때문에 거대해지고 유지 관리하기가 어려웠다.









## Model2 Architecture :clown_face:

모델2 아키텍처는 여러 책임이 있는 복잡한 JSP와 관련된 복잡성을 해결하기 위해 도입됐다. MVC 아키텍처 스타일이라고도 한다.





다음 그림은 일반적인 **모델2 아키텍처**를 나타낸다. 



![](https://docs.google.com/drawings/d/sESODwjA0pH0roqdXXlV9zA/image?parent=e/2PACX-1vRVvbIKomKLDTKGV_Zpd2jnSflB7IGvZt3_68Ps6WLyfUO3_b16JGyGje2BnfNAFQnyCXgohLQOH9QO&rev=65&h=269&w=407&ac=1)





**모델2 아키텍처**는 **모델**, **뷰**와 **컨트롤러** 간의 역할을 명확하게 구분해 유지 관리가 좀 더 쉬운 애플리케이션이 만들어진다.



모델2 아키텍처의 **특징**

- **모델**: 뷰를 생성하는 데 사용할 데이터를 나타냄

- **뷰**: 모델을 사용해 화면을 렌더링

- **컨트롤러**: 플로우를 제어함. 브라우저에서 요청을 가져와 모델을 채우고 뷰로 전환한다.

  위 그림에서의 Servlet1과 Servlet2가 컨트롤러에 해당한다.







## Model2 Front Controller Architecture :clap:

모델2 아키텍처의 기본 버전에서 **브라우저의 요청을 전담**하는 서블릿(**Front Controller**)를 만든 아키텍처다.

또한, 여러 비즈니스 시나리오에서 요청을 처리하기 전에 몇 가지 **공통적인 작업**을 수행해야 할 때 이를 **Front Controller에게 맡길 수 있다.**





다음 그림은 전형적인 **모델2 Front Controller** 아키텍처를 나타낸다.



![](https://docs.google.com/drawings/d/ssQjGscHiAhQ2xl5I1Sr_TQ/image?parent=e/2PACX-1vRVvbIKomKLDTKGV_Zpd2jnSflB7IGvZt3_68Ps6WLyfUO3_b16JGyGje2BnfNAFQnyCXgohLQOH9QO&rev=85&h=338&w=407&ac=1)





**Front Controller**가 하는일

- 어떤 **Controller**가 요청을 실행할지 결정한다.
- 렌더링할 **뷰**를 결정한다.
- **Controller**들이 해야 하는 **공통 작업**을 대신 수행한다.
- 스프링 MVC는 **Front Controller**에서 MVC 패턴을 사용한다. **Front Controller**를 **DispatcherServlet**이라고 한다.



> 보다시피 **Front Controller**는 많은 일을 하므로 **Front Controller**에 필요한 것 이상을 넣지 않도록 주의해야 한다. 적절한 경우 **Filter** 사용을 고려해야 한다.