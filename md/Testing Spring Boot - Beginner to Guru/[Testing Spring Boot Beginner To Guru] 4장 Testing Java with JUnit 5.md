# [Testing Spring Boot Beginner To Guru] 4장 Testing Java with JUnit 5



## 1. Overview of JUnit Assertions

### JUnit Assertions

- Assertion(단언)
  - 테스트 코드를 작성할 때 통과해야 하는 조건을 assert하게 된다.
- Examples:
  - assertEquals(2, 2);
  - assertNotEquals(2, 5)
- Assert method 들은 대부분 optional한 failure message를 받을 수 있도록 Override 되어 있따.
  - assertEquals(2, 2, "Values do not match")



### JUnit Assertion Lambdas

- JUnit 5는 assertions에 lambda를 쓰는 것을 지원한다.
- **Grouped Assertions** - 한 block 안에 있는 모든 assertion들이 성공해야 성공처리되고, 하나라도 실패하면 실패 처리한다.
  - 모든 assertion들이 한 block 안에서 병행으로 수행되며, 모든 실패들이 보고된다.
- **Dependent Assertions** - Grouped assertion들로 이루어진 block들을 허용한다.
- `assertThrow`라는 람다 표현식을 통해 예상되는 예외 값들을 테스트 해볼 수 있다.
- `assertTimeout`이라는 람다 표현식을 통해 시간 초과를 테스트 해볼 수 있다.





### Assertion Frameworks

- Popular Options
  - AssertJ
  - Hamcrest
  - Truth







## 2. Grouped Assertions

JUnit 5는 **Grouped Assertions**라고 하는 새로운 기능을 지원한다.

만약에 당신이 서로 관련 있는 여러 개의 assertion들을 함께 수행하고, 

하나의 통합된 결과로 받아보고 싶다면, Grouped Assertions를 이용하면 좋다.



Grouped Assertions 기능은 `assertAll()`이라는 메서드를 통해 지원한다.



`assertAll()` 메서드는 아래와 같이 6가지 버전이 있다.

| Method variations                                            | Description                                                  |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| static void assertAll (String heading, Collection executables) | Verifies that no exception is thrown by all the executables of type Collection provided as input parameter |
| static void assertAll (String heading, Stream executables)   | Verifies that no exception is thrown by all the assert functions of different Stream provided as input parameter |
| static void assertAll (String heading, Executable... executables) | Verifies that no exception is thrown by all the supplied executables provided as input parameter |
| static void assertAll (Collection executables)               | Verifies that no exception is thrown by all the executables of type Collection provided as input parameter. This method doesn’t include header as the parameter |
| static void assertAll (Stream executables)                   | Verifies that no exception is thrown by all the supplied executables of different streams provided as input parameter. This method doesn’t include header as the parameter |
| static void assertAll (Executable... executables)            | Verifies that no exception is thrown by all the supplied executable. This method doesn’t include header as the parameter |



**예시**

```java
@Test
void groupedAssertionsMsgs() {
    // given
    Person person = new Person(1l, "Joe", "Buck");


    // then
    assertAll("Test Props Set",
            () -> assertEquals(person.getFirstName(), "Joe", "First Name failed"),
            () -> assertEquals(person.getLastName(), "Buck", "Last Name failed"));
}
```





## 3. Nested Grouped Assertions (Dependent Grouped Assertions)

우리는 한 개의 `assertAll()` 함수 안에 여러 개의 `assert` 메서드를 사용할 수 있다.



용어가 헷갈릴 수 있어서 정리를 하자면,

- 한 개의 `assertAll()` 메서드는 한 개의 독립적인 grouped assertion으로 간주된다.

- 만약에 `assertAll()` 메서드가 한 개 이상의 `assertAll()` 메서드를 포함하고 있다면,

  포함된 `assertAll()` 메서드들은 nested grouped assertion으로 간주된다.



**예시**

```java
class JUnit5Assertion1 {
    String ename;
    int ecode;
    String name=null;
     @Test
     public void dependentGrpAssert() {      
        Employee emp=new Employee("Nidhi Singh",1028838);   
        assertAll("ValidateEmpNameNotNull",
                () -> {  
                    assertNotNull(name); 
                    // 실패하므로 아래의 grouped assertion은 실행되지 않는다.
                    // 한 블록 내에 있는 assertion들은 dependent.
                     
                    assertAll("ValidateEmpCode",
                            () -> {
                                ecode=emp.getEcode();
                                assertEquals(1028838,1028838);
                            }
                             
                    ); //end of inner assertAll();
                }
                     
        );//end of outer assertAll();   
    }    
}
```



**정리**

- 한 블럭 내에 있는 assertion들은 dependent
- `assertAll()` 메서드의 인자로 들어가는 람다 식들은 independent







### `assertAll()`로 independent assertion 여러 개 수행할 때 장점

- 각각의 assertion을 독립적으로 분리시켜서 수행할 수 있다.
- 이런 경우에는 하나의 assertion이 실패해도, 다음 assertion을 수행한다.
- 모든 assertion들을 하나씩 수행할 것이다.





### 언제 Nested Grouped Assertions를 사용하는가?

- 하나 이상의 테스트 케이스들이 다른 테스트 케이스에 의존하는 경우
- 하나의 assert 문의 결과가 다른 assert 문을 위한 deciding factor가 되는 경우