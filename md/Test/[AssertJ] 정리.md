# [AssertJ] 정리



> 참고:
>
> https://pjh3749.tistory.com/241
>
> https://www.baeldung.com/introduction-to-assertj



- Object Assertions
- Boolean Assertions
- Iterable/Array Assertions
- Character Assertions
- Class Assertions
- File Assertions
- Double / Float / Integer Assertions
- InputStream Assertions
- Map Assertions
- Throwable Assertions





## 1. AssertJ란?

AssertJ는 assertion들을 제공하는 라이브러리다.

JUnit의 assertion에 비해 가독성이 좋다고 한다.

```java
assertEquals(expected, actual);  // JUnit
assertThat(actual).isEqualTo(expected);  // AssertJ
```

AssertJ의 assertion 문이 더 자연스럽게 읽히는 것을 확인할 수 있다.





## 2. Assertion 작성하기

assertion을 작성하기 위해서는 테스트하고자 하는 객체를 `Assertions.assertThat()` 메서드에 넣으면서 시작한다.



## 3. Assertion Description

assertion이 수행될 때 상황을 설명하는 것은 중요하다.

이때 상황 설명을 `as()`라는 메서드로 지정할 수 있는데 

주의할 점은 **<u>assertion이 수행되기 전에 작성해야 한다</u>**는 점이다.



### 예시

```java
TolkienCharacter frodo = new TolkienCharacter("Frodo", 33, Race.HOBBIT); 

// 실패하는 테스트 예시 그리고 중요한 것은 as()를 assertion이전에 호출해야 한다! 
assertThat(frodo.getAge()).as("check %s's age", frodo.getName()).isEqualTo(100);
```

- 테스트가 실패하면 `as()` 메서드로 지정한 메시지가 출력된다.

  ```
  [check Frodo's age] expected:<100> but was:<33>
  ```

  



## 4. Object Assertions

```java
@Getter @Setter
public class Dog { 
    private String name; 
    private Float weight;
}

Dog fido = new Dog("Fido", 5.25);
Dog fidosClone = new Dog("Fido", 5.25);
```



위와 같은 코드가 있을 때, **equality(동등성) 비교**는 아래와 같은 assertion으로 할 수 있다.

```java
assertThat(fido).isEqualTo(fidosClone);
```

- 여기서는 `Dog` 클래스가 `Equals()` 메서드를 재정의하지 않았기 때문에 `==` 연산자로 비교하게 된다.

  이때 `fido`와 `fidosClone`은 다른 객체이므로 위의 assertion은 실패한다.





객체 참조값을 비교하는게 아니라 **객체의 content를 비교**하고 싶다면 아래와 같은 assertion을 할 수 있다.

```java
assertThat(fido).isEqualToComparingFieldByFieldRecursively(fidosClone);
```

- `fido`와 `fidosClone`은 field by field로 비교하면 같은 값을 가지므로 위의 assertion은 성공한다.



더 많은 object assertion들을 찾아 보고 싶다면, *AbstractObjectAssert* [documentation](https://joel-costigliola.github.io/assertj/core-8/api/org/assertj/core/api/AbstractObjectAssert.html)을 살펴보자..





## 5. Boolean Assertions

- `isTrue()`
- `isFalse()`



### 예시

```java
assertThat("".isEmpty()).isTrue();
```





## 6. Iterable/Array Assertions

- `contains()`
- `isNotEmpty()`
- `startsWith()`



더 많은 Iterable/Array assertion들을 찾아 보고 싶다면, *AbstractIterableAssert* [documentation](https://joel-costigliola.github.io/assertj/core-8/api/org/assertj/core/api/AbstractIterableAssert.html)을 살펴보자..



`filteredOn()`





## 7. Character Assertions







## 8. Class Assertions





## 9. File Assertions





## 10. Double/Float/Integer Assertions





## 11. InputStream Assertions





## 12. Map Assertions





## 13. Throwable Assertions







