# Mockito’s Mock Methods



> **[참고]**: Mockito’s Mock Methods
>
> https://www.baeldung.com/mockito-series
>
> 의역과 오역에 주의...





## 1. Overview

이번 포스팅은 ***Mockito* API**의 표준 **static *mock* method**들의 다양한 용례들에 대해 살펴본다.



아래의 *MyList* 클래스는 여러 개의 test case들에서 mocked될 collaborator다.

```java
public class MyList extends AbstractList<String> {
    @Override
    public String get(int index) {
        return null;
    }

    @Override
    public int size() {
        return 1;
    }
}
```







## 2. Simple Mocking

아래의 메서드의 파라미터에 mocking하고 싶은 클래스를 전달하면 mock 객체가 생성되어 반환된다.

```java
public static <T> T mock(Class<T> classToMock)
```



위의 메서드를 사용해서 class를 mocking하고 기댓값을 설정해보자.

```java
MyList listMock = mock(MyList.class);
when(listMock.add(anyString())).thenReturn(false);
```



그다음에는 mock 객체의 method를 실행해보자.

```java
boolean added = listMock.add(randomAlphabetic(6));
```



> RandomStringUtils.randomAlphabetic(count)   [org.apache.commons.lang3.RandomStringUtils]
>
> - 영문 대소문자를 count만큼 random으로 생성해준다.
> - count: 문자열 길이





아래의 코드는 mock 객체의 *add* 메서드가 호출되었음을 확인하고,

그 호출이 우리가 전에 세팅한 값과 일치하는 값을 반환하는 것을 확인한다.

```java
verify(listMock).add(anyString());
assertThat(added, is(false));
```







## 3. Mocking With Mock's Name

mocking하고 싶은 **클래스**와 함께 생성될 **mock 객체의 이름**을 지정해줄 수도 있다.

```java
public static <T> T mock(Class<T> classToMock, String name)
```



일반적으로 mock 객체의 이름은 잘 돌아가는 코드에서는 하는 일이 없다.

하지만 **<u>debugging할 때</u>**는 유용하다. 

왜냐면 verification error들을 추적할 때 mock 객체의 이름이 사용되기 때문이다.



실패한 verification에서 던져진 exception의 메시지에 우리가 제공한 mock 객체의 이름이 포함되어 있는지를 확인하기 위해서,

우리는 JUnit의 *TestRule* 인터페이스의 구현체인 *ExpectedException*을 이용할 것이다.



아래처럼 테스트 클래스에 *ExpectedException*을 포함시켜라.

```java
@Rule
public ExpectedException thrown = ExpectedException.none();
```

이 rule은 test method들로부터 던져진 exception들을 처리하는데 사용될 것이다.



아래의 코드에서, 우리는 *MyList* 클래스의 mock 객체를 만들고 *myMock*이라는 이름을 지어줬따.

```java
MyList listMock = mock(MyList.class, "myMock");
```



그 후에, mock 객체의 메서드의 반환 기대 값을 설정하고 메서드를 수행한다.

```java
when(listMock.add(anyString())).thenReturn(false);
listMock.add(randomAlphabetic(6));
```



우리는 실패하는 verification을 작성할 것이고,

이 verification은 mock 객체에 대한 정보를 포함하고 있는 메시지와 함께 exception을 던져야 한다.

이를 위해서는, exception에 대한 expectation들이 먼저 세팅되어야 한다.

```java
thrown.expect(TooLittleActualInvocations.class);
thrown.expectMessage(containsString("myMock.add"));
```





아래의 verification은 실패할 것이다.

```java
verify(listMock, times(2)).add(anyString());
```





그리고 던져진 exception에 대한 메시지는 아래와 같다.

```java
org.mockito.exceptions.verification.TooLittleActualInvocations:
myMock.add(<any>);
Wanted 2 times:
at com.baeldung.mockito.MockitoMockTest
  .whenUsingMockWithName_thenCorrect(MockitoMockTest.java:...)
but was 1 time:
at com.baeldung.mockito.MockitoMockTest
  .whenUsingMockWithName_thenCorrect(MockitoMockTest.java:...)
```



보시다싶이, mock 객체의 이름이 exception message에 포함되어 

verification이 실패했을 때 실패 요인을 찾는 데 유용하다.







## 4. Mocking With Answer

클래스와 함께 Answer 구현체를 같이 넘겨주면 default answer를 설정할 수 있다.

```java
public static <T> T mock(Class<T> classToMock, Answer defaultAnswer)
```



우선, *Answer* 인터페이스의 구현체 정의부터 살펴보자.

```java
class CustomAnswer implements Answer<Boolean> {
    @Override
    public Boolean answer(InvocationOnMock invocation) throws Throwable {
        return false;
    }
}
```



위의 *CustomAnswer* 클래스는 mock 객체를 생성할 때 사용된다.

```java
MyList listMock = mock(MyList.class, new CustomAnswer());
```



만약 우리가 method에 대한 expectation을 설정하지 않는다면, 

default answer가 *CustomAnswer*에서 설정된 값으로 반환된다.





이를 증명하기 위해, expectation setting 과정을 건너뛰고 메서드를 바로 실행시켜보자.

```java
boolean added = listMock.add(randomAlphabetic(6));
```



아래의 verification과 assertion은 *Answer* argument와 함께 생성한 mock 객체의 method가 예상한 대로 작동한다는 것을 보여준다.

```java
verify(listMock).add(anyString());
assertThat(added, is(false));
```





## 5. Mocking With MockSettings

(생략)







