# MockMvc Test



> https://ktko.tistory.com/entry/%EC%8A%A4%ED%94%84%EB%A7%81Spring-MockMvc-%ED%85%8C%EC%8A%A4%ED%8A%B8





이번 포스팅에는 MVC 컨트롤러를 테스트하는 방법을 설명한다. 컨트롤러에 대한 테스트를 이야기하면 항상 나오는 질문이 있는데, 그것은 바로 컨트롤러에 대한 단위 테스트가 필요할까?라는 것이다. 컨트롤러의 주요 역할은 요청 경로와 처리 내용의 매핑, 입력값 검사, 요청한 데이터의 취득, 비지니스 로직 호출, 다음 이동 화면의 제어와 같은 기능을 하기 때문에, 정작 컨트롤러 자체에는 단위 테스트가필요할 만한 비지니스 로직이 존재하지 않는다. 우선 요청 경로와 처리 내용의 매핑이나 요청 데이터의 취득, 입력값 검사와 같은 부분은 스프링 MVC의 프레임워크 기능을 사용해야만 그 처리 결과가 제대로 됐는지 검증할 수 있다. 결국 이런 상황을 고려한다면 컨트롤러의 테스트는 일반적인 단위 테스트의 형태가 아니라, 스프링 MVC의 프레임워크 기능까지 통합된 상태인 통합 테스트의 관점으로 보는 것이 맞다. 



**Dependency**

```xml
<dependency>
    <groupId>org.mockito</groupId>
    <artifactId>mockito-all</artifactId>
    <version>1.9.5</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.12</version>
    <scope>test</scope>
</dependency> 
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-test</artifactId>
    <version>${org.springframework-version}</version>
</dependency>


출처: https://ktko.tistory.com/entry/스프링Spring-MockMvc-테스트 [KTKO 개발 블로그와 여행 일기]
```



**MockMvc 란 ?**

**MockMvc**는 <u>웹 애플리케이션을 애플리케이션 서버에 배포하지 않고도</u> 스프링 MVC의 동작을 재현할 수 있는 클래스이다.





**요청 데이터 설정**

**요청 데이터를 설정**할 때는 org.springframework.test.web.servlet.request.**MockMvcRequestBuilders**를 import 해주어야 한다. **파일**과 관련된 것은 org.springframework.test.web.servlet.request.**MockMultipartHttpServletRequestBuilder**을 import 해주어야 한다.





**MockMvcRequestBuilders 주요 메서드** 

| 메서드명         | 설명                                                         |
| ---------------- | ------------------------------------------------------------ |
| param / params   | 요청 파라미터를 설정한다.                                    |
| header / headers | 요청 헤더를 설정한다. contentType이나 accept와 같은 특정 헤더를 설정할 수 있는 메서드도 제공한다. |
| cookie           | 쿠키를 설정한다.                                             |
| content          | 요청 본문을 설정한다.                                        |
| requestAttr      | 요청 스코프에 객체를 설정한다.                               |
| flashAttr        | 플래시 스코프에 객체를 설정한다.                             |
| sessionAttr      | 세션 스코프에 객체를 설정한다.                               |



**MockMultipartHttpServletRequestBuilder 주요 메서드**

| 메서드명 | 설명                    |
| -------- | ----------------------- |
| file     | 업로드 파일을 지정한다. |











**실행 결과의 검증**

실행 결과를 검증할 때는 org.springframework.test.web.servlet.**ResultActions**를 이용하여 **andExpect** 메서드를 사용한다. 

**andExpect** 메서드의 인수에는 실행 결과를 검증하는 org.springframework.test.web.servlet.**ResultMatcher**를 지정한다. 

스프링 테스트는 **mockMvcResultMatchers**의 팩토리 메서드를 사용해 다양한 **ResultMatcher**를 제공한다.



| 메서드명      | 설명                                                         |
| ------------- | ------------------------------------------------------------ |
| status        | HTTP 상태 코드를 검증한다.                                   |
| header        | 응답 헤더의 상태를 검증한다.                                 |
| cookie        | 쿠키 상태를 검증한다.                                        |
| content       | 응답 본문 내용을 검증한다. jsonPath나 xpath와 같은 특정 콘텐츠를 위한 메서드도 제공한다. |
| view          | 컨트롤러가 반환한 뷰 이름을 검증한다.                        |
| forwardedUrl  | 이동 대상 경로를 검증한다. 패턴으로 검증할 때는 forwardedUrlPattern 메서드를 사용한다. |
| redirectedUrl | 리다이렉트 대상의 경로 또는 URL을 검증한다. 패턴으로 검증할 때는 redirectedUrlPattern 메서드를 사용한다. |
| model         | 스프링 MVC 모델 상태를 검증한다.                             |
| flash         | 플래시 스코프의 상태를 검증한다.                             |
| request       | 서블릿 3.부터 지원되는 비동기 처리의 상태나 요청 스코프의 상태, 세션 스코프의 상태를 검증한다. |











**실행 결과 출력**

실행 결과를 로그로 출력할 때는 org.springframework.test.web.servlet.**ResultActions**의 **andDo** 메서드를 이용한다. 

**andDo** 메서드의 인수에는 실행 결과를 처리할 수 있는 org.springframework.test.web.servlet.**ResultHandler**을 지정한다. 

스프링 테스트는 MockMvc **ResultHandlers**의 팩토리 클래스를 통해 다양한 **ResultHandler**을 제공한다.





**MockMvcResultHandlers의 주요 메서드**

| 메서드명 | 설명                                                         |
| -------- | ------------------------------------------------------------ |
| log      | 실행 결과를 디버깅 레벨에서 로그로 출력한다. 로그를 출력할 때 사용되는 로거의 이름은 org.springframework.test.web.servlet.result다 |
| print    | 실행 결과를 임의의 출력 대상에 출력한다. 출력 대상을 지정하지 않으면 기본적으로 표준 출력(System.out.)이 출력 대상이 된다. |



**간단한 예제**

```java
import static org.springframework.test.web.servlet.result.MockMvcResultHandlers.print;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;
 
import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.MediaType;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.RequestBuilder;
import org.springframework.test.web.servlet.request.MockMvcRequestBuilders;
import org.springframework.test.web.servlet.setup.MockMvcBuilders;
 
 
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = { "file:src/main/webapp/WEB-INF/spring/appServlet/servlet-context.xml" })
public class ControllerTest {
    
    @Autowired
    public RestAPIController homeController;
    private MockMvc mockMvc;
    
    @Before
    public void createController() {
        mockMvc = MockMvcBuilders.standaloneSetup(homeController).build();
    }
    
    @Test
    public void restTest() throws Exception {
        RequestBuilder reqBuilder = MockMvcRequestBuilders.get("/todo/get/1").contentType(MediaType.APPLICATION_JSON);
        mockMvc.perform(reqBuilder).andExpect(status().isOk()).andDo(print()); 
    }
}
```



위의 import내용 특히 static으로 import된 것들이 있어야 예제에 포함된 status().isok() 또는 andDo 메서드 안에 있는 print()를 실행할 수 있다.



테스트할 Controller를 @Autowired하였고, 28번 라인처럼 build() 해야 한다.









