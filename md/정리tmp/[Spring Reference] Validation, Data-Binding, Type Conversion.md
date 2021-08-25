# [Spring Reference] Validation, Data-Binding, Type Conversion



## 6.1 Introduction (소개)

스프링은 어플리케이션의 모든 레이어에서 기본이 되고 아주 사용하기 편리한 **Validator** 인터페이스를 제안했다.



데이터바인딩은 어플리케이션의 도메인 모델(또는 사용자 입력을 처리하려고 사용하는 어떤 객체)에 사용자 입력을 동적으로 바인딩하는데 유용하다. 스프링은 이 작업을 하기 위해서 **DataBinder**를 제공한다. 



**BeanWrapper**는 스프링 프레임워크의 기본 개념이고 많은 곳에서 사용된다. 하지만 BeanWrapper를 직접 사용할 일은 거의 없을 것이다. 하지만 이 문서는 레퍼런스 문서이기 때문에 약간 설명하는 것이 적절하다고 생각한다. 이번 장에서 BeanWrapper를 설명할 것이고 BeanWrapper를 사용할 것이라면 객체에 데이터를 바인딩할 때 사용할 가능성이 높다.



스프링의 **DataBinder**와 저수준 **BeanWrapper**는 둘 다 프로퍼티의 값들을 파싱하고 포매팅하는데 **PropertyEditor**를 사용한다. PropertyEditor 개념은 JavaBeans 명세의 일부이고 역시 이번 장에서 설명한다. 



스프링 3는 UI 필드 값을 포매팅하는 고수준 "format" 패키지 뿐만 아니라 일반적인 타입 변환의 기반을 제공하는 "core.convert" 패키지를 도입했다. 이 새로운 패키지들을 PropertyEditor의 더 간단한 대안으로 사용할 것이고 이에 대해서 이번 장에서 얘기할 것이다.







## 6.2 스프링의 Validator 인터페이스를 사용하는 유효성 검사

스프링은 객체의 유효성검사에 사용할 수 있는 Validator 인터페이스를 제공한다. 

Validator 인터페이스는 유효성검사를 하면서 

Validator가 Errors 객체에 유효성검사의 실패내역을 보고할 수 있도록 Errors 객체를 사용해서 동작한다.





### 예제1

다음과 같은 객체를 생각해 보자.

```java
public class Person {

  private String name;
  private int age;

  // getter와 setter...
}
```



`org.springframework.validation.Validator` 인터페이스의 다음 두 가지 메서드를 구현해서

`Person` 클래스에 대한 유효성 검사 동작을 제공할 것이다.





**우리가 구현해야 하는 Validator 인터페이스의 두 가지 메서드**

- `boolean supports(Class)` 
  - 이 Validator가 제공된 Class의 인스턴스를 유효성검사할 수 있는가?
- `void validate(Object, Errors)` 
  - 주어진 객체에 유효성검사를 하고 유효성검사에 오류가 있는 경우 주어진 객체에 이 오류들을 등록한다.



```java
public class PersonValidator implements Validator {

  /**
  * 이 Validator는 단순히 Person 인스턴스를 유효성검사한다
  */
  public boolean supports(Class clazz) {
    return Person.class.equals(clazz);
  }

  public void validate(Object obj, Errors e) {
    ValidationUtils.rejectIfEmpty(e, "name", "name.empty");
    Person p = (Person) obj;
    if (p.getAge() < 0) {
      e.rejectValue("age", "negativevalue");
    } else if (p.getAge() > 110) {
      e.rejectValue("age", "too.darn.old");
    }
  }
}
```



여기서 볼 수 있듯이 `ValidationUtils` 클래스의 `static rejectIfEmpty(..)` 메서드는 

'name' 프로퍼티가 null이거나 ""(빈 문자열)일 때, 

'name' 프로퍼티를 reject(거절)하는 데 사용한다.







### 예제2 - 중첩 객체 유효성 검사

