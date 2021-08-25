# Mockito를 직접 사용해보자!



### mock(), @Mock, @InjectMocks

- @Mock
  - mock 객체를 만들어서 반환합니다.
- @InjectMocks
  - @Mock이나 @Spy 객체를 자신의 멤버 클래스와 일치하면 주입시킵니다.



- verify()
  - verify()를 이용하여 mock 객체에 대한 원하는 메서드가 특정 조건으로 실행되었는지 검증할 수 있다.
  - mock 작업이 수행되었는지 검증한다.
  - **verify(T mock).method();**
  - **verify(T mock, VerificationMode mode).method();**
  - VerificationMode의 종류는 아래와 같다.
    - atLeastOnce(): 적어도 한 번 수행했는지 검증
    - atLeast(int n): 적어도 n 번 수행했는지 검증
    - times(int n): 무조건 n번 수행했는지 검증 
    - atMost(int n): 최대 n번 수행했는지 검증
    - never(): 수행되지 않았는지 검증(수행했으면 오류)



```java
// static으로 import를 하게 되면 조금 더 깨끗하게 볼 수 있습니다.
import static org.mockito.Mockito.*;

// mock 처리를 합니다.
// Annotation, mock() Method 같은 표현입니다.
@Mock
List annotationMockedList
List mockedList = mock(List.class);


//using mock object
mockedList.add("one");
mockedList.clear();

//검증 - 성공
verify(mockedList).add("one"); // add("one")가 호출되었는지 검증합니다.
verify(mockedList).clear(); // clear()가 호출되었는지 검증합니다.
//검증 - 실패
verify(mockedList).add("two"); // add("two")이 호출이 되지 않았기에 실패합니다.
```





### stubbing

- when()
  - when() 메서드는 Mock이 감싸고 있는 메서드가 호출되었을 때, Mock 객체의 메서드(행위)를 호출 선언할 때 사용한다.
  - when() 메서드는 OngoingStubbing이라는 인터페이스를 리턴한다.



- OngoingStubbing 인터페이스

  - thenAnswer(Answer<?> answer): Answer라는 인터페이스를 구현하며, 원하는 작업을 수행할 수 있다.
  - thenCallRealMethod(): 해당 메서드가 구현되어 있다면, 실제 메서드를 호출한다.
  - thenReturn(T value): 지정한 값을 리턴
  - thenReturn(T value, T... values): 지정되어 있는 값을 순차적으로 리턴한다.
  - thenThrow(java.lang.Throwable... throwables): 예외를 야기시키는 Throwable 객체를 지정한다.

  

```java
// 구체적인 클래스를 mock 처리합니다.
LinkedList mockedList = mock(LinkedList.class);

//stubbing
when(mockedList.get(0)).thenReturn("first"); // get(0)이 호출되면 "first"를 반환합니다.
when(mockedList.get(1)).thenThrow(new RuntimeException()); // get(1)이 호출되면 RuntimeException 에러를 발생합니다.

System.out.println(mockedList.get(0)); // "first"
System.out.println(mockedList.get(1)); // RuntimeException()
System.out.println(mockedList.get(999)); // 999에 대한 stub 처리를 하지 않았기 때문에 null값이 return됩니다.
```



## Stubbing void methods with exceptions

void를 리턴형식으로 갖는 메소드는 stub하는 법이 약간 다르다. 위에서 설명한 일반 stubbing은 `when(mock.method()).thenReturn(value)` 형식인데, `mock.method()`이 void값을 가지면 `when(void)`처럼 되기 때문에 Java 문법에 맞지않기 때문이다. 대신 `doThrow(new Exception()).when(mock).method()`처럼 사용한다.



```java
doThrow(new RuntimeException()).when(mockedList).clear();

mockedList.clear(); // RuntimeException
```







### 매개변수 검증

