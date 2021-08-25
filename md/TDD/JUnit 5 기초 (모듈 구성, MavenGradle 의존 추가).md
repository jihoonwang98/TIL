## JUnit 5 기초 (모듈 구성, Maven&middot;Gradle 의존 추가)



> JUnit 5.5 버전 기준으로 설명한다.







### JUnit 5 모듈 구성

**JUnit 5**는 크게 3개의 요소로 구성되어 있다.

- **JUnit Platform:** 테스팅 프레임워크를 구동하기 위한 **launcher**와 테스트 엔진을 위한 **API**를 제공한다.
- **JUnit Jupiter:** **JUnit 5**를 위한 테스트 **API**와 **실행 엔진**을 제공한다.
- **JUnit Vintage:** **JUnit 3과 4**로 작성된 테스트를 **JUnit 5 Platform**에서 실행하기 위한 모듈을 제공한다.



이들 구성 요소의 주요 모듈 구조는 아래와 같다.

![](https://docs.google.com/drawings/d/sPIsT8wqeF7nHGqqSeCb_GQ/image?parent=e/2PACX-1vSgWs9-SkKDVDplyC96i2vj5TuZvbmLHvACG_NMQ3eEa1hFtJwCGbFZVRaHriS3HOeS8O0XIQ6gqqII&rev=266&h=288&w=601&ac=1)



**JUnit 5**는 테스트를 위한 **API**로 **jupiter-api**를 제공한다. **jupiter-api**를 사용해서 테스트를 작성하고 실행하려면 위 그림에 보이는 주피터 관련 모듈을 의존에 추가하면 된다. 





#### [Maven에 JUnit 5 의존 추가하기]

```xml
// pom.xml
~생략~
<dependencies>
	<dependency>
    	<groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter</artifactId>
        <version>5.5.0</version>
        <scope>test</scope>
    </dependency>
</dependencies>

<build>
	<plugins>
    	<plugin>
        	<artifactId>maven-surefire-plugin</artifactId>
            <version>2.22.1</version>
        </plugin>
        ~ 생략 ~
    </plugins>
</build>
```



- 여기서 **junit-jupiter** 모듈은 **junit-jupiter-api** 모듈, **junit-jupiter-params **모듈, **junit-jupiter-engine** 모듈을 포함한다. 이 세 모듈은 **JUnit**으로 테스트 코드를 작성하고 실행하기 위한 모듈이다.

  

- **JUnit 5**를 이용해서 테스트를 실행하려면 **JUnit 5 Platform**이 제공하는 **platform-launcher**를 사용해야 한다. **Maven**은 **maven-surefire-plugin 2.22.0** 버전부터 **JUnit 5 Platform**을 지원하므로 따로 **Platform**을 설정하지 않아도 된다.









#### [Gradle에 JUnit 5 의존 추가하기]

**Gradle**도 유사하다. **junit-jupiter-api**를 테스트 구현으로 사용하고 **JUnit Platform**을 이용해서 테스트를 실행하도록 설정하면 된다.



```gradle
plugins {
	id 'java'
	id 'eclipse'
	id 'idea'
}

sourceCompatibility = '1.8'
targetCompatibility = '1.8'

repositories {
	mavenCentral()
}

dependencies {
	testImplementation('org.junit.jupiter:junit-jupiter:5.5.0')
}

test{
	useJUnitPlatform()
	testLogging {
		events "passed", "skipped", "failed"
	}
}
```



- **testImplementation**을 사용해서 **junit-jupiter** 의존을 추가한다. **test** 태스크는 **JUnit 5 Platform**을 사용하도록 설정한다.



> Gradle 4.6 버전부터 JUnit 5 플랫폼을 지원한다.





버전별 **Maven/Gradle** 설정은 다음 문서를 참고한다.

- **https://junit.org/junit5/docs/버전/user-guide/**













### @Test 애너테이션과 Test 메서드

**JUnit** 모듈을 설정했다면 **JUnit**을 이용해서 테스트 코드를 작성하고 실행할 수 있다. **JUnit** 코드의 기본 구조는 간단하다. 테스트로 사용할 클래스를 만들고 **@Test** 애너테이션을 메서드에 붙이기만 하면 된다.



```java
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.assertEquals;

public class SumTest {
    
    @Test 
    void sum() {
        int result = 2 + 3;
        assertEquals(5, result);
    }
}
```



**테스트 클래스의 이름**을 작성하는 특별한 규칙은 없지만 보통 다른 클래스와 구분을 쉽게 하기 위해 **'Test'를 접미사로 붙인다**. 테스트를 실행할 메서드에는 **@Test** 애너테이션을 붙인다. 이때 <u>**@Test** 애너테이션을 붙인 메서드는 **private**이면 안된다</u>.

















### 주요 단언(Assertion) 메서드

**Assertions** 클래스는 **assertEquals()**를 포함해 아래와 같은 **단언 메서드**를 제공한다.



| **메서드**                                          | **설명**                                                     |
| --------------------------------------------------- | ------------------------------------------------------------ |
| **assertEquals(expected, actual)**                  | 실제 값(actual)이 기대하는 값(expected)과 같은지 검사한다.   |
| **assertNotEquals(unexpected, actual)**             | 실제 값(actual)이 특정 값(unexpected)과 같지 않은지 검사한다. |
| **assertSame(Object expected, Object actual)**      | 두 객체가 동일한 객체인지 검사한다.                          |
| **assertNotSame(Object unexpected, Object actual)** | 두 객체가 동일하지 않은 객체인지 검사한다.                   |
| **assertTrue(boolean condition)**                   | 값이 true인지 검사한다.                                      |
| **assertFalse(boolean condition)**                  | 값이 false인지 검사한다.                                     |
| **assertNull(Object actual)**                       | 값이 null인지 검사한다.                                      |
| **assertNotNull(Object actual)**                    | 값이 null이 아닌지 검사한다.                                 |
| **fail()**                                          | 테스트를 실패 처리한다.                                      |



주요 타입별로 **assertEquals()** 메서드가 존재한다. 예를 들어 **int** 타입을 위한 **assertEquals()** 메서드, **Long** 타입을 위한 **assertEquals()** 메서드, **Object**를 위한 **assertEquals()** 메서드 등이 존재한다.**assertEquals()** 메서드를 사용할 때 주의할 점은 첫 번째 인자가 기대하는 값이고 두 번째 인자가 검사하려는 값(실제 값)이라는 점이다.







**fail()** 메서드는 테스트에 실패했음을 알리고 싶을 때 사용한다. 예를 들어 ID와 암호로 전달받은 파라미터 값이 **null**이면 **IllegalArgumentException**이 발생하도록 인증 기능을 구현했다고 가정하자. 테스트 코드는 ID와 암호로 **null**을 전달했는데 익셉션이 발생하지 않으면 테스트에 실패했다고 볼 수 있다. 이럴 때 **fail()**를 사용할 수 있다. 



```java
try {
	AuthService authService = new AuthService();
    authService.authenticate(null, null);
    fail();  // 이 지점에 다다르면 fail() 메서드는 테스트 실패 에러를 발생
} catch(IllegalArgumentException e) {
    
}
```



**Exception 발생 유무**가 검증 대상이라면 **fail()** 메서드를 사용하는 것보다 아래의 두 메서드를 사용하는 것이 더 명시적이다.



| 메서드                                                       | 설명                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| **assertThrows(Class\<T> expectedType, Executable executable)** | executable을 실행한 결과로 지정한 타입의 익셉션이 발생하는지 검사한다. |
| **assertDoesNotThrow(Executable executable)**                | executable을 실행한 결과로 익셉션이 발생하지 않는지 검사한다. |



다음은 **assertThrows()**를 이용해서 지정한 익셉션이 발생하는지 검사하는 코드 예이다.

```java
assertThrows(IllegalArgumentException.class,
	() -> {
        AuthService authService = new AuthService();
        authService.authenticate(null, null);
    });
```





**assertThrows()** 메서드는 <u>발생한 익셉션 객체를 리턴</u>한다. 발생한 익셉션을 이용해서 추가로 검증이 필요하면 **assertThrows()** 메서드가 리턴한 익셉션 객체를 사용하면 된다.

```java
IllegalArgumentException thrown = assertThrows(IllegalArgumentException.class,
	() -> {
        AuthService authService = new AuthService();
        authService.authenticate(null, null);
    });
assertTrue(thrown.getMessage().contains("id"));
```





참고로 **assertThrows()**와 **assertDoesNotThrow()** 메서드에서 사용하는 **Executable** 인터페이스는 다음과 같이 **execute()** 메서드를 가진 함수형 인터페이스이다.

```java
package org.junit.jupiter.api.function;

public interface Executable {
	void execute() throws Throwable;
}
```





**assert** 메서드는 실패하면 다음 코드를 실행하지 않고 바로 익셉션을 발생시킨다. 예를 들어 다음 코드는 첫 번째 **assertEquals()** 메서드에서 검증에 실패하기 때문에 그 시점에 **AssertionFailedError**를 발생시킨다. 따라서 두 번째 **assertEquals()** 메서드는 실행되지 않는다.

```java
assertEquals(3, 5/2);   // 검증 실패로 에러 발생
assertEquals(4, 2 * 2); // 이 코드는 실행되지 않음
```





그런데 경우에 따라 일단 모든 검증을 실행하고 그 중에 실패한 것이 있는지 확인하고 싶을 때가 있다. 이럴 때 사용할 수 있는 것이 **assertAll()** 메서드이다. 다음은 **assertAll()** 메서드의 사용 예이다.

```java
assertAll(
	() -> assertEquals(3, 5/2),
    () -> assertEquals(4, 2*2),
    () -> assertEquals(6, 11/2)
);
```





**assertAll()** 메서드는 **Executable** 목록을 가변 인자로 전달받아 각 **Executable**을 실행한다. 실행 결과로 검증에 실패한 코드가 있으면 그 목록을 모아서 에러 메시지로 보여준다. 예를 들어 위 코드 실행 결과로 출력되는 테스트 실패 오류 메시지는 다음과 같다.

```java
org.opentest4j.MultipleFailuresError: Multiple Failures (2 failures)
expected: <3> but was: <2>
expected: <6> but was: <5>
```











### Test LifeCycle



#### [@BeforeEach 애너테이션과 @AfterEach 애너테이션]

**JUnit**은 각 테스트 메서드마다 다음 순서대로 코드를 실행한다.

1. 테스트 메서드를 포함한 **객체 생성**
2. (존재하면) **@BeforeEach** 애너테이션이 붙은 메서드 **실행**
3. **@Test** 애너테이션이 붙은 메서드 **실행**
4. (존재하면) **@AfterEach** 애너테이션이 붙은 메서드 **실행**



```java
package chap05;

import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;

public class LifeCycleTest {

    public LifeCycleTest() {
        System.out.println("new LifeCycleTest()");
    }

    @BeforeEach
    void setUp() {
        System.out.println("setUp()");
    }

    @Test
    void a() {
        System.out.println("a()");
    }

    @Test
    void b() {
        System.out.println("b()");
    }

    @AfterEach
    void tearDown() {
        System.out.println("tearDown()");
    }
}


// output
new LifeCycleTest()
setUp()
a()
tearDown()
new LifeCycleTest()
setUp()
b()
tearDown()
```



이 결과를 보면 **@Test** 메서드를 실행할 때마다 **<u>객체를 새로 생성</u>**하고 테스트 메서드를 실행하기 전과 후에 **@BeforeEach** 애너테이션과 **@AfterEach** 애너테이션을 붙인 메서드를 실행한다는 것을 알 수 있다.



**@BeforeEach** 애너테이션과 **@AfterEach** 애너테이션을 붙인 메서드는 **@Test** 애너테이션과 마찬가지로 **private**이면 안 된다.







#### [@BeforeAll 애너테이션과 @AfterAll 애너테이션]

한 클래스의 모든 테스트 메서드가 실행되기 전에 특정 작업을 수행해야 한다면 **@BeforeAll** 애너테이션을 사용한다. **@BeforeAll** 애너테이션은 **정적 메서드(static method)**에 붙이는데 이 메서드는 클래스의 모든 테스트 메서드를 실행하기 전에 **<u>한 번만</u>** 실행된다.

**@AfterAll** 애너테이션은 반대로 클래스의 모든 테스트 메서드를 실행한 뒤에 실행된다. 이 메서드 역시 **정적 메서드**에 적용한다.













### 테스트 메서드 간 실행 순서 의존과 필드 공유하지 않기

다음 코드를 보자.

```java
public class BadTest {
    private FileOperator op = new FileOperator();
    private static File file; // 두 테스트가 데이터를 공유할 목적으로 필드 사용
    
    @Test
    void fileCreationTest() {
        File createdFile = op.createFile();
        assertTrue(createdFile.length() > 0);
        this.file = createdFile;
    }
    
    @Test
    void readFileTest() {
        long data = op.readData(file);
        assertTrue(data > 0);
    }
}
```



이 테스트는 **fileCreationTest()** 메서드가 **readFileTest()** 메서드보다 먼저 실행된다는 것을 가정한다. 하지만 테스트 메서드가 **<u>특정 순서대로 실행된다는 가정하에 테스트 메서드를 작성하면 안 된다.</u>** 



각 테스트 메서드는 서로 **독립적**으로 동작해야 한다. 한 테스트 메서드의 결과에 따라 다른 테스트 메서드의 실행 결과가 달라지면 안 된다. 그런 의미에서 테스트 메서드가 서로 필드를 공유한다거나 실행 순서를 가정하고 테스트를 작성하지 말아야 한다.









### 추가 애너테이션: @DisplayName, @Disabled

테스트 실행 결과를 보면 **테스트 메서드 이름**을 사용해서 테스트 결과를 표시한다.

**@DisplayName** 애너테이션을 사용하면 테스트 결과에 애너테이션에 지정한 값을 표시한다.



특정 테스트를 실행하기 싫을 때는 **@Disabled** 애너테이션을 사용한다. **JUnit**은 **@Disabled** 애너테이션이 붙은 클래스나 메서드는 테스트 실행 대상에서 제외한다.









### 모든 테스트 실행하기

코드를 원격 레포지토리에 푸시하거나 코드를 빌드해서 운영 환경에 배포하기 전에는 모든 테스트를 실행해서 깨지는 테스트가 없는지 확인한다. 



모든 테스트를 실행하는 방법은 아래와 같다.

- **mvn test (wrapper를 사용하는 경우 mvnw test)**
- **gradle test (wrapper를 사용하는 경우 gradlew test)**