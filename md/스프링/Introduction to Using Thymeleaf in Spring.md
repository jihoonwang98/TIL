# Introduction to Using Thymeleaf in Spring :crab:



> 출처: https://www.baeldung.com/thymeleaf-in-spring-mvc
>
> 의역, 오역에 주의...





## 1. Introduction

**Thymeleaf**란? :thinking:

- HTML, XML, JS, CSS 등을 만들고 처리하는 **템플릿 엔진**



이번 포스팅에서는 **Thymeleaf**를 **Spring**과 함께 사용하는 방법에 대해 살펴본다. :mag:





## 2. Integrating Thymeleaf With Spring

먼저 **Thymeleaf**를 스프링 프로젝트에 사용하기 위해 의존성을 추가하는 방법을 살펴본다. 



Maven **pom.xml** 파일에 아래와 같이 의존성을 추가한다.

```xml
<dependency>
    <groupId>org.thymeleaf</groupId>
    <artifactId>thymeleaf</artifactId>
    <version>3.0.11.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.thymeleaf</groupId>
    <artifactId>thymeleaf-spring5</artifactId>
    <version>3.0.11.RELEASE</version>
</dependency>
```



> Spring 4 프로젝트에서는, thymeleaf-spring5 대신 thymeleaf-spring4를 입력







**SpringTemplateEngine** 클래스에서 **Thymeleaf** 템플릿 엔진 사용과 관련한 모든 환경 설정을 수행한다. 이 클래스를 **bean**으로 등록하자.

```java
@Bean
@Description("Thymeleaf Template Engine")
public SpringTemplateEngine templateEngine() {
    SpringTemplateEngine templateEngine = new SpringTemplateEngine();
    templateEngine.setTemplateResolver(templateResolver());
    templateEngine.setTemplateEngineMessageSource(messageSource());
    return templateEngine;
}

@Bean
@Description("Thymeleaf Template Resolver")
public ServletContextTemplateResolver templateResolver() {
    ServletContextTemplateResolver templateResolver = new ServletContextTemplateResolver();
    templateResolver.setPrefix("/WEB-INF/views/");
    templateResolver.setSuffix(".html");
    templateResolver.setTemplateMode("HTML5");

    return templateResolver;
}
```



**templateResolver** 빈의 **setPrefix**와 **setSuffix** 메서드의 역할은

- **setPrefix**의 역할
  - *webapp* 디렉터리 내의 *view* 페이지의 **위치 지정**
  - `templateResolver.setPrefix("/WEB-INF/views/");`
- **setSuffix**의 역할
  - *view* 파일의 **파일 확장자 지정**
  - `templateResolver.setSuffix(".html");`





> [참고] 
>
> *Spring MVC*의 **ViewResolver** 인터페이스는 *controller*가 반환하는 *view* 이름을 실제 *view object*에 *mapping*하는 역할을 한다. 
>
> 
>
> ```java
> @Controller
> public class TestController {
> 	@RequestMapping(value = "/home")
> 	public String home() {
> 		return "index";
> 	}
> }
> ```
>
> 
>
> **Controller**가 위의 코드처럼 논리적 이름인 *index*를 반환하면 **ViewResolver**가 *index*에다가 *Prefix*와 *Suffix*를 붙여서 실제 *view object*에 매핑해준다.





**ThymeleafViewResolver**는 **ViewResolver** 인터페이스를 *implements*하고, 주어진 논리적 *view name*으로 어떤 *Thymeleaf view*에 매핑할지를 결정해준다.

```java
@Bean
@Description("Thymeleaf View Resolver")
public ThymeleafViewResolver viewResolver() {
    ThymeleafViewResolver viewResolver = new ThymeleafViewResolver();
    viewResolver.setTemplateEngine(templateEngine());
    viewResolver.setOrder(1);
    return viewResolver;
}
```





이렇게 총 3가지의 *Bean*(*templateEngine*, *templateResolver*, *viewResolver*)을 등록하면 Spring에서 Thymeleaf를 사용하기 위한 사전 준비 작업이 끝난다.





## 3. Thymeleaf in Spring Boot

그렇다면 *Spring* 프로젝트가 아닌 **Spring Boot** 프로젝트에서는 어떻게 환경 설정을 할까?



**Spring Boot**는 *Thymeleaf*를 위해 *auto-configuration*을 제공한다. 