```java
public class Customer {

    private String firstName;
    private String surname;
    private Address address;
}
```



```java
public class CustomerValidator implements Validator {

  private final Validator addressValidator;

  public CustomerValidator(Validator addressValidator) {
    if (addressValidator == null) {
      throw new IllegalArgumentException(
        "The supplied [Validator] is required and must not be null.");
    }
    if (!addressValidator.supports(Address.class)) {
      throw new IllegalArgumentException(
        "The supplied [Validator] must support the validation of [Address] instances.");
    }
    this.addressValidator = addressValidator;
  }

  /**
  * 이 Validator는 Customer 인스턴스의 유효성을 검사하고 Customer의 모든 하위 클래스도 유효성 검사한다
  */
  public boolean supports(Class clazz) {
    return Customer.class.isAssignableFrom(clazz);
  }

  public void validate(Object target, Errors errors) {
    ValidationUtils.rejectIfEmptyOrWhitespace(errors, "firstName", "field.required");
    ValidationUtils.rejectIfEmptyOrWhitespace(errors, "surname", "field.required");
    Customer customer = (Customer) target;
    try {
      errors.pushNestedPath("address");
      ValidationUtils.invokeValidator(this.addressValidator, customer.getAddress(), errors);
    } finally {
      errors.popNestedPath();
    }
  }
}
```



Address 객체들은 Customer 객체들과 관계없이 사용될 것이므로 별도의 AddressValidator를 구현했다. 

위와 같이 CustomerValidator내에서 AddressValidator를 의존성 주입하거나 인스턴스화 해서 사용할 수 있다. 



>  AddressValidation 로직을 CustomerValidator에 복사-붙여넣기 할 필요 없이 
>
> 의존성 주입받거나 인스턴스화 하면 코드 중복을 제거할 수 있다.







## 6.3 오류 메시지에 대한 코드 처리

데이터바인딩과 유효성검사에 대해서 이야기했다. 

마지막으로 유효성 오류에 대응되는 **출력 메세지**에 대해서 얘기해야 한다.



위에서 본 예제에서 name와 age 필드를 거절했다. 

MessageSource를 사용해서 오류 메세지를 출력하려면 

필드(이 경우에는 'name'와 'age')를 거절했을 때 전달받은 오류 코드를 사용해야 할 것이다.



rejectValue를 호출하거나 (직접적이든 간접적이든 ValidationUtils같은 클래스를 사용해서) 

Errors 인터페이스의 다른 reject 메서드 중의 하나를 호출했을 때 

기반이 되는 구현체는 전달한 코드뿐만 아니라 추가적인 다수의 오류 코드를 등록한다. 

어떤 오류 코드를 등록하는 지는 사용하는 MessageCodesResolver가 결정한다. 

기본적으로는 DefaultMessageCodesResolver를 사용한다. 



예를 들어 DefaultMessageCodesResolver는 전달한 코드와 메세지를 등록하고

reject 메서드에 전달한 필드명을 포함한 메세지를 등록한다. 

그래서 rejectValue("age", "too.darn.old")를 사용해서 필드를 거절하는 경우 

스프링은 too.darn.old 코드 이외에 too.darn.old.age 와 too.darn.old.age.int도 등록할 것이다.

(그래서 첫번째 것은 필드 명을 담고 있고 두번째 것은 필드의 타입을 담고 있다.) 



오류 메시지에 대한 처리가 궁금하면 `MessageCodesResolver`에 대해 살펴보자.







## 6.4 빈 조작과 BeanWrapper

`org.springframework.beans` 패키지는 Sun의 자바 빈 표준을 충실히 따른다.



**JavaBean이란?**

- Argument가 없는 기본 생성자를 가진 클래스
- get/set 메서드 네이밍 관례를 따른다. 
  - 예시: bingoMadness라는 프로퍼티는 setBingoMadness, getBingoMadness라는 메서드 네이밍을 가진다.



`beans` 패키지에서 아주 중요한 클래스는

