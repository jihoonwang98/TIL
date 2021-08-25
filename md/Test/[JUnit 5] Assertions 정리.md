# [JUnit 5] Assertions 정리



> 참고: https://howtodoinjava.com/junit5/junit-5-assertions-examples/



이번 포스팅에서 정리할 **JUnit Jupiter assertion**들은 모두 `org.junit.jupiter.Assertions` 클래스에 static method로 정의되어 있다.



## 1. `assertEquals()`, `assertNotEquals()`

말 그대로 동등한지 안 동등한지 assert 한다.

동등성 비교할 때는 `equals()` 메서드를 사용한다.





### `assertEquals()` 메서드

`assertEquals()` 메서드는 아래와 같이 3가지 메서드 형태가 존재한다.

```java
public static void assertEquals(int expected, int actual)
public static void assertEquals(int expected, int actual, String message)
public static void assertEquals(int expected, int actual, Supplier<String< messageSupplier)
```

- 첫번째가 예상값, 두번째가 실제값임!! 헷갈리기 쉬움
- 세번째 파라미터에는 실패할 시 출력할 메시지가 들어간다. 람다나 String을 넣어주면 된다.



위에서는`int`형 `assertEquals()` 메서드를 살펴봤는데, 

이 외에도 `int`, `short`, `char`, ... 등등 여러가지의 자료형을 지원하는 메서드들이 overload 되어 있다.





### `assertEquals()` 메서드 예시

```java
@Test
void testCase() {
    //Test will pass
    Assertions.assertEquals(4, Calculator.add(2, 2));
     
    //Test will fail 
    Assertions.assertEquals(3, Calculator.add(2, 2), "Calculator.add(2, 2) test failed");
     
    //Test will fail 
    Supplier<String> messageSupplier  = () -> "Calculator.add(2, 2) test failed";
    Assertions.assertEquals(3, Calculator.add(2, 2), messageSupplier);
}
```





### `assertNotEquals()` 메서드

`assertNotEquals()` 메서드는 `assertEquals()` 메서드랑 비슷하다.

근데 한 가지 다른 점은 얘는 다양한 자료형에 대한 메서드를 overload 해놓지 않았다.

아래처럼 오직 `Object`형 객체만 받을 수 있다.

```java
public static void assertNotEquals(Object expected, Object actual)
public static void assertNotEquals(Object expected, Object actual, String message)
public static void assertNotEquals(Object expected, Object actual, Supplier<String> messageSupplier)
```







## 2. `assertArrayEquals()` 

배열이 동등한지 assert한다.

`assertEquals()`랑 비슷하다.

`boolean[]`, `char[]`, `int[]`, ... 등등 여러가지 자료형에 대해 overload 되어 있다.



### 메서드 형태

```java
public static void assertArrayEquals(int[] expected, int[] actual)
public static void assertArrayEquals(int[] expected, int[] actual, String message)
public static void assertArrayEquals(int[] expected, int[] actual, Supplier<String> messageSupplier)
```

- 얘도 위에 꺼랑 비슷하다.



### 예시

```java
@Test
void testCase() {
    //Test will pass
    Assertions.assertArrayEquals(new int[]{1,2,3}, new int[]{1,2,3}, "Array Equal Test");
     
    //Test will fail because element order is different
    Assertions.assertArrayEquals(new int[]{1,2,3}, new int[]{1,3,2}, "Array Equal Test");
     
    //Test will fail because number of elements are different
    Assertions.assertArrayEquals(new int[]{1,2,3}, new int[]{1,2,3,4}, "Array Equal Test");
}
```





## 3. `assertIterableEquals()` 

이번에는 `Iterable` 객체가 동등한지 assert 하는 메서드다.

예상 `Iterable`과 실제 `Iterable`의 값들이 **deeply equals** 한지 assert한다.



**deeply equals**란??

- collection의 각각의 원소가 동일한 것은 물론이고,
- collection의 원소의 개수와 순서까지 똑같아야 한다.





### 메서드 형태

```java
public static void assertIterableEquals(Iterable<?> expected, Iterable> actual)
public static void assertIterableEquals(Iterable<?> expected, Iterable> actual, String message)
public static void assertIterableEquals(Iterable<?> expected, Iterable> actual, Supplier<String> messageSupplier)
```