아래와 같이 *pom.xml* 메이븐 파일에 의존성을 추가하자.

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
    <version>2.3.3.RELEASE</version>
</dependency>
```



또한, 일반 *Spring* 프로젝트와는 다르게 환경 설정도 무지 간편하다.

어떠한 *configuration*도 필요하지 않다. Default로, HTML 파일들은 **resources/templates** 폴더에 위치시키면 된다.







## 4. Displaying Values from Message Source (Property Files)

*Thymeleaf*의 `th:text="#{key}"` 태그 속성은 *property file*로부터 값을 추출해 표시할때 사용한다.



이렇게 *property file*을 이용하기 위해서는 먼저 *property file*을 **messageSource** 빈으로 등록해야 한다.

```java
@Bean
@Description("Spring Message Resolver")
public ResourceBundleMessageSource messageSource() {
    ResourceBundleMessageSource messageSource = new ResourceBundleMessageSource();
    messageSource.setBasename("messages");
    return messageSource;
}
```





아래는 `welcome.message` 키로 등록한 값을 추출해 표시하는 Thymeleaf HTML code이다.

```html
<span th:text="#{welcome.message}" />
```







## 5. Displaying Model Attributes



### 5.1 Simple Attributes

`th:text="${attributename}"` 태그 속성은 **Model attributes**들의 값을 표시하는데 사용된다.



**Controller** 클래스에서 아래와 같이 *serverTime*이라는 이름으로 *model attribute*을 등록했다고 해보자.

```java
model.addAttribute("serverTime", dateFormat.format(new Date()));
```



위에서 등록한 *model attribute*을 추출하기 위해 아래와 같은 HTML code를 작성한다

```html
Current time is <span th:text="${serverTime}" />
```







### 5.2 Collection Attributes

만약 *model attributes*들이 **Collection** 형태라면, 

`th:each` 태그 속성을 사용해서 *collection* 내의 객체들을 반복 처리할 수 있다.





아래와 같이 *Student* 모델 클래스를 정의하자. 

```java
public class Student implements Serializable {
    private Integer id;
    private String name;
    // standard getters and setters 
    // setter와 getter는 지면상 생략한다.
}
```



이제 **Controller** 클래스에서 *Student* 객체들을 *list*에 담아서 이 *list*를 **model attribute**로 등록해보자.

```java
List<Student> students = new ArrayList<Student>();
// logic to build student data
model.addAttribute("students", students);
```



*Thymeleaf template code*를 아래와 같이 작성하면 *list*내의 *students*들의 field 값을 모두 표시하는 코드를 짤 수 있다.

```html
<tbody>
	<tr th:each="student: ${students}">
    	<td th:text="${student.id}" />
        <td th:text="${student.name}" />
    </tr>
</tbody>
```









## 6. Conditional Evaluation



### 6.1 *if* and *unless*

`th:if="${condition}"` 태그 속성은 `${ }` 안에 들어가 있는 **condition**이 만족될 때, 해당 태그 속성이 포함되어 있는 태그를 표시한다.

`th:unless="${condition}"` 태그 속성은 반대로 *condition*이 만족되지 않을 때, 해당 태그 속성이 포함되어 있는 태그를 표시한다.





*Student* 모델 클래스에 *gender* 필드를 추가하자.

```java
public class Student implements Serializable {
    private Integer id;
    private String name;
    private Character gender;  // 'M' or 'F'
    
    // standard getters and setters
}
```



만약 우리가 *Student*의 *gender* 필드에 따라 "Male" 또는 "Female"을 출력하고 싶다면, 아래와 같이 *Thymeleaf code*를 작성할 수 있다.

```html
<td>
    <span th:if="${student.gender} == 'M'" th:text="Male" /> 
    <span th:unless="${student.gender} == 'M'" th:text="Female" />
</td>
```





### 6.2 *switch* and *case*

`th:switch`와 `th:case` 태그 속성들도 역시 조건을 만족하는지 여부에 따라 *content*를 출력할지를 결정한다.



방금 위에서 작성한 코드를 `th:switch`와 `th:case` 속성으로 다시 작성하면 아래와 같다.

```html
<td th:switch="${student.gender}">
    <span th:case="'M'" th:text="Male" /> 
    <span th:case="'F'" th:text="Female" />
</td>
```





## 7. Handling User Input





