`BeanWrapper` 인터페이스와 그에 대한 구현체 `BeanWrapperImpl`이다.



Javadoc에 나와있듯이 `BeanWrapper`는 

프로퍼티 값을 (개별적으로나 한꺼번에) 설정하고 가져오는 기능과,

프로퍼티 descriptor를 가져오는 기능이나, 

프로퍼티가 읽을 수 있는지 쓸 수 있는지 결정하기 위해 쿼리할 수 있는 기능을 제공한다.

`BeanWrapper`는 무한 계층까지 하위 프로퍼티에서 프로퍼티를 설정하는 것이 가능하도록 중첩된 프로퍼티도 지원한다.

그리고 `BeanWrapper`는 대상 클래스에 지원 코드를 두지 않고도 

표준 자바빈 `PropertyChangeListeners`와 `VetoableChangeListeners`를 추가하는 기능도 지원한다. 

마지막으로 가장 중요한 것은 `BeanWrapper`가 색인된 프로퍼티를 설정하는 지원을 제공한다는 것이다.



`BeanWrapper`는 보통 어플리케이션 코드에서 직접 사용하지 않고 `DataBinder`와 `BeanFactory`에서 사용한다.

`BeanWrapper`의 동작 방식은 그 이름이 어느 정도 알려주고 있다.

프로퍼티를 설정하고 획득하는 것처럼 해당 빈에 액션을 수행하기 위해서 빈을 감싼다.





### 6.4.1 기본적인 프로퍼티와 중첩된 프로퍼티를 설정하고 가져오기

프로퍼티를 설정하고 가져오는 것은 `setPropertyValue(s)`와 `getPropertyValue(s)` 메서드를 사용해서 이루어진다.

둘 다 다수의 Overload 된 메서드들이 있다.

자세한 내용은 스프링 JavaDoc에 모두 설명되어 있다.

객체의 프로퍼티를 나타내는 여러 가지 관례가 있다는 사실은 중요하다.



다음은 몇 가지 예제이다.



#### 프로퍼티 예제

| 표현식               | 설명                                                         |
| -------------------- | ------------------------------------------------------------ |
| name                 | getName()이나 isName()이나 setName(..) 메서드와 대응되는 name 프로퍼티를 나타낸다. |
| account.name         | account 프로퍼티의 중첩된 name 프로퍼티를 나타낸다. 예를 들면, getAccount().setName()이나 getAccount().getName()에 대응된다. |
| account[2]           | 색인된 account 프로퍼티의 세 번째 요소를 나타낸다. 색인된 프로퍼티는 array, list나 자연스럽게 정렬된 컬렉션이 될 수 있다. |
| account[COMPANYNAME] | Map 프로퍼티 account의 COMPANYNAME 키로 찾은 값을 나타낸다.  |





아래에서는 프로퍼티를 얻거나 설정하는 `BeanWrapper` 동작 예제를 살펴본다.

(`BeanWrapper`를 직접 사용해서 작업하지 않는다면 다음 부분은 아주 중요하진 않다.)





#### 예제

```java
public class Company {
  private String name;
  private Employee managingDirector;

  public String getName() {
    return this.name;
  }
  public void setName(String name) {
    this.name = name;
  }
  public Employee getManagingDirector() {
    return this.managingDirector;
  }
  public void setManagingDirector(Employee managingDirector) {
    this.managingDirector = managingDirector;
  }
}
```



```java
public class Employee {
  private String name;
  private float salary;

  public String getName() {
    return this.name;
  }
  public void setName(String name) {
    this.name = name;
  }
  public float getSalary() {
    return salary;
  }
  public void setSalary(float salary) {
    this.salary = salary;
  }
}
```





다음 코드는 인스턴스화된 Companies와 Employees의 프로퍼티를 어떻게 획득하고 조작하는 가를 보여주는 예제이다.

