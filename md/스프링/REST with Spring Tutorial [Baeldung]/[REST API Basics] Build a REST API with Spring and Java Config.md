# Spring으로 REST API 빌드하기​ :deer:



> **[참고]**: Build a REST API with Spring and Java Config
>
>  https://www.baeldung.com/building-a-restful-web-service-with-spring-and-java-based-configuration
>
> 의역과 오역에 주의...





## 1. Overview :sailboat:

이번 포스팅에서는 **Spring**을 이용해 **REST API**를 구현하는 방법을 알아보자.



> **REST**란?
>
> **Re**presentational **S**tate **T**ransfer의 약자로 
>
> **자원(resource)의 표현(representation)**에 의한 **상태 전달**이라고 보면 된다.
>
> 무슨 말인지 모르겠다면 
>
> https://gmlwjd9405.github.io/2018/09/21/rest-and-restful.html
>
> 위 링크에 자세히 설명되어 있다 :clap: 





## 2. Understanding REST in Spring :sake:

Spring framework을 이용해 **RESTful** 서비스를 구축하는 데에는 2가지 방법이 있다.

- MVC의 *ModelAndView*를 이용하는 것
- **HTTP message converters를 이용하는 것**



*ModelAndView*를 이용하는 접근법은 더 **구식**이고 설정하기 ~~**귀찮**~~지만 참고 자료가 많다. 이 방법은 old model에 REST 패러다임을 쑤셔넣으려고 해서 문제가 있다고 한다.



그래서 Spring team은 Spring 3.0 버전부터 REST에 대한 새로운 support를 지원하기 시작했다.

이 새로운 접근법은 **HttpMessageConverter**와 **Annotation**들을 사용해 훨씬 가볍고 구현하기 쉽다. :nerd_face:

Configuration은 최소화되고 설정들 중 상당 부분이 <u>default 설정</u>으로 구현되어 필요한 부분만 설정해주면 되게 바뀌었다. :raised_hands:







## 3. The Java Configuration :watermelon:



```java
@Configuration
@EnableWebMvc
public class WebConfig{
   //
}
```

새로운 어노테이션인 **@EnableWebMvc**는 몇가지 유용한 작업들을 해준다.

- classpath의 **Jackson** 라이브러리와 JAXB2의 존재를 알아서 감지해주고 

  기본적으로 **JSON**과 **XML** converter를 생성하고 등록해준다.

- 이 어노테이션의 기능은 XML에서의 `<mvc:annotation-driven />`과 동일하다.



이 어노테이션을 이용하는 것이 훨씬 쉽고 다양한 상황에서 유용할 수 있지만, <u>완벽하지는 않다</u>.

더욱 복잡한 설정이 필요할 때, 이 어노테이션을 지우고 **WebMvcConfigurationSupport** 클래스를 상속받아 사용하자.





### 3.1 Using Spring Boot :yum:

만약 우리가 **@SpringBootApplication** 어노테이션과 `spring-webmvc` 라이브러리를 사용하고 있다면, 

**@EnableWebMvc** 어노테이션이 **default autoconfiguration**과 함께 자동적으로 추가된다.



이 **default 자동 설정**에 MVC 기능을 더 추가하고 싶다면 **@Configuration**이 붙은 클래스가

**WebMvcConfigurer** 인터페이스를 implements 하게 해주면 된다.



> 참고: https://m.blog.naver.com/nuberus/222032697003



또한 **WebMvcRegistrationsAdapter**를 사용해 **RequestMappingHandlerMapping, RequestMappingHandlerAdapter, ExceptionHandlerExceptionResolver** 등의 구현을 원하는 대로 설정할 수도 있다.



**Spring Boot**가 기본적으로 제공하는 MVC 기능들을 버리고 custom 설정을 하고 싶다면, 그냥 **@EnableWebMvc** 어노테이션을 사용하면 된다.







## 4. Testing the Spring Context :egg:

먼저 Spring Boot 없이 Spring Context를 테스트 해보자. 

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration( 
  classes = {WebConfig.class, PersistenceConfig.class},
  loader = AnnotationConfigContextLoader.class)
public class SpringContextIntegrationTest {

