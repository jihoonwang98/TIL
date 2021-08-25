# [Testing Spring Boot Beginner To Guru] 9장 Getting Started with Mockito



## 1. Introduction to Mockito

### What is Mockito?

- Mockito는 가장 인기 있는 mocking 프레임워크다.
- Mocks (aka Test Doubles)들은 실제 객체를 대신 해주는 대체 구현체들이다.
- Works well with DI.
- Injected Dependencies can be mocks.



### Types of Mocks (aka Test Doubles)

- **Dummy**: 단순히 코드를 컴파일 시키기 위해 사용되는 객체
- **Fake**: 구현은 되어 있지만, production으로 내기에는 준비되지 않은 객체
- **Stub**: method call에 대해 pre-defined answer들을 가지고 있는 객체
- **Mock**: method call에 대해 pre-defined answer들을 가지고 있고, execution에 대한 기대값도 가지고 있다.
  - 예상치 못한 invocation(호출)이 감지되면 예외를 던질 수 있다.
- **Spy**: Mockito에서 Spy란 실제 객체를 둘러싸고 있는 wrapper 같은 Mock을 말한다.





### Important Terminology

- **Verify**: mocked된 method가 호출된 횟수를 verify 하는데 사용된다.
- **Argument Matcher**: Mocked method에 전달된 argument을 matching하고, 이 argument 들을 allow할지 disallow할지 결정한다.
- **Argument Captor**: Mocked method에 전달된 argument들을 Capture한다.
  - method로 전달된 arguments들에 대한 assertion을 수행할 수 있게 해준다.





### Mockito Annotations

| 애너테이션       | 설명                                                    |
| ---------------- | ------------------------------------------------------- |
| **@Mock**        | Mock 객체를 생성할 때 사용된다.                         |
| **@Spy**         | Spy 객체를 생성할 때 사용된다.                          |
| **@InjectMocks** | Test 클래스에 mock이나 spy 객체들을 주입할 때 사용한다. |
| **@Captor**      | Mock 객체의 argument들을 caputure한다.                  |

