```java
BeanWrapper company = BeanWrapperImpl(new Company());
// 회사이름을 설정한다..
company.setPropertyValue("name", "Some Company Inc.");
// ... 또는 다음과 같이 할 수 있다:
PropertyValue value = new PropertyValue("name", "Some Company Inc.");
company.setPropertyValue(value);

// 감독관을 생성하고 회사에 연결한다:
BeanWrapper jim = BeanWrapperImpl(new Employee());
jim.setPropertyValue("name", "Jim Stravinsky");
company.setPropertyValue("managingDirector", jim.getWrappedInstance());

// company를 통해 managingDirector의 봉급을 획득한다
Float salary = (Float) company.getPropertyValue("managingDirector.salary");
```





### 6.4.2 내장 PropertyEditor 구현체

스프링은 Object와 String 간의 변환에 PropertyEditors의 개념을 사용한다.

생각해보면 때때로 객체 자체와는 다른 방법으로 프로퍼티를 표현할 수 있는 손쉬운 방법이다.

예를 들어, Date는 사람이 읽을 수 있게 표현할 수 있고 (String '2007-08-09' 처럼) 

동시에 여전히 사람이 읽을 수 있는 형식을 다시 원래의 날짜로 변환할 수도 있다.



이 동작들은 `java.beans.PropertyEditor` 타입의 커스텀 에디터를 등록함으로써 이뤄질 수 있다.

`BeanWrapper`나 이전 챕터에서 얘기했던 대안적인 특정 IoC 컨테이너에 커스텀 에디터를 등록하면,

어떻게 프로퍼티를 원하는 타입으로 변환하는지 알려준다.

더 자세한 내용은 Sun이 제공하는 `java.beans` 패키지의 Javadoc의 PropertyEditors 부분을 살펴보자.





#### 스프링에서 프로퍼티 수정을 사용하는 몇 가지 예제

- PropertyEditors를 사용해서 빈에 프로퍼티를 설정한다.

  XML 파일에 선언한 어떤 빈의 프로퍼티 값으로 String을 사용했을 때

  스프링은 (해당 프로퍼티의 setter가 Class-parameter를 가지고 있다면) 파라미터를

  Class 객체로 처리하려고 ClassEditor를 사용할 것이다.

- 스프링 MVC 프레임워크에서 HTTP 요청 파라미터의 파싱은 

  CommandController의 모든 하위클래스에 수동으로 연결할 수 있는 모든 종류의

  PropertyEditors를 사용해서 이루어진다.



스프링은 쉽게 사용할 수 있도록 다수의 내장 PropertyEditors를 가진다.

이 내장 PropertyEditors는 아래 목록에 나와있고 이 모두는 `org.springframework.beans.propertyeditors` 패키지에 존재한다.

모두는 아니지만 대부분은 `BeanWrapperImpl`이 기본으로 등록된다.



#### 내장 PropertyEditors