   @Test
   public void contextLoads(){
      // When
   }
}
```



**@ContextConfiguration** 어노테이션을 통해 **자바 기반 설정 파일**들을 명시해주고 있다.

**AnnotationConfigContextLoader**는 **@Configuration** 클래스들로부터 bean 정의들을 불러오는 역할을 한다.

**WebConfig** 설정 클래스는 test에 포함되지 않음에 주의해라. 왜냐하면 WebConfig 클래스는 Servlet context에서 실행되어야하기 때문이다.





### 4.1 Using Spring Boot :facepunch:

**Spring Boot**는 Spring **ApplicationContext**를 set up 하는데 유용한 몇가지의 어노테이션들을 제공한다. 이 어노테이션들이 우리의 테스트들을 좀 더 직관적으로 만들어준다.



우리는 **Application Configuration**의 **특정 부분**만을 불러와 테스트할 수도 있고, 

**전체 Context**가 startup되는 process를 테스트해볼 수도 있다.







- **@SpringBootTest** 어노테이션 사용
  - 서버의 실행없이 **전체 Context**를 테스트 가능



```java
@RunWith(SpringRunner.class)
@SpringBootTest
@AutoConfigureMockMvc
public class FooControllerAppIntegrationTest {

    @Autowired
    private MockMvc mockMvc;

    @Test
    public void whenTestApp_thenEmptyResponse() throws Exception {
        this.mockMvc.perform(get("/foos")
            .andExpect(status().isOk())
            .andExpect(...);
    }

}
```

**@AutoConfigureMockMvc** 어노테이션을 추가해 **MockMvc** 인스턴스를 주입하고 HTTP 요청들을 날릴 수 있다. :+1:











- **@WebMvcTest** 어노테이션 사용

  - **전체 Context**를 생성하기 싫고 **MVC Controller**들만 테스트하고 싶을때 사용

    

```java
@RunWith(SpringRunner.class)
@WebMvcTest(FooController.class)
public class FooControllerWebLayerIntegrationTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private IFooService service;

    @Test()
    public void whenTestMvcController_thenRetrieveExpectedResult() throws Exception {
        // ...

        this.mockMvc.perform(get("/foos")
            .andExpect(...);
    }
}
```



>  좀 더 디테일한 information은 
>
> https://www.baeldung.com/spring-boot-testing 
>
> 위 링크를 참조하자.





## 5. The Controller :control_knobs:

**@RestController**는 **RESTful API**를 만들때 아주 중요한 *artifact*다.



이번 절에서는 간단한 **REST 자원**인 '*Foo*'를 모델링하는 **controller**를 작성해보자.

```java
@RestController
@RequestMapping("/foos")
class FooController {

    @Autowired
    private IFooService service;

    @GetMapping
    public List<Foo> findAll() {
        return service.findAll();
    }

    @GetMapping(value = "/{id}")
    public Foo findById(@PathVariable("id") Long id) {
        return RestPreconditions.checkFound(service.findById(id));
    }

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public Long create(@RequestBody Foo resource) {
        Preconditions.checkNotNull(resource);
        return service.create(resource);
    }

    @PutMapping(value = "/{id}")
    @ResponseStatus(HttpStatus.OK)
    public void update(@PathVariable( "id" ) Long id, @RequestBody Foo resource) {
        Preconditions.checkNotNull(resource);
        RestPreconditions.checkNotNull(service.getById(resource.getId()));
        service.update(resource);
    }