```java
// 모든 Integer 타입 매개변수를 받을 경우 "element"를 돌려줍니다.
when(mockedList.get(anyInt())).thenReturn("element");

//커스텀 Matcher를 사용하여 stubing합니다.
when(mockedList.contains(argThat(isValid()))).thenReturn("element");

// "element"
System.out.println(mockedList.get(999));

// 검증 - Integer 타입 매개변수를 받으며 호출되었는지 검증합니다.
verify(mockedList).get(anyInt());
// 검증 - argument matchers
verify(mockedList).add(argThat(someString -> someString.length() > 5));

// 값을 검증할때에는 eq()로 감싸야 호출해야 합니다.
verify(mock).someMethod(anyInt(), anyString(), eq("third argument"));
// 그렇지 않을 경우 실패하게 됩니다.
verify(mock).someMethod(anyInt(), anyString(), "third argument");
```



```java
when(mockedList.get(anyInt())).thenReturn("int");
when(mockedList.add(anyFloat())).thenReturn(true);
when(mockedList.add(anyString())).thenReturn(true);

System.out.println(mockedList.get(999)); // int
System.out.println(mockedList.add(3.3)); // true
System.out.println(mockedList.add("string")); // true

verify(mockedList).get(anyInt());
verify(mockedList).add(anyFloat());
verify(mockedList).add(eq("string"));
```





### Verifying exact number of invocations / at least / at most / never, 호출 시간 검증



```java
@Test
public void verifyTest() {
	@SuppressWarnings("unchecked")
    List<String> testMock = mock(ArrayList.class);
    testMock.add("1");
    testMock.add("2");
    testMock.add("3");
    
    // add 메서드가 최소 1번 호출되었음을 검증
    verify(testMock, atLeastOnce()).add(anyString());
    verify(testMock, atLeast(3)).add(anyString());
    verify(testMock, atMost(3)).add(anyString());
    verify(testMock, times(3)).add(anyString());
    
    // add("1")가 1번 호출되었음을 검증
    verify(testMock, times(1)).add("1");
    verify(testMock, times(1)).add("2");
    verify(testMock, times(1)).add("3");
    
    
    // add("4")가 수행되지 않았음을 검증.
    verify(testMock, never()).add("4");
    
    
    // mock 순서 다르게 verify 정의하면 오류난다.
    InOrder inOrder = inOrder(testMock);
    
    inOrder.verify(testMock).add("1");
    inOrder.verify(testMock).add("2");
    inOrder.verify(testMock).add("3");
}
```



```java
mockedList.add("once");

mockedList.add("twice");
mockedList.add("twice");

mockedList.add("three times");
mockedList.add("three times");
mockedList.add("three times");

verify(mockedList).add("once"); // times(1) 기본값
verify(mockedList, times(1)).add("once");

verify(mockedList, times(2)).add("twice");
verify(mockedList, times(3)).add("three times");

verify(mockedList, never()).add("never happened"); // 호출된 적 없음

verify(mockedList, atLeastOnce()).add("three times"); // 최소 한 번
verify(mockedList, atLeast(2)).add("five times"); // 최소 두 번
verify(mockedList, atMost(5)).add("three times"); // 최대 다섯 번
```





## Verification in order

메소드에 호출 시 넘긴 값 뿐만이 아니라 메소드 호출 순서도 검증가능하고, `inOrderObj.verify(mock.method())`같은 형식으로 쓰면 된다. 여러 mock의 메소드 호출 순서도 검증할 수 있다.



```java
// single mock
List singleMock = mock(List.class);

singleMock.add("first");
singleMock.add("second");

InOrder inOrder = inOrder(singleMock);

inOrder.verify(singleMock).add("first");
inOrder.verify(singleMock).add("second");

// multiple mocks
List firstMock = mock(List.class);
List secondMock = mock(List.class);

//using mocks
firstMock.add("first");
secondMock.add("second");

InOrder inOrder = inOrder(firstMock, secondMock); // pass multiple mocks to verify

inOrder.verify(firstMock).add("first");
inOrder.verify(secondMock).add("second");
```



