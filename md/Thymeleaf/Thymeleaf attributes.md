# Thymeleaf attributes



> https://frontbackend.com/thymeleaf/thymeleaf-tutorial



## 1. Getting Started



### `application.yml ` 설정 파일

```yml
# THYMELEAF (ThymeleafAutoConfiguration)
spring:
  thymeleaf:
    cache: true  # Enable template caching
    check-template: true  # Check that the template exists before rendering it
    check-template-location: true # Check that the templates location exists
    content-type: text/html # Content-Type value
    enabled: true # Enable MVC Thymeleaf view resolution
    encoding: UTF-8 # Template encoding
    excluded-view-names: # Comma-separated list of view names that should be excluded from resolution
    mode: HTML5
    prefix: classpath:/templates/ 
    suffix: .html
    template-resolver-order:  # Order of the template resolver in the chain
    view-names: # Comma-separated list of view names that can be resolved.
```

위와 같은 설정을 Spring Boot가 자동으로 설정해준다.





## 2. `th:href` 속성 / URL 지정

Thymeleaf는 link expressions(`@{...}`)를 사용해서 URL을 쉽게 생성하는 방법을 제공한다.



### 절대 경로 (Absolute URLs)

Absolute URL들은 다른 server의 링크를 가리키는데 쓰이곤 한다.

얘네들은 `http://` 또는 `https://` 같은 프로토콜 이름으로 시작한다.



아래의 예시는 `https://frontbackend.com/tag/thymeleaf`에 대한 절대 경로를 생성한다.

```html
<a th:href="@{https://frontbackend.com/tag/thymeleaf}">Thymeleaf</a>
```



위의 코드를 렌더링하면 아래와 같이 나올 것이다.

```html
<a href="https://frontbackend.com/tag/thymeleaf">Thymeleaf</a>
```





### 상대 경로 (Context-relatvie URLs)

상대 경로란 서버에 정의된 웹 애플리케이션 root context에 상대적인 경로를 뜻한다.

예를 들어서, 만약 당신의 Spring Boot 애플리케이션이 context path를 사용하고 있어서

`application.properties` 파일에 `server.contextPath=/myapp`과 같이 설정했다면

`myapp`이 context name이 될 것이다.





상대 경로는 `/`로 시작한다.



아래와 같은 thymeleaf 코드가 있다고 하자.

```java
<a th:href="@{/tag/thymeleaf}">Thymeleaf</a>
```



위의 코드는 아래와 같은 코드로 렌더링된다. (앞에 context path가 붙는다)

```java
<a href="/myapp/tag/thymeleaf">Thymeleaf</a>
```



물론 context path가 정의되어 있지 않으면 아래와 같이 된다.

```html
<a href="/tag/thymeleaf">Thymeleaf</a>
```





### 서버 상대 경로 (Server-relative URLs)

Server-relative URL이란 Context-relative URL과 비슷하지만 다른 context를 가리킬 수 있다.



아래와 같은 코드를 `myapp`이란 context에서 돌렸을 때,

```html
<a th:href="@{~/tag/thymeleaf}">Thymeleaf</a>
```



context를 무시하고 아래와 같이 렌더링된다.

```html
<a href="/tag/thymeleaf">Thymeleaf</a>
```





### 프로토콜 상대 경로 (Protocol-relative URLs)

Protocol-relative URL이란 `styles`, `scripts`, `images`와 같은 외부 resource들을 포함하는데 사용된다.

이런 종류의 URL은 파일 시스템 내의 절대 경로처럼 작동하고 설정된 protocol을 유지한다.



아래 예시는 `https://frontbackend.com`의 `script.js` 파일을 Protocol-relative URL을 사용해서 포함시키는 예제다.

```html
<script th:src="@{//frontbackend.com/script.js}"></script>
```



위의 코드는 unmodified하게 렌더링된다.

```html
<script src="//frontbackend.com/script.js"></script>
```





### 요청 파라미터 지정하기 (URLs with parameters)

URL에 query parameter를 지정하려면 얘네들을 괄호`()`안에 넣어야 한다.



tag라는 name에 thymeleaf라는 value를 가지는 parameter를 추가하는 예시

```java
<a th:href="@{/tags(tag='thymeleaf')}">thymeleaf</a>
```



위의 코드는 아래와 같이 렌더링된다.

