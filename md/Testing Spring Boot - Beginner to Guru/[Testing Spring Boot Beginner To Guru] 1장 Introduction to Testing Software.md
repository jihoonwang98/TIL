# [Testing Spring Boot: Beginner To Guru] 1장 Introduction to Testing Software



> 참고:
>
> https://zorba91.tistory.com/304
>
> https://www.udemy.com/course/testing-spring-boot-beginner-to-guru/



## 1. Introduction To Testing Software



### 테스트를 왜 작성해야 하는가?

- Software 퀄리티 UP
- 자신의 코드가 생각한대로 돌아가는지 증명 가능
- 코드 하나 고치면 다른게 안돌아가는 문제를 해결 가능
- 산업 전반에 best practice로 생각되고 있다.
- 기존에 존재하는 코드를 자신감 있게 변경 가능



### 테스트 관련 용어

- **Code Under Test:** 테스트하고 있는 코드 (또는 애플리케이션)

- **Test Fixture:** 

  >"A test fixture is a fixed state of a set of objects used as a baseline for running tests. The purpose of a test fixture is to ensure that there is a well known and fixed environment in which tests are run so that results are repeatable." 
  >
  >\- Junit Doc

  Test Fixture란, 테스트 수행을 위해 기준선으로 사용되는 객체들의 고정된 상태를 말한다.

  Test Fixture의 목적은 테스트를 여러번 반복해도 같은 결과가 나올 수 있는 고정된 환경을 만들기 위함이다.

  

  - 예시:
    - Mock 또는 가짜 객체의 세팅이나 생성 그리고 삽입할 데이터의 준비
    - Test Fixture를 만들어내는 특정 파일들을 복사하면 특정 상태로 초기화된 객체들이 생성됨.
    - Junit의 Fixture 애너테이션: `@Before`, `@BeforeClass`, `@After`, `@AfterClass`



- **Unit Tests / Unit Testing:** Code written to test code under test

  - 코드의 특정 부분을 테스트하기 위해 디자인된다.

  - 테스트가 완료된 코드 line의 비율이 바로 **Code Coverage**다.

    > **Code Coverage란?**
    >
    > 코드 커버리지는 소프트웨어의 테스트를 논할 때 얼마나 테스트가 충분한지를 나타내는 지표 중 하나다.
    >
    > 말 그대로 코드가 얼마나 커버되었는가이다.
    >
    > **소프트웨어 테스트를 진행했을 때 코드 자체가 얼마나 실행되었냐**는 것이다.
    >
    > 
    >
    > 코드의 구조를 이루는 것은 크게 구문(Statement), 조건(Condition), 결정(Decision)이다. 
    >
    > 이러한 구조를 얼마나 커버했느냐에 따라 코드커버리지의 측정기준은 나뉘게 된다. 

  - 이상적인 Code Coverage 비율은 70~80%이다.

  - 'unity'해야 하며, 실행 속도가 매우 빨라야 한다.

    - unity하다는 것은 매우 매우 작은 단위여야 한다는 뜻이다.

  - 단위 테스트는 외부 종속성(dependency)을 가져서는 안된다.

    - DB 연결 X, Spring Context X
    - 단위 테스트는 완전히 독립적이어야 한다.



- **Integration Tests:** 시스템 전반의 구성요소와 객체들 사이의 행동을 테스트하기 위해 설계됨.
  - 단위 테스트에 비해 더 넓은 영역을 한 번에 테스트
  - Spring Context, DB, Message brokers 등을 포함 가능
  - 단위 테스트에 비해 더 많은 시간이 필요하게 됨



- **Functional Tests:** 쉽게 말해 애플리케이션을 직접 실행해서 테스트하는 것을 말함.





### Unit&middot;Integration&middot;Functional Tests 비교

![](https://napari.org/_images/tests.png)



- 세 가지의 테스트 모두 Software Quality에 중요한 역할을 한다.
- 하지만, 테스트 작성의 주가 되는 것은 **Unit Tests**가 되어야 한다.
  - 작은 단위, 빠르고, 가벼운 테스트
  - 매우 세밀하고 구체적인 테스트





### Agile Testing Methods

- **TDD: **테스트 주도 개발 (Test Driven Development)
  - 1. 테스트 코드 작성 (실패)
    2. 테스트가 돌아가도록 코드를 수정
    3. 코드 리팩토링



- **BDD:** 행동 주도 개발 (Behavior Driven Development)

  - TDD와 매우 유사하다.

  - BDD를 통해 개발을 할 시 테스트 메서드의 이름을 "이 클래스가 어떤 행위를 해야 한다. (should do something)" 식의 문장으로 작성해 행위에 대한 테스트에 집중할 수 있다.

  - when / then 또는

    given / when / then 꼴로 표현되기도 한다.

  - BDD를 도와주는 <u>Spock</u>이라는 framework도 있다!





### Testing Components

- **Mocks:** 테스트에 사용되는 클래스의 가짜 구현체

  - dependent object들(ex) datasource)에 대한 test double 

    - > **test double이란?**
      >
      > xUnit Test Patterns의 저자가 만든 용어로 테스트를 진행하기 어려운 경우 이를 대신해 테스트를 진행할 수 있도록 만들어주는 객체를 말한다.
      >
      > 영화 촬영 시 위험한 역할을 대신하는 스턴트 더블에서 비롯되었다.

  - 결과의 기댓값을 설정할 수 있다.

  - Mock을 사용하면 객체들간의 상호작용의 기댓값을 증명할 수 있다.