| Class                   | 설명                                                         |
| ----------------------- | ------------------------------------------------------------ |
| ByteArrayPropertyEditor | 바이트 배열에 대한 에디터. 문자열은 간단하게 대응되는 바이트 표현으로 변환될 것이다. BeanWrapperImpl의 기본값으로 등록된다. |
| ClassEditor             | 클래스를 나타내는 문자열을 실제 클래스로 파싱하거나 그 반대로 파싱한다. 클래스를 찾지 못하면 IllegalArgumentException를 던진다. BeanWrapperImpl의 기본값으로 등록된다. |
| CustomBooleanEditor     | Boolean 프로퍼티에 대해 커스터마이징할 수 있는 프로퍼티 데이터다. BeanWrapperImpl의 기본값으로 등록되지만 커스텀 에디터처럼 커스텀 인스턴스를 등록함으로써 오버라이드할 수 있다. |
| CustomCollectionEditor  | 컬렉션에 대한 프로퍼티 에디터로 모든 소스 Collection를 전달한 타겟 Collection 타입으로 변환한다. |
| CustomDateEditor        | java.util.Date에 대한 커스터마이징 할 수 있는 프로퍼티 에디터로 커스텀 DateFormat을 지원한다. 기본값으로 등록되지 않는다. 적절한 형식으로 필요한 만큼 사용자가 등록해야 한다. |
| CustomNumberEditor      | Integer, Long, Float, Double같은 숫자타입의 하위클래스에 대한 커스마타이징할 수 있는 프로퍼티 에디터이다. BeanWrapperImpl의 기본값으로 등록되지만 커스텀 에디터처럼 커스텀 인스턴스를 등록해서 오바라이드할 수 있다. |
| FileEditor              | 문자열을 java.io.File 객체로 처리할 수 있다. BeanWrapperImpl의 기본값으로 등록된다. |
| InputStreamEditor       | InputStream 프로퍼티를 문자열로 직접 설정할 수 있도록 텍스트 문자열을 받아서 InputStream을 생성하는(중간에 ResourceEditor와 Resource를 통해서) 단방향 프로퍼티 에디터이다. 기본 사용법은 InputStream를 닫지 않을 것이다. BeanWrapperImpl의 기본값으로 등록된다. |
| LocaleEditor            | 문자열을 Locale 객체로 처리하거나 그 반대로 할 수 있다.(문자열 형식은 Locale의 toString() 메서드가 제공하는 형식과 같은 [language]_[country]_[variant]이다.) BeanWrapperImpl의 기본값으로 등록된다. |
| PatternEditor           | 문자열을 JDK 1.5 Pattern 객체로 처리하거나 그 반대로 처리할 수 있다. |
| PropertiesEditor        | 문자열(java.lang.Properties 클래스의 Javadoc에서 정의된 것과 같은 형식으로 포매팅된)을 Properties 객체로 변환할 수 있다.BeanWrapperImpl의 기본값으로 등록된다. |
| StringTrimmerEditor     | 스트림을 trim하는 프로퍼티 에디터이다. 선택적으로 비어있는 문자열을 null 값으로 변형할 수도 있다. 기본적으로는 등록되지 않는다. 필요에 따라 사용자가 등록해야 한다. |
| URLEditor               | URL의 문자열 표현을 실제 URL 객체로 처리할 수 있다. BeanWrapperImpl의 기본값으로 등록된다. |



이후 중략

https://blog.outsider.ne.kr/825

https://blog.outsider.ne.kr/826





## 6.7 Spring 3 Validation

스프링 3에서는 유효성 검사에 대한 지원이 여러모로 강화되었다.

우선 JSR-303 Bean Validation API를 이제 완전히 지원한다.



두번째로 프로그래밍적으로 사용할 때 스프링의 DataBinder는 객체에 대한 바인딩 뿐만 아니라

객체의 유효성 검사도 할 수 있다.



세번째로 스프링 MVC는 @Controller 입력에 대한 유효성 검사를 선언적으로 할 수 있다.





### 6.7.1 JSR-303 Bean Validation API의 개요

JSR-303는 자바 플랫폼의 유효성 검사 제약사항 선언과 메타데이터를 표준화한다.

JSR-303 API를 사용해서 선언적인 유효성 제약사항으로 도메인 모델 프로퍼티에 어노테이션을 붙이고

런타임시에 이를 강제할 수 있다.

사용할 만한 다수의 내장 제약사항이 존재한다. 물론 자신만의 커스텀 제약사항도 정의할 수 있다.





#### 예제

설명을 위해 2개의 프로퍼티를 가진 간단한 `PersonForm` 모델을 생각해보자.

```java
public class PersonForm {
  private String name;
  private int age;
}
```



JSR-303으로 이러한 프로퍼티에 대한 유효성 검사 제약사항을 선언적으로 정의할 수 있다.

```java
public class PersonForm {

  @NotNull
  @Size(max=64)
  private String name;

  @Min(0)
  private int age;

}
```