### 예시

```java
@Test
void testCase() {
     Iterable<Integer> listOne = new ArrayList<>(Arrays.asList(1,2,3,4));
     Iterable<Integer> listTwo = new ArrayList<>(Arrays.asList(1,2,3,4));
     Iterable<Integer> listThree = new ArrayList<>(Arrays.asList(1,2,3));
     Iterable<Integer> listFour = new ArrayList<>(Arrays.asList(1,2,4,3));
      
    //Test will pass
    Assertions.assertIterableEquals(listOne, listTwo);
     
    //Test will fail
    Assertions.assertIterableEquals(listOne, listThree);
     
    //Test will fail
    Assertions.assertIterableEquals(listOne, listFour);
}
```







## 4. `assertLinesMatch()` 

Line들이 매칭되는지 assert한다.

Line들이란 String으로 이루어진 List를 말한다.



Line들을 비교하는 로직은 다음과 같다

1. `expected.equals(actual)`을 체크한다. 
   - 참이면 나머지 스텝을 건너뛰고 다음 String을 비교하러 간다.
2. 1번 결과가 거짓이면 `expected`를 정규표현식으로 간주하고 `String.matches(String)` 메서드를 통해 체크한다.
   - 참이면 3번 스텝을 건너뛰고 다음 String을 비교하러 간다.
3. 2번 결과가 거짓이면 `expected` 라인이 **fast-forward marker**인지 체크한다. 만약 그렇다면 그냥 비교 안하고 넘어가는 것 같다.



**fast-forward marker**란,

- `>>`로 시작하고 `>>`로 끝나는 String을 말한다.
- 최소 4글자를 포함하고 있어야 한다. (>>가 앞 뒤로 두번 들어가면 최소 네 글자니까..)
- fast-forward 사이에 있는 모든 Character들은 버려진다.





```java
@Test
public void test() {
    List<String> list1 = Arrays.asList(">> skip >>", "string");
    List<String> list2 = Arrays.asList("아무글자", "string");

    org.junit.jupiter.api.Assertions.assertLinesMatch(list1, list2);
}
```

- 위의 결과가 true가 된다.





## 5. `assertNotNull()`, `assertNull()`

말 그대로 `null`인지 아닌지 assert 한다.





### 메서드 형태

```java
public static void assertNotNull(Object actual)
public static void assertNotNull(Object actual, String message)
public static void assertNotNull(Object actual, Supplier<String> messageSupplier)

public static void assertNull(Object actual)
public static void assertNull(Object actual, String message)
public static void assertNull(Object actual, Supplier<String> messageSupplier)
```





### 예시

```java
@Test
void testCase() 
{    
    String nullString = null;
    String notNullString = "howtodoinjava.com";
     
    //Test will pass
    Assertions.assertNotNull(notNullString);
     
    //Test will fail
    Assertions.assertNotNull(nullString);
     
    //Test will pass
    Assertions.assertNull(nullString);
 
    // Test will fail
    Assertions.assertNull(notNullString);
}
```







## 6. `assertNotSame()`, `assertSame()`

엥 아까 `assertEquals()`란 메서드를 본 것 같은데 비스무리한 놈이 또 나왔다..

차이점이 뭘까?

`assertSame()` 메서드는 expected 객체와 actual 객체가 '같은' 객체인지 assert 한다.



여기서 **같다**(same)는 의미는 **동등하다**(equals)와 구분해야 한다.

객체가 같다는 건 객체가 저장된 주소값이 같다는 것을 의미한다.

객체가 동등하다는 것은 객체가 가지고 있는 프로퍼티 값이 서로 동일하다는 것을 의미한다.



### 메서드 형태

```java
public static void assertNotSame(Object expected, Object actual)
public static void assertNotSame(Object expected, Object actual, String message)
public static void assertNotSame(Object expected, Object actual, Supplier<String> messageSupplier)

public static void assertSame(Object expected, Object actual)
public static void assertSame(Object expected, Object actual, String message)
public static void assertSame(Object expected, Object actual, Supplier<String> messageSupplier)
```





### 예시

```java
@Test
void testCase() {
    String str1 = "hello";
    String str2 = str1;
    String str3 = new String("hello");

    //Test will pass
    assertSame(str1, str2);

    //Test will pass
    assertNotSame(str1, str3);
}
```