```java
List singleMock = mock(List.class);

singleMock.add("was added first");
singleMock.add("was added second");

// 중위 순회 Case
InOrder inOrder = inOrder(singleMock);

// "was added first"부터 검증후 "was added second"를 검증하여 순서에 대한 검증을 진행합니다.
inOrder.verify(singleMock).add("was added first");
inOrder.verify(singleMock).add("was added second");
```



## Finding Redundant Invocations

mock의 행동이 모두 검증 되었는지 확인한다. 모든 테스트에 사용할 필요는 없고 정말 필요한 곳에만 사용하길 추천한다.



```java
mockedList.add("one");
mockedList.add("two");

verify(mockedList).add("one");

verifyNoMoreInteractions(mockedList); // fail here
```





### 연속적인 Stubbing

```java
when(mock.someMethod("some arg"))
.thenThrow(new RuntimeException()) // 첫번째 Return
.thenReturn("foo");                // 두번째 Return

//첫번째 call: throws runtime exception:
mock.someMethod("some arg");

//두번째 call: "foo"
System.out.println(mock.someMethod("some arg"));

//가장 마지막 Stubbing한 값을 Return 합니다.
System.out.println(mock.someMethod("some arg"));

// 호출한 순서대로 결과값을 Return 합니다.
when(mock.someMethod("some arg"))
.thenReturn("one", "two", "three");
```





### Spying, @Spy

- Spy를 통해 실제 객체를 생성하고 필요한 부분에만 mock처리하여 검증을 진행할 수 있습니다.

```java
List list = new LinkedList();
List spy = spy(list);

// Method에 대한 stubbing
when(spy.size()).thenReturn(100);

// 실제 객체의 Method를 호출합니다.
spy.add("one");
spy.add("two");

// "one"
System.out.println(spy.get(0));

// Stubbing된 Method의 결과 : 100
System.out.println(spy.size());


// Spy의 주의할 점.
List list = new LinkedList();
List spy = spy(list);

// 실제 객체에 아무것도 없기때문에 when을 호출하게 되면 IndexOutofBoundsException이 발생합니다.
when(spy.get(0)).thenReturn("foo");

// 그렇기 때문에 doReturn으로 stubbing을 진행합니다.
doReturn("foo").when(spy).get(0);
```





## @Mock Annotation

Mock 생성은 쓸데없이 반복적이다. `@Mock` Annotation을 사용하면 좀 더 간단하게 mock을 할 수 있고 코드가독성도 좋아진다. `MockitoAnnotations.initMocks()` 또는 `@RunWith(MockitoJUnitRunner.class)`를 사용하면 자동으로 초기화를 해준다.



```java
public class ListTest {
  @Mock private List mockedList;

  @Before
  public void initMocks() {
      MockitoAnnotations.initMocks(this); // mock all the field having @Mock annotation
  }
  
  @Test
  public void test() {
    // test here
  }
}

// also with @RunWith annotation
@RunWith(MockitoJUnitRunner.class)
public class ListTest {
  @Mock private List mockedList;
 
  @Test
  public void test() {
    // test here
  }
}
```



## Iterator-style stubbing

가끔씩 하나의 메소드에서 순차적으로 여러 다른 값을 돌려주거나 예외를 발생시켜야 할 때가 있다. 한 번 stub한 값은 바뀌지 않지만 다음과 같이 매번 호출할 때 마다 다른 행동을 하도록 정할 수 있다.



```java

// chaining
when(mock.someMethod(anyString()))
  .thenReturn("foo")
  .thenReturn("bar")
  .thenThrow(new RuntimeException());

System.out.println(mock.someMethod("some arg")); // foo
System.out.println(mock.someMethod("some arg")); // bar
System.out.println(mock.someMethod("some arg")); // Runtime Exception

// multi arguments
when(mock.someMethod(anyString()))
  .thenReturn("one", "two");

System.out.println(mock.someMethod("some arg")); // one
System.out.println(mock.someMethod("some arg")); // two
System.out.println(mock.someMethod("some arg")); // two - 마지막 stub한 값으로 고정
```





## Stubbing with callbacks

