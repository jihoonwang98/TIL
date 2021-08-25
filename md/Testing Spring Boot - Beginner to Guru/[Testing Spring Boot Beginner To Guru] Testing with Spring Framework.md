# [Testing Spring Boot: Beginner To Guru] Testing with Spring Framework



### Testing Utilities

- **ReflectionTestUtils** - allows use of reflection to modify private fields
  - Spring은 private property들을 autowire 할 수 있다. (하지만 poor practice로 간주된다..)
  - Can also be used to hook into bean lifecycle events
- **AOP Utils** 
  - AOP를 테스트하는 것을 도와준다.



### Spring MVC Test

- Spring MVC Test - 컨트롤러 interaction들을 테스트하는 매우 철저한 프레임워크
  - **`MockHttpServletRequest`**: request / response의 Mock 구현체
  - **`MockHttpSession`**: Http Session의 Mock 구현체
  - **`ModelAndViewAssert`**: assertion 유틸리티
- 이 프레임워크는 스프링 컨테이너를 작동시킬 필요없이 web request들을 테스트할 수 있게 해준다.
- Controller에 대한 진정한 의미의 'unit test'를 수행하게 해준다.
  - web context를 띄우지 않고 테스트하면 훨씬 빠르게 테스트할 수 있다.





### Spring Integration Testing

- Spring Context를 띄운 채로 진행하는 테스트를 Integration testing(통합 테스트)라고 한다.
- Spring Context를 띄우는 것은 expensive operation(시간이 많이 걸리는 연산)으로 간주된다.
  - Context를 띄우는 데 10 ~ 40초 정도 걸린다. (Project의 복잡도나 Hardware에 따라 다르다)

- Spring은 성능을 향상시키기 위해 테스트들 간에 Context를 캐싱해준다.
- Dependency Injection
  - Spring은 테스트 클래스에 빈을 주입시킬 수 있다.
- Transaction Management
  - 기본적으로 Spring은 테스트가 끝나면 자동으로 DB 트랜잭션을 롤백시킨다.







### JDBC Testing Support

- **JdbcTemplate**
- **JdbcTestUtils** - DB 테스트를 도와주는 utility 클래스를 모은 Collection
  - countRowsInTable
  - countRowsInTableWhere
  - deleteFromTables
  - deleteFromTablesWhere
  - dropTables





### Spring Framework Testing Annotations

| 애너테이션                  | 설명                                                         |
| --------------------------- | ------------------------------------------------------------ |
| **@BootstrapWith**          | Test Context가 어떻게 bootstrap 될 것인지 설정하는 Class-level 애너테이션 |
| **@ContextConfiguration**   | Application Context를 설정하는 Class-level 애너테이션        |
| **@WebAppConfiguration**    | Web Application Context를 설정하는 Class-level 애너테이션    |
| **@ContextHierarcy**        | 여러 개의 @ContextConfiguration들을 설정할 때 사용하는 Class-level 애너테이션 |
| **@ActiveProfiles**         | 테스트를 위해 profile을 activate할 때 사용되는 Class-level 애너테이션 |
| **@TestPropertySource**     | 테스트를 위해 property source들을 설정할 때 사용되는 Class-level 애너테이션 |
| **@DirtiesContext**         | Spring에게 테스트가 끝난 후 Context를 re-load 하라고 요청하는 Class 또는 Method Level 애너테이션 |
| **@TestExecutionListeners** | Test Execution Listener를 설정하는데 사용된다.               |
| **@Commit**                 | 테스트 수행 결과를 DB에 commit할 때 사용되는 Class 또는 Method Level 애너테이션 |
| **@Rollback**               | 테스트 수행 결과를 rollback할 때 사용되는 Class 또는 Method Level 애너테이션 |
| **@BeforeTransaction**      | 트랜잭션이 시작되기 전에 void를 리턴하는 method를 실행시켜 준다. |
| **@AfterTransaction**       | 트랜잭션이 끝난 후에 void를 리턴하는 method를 실행시켜 준다. |
| **@Sql**                    | 테스트를 실행하기 전에 수행할 SQL script들을 설정한다.       |
| **@SqlConfig**              | SQL script의 parsing 정보에 대한 설정을 한다.                |
| **@SqlGroup**               | SQL script들에 대한 grouping을 설정한다.                     |





### JUnit 4 Testing Annotations

| 애너테이션                           | 설명                                                         |
| ------------------------------------ | ------------------------------------------------------------ |
| **@IfProfileValue**                  | 특정한 환경들에 대해서만 테스트를 enable 한다.               |
| **@ProfileValueSourceConfiguration** | 어떻게 profile value 들을 가져올 것인지를 설정하는 Class-level 애너테이션 |
| **@Timed**                           | 정해진 시간 내에 테스트를 완료할 것을 요구하는 애너테이션    |
| **@Repeat**                          | 테스트를 x번 반복해서 수행한다.                              |





### Spring JUnit 5 Testing Annoations

| 애너테이션                | 설명                                                         |
| ------------------------- | ------------------------------------------------------------ |
| **@SpringJUnitConfig**    | @ContextConfiguration과 @ExtendWith(SpringExtension.class) 애너테이션을 합친 애너테이션.<br />테스트를 위한 Spring Context를 설정한다. |
| **@SpringJUnitWebConfig** | @ContextConfiguration과 @WebAppConfiguration과 @ExtendWith(SpringExtension.class) 애너테이션을 합친 애너테이션.<br />테스트를 위한 Spring Context를 설정한다. |
| **@EnabledIf**            | 테스트를 수행할 조건을 설정한다.                             |
| **@DisabledIf**           | 테스트를 수행하지 않을 조건을 설정한다.                      |