## 7. `assertTimeout()`, `assertTimeoutPreemptively()`

`assertTimeout()` and `assertTimeoutPreemptively()` both are used to test long running tasks. If given task inside testcase takes more than specified duration, then test will fail.



Only different between both methods is that in `assertTimeoutPreemptively()`, execution of the `Executable` or `ThrowingSupplier` will be preemptively aborted if the timeout is exceeded. In case of `assertTimeout()`, `Executable` or `ThrowingSupplier` will NOT be aborted.



### 메서드 형태

```java
public static void assertTimeout(Duration timeout, Executable executable)
public static void assertTimeout(Duration timeout, Executable executable, String message)
public static void assertTimeout(Duration timeout, Executable executable, Supplier<String> messageSupplier)
public static void assertTimeout(Duration timeout, ThrowingSupplier<T> supplier, String message)
public static void assertTimeout(Duration timeout, ThrowingSupplier<T> supplier, Supplier<String> messageSupplier)
```





### 예시

```java
@Test
void testCase() {
 
    //This will pass
    Assertions.assertTimeout(Duration.ofMinutes(1), () -> {
        return "result";
    });
     
    //This will fail
    Assertions.assertTimeout(Duration.ofMillis(100), () -> {
        Thread.sleep(200);
        return "result";
    });
     
    //This will fail
    Assertions.assertTimeoutPreemptively(Duration.ofMillis(100), () -> {
        Thread.sleep(200);
        return "result";
    });
}
```





## 8. `assertTrue()`, `assertFalse()`



### 메서드 형태

```java
public static void assertTrue(boolean condition)
public static void assertTrue(boolean condition, String message)
public static void assertTrue(boolean condition, Supplier<String> messageSupplier)
public static void assertTrue(BooleanSupplier booleanSupplier)
public static void assertTrue(BooleanSupplier booleanSupplier, String message)
public static void assertTrue(BooleanSupplier booleanSupplier, Supplier<String> messageSupplier)

public static void assertFalse(boolean condition)
public static void assertFalse(boolean condition, String message)
public static void assertFalse(boolean condition, Supplier<String> messageSupplier)
public static void assertFalse(BooleanSupplier booleanSupplier)
public static void assertFalse(BooleanSupplier booleanSupplier, String message)
public static void assertFalse(BooleanSupplier booleanSupplier, Supplier<String> messageSupplier)
```





### 예시

```java
@Test
void testCase() {
 
    boolean trueBool = true;
    boolean falseBool = false;
 
    Assertions.assertTrue(trueBool);
    Assertions.assertTrue(falseBool, "test execution message");
    Assertions.assertTrue(falseBool, AppTest::message);
    Assertions.assertTrue(AppTest::getResult, AppTest::message);
     
    Assertions.assertFalse(falseBool);
    Assertions.assertFalse(trueBool, "test execution message");
    Assertions.assertFalse(trueBool, AppTest::message);
    Assertions.assertFalse(AppTest::getResult, AppTest::message);
}
 
private static String message () {
    return "Test execution result";
}
 
private static boolean getResult () {
    return true;
}
```





## 9. `assertThrows()` 

파라미터로 넘겨진 `Executable`을 수행했을 때 `expectedType` 형태의 예외가 던져짐을 assert 한다.



### 메서드 형태

```java
public static <T extends Throwable> T assertThrows(Class<T> expectedType, Executable executable)
```





### 예시

```java
@Test
void testCase() {
 
    Throwable exception = Assertions.assertThrows(IllegalArgumentException.class, () -> {
        throw new IllegalArgumentException("error message");
    });
}
```







## 10. `fail()`

`fail()` 메서드는 그냥 테스트를 실패시킨다.



### 메서드 형태

```java
public static void fail(String message)
public static void fail(Throwable cause)
public static void fail(String message, Throwable cause)
public static void fail(Supplier<String> messageSupplier)
```



### 예시

```java
public class AppTest {
    @Test
    void testCase() {
 
        Assertions.fail("not found good reason to pass");
        Assertions.fail(AppTest::message);
    }
     
    private static String message () {
        return "not found good reason to pass";
    }
}
```
