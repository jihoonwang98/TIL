

# [Testing Spring Boot: Beginner To Guru] 2장 Test Driven Development By Example



## 1. TDD By Example Kent Beck



### Test Driven Development By Example

- 책 이름임
- Published in 2002 by Kent Beck
- 소프트웨어 테스트 방법론에 있어서 'iconic'한 책으로 여겨진다.



### About Kent Beck

- Original 17 signatory of the Agile Manifesto(성명서)





### Kent Beck 왈

- If I have the same logic in two places, I work with the design to understand how I can have only one copy. Designs without duplication tend to be easy to change.
- Don't make more versions of your source code. Rather than add more code bases, fix the underlying design problem that is preventing you from running from a single code base.
- If there are forms of testing, like stress and load testing, that find defects after development is "complete", bring them into the development cycle. Run load and stress tests continuously and automatically.





### Objective of this section

- See how to evolve the simple example:
  - Start with - it compiles
  - Know it will not be complete
  - Use TDD to evolve to clean, quality code





### The Money Example

- Kent Beck의 저서에 나오는 예시
- TDD를 가르칠 때 많이 사용되는 예시다.
- In this example:
  - We will start with a very simple example, having tests before it can compile
  - Initial example <u>WILL NOT</u> follow commonly accepted practices
  - Code will evolve using TDD