```html
<a href="/tags?tag=thymeleaf">thymeleaf</a>
```





파라미터 여러개를 설정하려면, 콤마로 구분해서 써주면 된다.

```html
<a th:href="@{/tags(tag='thymeleaf',order=1,sort=true)}">thymeleaf sorted</a>
```



아래와 같이 렌더링된다.

```html
<a href="/tags?tag=thymeleaf&order=1&sort=true">thymeleaf sorted</a>
```





### URL fragment identifiers

Fragment identifier들은 URL에 포함될 수 있다.



```html
<a th:href="@{/tags#all_tags(action='show')}">tags</a>
```



아래와 같이 렌더링 된다.

```html
<a href="/tags?action=show#all_tags">tags</a>
```





### Using expression in URLs

```html
<a th:href="@{/customers(id=${customer.id}, action=(${customer.active} ? 'show_active' : 'show_limited'))}">customer details</a>
```



`${customer.id}`가 1000이고, `${customer.active}`가 true면 아래와 같이 렌더링된다.

```html
<a href="/customers?id=1000&action=show_active">customer details</a>
```





## 3. `th:each` 속성 / 반복문

Thymeleaf가 iterable하다고 판단하는 객체들은 아래와 같다.

- `Iterable` 인터페이스를 implementing 하고 있는 객체
- `Enumeration` 인터페이스를 implementing 하고 있는 객체
- `Iterator` 인터페이스를 implementing 하고 있는 객체
- `Map` 인터페이스를 implementing 하고 있는 객체
- `arrays`





### 예시

customer 리스트를 HTML 테이블로 표현하고 싶다고 하자.



**Customer**

```java
@Data
public class Customer {
    private String firstName;
    private String lastName;
    private String email;
    private int age;
    private List<Address> addressList;
}
```



**Address**

```java
@Data
public class Address {
    private String street;
    private String city;
    private String zip;
    private String state;
}
```





**CustomerController**

```java
@Controller
@RequiredArgsConstructor
public class CustomerController {
    
    private final CustomerService customerService;
    
    @GetMapping("customers")
    public String main(Model model) {
        model.addAttribute("customers", customerService.mockCustomers());
        
        return "customers";
    }
}
```





**customers.html**

```html
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8" />
</head>
<body>
    
    <table>
        <tr>
        	<th>No</th>
            <th>First Name</th>
            <th>Last Name</th>
            <th>Age</th>
            <th>Email</th>
            <th>Address List</th>
        </tr>
        
        <tr th:each="customer, custStat : ${customers}">
			<td th:text="${custStat.count}">1</td>
            <td th:text="${customer.firstName}">John</td>
            <td th:text="${customer.lastName}">Doe</td>
            <td th:text="${customer.age}">18</td>
            <td th:text="${customer.email}">john.doe@frontbackend.com</td>
            
            <td>
            	<p th:each="address : ${customer.addressList}">
                    <span th:text="${address.street}">Street</span>
                    <span th:text="${address.city}">City</span>
                    <span th:text="${address.zip}">ZIP</span>
                    <span th:text="${address.state}">State</span>
                </p>
            </td>
        </tr>
    </table>
</body>    
</html>
```



Thymeleaf 엔진은 `th:each` 속성을 가진 template 조각을 iterable collection의 각 요소마다 반복한다.





### Iteration Status

Thymeleaf는 반복 과정에 대한 정보를 알려주는  `status` 객체를 제공한다.

`status` 객체는 아래와 같은 정보를 포함하고 있다.



`status` 객체의 이름을 `custStat`이라고 하자.

(`status` 객체 이름은 suffix로 `Stat`만 붙이면 되는 것 같다.)



- **iteration index**
  - `${custStat.index}`: 0 시작 인덱스 제공
  - `${custStat.count}`: 1 시작 인덱스 제공
- **total amount**
  - `${custStat.size}`
- **iteration variable**
  - `${custStat.current}`
- **첫 번째 요소 / 마지막 요소 판단**
  - `${custStat.first}`
  - `${custStat.last}`





## 4. 조건문 `th:if`(`th:unless`), `th:switch`





### `th:if`와 `th:unless`

#### 예제

**Snippet 클래스**

```java
@Getter 
@Builder
public class Snippet {
    private String title;
    private String code;
    private String language;
    private boolean published;
    private String author;
}
```