    @DeleteMapping(value = "/{id}")
    @ResponseStatus(HttpStatus.OK)
    public void delete(@PathVariable("id") Long id) {
        service.deleteById(id);
    }

}
```



위 예에서는 Guava style의 *RestPreconditions* utility를 사용하고 있다.

```java
public class RestPreconditions {
    public static <T> T checkFound(T resource) {
        if (resource == null) {
            throw new MyResourceNotFoundException();
        }
        return resource;
    }
}
```





위의 예에서 눈여겨볼 점은 **Controller** 클래스가 **non-public**이라는 점이다. (왜냐면 public으로 할 이유가 없기 때문... :sunglasses:)

보통, **controller**는 의존성 체인의 맨 마지막에 있기 때문이다. (무슨 말인지 이해가 안가면 아래의 글을 끝까지 읽어보자)

**Controller**는 보통 **Spring Front Controller**(*DispatcherServlet*)으로부터 HTTP request들을 받고 그냥 단순히 Service Layer로 위임해버린다. 

만약 **Controller**가 주입되거나 직접적으로 참조 변수를 통해 이용될 Case가 없다면, public으로 선언하지 않는 것을 선호한다.





또한, 위의 예에서 보이는 것처럼 **request mapping**이 직관적이다. 

요청된 URI에 따라 요청을 처리할 target method가 결정된다. 

요청을 처리할 URI를 지정하는 것은 **@RequestMapping**의 value 값이나 **@GetMapping, @PostMapping, @PutMapping, @DeleteMapping** 들의 value 값에 따라 결정된다.





**@RequestBody** 어노테이션은 **HTTP request**의 ***body***를 메서드 파라미터에 binding하는 역할을 한다.

반면, **@ResponseBody** 어노테이션은 response와 return type에 대해 똑같은 처리를 한다.



**@RestController**는 **@Controller + @ResponseBody**의 효과를 낸다.







## 6. Mapping the HTTP Response Codes :melon:

**HTTP 응답 Status code**를 잘 숙지하는 것은 REST service를 구현할 때 아주 중요한 부분 중 하나다.





### 6.1 Unmapped Requests (매핑되지 않은 요청들) :apple:

만약 Spring MVC가 mapping되지 않는 요청을 받았다면, 

요청이 허용되지 않는 것으로 간주하고 **405 METHOD NOT ALLOWED** 코드를 client에게 보낸다.



405 METHOD NOT ALLOWED를 client에게 보낼 때, 어떤 operation들이 허용되는지 알려주기 위해 *Allow* HTTP header를 포함해 보낸다.

Spring MVC는 어떠한 추가 설정도 필요없이 이 작업을 default로 해준다.





### 6.2 Valid Mapped Requests (매핑된 유효한 요청들) :baby:

매핑이 성공한 모든 요청에 대해서,

Spring MVC는 요청이 valid하다고 간주하고, 다른 status code가 명시되어 있지 않다면 **200 OK** 코드를 보낸다.



이는 *Controller*가 *create, update, delete*들에 대해서는 다른 **@ResponseStatus**를 선언하지만, ***get***에 대해서는 그렇지 않기 때문에 default로 **200 OK**를 반환해야 한다.







### 6.3 Client Error :bacon:

*client error*가 발생했을 때는, **custom exception**들을 정의하고 이를 적절한 error code들로 mapping 할 수 있다.



아래와 같은 exception들을 *web tier*의 어떤 layer에서든 throwing 하기만 해도,

Spring이 HTTP 응답에 대응하는 status code와 매핑해준다.

```java
@ResponseStatus(HttpStatus.BAD_REQUEST)
public class BadRequestException extends RuntimeException {
   //
}
@ResponseStatus(HttpStatus.NOT_FOUND)
public class ResourceNotFoundException extends RuntimeException {
   //
}
```







### 6.4 Using @ExceptionHandler :badminton:

**custom exception**들을 특정한 status code에 mapping하는 다른 방법은 

*controller*에서 **@ExceptionHandler** 어노테이션을 사용하는 것이다.



그런데 이 방법의 문제는 **@ExceptionHandler** 어노테이션이 

해당 어노테이션이 붙은 *controller*에만 적용된다는 것이다.

즉, 각각의 *controller*마다 우리가 어노테이션을 달아줘야 한다. :sob:







> 물론, Spring과 Spring boot가 제공하는 더 유연한 예외 처리 방법들이 존재한다.
>
> https://www.baeldung.com/exception-handling-for-rest-with-spring
>
> 그 방법들은 위 링크를 참조해보자.











## 7. Additional Maven Dependencies :goat:

Spring만 이용하면 **REST API**를 만들기 위해서 `spring-webmvc` 의존성 외에 추가로 더 의존성이 필요하다.



```xml
<dependencies>
   <dependency>
      <groupId>com.fasterxml.jackson.core</groupId>
      <artifactId>jackson-databind</artifactId>
      <version>2.9.8</version>
   </dependency>
   <dependency>
      <groupId>javax.xml.bind</groupId>
      <artifactId>jaxb-api</artifactId>
      <version>2.3.1</version>
      <scope>runtime</scope>
   </dependency>
</dependencies>
```



위의 라이브러리들은 **REST 자원**을 *JSON*이나 *XML* 형식으로 표현하는데 사용된다.



~~Spring은 설정하기가 너무 귀찮다~~ :tired_face:







### 7.1 Using Spring Boot :airplane:

만약 우리가 JSON 형태의 자원들을 얻고 싶을 때, 

**Spring Boot**는 *Jackson, Gson, JSON-B*와 같은 라이브러리들을 제공해준다.



단순히 classpath에 위의 라이브러리들 중 하나를 포함하는 것만으로도 Auto-configuration이 수행된다.



보통 웹 애플리케이션을 개발할 때, 우리는 그냥 `spring-boot-starter-web` 의존성을 추가하고 얘가 알아서 필요한 artifact들을 포함시켜주기를 기대한다.

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <version>2.4.0</version>
</dependency>
```



이렇게 추가하면 **Spring Boot**는 친절하게도 *Jackson* 라이브러리를 default로 사용하게 해준다. :grinning: