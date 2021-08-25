# 3장 Junit 5 시작하기



## 1. Introduction to JUnit 5

### JUnit History

### JUnit 4



### JUnit 5

- Started as a project called JUnit Lambda



### Goals of JUnit 5

- Leverage features of Java 8
  - Lambda expressions
  - Streams
  - Java 8 or higher is required
- Redesigned for better integration and extensibility





### JUnit 5 Modules

- **JUnit Platform**
  - JVM 환경에서 테스트를 돌리기 위한 플랫폼(환경)
  - 테스트를 콘솔에서 돌리게 해준다.
  - Maven이나 Gradle 같은 build tool에서 테스트를 돌릴 수 있게 해준다.
- **JUnit Jupiter**
  - 테스트를 작성하기 위한 프로그래밍 모델

- **JUnit Vintage**
  - 빈티지니깐 옛날 버전들 호환해주는 놈임
  - JUnit3이나 JUnit 4 테스트들을 돌려주는 테스트 엔진임





### JUnit Annotations

| 애너테이션         | 설명                                                 |
| ------------------ | ---------------------------------------------------- |
| @Test              | 테스트 메서드 위에 붙인다.                           |
| @ParameterizedTest | parameterized 테스트 메서드 위에 붙인다.             |
| @RepeatedTest      | 테스트를 N번 반복한다.                               |
| @TestFactory       | dynamic test를 위한 Test Factory 메서드 위에 붙인다. |
| @TestInstance      | 테스트 인스턴스의 lifecycle을 설정할 때 사용한다.    |



| 애너테이션    | 설명                                                         |
| ------------- | ------------------------------------------------------------ |
| @TestTemplate | 여러 개의 테스트 케이스들이 사용하는 template을 만든다.      |
| @DisplayName  | 테스트 이름을 지정해준다. (사람이 알아먹기 편하게)           |
| @BeforeEach   | 각각의 테스트를 실행하기 전에 실행된다                       |
| @AfterEach    | 각각의 테스트를 실행한 후에 실행된다.                        |
| @BeforeAll    | 현재 클래스에 있는 모든 테스트 케이스를 실행하기 전 딱 한번 실행되는 static method |



| 애너테이션  | 설명                                                         |
| ----------- | ------------------------------------------------------------ |
| @AfterAll   | 현재 클래스에 있는 모든 테스트 케이스를 실행한 후 딱 한번 실행되는 static method |
| @Nested     | nested 테스트 클래스를 만든다.                               |
| @Tag        | 테스트를 필터링하기 위한 'tag' 들을 선언한다.                |
| @Disabled   | 테스트나 테스트 클래스를 Disable 시킨다.                     |
| @ExtendWith | extension들을 등록할 때 사용된다.                            |





### JUnit Test Lifecycle

1. `@BeforeAll`: 모든 테스트 실행하기 전 딱 한 번
2. `@BeforeEach`: 테스트 한 개 실행하기 전마다 실행
3. `@Test`
4. `@AfterEach`
5. `@AfterAll`



![lifecycle](https://lh5.googleusercontent.com/73T6Udg7WsJlEmALGPDEEj_lyDd14Bq5R2ot9r06RWGIAwUe2po-ihJPAx9ziWqHl9LLBNokZZ6kIqUOWDfrfTx2dS7ImyaW5B--ccwohlH4cLDpZsF8jd11U2LsLY3KW3p_S98K)