위의 Snippet 클래스를 이용해서 오직 published된 snippet들만 출력하는 예제를 만들어 보자.



```html
<!DOCTYPE HTML>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8"/>
    <title>Spring Boot Thymeleaf Application</title>
</head>
<body>
<h1>List of Snippets</h1>
<table>
    <tr>
        <th>No</th>
        <th>Title</th>
        <th>Author</th>
        <th>Code</th>
        <th>Language</th>
    </tr>
    <tr th:each="snippet, snippStat : ${snippets}" th:if="${snippet.published}">
        <td th:text="${snippStat.index + 1}">1</td>
        <td th:text="${snippet.title}">JavaScript example</td>
        <td th:text="${snippet.author}">Doe</td>
        <td th:text="${snippet.code}">alert('test');</td>
        <td th:text="${snippet.language}">javascript</td>
    </tr>
</table>
</body>
</html>
```



Thymeleaf는 조건문 `th:if="${snippet.published}"`를 평가해서 true일 때만 HTML `tr` 요소로 렌더링한다.



`th:if=${value}` 문에서 `value`에 들어가는 객체 타입에 따라 true, false 기준이 다르다.

- value가 int인 경우
  - value가 양수: true
  - value가 음수: false
- value가 char인 경우
  - value가 non-zero인 경우: true
  - 그 외: false
- value가 String인 경우
  - value가 "false", "off", "no"가 아닌 경우: true
  - value가 "false", "off", "no"인 경우: false
- value가 Object인 경우
  - value가 null이 아닌 경우: true
  - value가 null인 경우: false
- value가 boolean인 경우
  - value가 true: true
  - value가 false: false



`th:unless`는 `th:if`의 반대다.





### Multiple conditions

`and`나 `or` 키워드를 사용해서 여러 개의 조건을 결합할 수 있다.



snippet.published가 true이고  snippet.language가 'java'로 시작하는 경우

```html
th:if="${snippet.published and #strings.startsWith(snippet.language, 'java')}"
```







### `th:switch` 조건문

`th:switch`, `th:case`는 JavaScript의 `switch`, `case`와 동일하다.

상수(enum 같은)에도 적용할 수 있을 뿐만 아니라 String이나 Number 타입에도 적용 가능하다.

`th:case="*"`은 `th:switch` 조건문에서 default 역할을 한다.



```html
<div th:each="snippet : ${snippets}">
    <span th:switch="${snippet.language}">
        <p th:case="'java'">Java Code: <th:block th:text="${snippet.code}">java code</th:block></p>
        <p th:case="'javascript'">JavaScript Code:<th:block th:text="${snippet.code}">javascript code</th:block></p>
        <p th:case="*">Other language</p>
    </span>
</div>
```







### Elvis Operator

Elvis Operator는 `A ?: B` 연산자를 말한다.

얘는 A, B 두 개의 인자를 받고, A가 false나 null이면 B를 리턴한다.



```html
<div th:each="snippet : ${snippets}">
    <div th:text="${snippet.author}?: '(no author specified)'">John Doe</td>
</div>
```

위의 예시는 author가 null이나 empty면 (no author specified) 텍스트가 들어간다.







## 5. 태그에 속성 추가하기 `th:attr`

### 예시

**Post 클래스**

```java
class Post {
    private Long id;
    private String title;
    private Date date;
}
```



```html
<div th:attr="id=${post.id},title=${post.title},date=${#dates.format(post.date, 'yyyy-MM-dd')}">
</div>
```



위 코드는 아래와 같이 렌더링 된다.

```html
<div id="1" title="This is the title" date="2019-01-01"></div>
```







## 6. Form 관련 속성들



### Form 만들기

Thymeleaf는 form을 만드는데 사용할 수 있는 속성들을 제공한다.

- `th:field`: form-backing bean의 property에 input을 바인딩하는데 사용된다.
- `th:errors`: form validation error들을 들고 있는 속성
- `th:errorclass`: 명시된 필드가 validation error가 존재할때 form input에 추가될 CSS class
- `th:object`: command 객체를 들고 있는 속성. backend 측에서 form을 표현하는 객체다.





### Command 객체

커맨드 객체는 Form의 input field들과 관련된 속성들을 포함하고 있는 빈 객체다.

이 객체는 getter, setter가 선언된 POJO 클래스다.