- **Spy:** Mock이랑 비슷하지만, 실제 객체가 사용된다.
  - Mock들은 해당 객체를 완전히 대체해버린다.
  - Spy들은 실제 객체가 안에 들어있는 wrapper 역할을 한다.





## 2. Common Java Testing Frameworks

### JUnit

- 가장 유명한 테스트 프레임워크
- JUnit 5는 2017년 9월에 나왔다. 아직 JUnit 4를 쓰는데도 많다.



### TestNG

- 2004년에 나옴.
- JUnit에 밀리고 있음



### Spock

- Groovy로 만들어진 자바 코드를 위한 테스트 프레임워크 (Groovy 테스트도 가능)
- Groovy에 대한 지식이 필요하다.
- BDD 접근을 따른다.
- 자신만의 Mocking framework를 포함하고 있다.



### Cucumber

- BDD Testing Framework
- Java, Javascript, Ruby 언어에 대해 이용가능
  - Java 커뮤니티에서도 인기를 얻고 있는 중
- Gherkin 문법을 사용한다.
  - Natural English like syntax
  - Describe the what, not the how



### Mockito

- Mocking framework for testing
  - Only does mocks
  - JUnit이나 TestNG 같은 Testing framework와 함께 사용해야 한다.
- Top 10 Java Library
- Spring 애플리케이션을 테스트할 때 매우 인기가 좋다.



### Spring MVC Test

- Spring Framework이 제공하는 Testing module
- Spring MVC Controller들을 테스트할 때 매우 유용하다.
- mock Servlet 환경을 제공한다.
- JUnit, TestNG, Spock과 같은 Testing Framework와 함께 사용된다.



### Rest Assured

- RESTful 웹 서비스를 테스트하기 위한 Java Framework
- Restful 인터페이스 테스트를 위한 매우 강력한 DSL을 제공한다.
- Spring Mock MC와 함께 사용될 수 있다.
- BDD Syntax를 따른다.



### Selenium

- 웹 브라우저 자동화

- 웹 애플리케이션에 대한 functional test를 작성할 수 있게 해준다.

  

### GEB

- Groovy Browser Automation
- Selenium을 사용하고 있다.
- JQuery-ish page element selector들을 가지고 있다.



### Test Containers

- Allows you to launch Docker containers from JUnit Tests

  JUnit Test로 Docker container를 실행하는 것을 도와준다.

- Allows you to start DB, message brokers, etc for integration and functional tests.

- 웹 애플리케이션을 테스트하기 위해 Selenium과 결합되어 사용될 수 있다.





## 3. Beyond Testing with CI and CD

### CI - Continuous Integration

- **Continuous Integration(CI):**

  새로운 코드 변경 사항이 정기적으로 빌드 및 테스트되어 공유 레포지토리에 통합되는 것.

  각각의 check-in은 자동화된 빌드에 의해 검증되고, 팀원들이 발생한 문제를 쉽게 알아차릴 수 있게 함.

  > "CI는 bug를 제거하는 것이 아니지만, bug를 찾아서 제거하기 쉽게 만들어 준다. "
  >
  > \- 마틴 파울러





### CI Practices

- 마틴 파울러가 정한 CI Practices
  - Single Source Repository를 유지하라
  - Build를 자동화하라
  - 당신의 Build를 Self-Testing하게 만들어라
  - 모든 Commit은 Integration Machine에서 이루어지게 하라
  - Broken Build들을 즉시 고쳐라
  - Build를 빠르게 유지하라
  - Production 환경의 복사본 환경에서 테스트하라
  - 누구나 최근 실행 버전에 접근하기 쉽게 만들어라
  - 모든 사람이 무슨 일이 일어나고 있는지 알기 쉽게 하라



### Common CI Build Servers

- **Self-Hosted:**
  - Jenkins, Bamboo, TeamCity, Hudson
- **Cloud Based:**
  - CircleCI, TravisCI, Codeship, GitLab CI, AWS CodeBuild
  - And many, many more



### CD - Continuous Deployment

- Continuous Deployment는 모든 CI 테스트들이 수행된 후에 자동으로 빌드된 artifacts들을 배포할 것이다.
- 모든 Commit마다 CD가 일어나야 한다.
- 완전히 자동화되어야 한다.
- Continuous Delivery와 헷갈리기 쉽다.
- Example: AWS CodePipeline





### CD - Continuous Delivery

- 변경된 Code를 Production 환경에 자동으로 곧바로 전달하는 프로세스

- Testing과 Deployment 자동화에 있어서 매우 높은 지식 수준을 요구한다.
- 매우 성숙한 Process를 요구한다.