[링크](https://github.com/springframeworkguru/tdd-by-example)에서 clone 받아서 시작..





## 2. TDD 1 - Multi Currency Money



### Multi Currency Money



![](https://lh6.googleusercontent.com/BazmzHCfOtM4CYoZzr-oUqhhnLbspoFpZ83fsHCIxwYwiS1sZYkm-i2hpuEqLN3nYVOXZFIbgRQ-1Qvey4fjdIH-g1wP_CHNLSd4J0PlDjpQpRlp529Kg_abyNuZo26p36X5Geqw)

- 단일 화폐 보고서
  - 여기서 total을 구하는 것은 쉽다.
  - Total = Shares * Price





![](https://lh4.googleusercontent.com/DbROtseNPlZ1wS72_cdWPLTiMXNus9xxVS_xjPYF40Cz4h8It5V-sP7Ebg7FYFYFo8rF62pS9_biGEHoajY_76nrui09ZO05M9z5ebgF4prHkeYDZA_7BRWphd1-SA8GlmvxnSPW)



- 다중 화폐 보고서

  - total을 구하려면 exchange rate(환율)가 필요하다.

  - Assume exchange rate of 1.5 from CHF to USD

  - Total (USD) = 1000 * 25 (USD) + 400 * 150 * (2/3) (CHF) 

    ​                     = 65,000 (USD)







### Technical Requirements For Report

- 두 개의 다른 화폐들을 더할 수 있어야 하고, 주어진 exchange rate에 따라 Total 결과 값을 변환할 수 있어야 한다.
- Need to be able to multiply an amount by a number (number of shares) and receive an amount.





### Setting Up Our TDD To-Do List

- Going forward we will setup a To-Do list.
  - Normal - On the list
  - **Bold** - What we are working on
  - ~~Strike Through~~ - Completed





### 현재 TDD To-Do List

- $5 + 10 CHF = $10 (with rate of 2:1)
- **$5 * 2 = $10**
- Make "amount" private
- Dollar side effects?
- Money Rounding?





### 코딩해보자

**MoneyTest 클래스**

```java
import static org.junit.jupiter.api.Assertions.assertEquals;

public class MoneyTest {

    @Test
    void testMultiplication() {
        Dollar five = new Dollar(5);
        five.times(2);
        assertEquals(10, five.amount);
    }
}
```

- 이 상태에서 Test를 돌려보면, 클래스를 찾을 수 없어서 컴파일이 안된다.
  - 우리의 목표는 이 테스트 코드를 컴파일이 되게 하는 것이다.
  - 컴파일을 되게 하기 위해서 아래와 같이 `Dollar` 클래스를 만들어 주자.



**Dollar 클래스**

```java
public class Dollar {

    int amount;

    public Dollar(int amount) {
    }

    void times(int multiplier) {
    }
}
```

- 이제 컴파일 오류가 사라졌으니 테스트를 돌려보자.

  - ```java
    org.opentest4j.AssertionFailedError: 
    Expected :10
    Actual   :0
    ```

  - 위와 같이 테스트가 실패한다.



```java
public class Dollar {

    int amount;

    public Dollar(int amount) {
        this.amount = amount;
    }

    void times(int multiplier) {
    }
}
```

- 그러면 이번엔 생성자를 바꿔보자.

  - ```java
    org.opentest4j.AssertionFailedError: 
    Expected :10
    Actual   :5
    ```

  - 그래도 테스트가 실패한다.



```java
public class Dollar {

    int amount;

    public Dollar(int amount) {
        this.amount = amount;
    }

    void times(int multiplier) {
    	this.amount *= 2;
    }
}
```

- 이번엔 times 메서드를 바꿔주자.

- 테스트가 통과한다.

- 이렇게 테스트를 통과시킨 후 리팩터링 과정을 거친다.

  - ```java
    public class Dollar {
    
        int amount;
    
        public Dollar(int amount) {
            this.amount = amount;
        }
    
        void times(int multiplier) {
        	this.amount *= multiplier;  // 리팩터링
        }
    }
    ```

    



### To-do List

- $5 + 10 CHF = $10 (with rate of 2:1)
- ~~$5 * 2 = $10~~
- Make "amount" private
- Dollar side effects?
- Money Rounding?





## 3. TDD 2 - Degenerate Objects

### TDD Cycle Review

1. **Write a Test** - Think about how the code should work.
2. **Make it Run** - Quickly get the code working. Make the test "Green".
3. **Make it Right** - Refactor the running code to quality, proper code.



- **Goal** - "Clean Code that works." - Ron Jefferies





### TDD To-Do List

- $5 + 10 CHF = $10 (with rate of 2:1)
- ~~$5 * 2 = $10~~
- Make "amount" private
- **Dollar side effects?** (이번 장에서 할 것)
- Money Rounding?





**MoneyTest 클래스**

```java
public class MoneyTest {

    @Test
    void testMultiplication() {
        Dollar five = new Dollar(5);
        five.times(2);
        assertEquals(10, five.amount);

        /* 코드 추가 */
        five.times(3);
        assertEquals(15, five.amount);
        
        // 우리의 Dollar Object는 변하고 있다(mutating). 그래서 테스트가 깨진다.
        // 해결책은 Dollar를 immutable Object로 만들고, 
        // Dollar에 multiplication 연산을 할 때, 새로운 Dollar Object를 리턴하는 것이다.
    }
}
```



아래와 같이 테스트를 고쳐보자.

```java
public class MoneyTest {

    @Test
    void testMultiplication() {
        Dollar five = new Dollar(5);
        Dollar product = five.times(2);
        assertEquals(10, product.amount);

        product = five.times(3);
        assertEquals(15, product.amount);
    }
}
```

`Dollar#times()` 메서드의 반환형이 `void`이기 때문에 컴파일 에러가 난다.

고치러 가자.



```java
public class Dollar {

    int amount;

    public Dollar(int amount) {
        this.amount = amount;
    }

    Dollar times(int multiplier) {
        return new Dollar(this.amount * multiplier);
    }
}
```

- 테스트가 통과한다.

- 객체의 상태를 바꾸는 것보다 불변 객체로 다루는 것이 더 나은 접근이다. 





### TDD To-Do List

- $5 + 10 CHF = $10 (with rate of 2:1)
- ~~$5 * 2 = $10~~
- Make "amount" private
- ~~Dollar side effects?~~ 
- Money Rounding?





## 4. TDD 3 - Equality for All

### Value Objects (VO)

- Dollar 객체는 이제 'value' object다.
- i.e. values "5" and "10" would be represented as two different objects
- 만약 다른 값을 표현하고 싶다면, 새로운 객체를 생성하면 된다.
  - 객체의 상태를 변화시키는 것이 아니다.
- Value Object를 선언하면 당신은 두 개의 Value Object가 동등(equal)한지 구별할 수 있는 능력이 반드시 필요합니다.
  - "5" Dollar 짜리 객체 두 개가 동등한지 구별할 수 있어야 함.
  - `EqualsAndHashCode` 필요
  - Java 객체는 default `equals()` 메서드를 가지고 있다. 
    - 근데 이 default `equals()` 메서드는 재정의를 해주지 않으면 `==` 연산자와 같은 역할을 한다.
    - 즉, 참조형 변수에 대해서는 메모리 주소 값을 비교한다.



### TDD To-Do List

- $5 + 10 CHF = $10 (with rate of 2:1)
- ~~$5 * 2 = $10~~
- Make "amount" private
- ~~Dollar side effects?~~ 
- Money Rounding?

- **equals()**





### 코딩해보자

**MoneyTest 클래스에 코드 추가**

```java
public class MoneyTest {

    @Test
    void testMultiplication() {
    	// ...
    }

    
    /* 코드 추가 */
    @Test
    void testEquality() {
        assertEquals(new Dollar(5), new Dollar(5));
        assertNotEquals(new Dollar(5), new Dollar(8));
    }
}
```

- ```java
  Expected :guru.springframework.Dollar@7adda9cc
  Actual   :guru.springframework.Dollar@5cee5251
  ```

  테스트가 실패한다. `equals()` 재정의를 안해줘서 그런다.



**Dollar 클래스**

```java
public class Dollar {

    ...

    @Override
    public boolean equals(Object o) {
        Dollar dollar = (Dollar) o;
        return amount == dollar.amount;
    }
}
```

이렇게 재정의해주면 테스트가 잘 통과한다.



### TDD To-Do List

- $5 + 10 CHF = $10 (with rate of 2:1)
- ~~$5 * 2 = $10~~
- Make "amount" private
- ~~Dollar side effects?~~ 
- Money Rounding?

- ~~equals()~~
- hashCode()
- Equal Null





## 5. TDD 4 - Privacy

### Privacy

- 현재 `amount` 프로퍼티가 public이다.
- public 접근자로 설정하면 다른 클래스들이 `amount` 프로퍼티를 마음대로 조회하고 수정할 수 있게된다.
- Object Oriented Best Practice는 당신이 노출시키고 싶은 프로퍼티만 노출시키는 것이다.
- 현재 `equals()` 메서드를 구현했으므로 `amount` 프로퍼티에 대한 접근이 필요하지 않다.



### 코딩해보자

```java
public class MoneyTest {

    @Test
    void testMultiplication() {
        Dollar five = new Dollar(5);
        Dollar product = five.times(2);
        assertEquals(new Dollar(10), product); // assertEquals(10, product.amount);

        product = five.times(3);
        assertEquals(new Dollar(15), product); // assertEquals(15, product.amount);
    }

    @Test
    void testEquality() {
        ...
    }
}
```

- 위처럼 재정의한 `equals()` 메서드를 이용하면, 더 이상 `Dollar`의 `amount` 프로퍼티에 접근할 필요가 없게 된다.

  - ```java
    public class Dollar {
    
        private int amount;
        ...
    }
    ```

    `amount`를 private으로 고쳐주자.

    테스트가 통과된다.





### TDD To-Do List

- $5 + 10 CHF = $10 (with rate of 2:1)
- ~~$5 * 2 = $10~~
- ~~Make "amount" private~~
- ~~Dollar side effects?~~ 
- Money Rounding?
- ~~equals()~~
- hashCode()
- Equal Null
- Equal Object





## 6. TDD 5 - Franc-ly Speaking

### TDD Cycle Revisited

- Write a Test
- Make it compile
- Run to see that it fails
- Make it run
- Remove duplication / Make it right



### TDD To-Do List

- $5 + 10 CHF = $10 (with rate of 2:1)
- ~~$5 * 2 = $10~~
- ~~Make "amount" private~~
- ~~Dollar side effects?~~ 
- Money Rounding?
- ~~equals()~~
- hashCode()
- Equal Null
- Equal Object

- **5 CHF * 2 = 10 CHF** (`Franc` 객체를 구현해보자!)



### 코딩해보자

**MoneyTest 클래스에 추가**

```java
public class MoneyTest {
    // ...
    
    @Test
    void testMultiplicationFranc() {
        Franc five = new Franc(5);
        Franc product = five.times(2);
        assertEquals(new Franc(10), product);

        product = five.times(3);
        assertEquals(new Franc(15), product);
    }

    @Test
    void testEqualityFranc() {
        assertEquals(new Franc(5), new Franc(5));
        assertNotEquals(new Franc(5), new Franc(8));
    }
}
```



**Franc 클래스 추가**

```java
public class Franc {

    private int amount;

    public Franc(int amount) {
        this.amount = amount;
    }

    Franc times(int multiplier) {
        return new Franc(this.amount * multiplier);
    }

    @Override
    public boolean equals(Object o) {
        Franc franc = (Franc) o;
        return amount == franc.amount;
    }
}
```

- 추가하고 나면 테스트가 모두 통과한다.



#### 문제점

- `Franc`과 `Dollar` 객체 사이에 중복되는 코드가 많다.
  - To-do List에 넣어놓자.



### TDD To-Do List

- $5 + 10 CHF = $10 (with rate of 2:1)
- ~~$5 * 2 = $10~~
- ~~Make "amount" private~~
- ~~Dollar side effects?~~ 
- Money Rounding?
- ~~equals()~~
- hashCode()
- Equal Null
- Equal Object

- ~~5 CHF * 2 = 10 CHF~~ 
- Dollar/Franc Duplication
- Common equals()
- Common times()





## 7. TDD 6 - Eqaulity For All Redux



### TDD To-Do List

- $5 + 10 CHF = $10 (with rate of 2:1)
- ~~$5 * 2 = $10~~
- ~~Make "amount" private~~
- ~~Dollar side effects?~~ 
- Money Rounding?
- ~~equals()~~
- hashCode()
- Equal Null
- Equal Object

- ~~5 CHF * 2 = 10 CHF~~ 
- Dollar/Franc Duplication
- **Common equals()**
- Common times()



### OOP Inheritance

![](https://lh3.googleusercontent.com/3fi2UYItl6OwhkR_a6lO_Pzvs1vcGv2RaeDq97NE9SNBFuQtIc-0klot9FyljR7ngqTcobVXNwJE8TCyt93MSQIJJiQv8aLbE-9A1v3Momt4Xramw-Gmpqc2UgWU5xzlwjuKorcU)

- 앞 장에서 발생했던 `Dollar`와 `Franc`의 코드 중복을 제거하기 위해 OOP의 상속 기능을 이용하자.



**Money 클래스**

```java
public class Money {

    protected int amount;

    public Money(int amount) {
        this.amount = amount;
    }

    Money times(int multiplier) {
        return new Money(this.amount * multiplier);
    }

    @Override
    public boolean equals(Object o) {
        Money money = (Money) o;
        return amount == money.amount;
    }
}

```



**Franc 클래스**

```java
public class Franc extends Money {

    public Franc(int amount) {
        super(amount);
    }

    @Override
    Franc times(int multiplier) {
        return new Franc(this.amount * multiplier);
    }
}
```



**Dollar 클래스**

```java
public class Dollar extends Money{

    public Dollar(int amount) {
        super(amount);
    }

    @Override
    Dollar times(int multiplier) {
        return new Dollar(amount * multiplier);
    }
}
```



### TDD To-Do List

- $5 + 10 CHF = $10 (with rate of 2:1)
- ~~$5 * 2 = $10~~
- ~~Make "amount" private~~
- ~~Dollar side effects?~~ 
- Money Rounding?
- ~~equals()~~
- hashCode()
- Equal Null
- Equal Object

- ~~5 CHF * 2 = 10 CHF~~ 
- Dollar/Franc Duplication
- ~~**Common equals()**~~
- Common times()
- Compare Francs With Dollars





## 8. TDD 7 - Apples And Oranges

### TDD To-Do List

- $5 + 10 CHF = $10 (with rate of 2:1)
- ~~$5 * 2 = $10~~
- ~~Make "amount" private~~
- ~~Dollar side effects?~~ 
- Money Rounding?
- ~~equals()~~
- hashCode()
- Equal Null
- Equal Object

- ~~5 CHF * 2 = 10 CHF~~ 
- Dollar/Franc Duplication
- ~~Common equals()~~
- Common times()
- **Compare Francs With Dollars**



### 코딩해보자

```java
@Test
void testEqualityDollar() {
    assertEquals(new Dollar(5), new Dollar(5));
    assertNotEquals(new Dollar(5), new Dollar(8));
    
    // 추가
    assertNotEquals(new Dollar(5), new Franc(5));
}
```



```java
public class Money {
	// ...
	
	
	// 추가
    @Override
    public boolean equals(Object o) {
        Money money = (Money) o;
        return amount == money.amount && this.getClass().equals(o.getClass());
    }
}
```



### TDD To-Do List

- $5 + 10 CHF = $10 (with rate of 2:1)
- ~~$5 * 2 = $10~~
- ~~Make "amount" private~~
- ~~Dollar side effects?~~ 
- Money Rounding?
- ~~equals()~~
- hashCode()
- Equal Null
- Equal Object

- ~~5 CHF * 2 = 10 CHF~~ 
- Dollar/Franc Duplication
- ~~Common equals()~~
- Common times()
- ~~Compare Francs With Dollars~~
- Currency





## 9. TDD 8 - Making Objects

### TDD To-Do List

- $5 + 10 CHF = $10 (with rate of 2:1)
- ~~$5 * 2 = $10~~
- ~~Make "amount" private~~
- ~~Dollar side effects?~~ 
- Money Rounding?
- ~~equals()~~
- hashCode()
- Equal Null
- Equal Object

- ~~5 CHF * 2 = 10 CHF~~ 
- **Dollar/Franc Duplication**
- ~~Common equals()~~
- Common times()
- ~~Compare Francs With Dollars~~
- Currency



### Makin' Objects

- `Money` 클래스의 두 subclass들(`Dollar` & `Franc`)은 거의 똑같다.
- 목표는 이 subclass들이 존재하지 않게 리팩터링 하는 것이다. 
- 첫 번째 단계는 subclass들에 대한 직접적인 참조를 줄이는 것이다.
- 우리는 `Dollar`와 `Franc`들을 리턴하는 `Money` 클래스에 대한 factory method를 만들 수 있다.
- This is an incremental step.



### 코딩해보자

**Dollar 클래스**

```java
public class Dollar extends Money{

    public Dollar(int amount) {
        super(amount);
    }

    @Override
    Money times(int multiplier) {
        return new Dollar(amount * multiplier);
    }
}
```



**Franc 클래스**

```java
public class Franc extends Money {

    public Franc(int amount) {
        super(amount);
    }

    @Override
    Money times(int multiplier) {
        return new Franc(this.amount * multiplier);
    }
}
```



**MoneyTest 클래스**

```java
public class MoneyTest {

    @Test
    void testMultiplicationDollar() {
        Dollar five = new Dollar(5);
        assertEquals(new Dollar(10), five.times(2));
        assertEquals(new Dollar(15), five.times(3));
    }

    @Test
    void testMultiplicationFranc() {
        Franc five = new Franc(5);
        assertEquals(new Franc(10), five.times(2));
        assertEquals(new Franc(15), five.times(3));
    }
    
    // ...
    
}
```





**Money 클래스**

```java
public abstract class Money {

    protected int amount;

    public static Money dollar(int amount) {
        return new Dollar(amount);
    }

    public static Money franc(int amount) {
        return new Franc(amount);
    }

    public Money(int amount) {
        this.amount = amount;
    }

    public abstract Money times(int multiplier);


    @Override
    public boolean equals(Object o) {
        Money money = (Money) o;
        return amount == money.amount && this.getClass().equals(o.getClass());
    }
}
```



**Dollar 클래스**

```java
public class Dollar extends Money{

    public Dollar(int amount) {
        super(amount);
    }

    @Override
    public Money times(int multiplier) {
        return new Dollar(amount * multiplier);
    }
}
```



**Franc 클래스**

```java
public class Franc extends Money {

    public Franc(int amount) {
        super(amount);
    }

    @Override
    public Money times(int multiplier) {
        return new Franc(this.amount * multiplier);
    }
}
```



**MoneyTest 클래스**

```java
public class MoneyTest {

    @Test
    void testMultiplicationDollar() {
        assertEquals(Money.dollar(10), Money.dollar(5).times(2));
        assertEquals(Money.dollar(15), Money.dollar(5).times(3));
    }

    @Test
    void testEqualityDollar() {
        assertEquals(Money.dollar(5), Money.dollar(5));
        assertNotEquals(Money.dollar(5), Money.dollar(8));
        assertNotEquals(Money.dollar(5), Money.franc(5));
    }

    @Test
    void testMultiplicationFranc() {
        assertEquals(Money.franc(10), Money.franc(5).times(2));
        assertEquals(Money.franc(15), Money.franc(5).times(3));
    }

    @Test
    void testEqualityFranc() {
        assertEquals(Money.franc(5), Money.franc(5));
        assertNotEquals(Money.franc(5), Money.franc(8));
    }
}
```

- `Dollar`와 `Franc`에 대한 직접적인 참조를 다 제거했다.





### TDD To-Do List

- $5 + 10 CHF = $10 (with rate of 2:1)
- ~~$5 * 2 = $10~~
- ~~Make "amount" private~~
- ~~Dollar side effects?~~ 
- Money Rounding?
- ~~equals()~~
- hashCode()
- Equal Null
- Equal Object

- ~~5 CHF * 2 = 10 CHF~~ 
- ~~Dollar/Franc Duplication~~
- ~~Common equals()~~
- Common times()
- ~~Compare Francs With Dollars~~
- Currency
- Delete testFrancMultiplication?





## 10. TDD 9 - Times We're Livin' In

### TDD To-Do List

- $5 + 10 CHF = $10 (with rate of 2:1)
- ~~$5 * 2 = $10~~
- ~~Make "amount" private~~
- ~~Dollar side effects?~~ 
- Money Rounding?
- ~~equals()~~
- hashCode()
- Equal Null
- Equal Object

- ~~5 CHF * 2 = 10 CHF~~ 
- ~~Dollar/Franc Duplication~~
- ~~Common equals()~~
- Common times()
- ~~Compare Francs With Dollars~~
- **Currency**
- Delete testFrancMultiplication?

