커맨드 객체는 어떠한 비즈니스 로직도 들고 있으면 안된다.



아래의 예시는 커맨드 객체에 대한 reference를 `th:object` 속성으로 들고 있게 한다.

```html
<form th:action="@{/registration}" th:object="${registration}" method="post">
    ...
</form>
```



주의

- 하나의 form 요소 안에는 오직 하나의 `th:object` 속성만이 존재할 수 있다.

- bean object를 정의하는 expression이 해당 bean 객체를 direct로 가리켜야 한다.

  - Property navigation(`머시기.머시기` 같은 꼴)은 error를 야기한다.

  ```html
  <form th:object="${customer}"></form>  <!-- direct로 가리키는 건 가능 -->
  <form th:object="${base.customer}"></form>  <!-- property navigation으로 가리키는 건 불가능 -->
  ```





### Inputs

`th:field` 속성은 html의 `<input>` 태그의 속성으로 사용할 수 있다.

얘는 bean 객체의 property와 input 값을 연결시켜주는 역할을 한다.

이 속성은 html의 `<input type=?>` 태그의 type이 뭐냐에 따라 다르게 동작한다.

Thymeleaf는 HTML5의 새로운 input type들(type="color", type="datetime" 등등)을 모두 지원한다.



아래 예시는 HTML `text` 입력값을 빈 객체의 username 프로퍼티에 binding하는 것을 보여준다.

```html
<input type="text" th:field="*{username}" />
```



number, datetime, color 같은새로운 HTML5의 input type을 이용해 더 복잡한 예시를 들어보자.

```html
<form th:action="@{/sampleInputs}" th:object="${sampleInputs}" method="post">
    <div>
        <label>Date:</label>
        <input type="text" th:field="*{dateField}" placeholder="yyyy-MM-dd" />
    </div>
    <div>
        <label>Integer:</label>
        <input type="number" th:field="*{numberField}" placeholder="integer" />
    </div>
    <div>
        <label>Double:</label>
        <input type="text" th:field="*{doubleField}" placeholder="double" />
    </div>
    <div>
        <label>String:</label>
        <input type="text" th:field="*{textField}" placeholder="string" />
    </div>
    <div>
        <label>Color:</label>
        <input type="color" th:field="*{colorField}" placeholder="color" />
    </div>
    <div>
        <label>Date time:</label>
        <input type="datetime-local" th:field="*{dateTimeField}" placeholder="date" />
    </div>

    <input type="submit" value="Submit"/>
</form>
```



위의 form 태그에 부착되는 커맨드 객체는 아래와 같다.

```java
@NoArgsConstructor
@Getter @Setter @ToString
public class SampleInputs {

    @DateTimeFormat(pattern = "yyyy-MM-dd")
    private Date dateField;

    private Double doubleField;
    private Integer numberField;
    private String textField;
    private String colorField;

    @DateTimeFormat(pattern = "yyyy-MM-dd'T'HH:mm")
    private Date dateTimeField;

}
```