JSR-303 Validator가 이 클래스의 인스턴스를 검사할 때 이러한 제약사항을 강제할 것이다.

JSR-303에 대한 일반적인 내용은 [Bean Validation Specification](http://jcp.org/en/jsr/detail?id=303)를 봐라. 

기본 레퍼런스 구현체의 특정 능력에 대한 내용은 [Hibernate Validator](https://www.hibernate.org/412.html) 문서를 봐라.

JSR-303 구현체를 어떻게 스프링 빈으로 설정하는지 알고 싶으면 계속 읽어봐라.





### 6.7.2 Bean Validation 구현체 설정

스프링은 JSR-303 Bean Validation API를 완전히 지원한다. 

이는 JSR-303 구현체를 스프링 빈으로 편리하게 설정하도록 하는 것도 포함한다. 

또한 어플리케이션에서 유효성검사가 필요할 때마다 

`javax.validation.ValidatorFactory`나 `javax.validation.Validator`를 주입할 수 있다.



기본 JSR-303 Validator를 스프링 빈으로 설정하려면 `LocalValidatorFactoryBean`를 사용해라.

```xml
<bean id="validator"
    class="org.springframework.validation.beanvalidation.LocalValidatorFactoryBean"/>
```



위의 기본 설정은 JSR-303의 기본 부트스트랩 메카니즘을 사용해서 JSR-303을 초기화한다. 

Hibernate Validator같은 JSR-303 프로바이더는 클래스패스에 존재해야 하고 자동적으로 탐지될 것이다.





#### 6.7.2.1 Validator 주입

`LocalValidatorFactoryBean` 는

`org.springframework.validation.Validator`뿐만 아니라 

`javax.validation.ValidatorFactory`와` javax.validation.Validator`를 모두 구현한다. 







이러한 인터페이스에 대한 참조를 유효성검사 로직을 실행해야하는 빈에 주입할 것이다.

JSR-303 API를 직접 사용하는 걸 좋아한다면 `javax.validation.Validator`에 대한 참조를 주입해라.

```java
import javax.validation.Validator;

@Service
public class MyService {

  @Autowired
  private Validator validator;

  ...
}
```





빈(bean)이 Spring Validation API를 필요로 한다면 

`org.springframework.validation.Validator`에 대한 참조를 주입해라.

```java
import org.springframework.validation.Validator;

@Service
public class MyService {

  @Autowired
  private Validator validator;

}
```







#### 6.7.2.2 커스텀 제약사항(Constraints) 설정

각 JSR-303 유효성 검사 제약사항은 두 부분으로 구성되어 있다.



첫째는 제약사항과 설정가능한 제약사항의 프로퍼티를 설정하는 `@Constraint`. 

둘째는 제약사항의 동작을 구현하는 `javax.validation.ConstraintValidator` 인터페이스의 구현체이다. 



선언과 구현체의 연결을 위해 각 @Constraint 어노테이션은 대응되는 `ValidationConstraint` 구현체를 참조한다. 

런타임시에 `ConstraintValidatorFactory`는 제약사항 어노테이션이 도메인 모델을 만났을 때 참조된 구현체를 인스턴스화한다.



기본적으로 `LocalValidatorFactoryBean`는 

스프링이 `ConstraintValidator` 인스턴스를 생성하려고 사용하는 `SpringConstraintValidatorFactory`를 설정한다. 

이는 다른 스프링 빈처럼 의존성 주입의 이점을 가진 커스텀 `ConstraintValidator`를 사용할 수 있다.





##### 예제

```java
@Target({ElementType.METHOD, ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy=MyConstraintValidator.class)
public @interface MyConstraint {
}
```



```java
import javax.validation.ConstraintValidator;

public class MyConstraintValidator implements ConstraintValidator {

  @Autowired;
  private Foo aDependency;

  ...
}
```

여기서 보듯이 `ConstraintValidator` 구현체는 다른 스프링 빈처럼 `@Autowired`로 의존성을 가진다.





(생략)