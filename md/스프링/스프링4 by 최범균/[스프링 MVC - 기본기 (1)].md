# [스프링 MVC - 기본기 (1)]



## 1. 기본 흐름과 주요 컴포넌트



![](https://docs.google.com/drawings/d/e/2PACX-1vRoMPmjUHxgy6wncGh_wN3rHTKoON7M81I8sA3rGlOXrpghPbR5GCiH4hhM8IzhxpEy--HUB3XnaP8D/pub?w=1167&h=661)



- 회색으로 표시한 부분이 직접 구현해야 하는 부분이다.





**※ 스프링 MVC의 주요 구성 요소**

| 구성 요소            | 설명                                                         |
| -------------------- | ------------------------------------------------------------ |
| DispatcherServlet    | 클라이언트의 요청을 전달받는 Front Controller의 역할을 한다. 컨트롤러에게 클라이언트의 요청을 전달하고, 컨트롤러가 리턴한 결과값을 View에 전달하여 알맞은 응답을 생성하도록 한다. |
| HandlerMapping       | 클라이언트의 요청 URL을 어떤 컨트롤러가 처리할지를 결정한다. |
| HandlerAdapter       | DispatcherServlet의 처리 요청을 변환해서 컨트롤러에게 전달하고, 컨트롤러의 응답 결과를 DispatcherServlet이 요구하는 형식으로 변환한다. 웹 브라우저 캐시 등의 설정도 담당한다. |
| 컨트롤러(Controller) | 클라이언트의 요청을 처리한 뒤, 결과를 리턴한다. 응답 결과에서 보여줄 데이터를 모델에 담아 전달한다. |
| ModelAndView         | 컨트롤러가 처리한 결과 정보 및 뷰 선택에 필요한 정보를 담는다. |
| ViewResolver         | 컨트롤러의 처리 결과를 보여줄 뷰를 선택한다.                 |
| 뷰 (View)            | 컨트롤러의 처리 결과 화면을 생성한다. JSP나 Thymeleaf 등의 템플릿 엔진을 이용해서 클라이언트에 응답 결과를 전송한다. |





## 2. 스프링 부트 자동 등록 빈

스프링 MVC를 설정하려면 최소한 다음의 구성 요소를 빈 객체로 등록해주어야 한다.

- `HandlerMapping` 구현체 -> 주로 `RequestMappingHandlerMapping` 사용
- `HandlerAdapter` 구현체 -> 주로 `RequestMappingHandlerAdapter` 사용
- `ViewResolver` 구현체





> 참고: http://wonwoo.ml/index.php/post/2308



### HandlerMapping 인터페이스

이 인터페이스는 해당 요청 정보를 기준으로 어떤 컨트롤러를 사용할 것인가를 결정하는 인터페이스다.

이 인터페이스는 여러 구현체를 가지고 있는데 보통 `RequestMappingHandlerMapping`을 많이 사용한다.



#### \- RequestMappingHandlerMapping 구현체

아래와 같이 `@RequestMapping` 애너테이션을 사용해서 개발한다면 `RequestMappingHandlerMapping` 매핑 전략을 사용하고 있다는 뜻이다.

```java
@Controller
@RequestMapping("/accounts")
public class AccountController {
	...
}
```





### HandlerAdapter 인터페이스

이 인터페이스는 `HandlerMapping`에 의해 결정된 Handler 정보로 해당 메서드를 직접 호출해주는 역할을 한다. 

이 역시 여러 개의 구현체가 존재한다. `RequestMappingHandlerMapping`과 대응되는 `RequestMappingHandlerAdapter` 구현체를 가장 많이 사용한다.

`RequestMappingHandlerMapping` 전략을 사용하고 싶다면 `RequestMappingHandlerAdapter`를 Adapter로 이용해야 한다.





#### - RequestMappingHandlerAdapter

가장 많이 사용하는 Adapter이다.

해당 Adapter에선 어떤 메서드를 호출해야 할지 결정해야 하므로 이 클래스엔 여러 정보들이 담겨 있다. 

왜냐하면 특정한 인터페이스를 구현한게 아니라 리플렉션을 이용해서 메서드를 호출해야 하므로 

해당 파라미터 타입과 리턴타입들의 정보를 알아야 하기 때문에 

해당 정보들을 파싱해줄 `HandlerMethodArgumentResolver`, `HandlerMethodReturnValueHandler` 인터페이스가 존재한다. 

```java
@Controller
@RequestMapping("/accounts")
public class AccountController {

    @GetMapping("/{id}")
    public String hello(@PathVariable Long id) {

       //...
    }
}
```

위의 클래스는 `RequestMappingHandlerAdapter`를 통해 `hello(Long)` 이란 메서드가 호출된다.







### HandlerMapping과 HandlerAdapter의 관계

위에서도 언급했지만, `RequestMappingHandlerMapping`과 `RequestMappingHandlerAdapter`는 하나를 사용하고 싶으면 다른 하나를 무조건 같이 사용해야 한다.



하지만 다른 전략들은 연관이 있어도 되고 없어도 된다.

예를 들어 Mapping 전략으로 `BeanNameUrlHandlerMapping`을 사용하면, Adapter 전략은 `SimpleControllerHandlerAdapter`, `HttpRequestHandlerAdapter`, ... 등등 아무거나 다 사용 가능하다.







### ViewResolver

뷰리졸버는 공부한 다음에...







...(아는 건 일단 생략)...





## 3. 스프링의 타입 변환(PropertyEditor와 ConversionService)

다음 코드를 보자.

```java
public class DataCollector implements ThresholdRequired {
    
    private int threshold;
    
    @Override
    public void setThreshold(int threshold) {
        this.threshold = threshold;
    }
    ...
}
```



`DataCollector` 클래스를 XML 설정으로 스프링 빈으로 등록할 때 다음의 설정을 사용할 것이다.

```xml
<bean id="collector" class="com.example.DataCollector">
	<property name="threshold" value="5" />
</bean>
```



여기서 `threshold` 프로퍼티의 값을 설정할 때 사용한 "5"는 문자열이고, 실제 `setThreshold()` 메서드의 파라미터는 `int` 타입이다.

스프링은 내부적으로 문자열 "5"를 `int` 타입 5로 변환한 뒤에 `setThreshold()` 메서드에 값을 전달한다.

스프링은 `int` 타입 외에 `long`, `double`, `boolean` 타입 등 기본 데이터 타입으로의 변환을 지원할 뿐만 아니라 문자열을 `Properties` 타입이나 `URL` 등으로 변환할 수 있다.



스프링은 내부적으로 `PropertyEditor`를 이용해서 문자열을 알맞은 타입으로 변환한다.

또한, 스프링 3 버전부터는 `ConversionService`를 이용해서 타입 변환을 처리할 수 있게 되었다.

이 절에서는 두 방식과 관련된 스프링의 타입 변환 설정에 대해 살펴보도록 하자.





### (1) PropertyEditor를 이용한 타입 변환

자바빈 규약은 문자열을 특정 타입의 프로퍼티로 변환할 때 `java.beans.PropertyEditor` 인터페이스를 사용한다.

자바는 기본 데이터 타입을 위한 `PropertyEditor`를 이미 제공하고 있다.

`sun.beans.editors` 패키지에는 `BooleanEditor`, `LongEditor`, `EnumEditor` 등 기본 타입을 위한 `PropertyEditor`를 제공하고 잇으며,

이들을 통해서 문자열을 `boolean`, `long`, `enum` 등의 타입으로 변환할 수 있다.



자바에 기본으로 포함된 `PropertyEditor` 구현체는 기본적인 타입만 지원하기 때문에, 

스프링은 설정 등의 편의를 위해 `PropertyEditor`를 추가로 제공하고 있다.

예를 들어, 스프링은 URL을 위한 `URLEditor`를 제공하고 있다.

따라서, 다음과 같이 URL 타입의 프로퍼티를 정의하고 XML에서 문자열을 이용해서 값을 설정할 수 있다.

```java
import java.net.URL;

public class RestClient {
    private URL serverUrl;
    
    public void setServerUrl(URL serverUrl) {
        this.serverUrl = serverUrl;
    }
    
    // ...
}
```

```xml
<bean class="com.example.RestClient">
	<!-- URLEditor를 이용해서 문자열을 URL 타입으로 변환 -->
    <property name="serverUrl" value="https://www.google.com" />
</bean>
```





#### - 스프링이 제공하는 주요 `PropertyEditor`

스프링은 URL을 비롯해 주요 타입에 대한 `PropertyEditor`를 제공하고 있으며, 이들 목록은 아래와 같다.

모든 클래스는 `org.springframework.beans.propertyeditors` 패키지에 위치하며, '기본'이라고 표시한 것은 별도 설정 없이 사용되는 것들이다.





**※ 스프링이 제공하는 주요 `PropertyEditor` 목록 **(옛날 기준임)

| PropertyEditor          | 설명                                                         | 기본 사용 여부 |
| ----------------------- | ------------------------------------------------------------ | -------------- |
| ByteArrayPropertyEditor | String.getBytes()를 이용해서 문자열을 byte 배열로 변환       | 기본           |
| CharArrayPropertyEditor | String.toCharArray()를 이용해서 문자열을 char 배열로 변환    |                |
| CharsetEditor           | 문자열을 Charset으로 변환                                    | 기본           |
| ClassEditor             | 문자열을 Class 타입으로 변환                                 | 기본           |
| CurrencyEditor          | 문자열을 java.util.Currency로 변환                           |                |
| CustomBooleanEditor     | 문자열을 Boolean 타입으로 변환                               | 기본           |
| CustomDateEditor        | DateFormat을 이용해서 문자열을 java.util.Date로 변환         |                |
| CustomNumberEditor      | Long, Double, BigDecimal 등 숫자 타입을 위한 프로퍼티 에디터 | 기본           |
| FileEditor              | 문자열을 java.io.File로 변환                                 | 기본           |
| LocaleEditor            | 문자열을 Locale로 변환                                       | 기본           |
| PatternEditor           | 정규 표현식 문자열을 Pattern으로 변환                        | 기본           |
| PropertiesEditor        | 문자열을 Properties로 변환                                   | 기본           |
| URLEditor               | 문자열을 URL로 변환                                          | 기본           |





#### - 커스텀 `PropertyEditor` 구현하기

직접 구현한 클래스처럼 `PropertyEditor`가 존재하지 않는 경우, `PropertyEditor`를 직접 구현해야 한다.

예를 들어, 다음과 같은 `Money` 클래스가 있다고 해보자.

```java
@AllArgsConstructor
public class Money {
    private int amount;
    private String currency;
    // ...
}
```



문자열을 `Money` 클래스로 변환해주는 `PropertyEditor`를 구현하려면 `PropertyEditor` 인터페이스를 상속 받아 알맞은 메서드를 구현해주면 된다.

그런데 `PropertyEditor` 인터페이스는 문자열을 `Money` 타입으로 변환하기 위한 메서드 외에 추가적인 메서드를 많이 정의하고 있기 때문에,

단순히 타입 변환 기능만 필요하다면 `PropertyEditorSupport` 클래스를 상속받아 `setAsText()` 메서드를 재정의하면 된다.



**String을 Money로 변환하기 위한 `PropertyEditor` 구현**

```java
public class MoneyEditor extends PropertyEditorSupport {
	
    @Override
    public void setAsText(String text) throws IllegalArgumentException {
        Pattern pattern = Pattern.compile("([0-9]+)([A-Z]{3})");
        Matcher matcher = pattern.matcher(text);
        
        if(!matcher.matches()) {
            throw new IllegalArgumentException("invalid format");
        }
        
        int amount = Integer.parseInt(matcher.group(1));
        String currency = matcher.group(2);
        setValue(new Money(amount, currency));
    }
}
```



이제 `MoneyEditor` 클래스를 이용하면 `Money` 타입의 프로퍼티 값을 문자열로 설정할 수 있다.

```xml
<bean class="com.example.InvestmentSimulator">
	<!-- minimumAmount 프로퍼티가 Money 타입 -->
    <property name="minimumAmount" value="10000WON" />
</bean>
```



실제로 위 설정이 올바르게 동작하려면, 문자열을 `Money`로 변환할 때 `MoneyEditor`를 사용하도록 추가로 설정해주어야 한다.

지금부터는 이에 대한 내용을 살펴보자.





#### - PropertyEditor 추가: 같은 패키지에 PropertyEditor 위치시키기

`PropertyEditor`를 구현했다면, 실제로 `PropertyEditor`를 사용하도록 `PropertyEditor`를 등록해주어야 한다.

이를 위한 가장 쉬운 방법은 다음과 같은 규칙에 따라 `PropertyEditor`를 작성하는 것이다. (이 규칙은 자바빈 규약에 명시된 것이다.)

- 변환 대상 타입과 동일한 패키지에 '타입Editor' 이름으로 `PropertyEditor`를 구현



예를 들어, `Money` 클래스가 `com.example` 패키지에 존재한다고 할 경우, 같은 패키지에 `MoneyEditor`라는 이름으로 `PropertyEditor`를 구현한다.

그러면, 문자열을 `Money` 타입으로 변환할 때 `MoneyEditor`를 사용해서 변환을 수행한다.





#### - PropertyEditor 추가: CustomEditorConfigurer 사용하기

특정 타입을 위해 구현한 `PropertyEditor`가 다른 패키지에 위치하거나 이름이 '타입Editor'가 아니라면, `CustomEditorConfigurer`를 이용해서 설정해야 한다.

`CustomEditorConfigurer`는 `BeanFactoryPostProcessor`로서 스프링 빈을 초기화하기 전에 필요한 `PropertyEditor`를 등록할 수 있도록 해준다.



`CustomEditorConfigurer`를 이용해서 `PropertyEditor`를 추가하는 방법은 다음과 같다.

```xml
<bean class="org.springframework.beans.factory.config.CustomEditorConfigurer">
	<property name="customEditors">
    	<map>
        	<entry key="com.example.Money" value="com.example.MyMoneyEditor"/>
        </map>
    </property>
</bean>


<bean class="com.example.InvestmentSimulator">
	<!-- minimumAmount가 Money 타입일 경우, MyMoneyEditor를 이용해서 타입 변환 -->
    <property name="minimumAmount" value="10000WON" />
</bean>
```

`CustomEditorConfigurer` 클래스의 `customEditors` 프로퍼티는 (타입, 타입 PropertyEditor) 쌍을 맵으로 전달받는다.

위 코드의 경우 `Money` 타입을 위한 `PropertyEditor`로 `MyMoneyEditor`를 사용한다고 설정했으므로,

`Money` 타입인 `minimumAmount` 프로퍼티의 값인 "10000WON"을 변환할 때 `MyMoneyEditor`를 사용한다.







#### - PropertyEditor 추가: PropertyEditorRegistrar 사용하기

`CustomEditorConfigurer`의 `customEditors` 프로퍼티를 이용해서 설정할 때 단점은 `PropertyEditor`에 매개변수를 설정할 수 없다는 점이다.

예를 들어, 특정 패턴에 맞게 입력한 문자열을 `Date` 타입으로 변환하고 싶은 경우 패턴을 지정할 수 있어야 하는데, 

앞서 설정 방식은 `PropertyEditor`의 클래스 이름을 사용하므로 이런 매개변수를 지정할 수 없다.

이렇게 `PropertyEditor`에 매개 변수를 지정하고 싶을 때 사용할 수 있는 것이 `PropertyEditorRegistrar`이다.



`PropertyEditorRegistrar`는 인터페이스로서 다음과 같이 정의되어 있다.

```java
package org.springframework.beans;

public interface PropertyEditorRegistrar {
	void registerCustomEditors(PropertyEditorRegistry registry);
}
```



특정 타입을 위한 `PropertyEditor` 객체를 직접 생성해서 설정하고 싶다면 다음과 같은 방법으로 `PropertyEditorRegistrar`를 사용하면 된다.

1. `PropertyEditorRegistrar` 인터페이스를 상속받은 클래스에서 `PropertyEditor`를 직접 생성하고 등록한다.
2. 1번 과정에서 생성한 클래스를 빈으로 등록하고, `CustomEditorConfigurer`에 `propertyEditorRegistrars` 프로퍼티로 등록한다.





먼저 할 작업은 `PropertyEditorRegistrar` 인터페이스를 상속받아 구현하는 것이다.

상속 받은 클래스는 `registerCustomEditors()` 메서드에서 원하는 `PropertyEditor`를 생성해서 등록하면 된다.

예를 들어, `CustomDateEditor`를 사용하고 싶다면, 다음과 같이 구현할 수 있다.

```java
public class CustomPropertyEditorRegistrar implements PropertyEditorRegistrar {
    
    private String datePattern;
    
    @Override
    public void registerCustomEditors(PropertyEditorRegistry registry) {
        CustomDateEditor propertyEditor = new CustomDateEditor(new SimpleDateFormat(datePattern), true);
        registry.registerCustomEditor(Date.class, propertyEditor);
    }
}
```



`registry.registerCustomEditor()` 메서드에서 첫 번째 파라미터는 변환 대상이 되는 타입을 지정하고, 

두 번째 파라미터는 문자열을 지정 타입으로 변환할 때 사용할 `PropertyEditor` 객체를 지정한다.

위 코드에서는 `Date` 타입의 변환 처리를 위해 `CustomDateEditor` 객체를 지정하였다.



`PropertyEditorRegistrar` 인터페이스 구현 클래스를 작성했다면, 그 다음으로는 `CustomEditorConfigurer`가 해당 클래스를 사용하도록 설정해주면 된다.

다음 코드는 설정 예이다. 

이 설정에서 `CustomEditorConfigurer`의 `propertyEditorRegistrars` 프로퍼티에 앞서 구현한 `customPropertyEditorRegistrar`를 등록했다.

`CustomPropertyEditorRegistrar`는 `Date` 타입을 위한 `PropertyEditor`를 등록해주므로, 

아래 코드에서 `restClient` 빈의 `apiDate` 프로퍼티 값을 `Date` 타입으로 알맞게 변환할 수 있게 된다.

```xml
<bean class="org.springframework.beans.factory.config.CustomEditorConfigurer">
	<property name="customEditors">
    	<map>
        	<entry key="com.example.Money"
                   value="com.example.MyMoneyEditor"> </entry>
        </map>
    </property>
    
    <property name="propertyEditorRegistrars">
    	<list>
        	<ref bean="customPropertyEditorRegistrar" />
        </list>
    </property>
</bean>

<bean id="customPropertyEditorRegistrar" class="com.example.CustomPropertyEditorRegistrar">
	<property name="datePattern" value="yyyy-MM-dd HH:mm:ss" />
</bean>

<bean id="restClient" class="com.example.RestClient">
	<property name="serverUrl" value="https://www.google.com" />
    <property name="apiDate" value="2010-03-01 09:30:00" />
</bean>
```





### (2) ConversionService를 이용한 타입 변환

스프링 3 버전부터 `ConversionService`가 추가되었다.

`ConversionService`는 좀 더 범용적인 타입 변환을 처리하기 위한 인터페이스다.

앞서 살펴본 `PropertyEditor`는 자바빈 규약에 따라 문자열과 타입 간의 변환을 처리해주는 방식인데 반해, 

`ConversionService`는 타입과 타입 간의 변환을 처리하기 위한 기능을 정의하고 있다.



`ConversionService`를 스프링 빈으로 등록하면, 스프링은 `PropertyEditor` 대신 `ConversionService`를 이용해서 타입 변환을 처리한다.

`ConversionService` 인터페이스는 다음과 같이 타입 변환을 위해 필요한 메서드를 정의하고 있다.

```java
package org.springframework.core.convert;

public interface ConversionService {

	boolean canConvert(Class<?> sourceType, Class<?> targetType);
	boolean canConvert(TypeDescriptor sourceType, TypeDescriptor targetType);
	<T> T convert(Object source, Class<T> targetType);
	Object convert(Object source, TypeDescriptor sourceType, TypeDescriptor targetType);
	
}
```



스프링은 이미 `ConversionService` 인터페이스를 구현한 클래스를 제공하고 있으므로, 이 구현체를 사용하면 된다.

이 절에서는 기본 제공되는 `ConversionService`를 이용한 설정 방법에 대해 살펴볼 것이다.





#### - ConversionServiceFactoryBean을 이용한 ConversionService 등록

`DefaultConversionService` 클래스는 스프링이 제공하는 `ConversionService` 구현이다.

이 클래스를 `ConversionService`로 사용하려면 `ConversionServiceFactoryBean` 클래스를 빈으로 등록하면 된다.

`ConversionServiceFactoryBean`는 팩토리로서 `DefaultConversionService`를 빈 객체로 생성해준다.





```xml
<bean id="conversionService" class="org.springframework.context.support.ConversionServiceFactoryBean"></bean>
```

`ConversionServiceFactoryBean` 클래스를 설정할 때 주의할 점은 빈 객체의 이름을 "conversionService"로 등록해야 한다는 점이다.

그렇지 않을 경우 `PropertyEditor`를 사용한다.





위와 같이 "conversionSerivce" 빈 객체를 등록하면, 스프링은 빈 객체를 설정할 때 conversionService의 `convert()` 메서드를 이용해서 `String` 데이터를 `int` 타입으로 변환한다.

```xml
<bean id="collector" class="com.example.DataCollector">
	<!-- conversionService를 통해 타입 변환 -->
    <property name="threshold" value="5" />
</bean>
```



`DefaultConversionService` 클래스는 타입 변환을 직접 하지 않고 내부에 등록된 `GenericConverter`에 위임한다.

`DefaultConversionService`는 다수의 컨버터를 갖고 있으며, `convert()` 메서드를 실행하면 다음과 같은 방식으로 타입을 변환한다.

1. 등록된 `GenericConverter`들 중에서 소스 객체의 타입을 대상 타입으로 변환해주는 `GenericConverter`를 찾는다.
2. `GenericConverter`가 존재할 경우, 해당 `GenericConverter`를 이용해서 타입 변환을 수행한다.
3. 존재하지 않을 경우, Exception을 발생시킨다.



`PropertyEditor`와 마찬가지로 `DefaultConversionService`는 이미 기본적인 타입 변환을 위한 `GenericConverter`를 등록하고 있다.

따라서, `DefaultConversionService`만 등록하면 추가 설정 없이 기본 데이터 타입이나, `Properties`, URL 타입 등에 대한 변환을 처리할 수 있다.





#### - `GenericConverter`를 이용한 커스텀 변환 구현

`DefaultConversionService`가 타입 변환을 위해 사용하는 `GenericConverter` 인터페이스는 다음과 같다.

```java
package org.springframework.core.convert.converter;

public interface GenericConverter {
    
    Set<ConvertiblePair> getConvertibleTypes();
    Object convert(Object source, TypeDescriptor sourceType, TypeDescriptor targetType);
    
    public static final class ConvertiblePair {
        private final Class<?> sourceType;
        private final Class<?> targetType;
        
        public ConvertiblePair(Class<?> sourceType, Class<?> targetType) {
            this.sourceType = sourceType;
            this.targetType = targetType;
        }
        
        // ...
    }
}
```



`getConvertibleTypes()` 메서드는 변환 가능한 타입 쌍(`ConvertiblePair`)의 집합을 리턴한다.

`convert()` 메서드는 `TypeDescriptor` 정보를 이용해서 `source` 객체를 대상 타입으로 변환해서 리턴한다.

실제 `String`을 `Money`로 변환해주는 `GenericConverter`는 아래와 같이 구현할 수 있다.

```java
public class MoneyGenericConverter implements GenericConverter {
    
    private Set<ConvertiblePair> pairs;
    
    public MoneyGenericConverter() {
        Set<ConvertiblePair> pairs = new HashSet<>();
        pairs.add(new ConvertiblePair(String.class, Money.class));
        this.pairs = Collections.unmodifiableSet(pairs);
    }
    
    @Override
    public Set<ConvertiblePair> getConvertibleTypes() {
        return pairs;
    }
    
    
    @Override
    public Object convert(Object source, TypeDescriptor sourceType, TypeDescriptor targetType) {
        if(targetType.getType() != Money.class) throw new IllegalArgumentException("invalid type");
        if(sourceType.getType() != String.class) throw new IllegalArgumentException("invalid type");
        
        String moneyString = (String)source;
        Pattern pattern = Pattern.compile("([0-9]+)([A-Z]{3})");
        Matcher matcher = pattern.matcher(moneyString);
        
        if(!matcher.matches()) throw new IllegalArgumentException("invalid format");
        
        int amount = Integer.parseInt(matcher.group(1));
        String currency = matcher.group(2);
        
        return new Money(amount, currency);
    }
    
}
```

이렇게 타입 변환을 처리하는 `GenericConverter`를 구현했다면, `ConversionService`가 해당 `GenericeConverter`를 사용하도록 등록해주어야 하는데,

다음과 같이 `ConversionServiceFactoryBean`의 `converters` 프로퍼티에 구현한 `GenericConverter`를 설정해주면 등록된다.

```xml
<bean id="conversionService" class="org.springframework.context.support.ConversionServiceFactoryBean">
	<property name="converters">
    	<set>
        	<bean class="com.example.MoneyGenericConverter" />
        </set>
    </property>
</bean>

<bean class="com.example.InvestmentSimulator">
	<!-- "10000WON"을 Money로 변환할 때 MoneyGenericConverter 사용 -->
    <property name="minimumAmount" value="10000WON" />
</bean>
```



...





## 4. 요청 파라미터의 값 변환 처리

예약 시스템을 만들 때, 예약 날짜를 '2014-01-01'과 같은 형식으로 입력받고 싶을 때가 있을 것이다.

그리고, 커맨드 객체의 필드 타입은 `Date`나 `LocalDateTime` 등 날짜/시간 관련 타입을 사용하고 있다고 하자.

이런 경우에 스프링이 알아서 '2014-01-01' 값을 갖는 요청 파라미터를 커맨드 객체의 `Date` 타입 프로퍼티로 변환해주면 편리할 것이다.

스프링은 이미 문자열을 `Date`나 `LocalDateTime` 등의 타입으로 변환해주는 기능을 제공하고 있으며, 이 절에서는 그 방법에 대해 살펴본다.



> `PropertyEditor`와 `ConversionService`를 알아야 이 절을 이해하기 쉽다.



