![](https://frontbackend.com/storage/thymeleaf/forms.png)

위와 같은 Sample data를 입력하면,

커맨드 객체는 아래와 같은 값들을 얻게 된다.

```java
SampleInputs(
    dateField=Thu Oct 10 00:00:00 CEST 2019, 
    doubleField=123.12, 
    numberField=21, 
    textField=This is simple text, 
    colorField=#d91111, 
    dateTimeField=Thu Oct 10 23:00:00 CEST 2019
)
```



 

### Checkboxes

#### 예제

```html
<form th:action="@{/sampleCheckboxes}" th:object="${sampleCheckboxes}" method="post">

    <ul>
        <li th:each="color : ${allSampleColors}">
            <input type="checkbox" th:field="*{colors}" th:value="${color}"/>
            <label th:for="${#ids.prev('colors')}" th:text="${color}">RED</label>
        </li>
    </ul>

    <div>
        <label th:for="${#ids.next('first')}">First:</label>
        <input type="checkbox" th:field="*{first}" />
    </div>
    <div>
        <label th:for="${#ids.next('second')}">Second:</label>
        <input type="checkbox" th:field="*{second}"/>
    </div>

    <input type="submit" value="Submit"/>
</form>
```

`#ids.prev('colors')`와 `#idx.next('second')` -> Thymeleaf utility class





```java
@NoArgsConstructor
@Getter @Setter @ToString
public class SampleCheckboxes {

    private SampleColor[] colors;

    private Boolean first;
    private Boolean second;
}
```



![](https://frontbackend.com/storage/thymeleaf/checkboxes.png)



다음과 같은 샘플 데이터를 입력하면 아래와 같이 나온다.

```java
SampleCheckboxes(
    colors=BLUE, WHITE,
    first=true, 
    second=false
)
```





### Radio Buttons

```html
<form th:action="@{/sampleRadioButtons}" th:object="${sampleRadioButtons}" method="post">

    <ul>
        <li th:each="color : ${allSampleColors}">
            <input type="radio" th:field="*{color}" th:value="${color}" />
            <label th:for="${#ids.prev('color')}" th:text="${color}">RED</label>
        </li>
    </ul>

    <input type="submit" value="Submit"/>
</form>
```



```java
@NoArgsConstructor
@Getter @Setter @ToString
public class SampleRadioButtons {

    private SampleColor color;
}
```



![](https://frontbackend.com/storage/thymeleaf/radiobuttons.png)

위와 같은 샘플 데이터는 아래와 같이 나온다.

```java
SampleRadioButtons(
    color=WHITE
)
```





### Dropdown/List selectors

HTML dropdown을 만들기 위해서는 `select` 요소와 nested `option` 태그를 정의해야 한다.

`select` 요소는 `th:field` 속성을 포함해야 한다.



```html
<form th:action="@{/sampleDropdowns}" th:object="${sampleDropdowns}" method="post">

    <div>
        <label>Select one color:</label>
        <select th:field="*{color}">
            <option th:each="color : ${allSampleColors}"
                    th:value="${color}"
                    th:text="${color}">RED
            </option>
        </select>
    </div>

    <div>
        <label>Select colors:</label>
        <select th:field="*{colors}" multiple>
            <option th:each="color : ${allSampleColors}"
                    th:value="${color}"
                    th:text="${color}">RED
            </option>
        </select>
    </div>

    <input type="submit" value="Submit"/>
</form>
```





```java
@NoArgsConstructor
@Getter @Setter @ToString
public class SampleDropdowns {

    private SampleColor color;
    private SampleColor[] colors;
}
```



![](https://frontbackend.com/storage/thymeleaf/dropdowns.png)

위 그림처럼 선택하면 아래와 같이 나온다.

```java
SampleDropdowns(
    color=GREEN, 
    colors=RED, BLUE, WHITE
)
```





### Validation and Error Messages



#### (1) Field errors

어떤 필드가 에러를 포함하고 있는지 체크하려면 `#fields.hasErrors(field_expression)` 메서드를 사용하면 된다.



아래 예시는 `dateField` 입력값에 에러가 생기면 `fieldError` 클래스를 붙여준다.

```html
<input type="text" th:field="*{dateField}" th:class="${#fields.hasErrors('dateField')}? fieldError" />
```



아래 코드처럼 명시된 필드의 모든 에러들을 나열할 수도 있다.

```html
<div>
  <p th:each="error : ${#fields.errors('dateField')}" th:text="${error}"></p>
</div>
```



`th:errorclass`는 해당 필드에 어떠한 에러라도 발생하면 명시한 CSS 클래스를 붙여준다.

```html
<input type="text" th:field="*{dateField}" class="form-input" th:errorclass="error" />
```



위 코드에서 `dateField`에 에러가 있다면, 아래와 같이 렌더링 될 것이다.

```html
<input type="text" id="dateField" name="dateField" value="2013-01-01" class="form-input error" />
```





#### (2) All errors

우리는 폼에서 발생한 모든 에러를 보여줄 수 있다.

`#fields.hasErrors()` 유틸리티 메서드에 `*` 또는 `all` 파라미터를 주거나

`#fields.hasAnyErrors()` 유틸리티 메서드를 사용하면 된다.

```html
<ul th:if="${#fields.hasErrors('*')}">
  <li th:each="error : ${#fields.errors('*')}" th:text="${error}">error</li>
</ul>
```





#### (3) Global errors

Thymeleaf form에는 세 번째 에러 타입인 **global**도 있다.

이 global error들은 어떠한 form이나 field와도 연관되지 않은 에러다.

우리는 이러한 global error들을 애플리케이션의 백엔드 측에서 프로그래머티컬하게 집어넣을 수 있다.



이러한 에러들에 접근하기 위해서는 `#fieldhasErrors()` 메서드에  `global` 파라미터를 집어넣으면 된다.

```html
<ul th:if="${#fields.hasErrors('global')}">
  <li th:each="error : ${#fields.errors('global')}" th:text="${error}">error</li>
</ul>
```





## 7. Using Enums in Thymeleaf



아래와 같은 `Planet` Enum 객체를 준비하자.

```java
public enum Planet {

    MERCURY,
    VENUS,
    EARTH,
    MARS,
    JUPITER,
    SATURN,
    URANUS,
    NEPTUNE
}
```



제출된 폼을 저장하기 위해 `Home` 클래스를 커맨드 객체로 사용한다.

```java
@Getter @Setter @ToString
@NoArgsConstructor
public class Home {

    private String title;
    private Planet planet;

}
```





### Using enums in dropdowns

Java enum 객체들은 모든 이용가능한 item들을 반환하는 `values()`라는 static method를 제공한다.

이 메서드를 이용해서 모든 planet들을 선택 가능하게 dropdown으로 만들어보자.

```html
<form th:action="@{/selectedPlanet}" method="post" th:object="${home}">
    <input th:field="*{title}" placeholder="Title"/>

    <select name="planet" th:field="*{planet}">
        <option th:each="planet : ${T(com.frontbackend.thymeleaf.enums.model.Planet).values()}"
                th:value="${planet}" th:text="${planet}"></option>
    </select>

    <input type="submit" value="Submit"/>
</form>
```



`T` 연산자는 Spring EL (Expression Language)를 참고하면 알 수 있다.

우린 이 `T` 연산자를 사용해서 Enum 객체 `Planet`의 `values()` 메서드를 사용한다.

이 외에도 title을 설정할 수 있게 input field를 만들었다.



![](https://frontbackend.com/storage/thymeleaf/thymeleaf-enum-simple.png)

위와 같이 만들어졌다.

보시다싶이 Enum에 정의된 모든 행성들이 같은 이름으로 dropdown에 나타나는 것을 볼 수 있다.

만약 Enum에 정의된 값과 다른 이름으로 select option에 나타나게 하고 싶으면 어떡할까?

다음 절에서 설명한 locale messages 기능을 사용하면 된다.





### Using locale messages for enum values

기본 locale 설정을 바꾸기 위해서 `WebMvcConfigurer` 클래스를 정의한다.

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Bean
    public LocaleResolver localeResolver(){
        SessionLocaleResolver localeResolver = new SessionLocaleResolver();
        localeResolver.setDefaultLocale(Locale.US);
        return  localeResolver;
    }

    @Bean
    public LocaleChangeInterceptor localeChangeInterceptor() {
        LocaleChangeInterceptor localeChangeInterceptor = new LocaleChangeInterceptor();
        localeChangeInterceptor.setParamName("lang");
        return localeChangeInterceptor;
    }

    @Override
    public void addInterceptors(InterceptorRegistry registry){
        registry.addInterceptor(localeChangeInterceptor());
    }
}
```



이제 우리는 어떤 내부 URL이든 요청 파라미터에 `lang` 값을 지정해주면 locale을 바꿀 수 있다.

- `?lang=EN`: for English,
- `?lang=ES`: for Spanish



dropdown 메뉴의 option들이 이제는 message expression(`th:text="#{${'planet.' + planet}}"`)들을 가지고 있다.

```html
<form th:action="@{/selectedPlanet}" method="post" th:object="${home}">
    <input th:field="*{title}" placeholder="Title"/>

    <select name="planet" th:field="*{planet}">
        <option th:each="planet : ${T(com.frontbackend.thymeleaf.enums.model.Planet).values()}"
                th:value="${planet}" th:text="#{${'planet.' + planet}}"></option>
    </select>

    <input type="submit" th:value="#{submit.button}"/>
</form>

<br/>

<a th:href="@{/(lang=en)}">EN</a> | <a th:href="@{/(lang=es)}">ES</a>
```



그리고 아래와 같이 properties 파일을 작성하자.

```properties
# English (messages_en_US.properties)
planet.MERCURY=Mercury
planet.VENUS=Venus
planet.EARTH=Earth
planet.MARS=Mars
planet.JUPITER=Jupiter
planet.SATURN=Saturn
planet.URANUS=Uranus
planet.NEPTUNE=Neptune
submit.button=Submit

home.title=Title
home.planet=Planet
home.selected=Selected planet


# Spanish (messages_es.properties)
planet.MERCURY=Mercurio
planet.VENUS=Venus
planet.EARTH=Tierra
planet.MARS=Marte
planet.JUPITER=Júpiter
planet.SATURN=Saturno
planet.URANUS=Urano
planet.NEPTUNE=Neptuno
submit.button=Enviar

home.title = Título
home.planet = Planet
home.selected = Planeta seleccionado
```



그럼 이제 설정된 Locale에 따라 message가 다르게 나온다.

![](https://frontbackend.com/storage/thymeleaf/thymeleaf-enums-english.png)

![](https://frontbackend.com/storage/thymeleaf/thymeleaf-enums-spanish.png)





### Using enum in comparison statements



#### (1) `If` statement

```html
<th:block th:each="planet : ${T(com.frontbackend.thymeleaf.enums.model.Planet).values()}">
    <span th:if="${planet == T(com.frontbackend.thymeleaf.enums.model.Planet).EARTH}"
          th:text="${planet + ' = our home'}"></span>
</th:block>
```





#### (2) `Switch` statement

Thymeleaf는 enum에 대해서도 `switch-case`문을 지원한다.



```html
<th:block th:each="planet : ${T(com.frontbackend.thymeleaf.enums.model.Planet).values()}">
    <div>
        <span th:text="${planet}"></span>
        <th:block th:switch="${planet}">
            <span th:case="${T(com.frontbackend.thymeleaf.enums.model.Planet).MERCURY}"> - First planet from the sun</span>
            <span th:case="${T(com.frontbackend.thymeleaf.enums.model.Planet).VENUS}"> - Second planet from the sun</span>
            <span th:case="${T(com.frontbackend.thymeleaf.enums.model.Planet).EARTH}"> - Third planet from the sun</span>
            <span th:case="${T(com.frontbackend.thymeleaf.enums.model.Planet).MARS}"> - Fourth planet from the sun</span>
            <span th:case="${T(com.frontbackend.thymeleaf.enums.model.Planet).JUPITER}"> - Sixth planet from the sun</span>
            <span th:case="${T(com.frontbackend.thymeleaf.enums.model.Planet).SATURN}"> - Seventh planet from the sun</span>
            <span th:case="${T(com.frontbackend.thymeleaf.enums.model.Planet).URANUS}"> - Eighth planet from the sun</span>
            <span th:case="${T(com.frontbackend.thymeleaf.enums.model.Planet).NEPTUNE}"> - Ninth planet from the sun</span>
        </th:block>
    </div>
</th:block>
```





## 8. Working with local variables in Thymeleaf

teamplate의 가독성을 향상시키기 위해 local variable을 만들고 사용하는 방법을 알아보자.



### `th:with` 속성

Thymeleaf에서 local variable이란 template의 어떤 fragment를 위해서만 정의되고 그 fragment 내에서만 이용가능한 변수다.

local variable을 선언하고 싶다면 `th:with` 속성을 사용하면 된다.



아래와 같은 template이 있다고 하자.

```html
<div th:with="firstCustomer=${customers[0]}">
  <p>
    The name of the first customer is <span th:text="${firstCustomer.name}">John Doe</span>.
  </p>
</div>
```

`th:with` 속성은 `firstCustomer`라는 이름의 지역 변수를 만들고 그 안에는 customers 객체의 첫 번째 요소를 저장하고 있다.

이 `firstCustomer` 지역 변수는 해당 `div` 태그 안에 있는 모든 자식 태그 안에서 접근 가능하다.





지역 변수를 여러개 선언하고 싶으면 콤마로 구분해서 작성하면 된다.

```html
<div th:with="firstCustomer=${customers[0]}, secondCustomer=${customers[1]}">
   <p>
    The name of the first customer is <span th:text="${firstCustomer.name}">Monika John</span>.
  </p>
  <p>
    But the name of the second customer is 
    <span th:text="${secondCustomer.name}">Johanna Alba</span>.
  </p>
</div>
```



아래처럼 선언한 지역변수를 해당 속성에서 재활용하는 것도 가능하다.

```html
<div th:with="customer=${customer.id},account=${accounts[customer]}">...</div>
```