이때까지의 예제에서는 stub 할 때 모두 특정값을 넣었다. 만약 mock의 상태나 메소드 인자값에 따라 다른 값을 돌려주게 하게 만들고 싶다면 어떻게 해야할까. `Answer<?>` 클래스를 사용하면 가능하다. 하지만 아주 특별한 상황이 아니라면 크게 사용할 일은 없을 듯하다.



```java
when(mockedList.get(anyInt())).thenAnswer(new Answer<Integer>() {
  public Integer answer(InvocationOnMock invocation) {
    Object[] args = invocation.getArguments(); // arguments
    List mock = (List)invocation.getMock(); // mock itself
    int result = (Integer)args[0] + 1;
    return result;
  }
});

System.out.println(mockedList.get(1)); // called with argument: 2
```



이 예제에서는 넘겨 받은 값에다가 1을 더해서 넘겨주도록 했다.



## Spying on real objects

`spy()`는 진짜 인스턴스를 mock하는 것이다. 당연히 spy된 인스턴스를 stub 할 수 있다.



```java
List list = new LinkedList();
List spy = spy(list);

when(spy.size()).thenReturn(100); // stubbing

spy.add("one");
spy.add("two");

System.out.println(spy.get(0)); // one
System.out.println(spy.size()); // 100

verify(spy).add("one");
verify(spy).add("two");

// Wrong use case
when(spy.get(10)).thenReturn("foo"); // IndexOutOfBoundsException

// use doReturn() instead
doReturn("foo").when(spy).get(0);
```

단, 어떤 경우는 stub 할 때 조심해야 한다. 위의 `when(spy.get(10))`에서는 진짜 인스턴스의 메소드를 호출하기 때문에 IndexOutOfBountException이 발생하게 된다. 이럴 경우 `doReturn()`를 사용해서 문제를 회피할 수 있다.





## Argument Matcher

이전의 예제에서 말했던 `anyInt()`, `anyString()`의 커스터마이즈 버전이다. stub, verification 용으로 쓸 수 있다. 클래스를 상속해서 `matches()`를 직접 구현하면 된다. 여러 곳에 재사용할 일이 경우 사용하면 좋다.

```java
class ListOfTwoElements extends ArgumentMatcher<List> {
  @Override
  public boolean matches(Object argument) {
    List list = (List)argument;
    return list.size() == 2;
  }
}

List mock = mock(List.class);
when(mock.addAll(argThat(new ListOfTwoElements()))).thenReturn(true);

mock.addAll(Arrays.asList("one", "two")); // true

verify(mock).addAll(argThat(new ListOfTwoElements()));
```



## Capturing arguments for further assertions

Argument Matcher와 비슷한 기능을 가지고 있는 Argument Captor라는 것도 있다. Argument Matcher와 다른 점은 따로 클래스를 만들 필요가 없다는 점, 그리고 verification에만 사용할 수 있다는 점이다.

```java
List mock = mock(List.class);
mock.addAll(Arrays.asList("one", "two"));  // false

ArgumentCaptor<List> argument = ArgumentCaptor.forClass(List.class);
verify(mock).addAll(argument.capture());
Assert.assertTrue(argument.getValue().size() == 2);
```





## Verification with timeout

실행시간이 중요한 메소드라면 `timeout()`을 사용해서 검증할 수 있다. `times()`나 `atLeast()`와도 같이 사용할 수 있다.



```java
verify(mock, timeout(100)).size();
verify(mock, timeout(100).times(2)).size();
verify(mock, timeout(100).atLeast(2)).size();
```





## One-liner stubs

체이닝(chaining)을 이용해서 mock 생성과 stub 까지 한 줄에 만들 수 있다. 마지막에 `getMock()`이 중요한 포인트. 코드가독성도 좋은 편이다.



```java
List mock = when(mock(List.class).get(0)).thenReturn(1).thenReturn(2).thenThrow(Exception.class).getMock();
```





> https://softarchitecture.tistory.com/64
>
> https://nesoy.github.io/articles/2018-09/Mockito
>
> https://bestalign.github.io/2016/07/08/intro-mockito-1/