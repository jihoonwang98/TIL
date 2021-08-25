# Mockito(@Mock, @Spy, @Captor, @InjectMocks)



> 참고 사이트: https://www.baeldung.com/mockito-annotations
>
> 번역이 매끄럽지 않을 수 있음에 주의...





## 1. Overview

이번 포스팅에서는 **Mockito Library**의 **annotation**들(**@Mock, @Spy, @Captor, @InjectMocks)**에 대해 살펴본다.







## 2. Enable Mockito Annotations

**Mockito Annotation**들을 사용 가능하게 하는 방법들을 살펴보자.





### 2.1 MockitoJUnitRunner

첫 번째 방법은 **JUnit test class** 위에 **@RunWith(MockitoJUnitRunner.class)** 어노테이션을 붙여주는 것이다.

```java
@RunWith(MockitoJUnitRunner.class)
public class MockitoAnnotationTest {
	...
}
```







### 2.2 MockitoAnnotations.initMocks()

**Mockito annotation**을 사용 가능하게 하는 **programmatical**한 방식으로는 **MockitoAnnotations.initMocks()**를 호출하는 것이 있다.

```java
@BeforeEach
public void init() {
    MockitoAnnotations.initMocks(this);
}
```







### 2.3 MockitoJUnit.rule()

마지막으로, **MockitoJUnit.rule()**을 사용하는 방법이 있다.

```java
public class MockitoInitWithMockitoJUnitRuleUnitTest {
    @Rule
    public MockitoRule initRule = MockitoJUnit.rule();
    ...
}
```



이 방법을 쓰는 경우, **MockitoRule**을 반드시 **public**으로 설정해야 한다.









## 3. @Mock 어노테이션

**Mockito** 라이브러리에서 가장 많이 쓰이는 어노테이션인 **@Mock**에 대해서 알아보자. 우리는 **@Mock** 어노테이션을 사용해 ***mocked* instance**들을 생성하고 주입할 수 있다. (**Mockito.mock** 메서드를 통해 manual하게 호출도 가능하다.)



**@Mock(mock() 메서드)** -> **mock 객체**를 만들어 반환한다.





아래의 예에서 우선 manual한 방법으로, ***mocked*** **ArrayList**를 생성해보자.

```java
@Test
public void whenNotUseMockAnnotation_thenCorrect() {
    List mockList = Mockito.mock(ArrayList.class);
    
    mockList.add("one");
    Mockito.verify(mockList).add("one");
    assertEquals(0, mockList.size());
    
    Mockito.when(mockList.size()).thenReturn(100);
    assertEquals(100, mockList.size());
}
```





위의 코드를 **@Mock** 어노테이션을 통해 간결하게 나타내보면 아래와 같다.

```java
import static org.mockito.Mockito.*;

@Mock
List<String> mockedList;

@Test
public void whenUseMockAnnotation_thenMockIsInjected() {
    List mockList = mock(ArrayList.class);
    
    mockedList.add("one");
    verify(mockedList).add("one");
    assertEquals(0, mockedList.size());
    
    when(mockList.size()).thenReturn(100);
    assertEquals(100, mockList.size());
}
```



```
@Mock
List<String> mockedList;

List<String> mockedList = Mockito.mock(ArrayList.class)
```





### 4. @Spy 어노테이션

**@Spy** 어노테이션을 통해 실제 존재하는 instance를 spy 할 수 있다.



아래의 예에서는 **@Spy** 어노테이션을 쓰지 않고 옛날 방식으로 List의 **spy**를 만들어보겠다.

```java
import static org.mockito.Mockito.*;

@Test
public void whenNotUseSpyAnnotation_thenCorrect() {
    List<String> spyList = spy(new ArrayList<String>());
    
    spyList.add("one");
    spyList.add("two");
    
    verify(spyList).add("one");
    verify(spyList).add("two");
    
    assertEquals(2, spyList.size());
    
    doReturn(100).when(spyList).size();
    assertEquals(100, spyList.size());
}
```



위의 예를 **@Spy** 어노테이션을 이용해서 만들어보자.

```java
import static org.mockito.Mockito.*;

@Spy
List<String> spiedList = new ArrayList<String>();

@Test
public void whenUseSpyAnnotation_thenSpyIsInjectedCorrectly() {
    spiedList.add("one");
    spiedList.add("two");
    
    verify(spiedList).add("one");
    verify(spiedList).add("two");
    
    assertEquals(2, spiedList.size());
    
    doReturn(100).when(spiedList).size();
    assertEquals(100, spiedList.size());
}
```



Note how, as before – we're interacting with the spy here to make sure that it behaves correctly. In this example we:

- Used the **real** method *spiedList.add()* to add elements to the *spiedList*.
- **Stubbed** the method *spiedList.size()* to return *100* instead of *2* using *Mockito.doReturn()*.





### 5. @Captor 어노테이션

**@Captor** 어노테이션을 사용하면 **ArgumentCaptor** *instance*를 생성할 수 있다.



아래의 예에서는 **@Captor** 어노테이션을 사용하지 않고 옛날 방식으로 **ArgumentCaptor**를 만들어본다.

```java
import static org.mockito.Mockito.*;

@Test
public void whenNotUseCaptorAnnotation_thenCorrect() {
    List mockList = mock(List.class);
    ArgumentCaptor<String> arg = ArgumentCaptor.forClass(String.class);

    mockList.add("one");
    verify(mockList).add(arg.capture());

    assertEquals("one", arg.getValue());
}
```



이번에는 **@Captor** 어노테이션을 사용해 코드를 작성해보자.

```java
import static org.mockito.Mockito.*;

@Mock
List mockedList;

@Captor
ArgumentCaptor argCaptor;

@Test
public void whenUseCaptorAnnotation_thenTheSam() {
    mockedList.add("one");
    verify(mockedList).add(argCaptor.capture());
    
    assertEquals("one", argCaptor.getValue());
}
```







### 6. @InjectMocks 어노테이션

**@InjectMocks** 어노테이션은 **mock field**들을 **tested object**에 자동적으로 주입해준다.



**@InjectMocks** -> **@Mock**이나 **@Spy** 객체를 자신의 멤버 필드와 일치하면 주입시킨다.



```java
public class MyDictionary {
    Map<String, String> wordMap;

    public MyDictionary() {
        wordMap = new HashMap<String, String>();
    }
    public void add(final String word, final String meaning) {
        wordMap.put(word, meaning);
    }
    public String getMeaning(final String word) {
        return wordMap.get(word);
    }
}
```



```java
import static org.mockito.Mockito.*;

@Mock
Map<String, String> wordMap;

@InjectMocks
MyDictionary dic = new MyDictionary();

@Test
public void whenUseInjectMocksAnnotation_thenCorrect() {
    when(wordMap.get("aWord")).thenReturn("aMeaning");

    assertEquals("aMeaning", dic.getMeaning("aWord"));
}
```



**@InjectMocks** 어노테이션을 통해 **MyDictionary** 객체에 **wordMap** *mock field*가 주입된다.





### 7. Injecting a Mock into a Spy

(생략)





### 8. Running into NPE While Using Annotation

(생략)





























