# [Spring Test] How to test a Controller in Spring Boot - practical guide

> https://thepracticaldeveloper.com/guide-spring-boot-controller-tests/#introduction





## 1. Sample application

> 코드: https://github.com/mechero/spring-boot-testing-strategies



- Superhero를 id로 찾을 수 없는 경우, `NonExistingHeroException`이 던져진다.

  Spring의 `@RestControllerAdvice`는 해당 Exception을 가로채서 404 Status code로 변환시킨다.

- `SuperHeroFilter`라는 클래스는 HTTP 응답에 `X-SUPERHERO-APP`이라는 헤더를 추가하기 위해 사용된다.





## 2. Server-side Tests

우리가 server-side 테스트(Controller Test)를 자세히 들여다보면, Spring이 제공하는 두 가지 메인 전략을 확인할 수 있다.

첫째는 MockMVC를 이용해 작성하는 방법이고,

둘째는 RestTemplate을 이용해 작성하는 방법이다.



만약 당신이 "진짜" Unit Test를 작성하고 싶다면, MockMVC를 이용해야 한다.

반면에 RestTemplate을 이용한다면 당신은 통합 테스트를 작성하고 있는 것이다.

RestTemplate은 Spring의 WebApplicationContext를 (부분적으로 또는 완전히)이용할 것이기 때문이다.

> RestTemplate이 Standalone mode라면 WebApplicationContext를 부분적으로 이용하고,
>
> 그렇지 않다면 WebApplicationContext를 아예 완전히 다 올려버린다.





## 3. Inside-Server Tests

우리는 Web Server를 띄울 필요없이 Controller 로직을 직접 테스트할 수 있다.

이것이 바로 필자가 **inside-server testing**이라고 부르는 것이다.

그리고 이런 방식이야말로 Unit Test의 정의에 부합한다.



Inside-server test를 작성하기 위해 당신은 Web Server의 모든 행동을 mocking 해야 한다.

그래서 당신은 우리의 애플리케이션에서 테스트되어야 할 부분들을 놓칠 수도 잇다.

하지만 걱정하지 말아라. 이런 부분들은 통합 테스트를 통해 완전히 커버할 수 있다.





### 전략 1: Spring MockMVC example in Standalone Mode

![](https://thepracticaldeveloper.com/images/posts/uploads/2017/07/tests_mockmvc_wm-1024.webp)



스프링에서는 MockMVC를 standalone-mode로 사용하면 inside-server test를 작성할 수 있다.

이렇게 되면 Spring Context를 전혀 불러오지 않게 된다.





#### MockMVC standalone 코드 예시

다음 코드는 SuperHeroController에 대한 테스트를 진행한다.

```java
@ExtendWith(MockitoExtension.class)
public class SuperHeroControllerMockMvcStandaloneTest {

    private MockMvc mvc;

    @Mock
    private SuperHeroRepository superHeroRepository;

    @InjectMocks
    private SuperHeroController superHeroController;

    // This object will be magically initialized by the initFields method below.
    private JacksonTester<SuperHero> jsonSuperHero;

    @BeforeEach
    public void setup() {
        // We would need this line if we would not use the MockitoExtension
        // MockitoAnnotations.initMocks(this);
        // Here we can't use @AutoConfigureJsonTesters because there isn't a Spring context
        JacksonTester.initFields(this, new ObjectMapper());
        // MockMvc standalone approach
        mvc = MockMvcBuilders.standaloneSetup(superHeroController)
                .setControllerAdvice(new SuperHeroExceptionHandler())
                .addFilters(new SuperHeroFilter())
                .build();
    }

    @Test
    public void canRetrieveByIdWhenExists() throws Exception {
        // given
        given(superHeroRepository.getSuperHero(2))
                .willReturn(new SuperHero("Rob", "Mannon", "RobotMan"));

        // when
        MockHttpServletResponse response = mvc.perform(
                get("/superheroes/2")
                        .accept(MediaType.APPLICATION_JSON))
                .andReturn().getResponse();

        // then
        assertThat(response.getStatus()).isEqualTo(HttpStatus.OK.value());
        assertThat(response.getContentAsString()).isEqualTo(
                jsonSuperHero.write(new SuperHero("Rob", "Mannon", "RobotMan")).getJson()
        );
    }

    @Test
    public void canRetrieveByIdWhenDoesNotExist() throws Exception {
        // given
        given(superHeroRepository.getSuperHero(2))
                .willThrow(new NonExistingHeroException());

        // when
        MockHttpServletResponse response = mvc.perform(
                get("/superheroes/2")
                        .accept(MediaType.APPLICATION_JSON))
                .andReturn().getResponse();

        // then
        assertThat(response.getStatus()).isEqualTo(HttpStatus.NOT_FOUND.value());
        assertThat(response.getContentAsString()).isEmpty();
    }

    @Test
    public void canRetrieveByNameWhenExists() throws Exception {
        // given
        given(superHeroRepository.getSuperHero("RobotMan"))
                .willReturn(Optional.of(new SuperHero("Rob", "Mannon", "RobotMan")));

        // when
        MockHttpServletResponse response = mvc.perform(
                get("/superheroes/?name=RobotMan")
                        .accept(MediaType.APPLICATION_JSON))
                .andReturn().getResponse();

        // then
        assertThat(response.getStatus()).isEqualTo(HttpStatus.OK.value());
        assertThat(response.getContentAsString()).isEqualTo(
                jsonSuperHero.write(new SuperHero("Rob", "Mannon", "RobotMan")).getJson()
        );
    }

    @Test
    public void canRetrieveByNameWhenDoesNotExist() throws Exception {
        // given
        given(superHeroRepository.getSuperHero("RobotMan"))
                .willReturn(Optional.empty());

        // when
        MockHttpServletResponse response = mvc.perform(
                get("/superheroes/?name=RobotMan")
                        .accept(MediaType.APPLICATION_JSON))
                .andReturn().getResponse();

        // then
        assertThat(response.getStatus()).isEqualTo(HttpStatus.OK.value());
        assertThat(response.getContentAsString()).isEqualTo("null");
    }

    @Test
    public void canCreateANewSuperHero() throws Exception {
        // when
        MockHttpServletResponse response = mvc.perform(
                post("/superheroes/").contentType(MediaType.APPLICATION_JSON).content(
                        jsonSuperHero.write(new SuperHero("Rob", "Mannon", "RobotMan")).getJson()
                )).andReturn().getResponse();

        // then
        assertThat(response.getStatus()).isEqualTo(HttpStatus.CREATED.value());
    }

    @Test
    public void headerIsPresent() throws Exception {
        // when
        MockHttpServletResponse response = mvc.perform(
                get("/superheroes/2")
                        .accept(MediaType.APPLICATION_JSON))
                .andReturn().getResponse();

        // then
        assertThat(response.getStatus()).isEqualTo(HttpStatus.OK.value());
        assertThat(response.getHeaders("X-SUPERHERO-APP")).containsOnly("super-header");
    }
}
```





