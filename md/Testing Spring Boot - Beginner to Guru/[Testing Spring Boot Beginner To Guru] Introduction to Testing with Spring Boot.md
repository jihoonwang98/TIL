# [Testing Spring Boot: Beginner To Guru] Introduction to Testing with Spring Boot



### Spring Boot Testing

- Spring Boot Test 기능들은 아래와 같은 starter를 등록함으로써 이용할 수 있다.
  - spring-boot-starter-test
- 위의 starter는 아래와 같은 기능들을 제공한다:
  - Common testing dependencies
  - Spring Boot Testing dependencies - (예) Testing annotations and support)
  - Spring Boot Testing Auto-Configuration



### Spring Boot Test Scope Dependencies

- spring-boot-starter-test (2.1.x 버전)은 아래와 같은 Testing Library들을 가져온다.
  - **Junit**: 2.1 버전 이전은 JUnit 4,	2.2 버전 이후는 JUnit 5
  - **Spring Test**: Spring Framework Testing features
  - **AssertJ:** Fluent assertions
  - **Hamcrest**: Matchers for testing
  - **Mockito:** Mocking framework
  - **JSONAssert**: Assertions for JSON
  - **JsonPath**: XPath for JSON





### Spring Testing Context with Spring Boot

- **`@SpringBootTest`**: will enable Spring Context
  - 만약 Junit 4를 이용하고 있으면 `@RunWith(SpringRunner.class)` 애너테이션을 테스트 클래스에 붙여줘야 한다.
  - `@ExtendWith(SpringExtension.class)` 애너테이션을 포함하고 있다.
  - 기본적으로 `@SpringBootConfiguration` 애너테이션을 찾는다.
    - 해당 애너테이션은 `@SpringBootApplication`에 포함되어 있다.
  - 기본적으로 Spring Boot는 web server를 시작시키지 않는다.





### Web Environment

- Web Environment를 enable 하려면 - `@SpringBootTest(webEnvironment = <option>)` 애너테이션 사용
- Web Environment Options들:
  - **MOCK**: Default 값, Mock Web Environment를 불러온다.
  - **RANDOM_PORT**: random port를 listening하는 embedded web server를 제공한다.
  - **DEFINED_PORT**: 8080 포트(기본) 또는 `application.properties`에 `server.port`로 정의된 포트를 listening 하는 embedded web server를 제공한다
  - **NONE**: No Web Environment



### Spring Boot Test Annotations

- `@TestComponent`: Stereotype for test components
- `@TestConfiguration`: Java Configuration for tests
- `@LocalServerPort`: Inject port of running server
- `@MockBean`: Inject Mockito Mock
- `@SpyBean`: Inject Mockito Spy





### Spring Boot Test Slices

- **`@SpringBootTest`**: 기본적으로 project를 스캔해서 모든 available(enabled)한 auto configuration들을 사용해서 full context를 불러온다.

  - 복잡한 애플리케이션에서는 full context를 불러오는 것이 heavy하고 costly할 수 있다.

  

- **Spring Boot Test Slices**: targeted light-weight configurations.

  - defined된 auto configuration 전체를 enable 하지 않는다.
  - Example: `@JsonTest` - Jackson(default)이나 Gson을 위한 JSON 환경이 설정된 Spring Boot를 생성한다.
  - `@SpringBootTest` 대신 `@머시기...Test`를 사용하는 것이 좋다...
  - 종류: (다음 Test Slice 애너테이션들이 해주는 auto-configuration들은 구글링해서 찾아보자.)
    - `@DataJdbcTest`
    - `@DataJpaTest`
    - `@DataLdapTest`
    - `@DataMongoTest`
    - `@DataNeo4jTest`
    - `@DataRedisTest`
    - `@JdbcTest`
    - `@JooqTest`
    - `@JsonTest`
    - `@RestClientTest`
    - `@WebFluxTest`
    - `@WebMvcTest`



















