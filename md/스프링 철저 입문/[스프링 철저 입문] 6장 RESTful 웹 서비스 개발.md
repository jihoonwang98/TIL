## 6.1 REST API 아키텍처

**REST API**에 대한 구체적인 개발 방법을 설명하기 전에 먼저 **REST API**가 무엇인지 살펴보자. **REST**는 '**REpresentational State Transfer**'의 약자이며, 클라이언트와 서버 사이에서 데이터를 주고받는 애플리케이션을 만들기 위한 아키텍처 스타일 중의 하나다.



**REST 아키텍처** 스타일에서 가장 중요한 것은 '**리소스**'라는 개념이다. **REST API**는 데이터베이스 등에서 관리되는 정보에서 클라이언트에 제공할 정보를 '**리소스**'의 형태로 추출한다. 추출된 **리소스**는 웹에 공개되고 **리소스**에 접근(CRUD 조작)하기 위한 수단으로 '**REST API**'를 제공한다.



### 6.1.1 Resource Oriented Architecture (ROA)

**ROA**는 **RESTful** 웹 애플리케이션을 구축하기 위한 구체적인 아키텍처를 정의하고 있다. 이 절에서는 **ROA**의 중요한 특징 7가지를 소개하고 **REST API**를 설계하거나 개발할 때 고려해야 할 사항을 설명할 것이다.



#### 웹의 리소스로 공개

클라이언트에 제공할 정보는 웹에서 리소스로 공개한다. 이는 HTTP 프로토콜을 사용해 리소스에 접근할 수 있다는 것을 의미한다.



#### URI를 통한 리소스 식별

웹에 공개할 리소스에는 그 리소스를 고유하게 식별할 수 있는 **URI(Universal Resource Identifier)**를 할당해 같은 네트워크에 연결돼 있다면 어디서든 같은 리소스에 접근할 수 있게 한다.



> 리소스에 할당할 URI로는 '리소스의 종류를 나타내는 명사'와 '리소스를 고유하게 식별할 수 있는 값(ID 등)'을 조합하는 것이 일반적이다. `https://api.github.com/users/spring-projects`를 예로 들어 설명하면, '**users**' 부분이 '**리소스 종류**'에 해당하고, '**spring-project**' 부분이 '**리소스를 고유하게 식별할 수 있는 값**'이 된다.



#### HTTP 메서드를 통한 리소스의 조작

리소스에 대한 CRUD 조작은 HTTP 메서드(GET, POST, PUT, DELETE 등)를 용도에 맞게 잘 나눠서 구현해야 한다. **REST API**를 만들 때 자주 사용하는 HTTP 메서드는 다음과 같이 4가지가 있다.



| HTTP 메서드 | 설명                                                         |
| ----------- | ------------------------------------------------------------ |
| **GET**     | URI에 지정된 리소스를 가져온다.                              |
| **POST**    | 리소스를 생성하고 생성된 리소스에 접근할 수 있는 URI를 받아온다. |
| **PUT**     | URI에 지정한 리소스를 생성하거나 갱신한다.                   |
| **DELETE**  | URI에 지정한 리소스를 삭제한다.                              |



이 밖에도 **HEAD, PATCH, OPTIONS** 같은 HTTP 메서드가 있지만 **GET, POST, PUT, DELETE**에 비교하면 사용 빈도는 낮다.



#### 적절한 포맷을 사용

리소스 포맷으로는 읽기 쉽고 데이터 구조를 잘 표현하는 **JSON**이나 **XML**과 같은 포맷을 사용한다. 일반적으로는 **JSON**이나 **XML**을 사용하지만 **REST API** 자체는 특정 포맷을 사용하도록 규정하지 않기 때문에 애플리케이션의 요구사항에 따라 적절한 포맷을 선택하면 된다.



- **리소스 포맷의 예(깃허브에서 제공하는 사용자 정보 중에서 일부를 발췌)**

```json
{
	"login": "spring-projects",
	"id": 317776,
	"avatar_url": "https://avatars.githubusercontent.com/u/317776?v=3",
	"type": "Organization",
	"site_admin": false,
	"name": "Spring",
	"company": null,
	"public_repos": 177,
	"created_at": "2010-06-29T18:58:02Z",
	"updated_at": "2015-09-28201705-10T10:38:45Z"
}
```





#### 적절한 HTTP 상태 코드를 사용

클라이언트에 응답을 할 때는 **HTTP 상태 코드**를 설정해줘야 한다. **HTTP 상태 코드**는 서버 측의 처리 결과를 알려주기 위한 것으로, 의미하는 내용은 아래의 표와 같다. 서버 측의 처리 결과에 따라 적절한 반응을 보여줘야 하는 클라이언트 애플리케이션의 입장에서는, **REST API**가 응답하는 **HTTP 상태 코드의 정확성**이 상당히 중요하다. 참고로 **HTTP 상태 코드**는 어떤 경우에 어떤 코드가 사용되는지가 **RFC** 문서에 정의돼 있으므로 클라이언트가 서버의 상태를 오판하는 것을 미연에 방지할 수 있다.



**HTTP 상태 코드 분류**

| **분류** | **설명**                                                     |
| -------- | ------------------------------------------------------------ |
| **1xx**  | 요청을 접수하고 처리를 계속하고 있음을 알리는 응답 코드      |
| **2xx**  | 요청을 접수하고 처리가 완료됐음을 알리는 응답 코드           |
| **3xx**  | 요청을 완료하기 위해 추가적인 처리(리다이렉트 등)가 필요함을 알리는 응답 코드 |
| **4xx**  | 요청에 결함이 있으므로 처리를 중단함을 알리는 응답 코드      |
| **5xx**  | 요청에 대해 서버가 제대로 처리하지 못함을 알리는 응답 코드   |





#### 클라이언트와 서버 간의 무상태 통신

서버는 클라이언트와 요청한 데이터만으로 처리를 한다. 이 말은 애플리케이션 서버가 HTTP 세션과 같은 공유 메모리를 사용하지 않고 **요청 데이터만으로 리소스를 조작**하는 것을 의미한다. **무상태(stateless)** 통신을 구현할 때는 애플리케이션의 상태(화면 등의 상태)를 클라이언트 측의 애플리케이션(DOM 및 자바스크립트 변수 등)에서 관리해야 한다.





#### 연관된 리소스에 대한 링크

리소스에는 관련된 다른 리소스나 서브 리소스에 대한 **하이퍼미디어 링크(URI)**를 포함할 수 있다. 이것은 관련된 리소스끼리 서로 링크를 가짐으로써 링크만 따라가면 **연관된 모든 리소스에 접근**할 수 있게 만드는 것을 의미한다.



한편 리소스에 하이퍼미디어 링크(URI)를 만들고 그 링크를 따라가는 방식으로 다른 리소스에 접근하는 아키텍처를 **HATEOAS(Hypermedia as the Engine Of Application State)**라 한다. **HATEOAS** 아키텍처를 이용하면 클라이언트가 리소스에 접근할 때 필요한 URI를 미리 알아야 할 필요가 없으므로 클라이언트와 서버 간의 결합도를 낮출 수 있는 효과가 있다.



- **하이퍼미디어 링크(URI)를 포함한 리소스의 예**

```json
{ 
    "login": "spring-projects",
    "id": 317776,
    "_links": {
        "self": {
            "href": "https://localhost:8080/users/spring-projects"
        },
        "users": {
            "href": "https://localhost:8080/users"
        }
    }
}
```





### 6.1.2 프레임워크의 아키텍처

**REST API**는 **스프링 MVC**를 활용해서 구현한다. **스프링 MVC의 아키텍처**는 앞서 4.3절 '스프링 MVC 아키텍처'에서 다뤘는데, **REST API**를 사용할 때는 프레임워크 내부에서 다음과 같은 형태로 처리된다.



![](https://docs.google.com/drawings/d/swCuWE-9XItLGPY2rGf7MXA/image?parent=e/2PACX-1vSgWs9-SkKDVDplyC96i2vj5TuZvbmLHvACG_NMQ3eEa1hFtJwCGbFZVRaHriS3HOeS8O0XIQ6gqqII&rev=725&h=420&w=601&ac=1)



**(1)**: **DispatcherServlet** 클래스는 클라이언트에서 요청을 받는다.

**(2)**: **DispatcherServlet** 클래스는 **HandlerMapping** 인터페이스의 **getHandler** 메서드를 호출하고 요청을 처리할 **핸들러 객체(REST API용 컨트롤러)**를 가져온다.

**(3)**: **DispatcherServlet** 클래스는 **HandlerAdapter** 인터페이스의 **handle** 메서드를 호출해서 핸들러 객체의 메서드 호출을 의뢰한다.

**(4)**: **HandlerAdapter** 인터페이스의 구현 클래스는 **HttpMessageConverter** 메서드를 호출하고 요청 본문 데이터를 **리소스 클래스 객체로 변환한다.**

**(5)**: **HandlerAdapter** 인터페이스의 구현 클래스는 핸들러 객체에 구현돼 있는 메서드를 호출해서 요청을 처리한다.

**(6)**: **HandlerAdapter** 인터페이스의 구현 클래스는 **HttpMessageConverter** 메서드를 호출하고 핸들러 객체에서 반환된 리소스 클래스 객체를 응답 본문에 기록한다.

**(7)**: **DispatcherServlet** 클래스는 클라이언트에 응답을 반환한다.





화면으로 응답하는 웹 애플리케이션과 **다른 점**은 다음 두 가지다. 

- **응답 본문을 생성하기 위한 뷰를 사용하지 않는다.**
- **'요청 본문 해석'과 '응답 본문 생성'은 HttpMessageConverter라는 컴포넌트에서 처리해준다.**





#### HttpMessageConverter 활용

스프링 MVC는 스프링 웹에서 제공되는 **org.springframework.http.converter.HttpMessageConverter**를 사용해 요청 본문을 자바 객체로 변환하고 자바 객체를 응답 본문으로 변환한다. 특히 이 장의 마지막에 소개할 **REST 클라이언트**(**RestTemplate** 클래스)는 **HttpMessageConverter**를 사용해 자바 객체를 요청 본문으로 변환하고 응답 본문을 자바 객체로 변환할 때 사용된다.



**REST API**와 **REST API 클라이언트** 모두를 스프링으로 개발한다면 다음과 같은 구조가 된다.



![](https://docs.google.com/drawings/d/sGqQuOrLweK1KOiT7nC2Rww/image?parent=e/2PACX-1vSgWs9-SkKDVDplyC96i2vj5TuZvbmLHvACG_NMQ3eEa1hFtJwCGbFZVRaHriS3HOeS8O0XIQ6gqqII&rev=299&h=251&w=601&ac=1)



스프링은 다양한 **HttpMessageConverter**의 구현 클래스를 제공한다. 만약 주고받을 데이터가 일반적인 리소스 형식(**JSON**이나 **XML** 등)이면 스프링이 제공하는 구현 클래스만 사용해도 충분히 변환할 수 있다.



**의존 라이브러리가 필요없는 주요 HttpMessageConverter 구현 클래스**

| 클래스명                                    | 설명                                                         |
| ------------------------------------------- | ------------------------------------------------------------ |
| **ByteArrayHttpMessageConverter**           | '본문(임의의 미디어 타입) ↔ 바이트 배열' 변환용 클래스       |
| **StringHttpMessageConverter**              | '본문(텍스트 형식의 미디어 타입) ↔ String' 변환용 클래스     |
| **ResourceHttpMessageConverter**            | '본문(임의의 미디어 타입) ↔ org.springframework.core.io.Resource 구현 클래스' 변환용 클래스 |
| **AllEncompassingFormHttpMessageConverter** | '본문(폼 형식 또는 멀티파트 형식의 미디어 타입) ↔ org.springframework.util.MultiValueMap' 변환용 클래스. 멀티파트 형식을 사용할 때는 MultiValueMap에서 본문으로의 변환은 할 수 있으나, 그 반대인 본문에서 MultiValueMap으로의 변환은 지원하지 않는다. |
| **MappingJackson2HttpMessageConverter**     | FasterXML Jackson Databind를 이용한 '본문(JSON 형식의 미디어 타입) ↔ 임의의 자바빈즈' 변환용 클래스 |
| **GsonHttpMessageConverter**                | Google Gson을 이용한 '본문(JSON 형식의 미디어 타입) ↔ 임의의 자바 빈즈' 변환용 클래스 |
| **MappingJackson2XmlHttpMessageConverter**  | FasterXML Jackson XML Databind를 이용한 '본문(XML 형식의 미디어 타입) ↔ 임의의 자바빈즈' 변환용 클래스 |
| **Jaxb2RootElementHttpMessageConverter**    | 자바 표준의 JAXB2를 사용한 '본문(XML 형식의 미디어 타입) ↔ 임의의 자바빈즈' 변환용 클래스 |



어떤 **HttpMessageConverter**가 사용되는지는 본문 형식(미디어 타입)과 변환 대상이 되는 자바 클래스의 종류에 따라 달라질 수 있다.





#### 리소스 클래스

이 책에서는 리소스를 표현하는 자바 클래스를 '**리소스 클래스**'라고 한다. **Entity**와 같은 도메인 객체를 **리소스 클래스**로 쓰기도 하지만, 이 책에서는 **도메인 객체**와는 별개로 다른 클래스를 만들어서 쓴다는 전제로 설명한다.



예를 들어, 다음과 같은 **JSON **형식의 리소스를 다루는 경우를 생각해 보자.



- **JSON 형식 리소스의 예**

```json
{
	"login": "spring-projects",
    "id": 317776,
    "name": "Spring",
    "blog": "http://spring.io/projects"
}
```



이 경우에는 다음과 같은 **리소스 클래스**를 만들면 된다.



- **리소스 클래스의 구현 예**

```java
import java.io.Serializable;

public class UserResource implements Serializable {
    
    private static final long serialVersionUID = -3106927618180228823L;
    private String login;
    private Integer id;
    private String name;
    private String blog;
    // 생략
}
```



## 6.2 애플리케이션 설정

이번 절에서는 본격적으로 **REST API**를 개발하는 데 필요한 설정에 대해 설명한다. 



### 6.2.1 라이브러리 설정

### 6.2.2 서블릿 컨테이너 설정

### 6.2.3 프런트 컨트롤러 설정



## 6.3 @RestController 구현

여기서는 **REST API**를 개발하기 위한 구체적인 구현 방법을 설명한다.



**REST API**를 개발할 때 만들어야 하는 주요 컴포넌트는 **컨트롤러 클래스**와 **리소스 클래스** 두 가지다. 이번 절에서는 먼저 **컨트롤러 클래스**를 만드는 방법에 대해 알아보자.



### 6.3.1 컨트롤러에서 구현할 처리의 전체 구조

구체적인 작성 방법을 설명하기 전에 컨트롤러 클래스에서 구현할 주요 처리의 전체 구조를 파악해 두자. 컨트롤러 클래스에서 구현하는 처리는 크게 다음의 두 가지로 분류할 수 있다.

- **메서드 시그니처를 참조해서 프런트 컨트롤러가 처리하는 '선언형' 처리**
- **컨트롤러 클래스의 메서드 안에서 처리를 구현하는 '프로그래밍형' 처리**



컨트롤러에서 구현하는 주요 처리는 다음과 같다.

- 선언형
  - 요청 매핑
  - 요청 데이터(리소스) 취득
  - 입력값 검사 수행
- 프로그래밍형
  - 비즈니스 로직 호출
  - 응답 데이터(리소스) 반환



기본적인 내용은 '화면으로 응답하는 웹 애플리케이션'을 개발할 때 구현하는 컨트롤러와 비슷하지만 다음의 두 가지가 다르다.

- **요청 데이터와 응답 데이터는 HttpMessageConverter를 통해 가져오고, 반환한다.**
- **입력값 검사에 대한 오류 처리는 예외 핸들러에서 공통으로 수행한다.**



이런 차이점을 실제 소스코드를 통해 확인할 수 있도록 '화면으로 응답하는 웹 애플리케이션'의 컨트롤러와 비교해 보자.



**선언형 처리**

```java
@RequestMapping(path="/messages", method=RequestMethod.POST)
@ResponseBody
public Message post(@Validated @RequestBody Message newResource) {
    // 생략
}
```



**프로그래밍형 처리**

```java
@ResponseBody
public Message post(@Validated @RequestBody Message newResource) {
	// 생략
    Message createdResource = service.create(newMessage);
    return createdResource;
}
```



구체적으로는 다음과 같은 차이가 있다.

- 요청 데이터를 요청 본문 형태로 받아낼 인수에 **@RequestBody**를 붙인다.
- 응답할 데이터를 반환하되, 그 결과가 응답 본문 형태가 되도록 메서드에 **@RequestBody**를 붙인다. (**@Controller** 대신 **@RestController**를 사용하면 생략할 수 있다.)
- 입력값 검사 결과는 **BindingResult**로 받아 처리하는 대신 예외로 받아 처리한다.



> REST API에서 오류가 발생할 때는 **오류 통지 전용 메시지**(JSON 등)로 응답하는 것이 일반적인데, **입력값 검사 오류**도 예외는 아니다. **입력값 검사 오류**를 처리하는 방법에는 '예외 핸들러를 이용해 공통으로 처리하는 방법'과 'BindingResult를 이용해 개별적으로 처리하는 방법'이 있는데, **오류 통지 전용 메시지**로 하려면 첫 번째 '예외 핸들러를 이용해 공통으로 처리하는 방법'을 권장한다. 이 책에서 소개하는 **REST API**의 구현 방법은 '예외 핸들러를 이용해 공통으로 처리하는 방법'을 전제로 한다. 





### 6.3.2 컨트롤러 클래스 작성

컨트롤러에서 구현할 처리 내용을 전체적인 관점에서 파악했다면 이제는 실제로 컨트롤러를 구현하는 방법을 살펴보자.



기본적인 부분은 '화면으로 응답하는 웹 애플리케이션'을 개발할 때 만드는 컨트롤러와 비슷하지만 **@Controller** 대신 **@org.springframework.web.bind.annotation.RestController**를 사용한다.



- **컨트롤러 클래스의 구현 예**

```java
// 생략
@RestController  
// POJO로 클래스를 만들고 @RestController를 붙인다. 
// @RestController는 @Controller와 @ResponseBody를 합친 애너테이션이다.
@RequestMapping("books")
// 컨트롤러 클래스가 대응할 리소스 경로 정보(URI 경로 부분)를 @RequestMapping에 지정한다. 
// 이 컨트롤러 클래스에서 구현될 핸들러 메서드(REST API)는 http://localhost:8080/{contextPath}/books라는 URI에 반응하게 된다.
public class BooksRestController {
}
```





### 6.3.3 REST API(핸들러 메서드) 작성

지금부터 **REST API**를 만드는 방법을 소개하겠다. **@RestController**를 붙여준 컨트롤러 클래스에 **REST API**용 메서드(핸들러 메서드)를 만들어준다. 여기서는 '**도서 정보**'를 다루는 **REST API**를 스프링 MVC를 사용해서 개발하는 방법을 살펴볼 것이다.



#### 리소스 클래스 작성

**REST API**를 작성하기 전에 **리소스 클래스**(REST API에서 다룰 리소스를 표현하는 자바 클래스)를 작성한다. **리소스 클래스**를 만드는 방법에 대해서는 6.4절에서 자세히 설명한다.



다음은 **JSON**을 사용해서 도서 정보를 표현한 것이다. 실제로는 더 많은 정보를 담아야 하지만 설명을 간단히 하기 위해 다음 세 가지 정보만 담았다.



- **도서 정보를 표현하는 리소스 형식**

```json
{
	"bookId": "9791158390757",
	"name": "슬랙으로 협업하기",
	"publishedDate": "2017-08-10"
}
```



- **도서 정보를 표현하는 리소스 클래스 구현 예**

```java
public class BookResource implements Serializable {
    
    private static final long serialVerisonUID = 5535788271499457190L;
    private String bookId;
    private String name;
    private java.time.LocalDate publishedDate;
    // 생략
}
```



여기서 중요한 것은 **JSON 필드명**과 **자바빈즈 프로퍼티명**을 똑같이 맞추는 것이다. 참고로 프로퍼티 타입에는 **String** 외의 타입도 사용할 수 있다. 이번 절에서는 **출판일(publishedDate)**을 Java SE 8부터 추가된 **JSR 310:Date and Time API**의 날짜/시간 타입(**java.time.LocalDate**)을 사용했다. **Date and Time API**를 사용하려면 의존 라이브러리에 **jackson-datatype-jsr310**을 추가해야 한다.



- **pom.xml 설정 예**

```xml
<dependency>
	<groupId>com.fasterxml.jackson.datatype</groupId>
    <artifactId>jackson-datatype-jsr310</artifactId>
</dependency>
```





#### Book 리소스 취득

다음은 특정 **Book**의 리소스 정보를 **REST API**를 통해 가져오는 예다. 이 예에서는 특정 **Book** 리소스를 가리킬 때 **URI 형식(URI 템플릿)**으로 `http://localhost:8080/{contextPath}/books/{bookId}`를 사용한다. 이때 `{bookId}` 부분을 '**경로 변수**'라고 하며, **REST API** 상에서 특정 **Book** 리소스를 지정할 수 있는 **고유한 키 값(도서ID)**이 들어가는 자리다.



- **리소스 취득용 REST API의 구현 예**

```java
@Autowired
BookService bookService;

@RequestMapping(path="{bookId}", method=RequestMethod.GET) 
// @RequestMapping을 사용해 요청을 매핑한다. 이 예에서는 도서 ID를 받기 위한 경로 변수 {bookId}를 path 속성에 지정했다.
// 참고로 @RestController를 사용하기 때문에 @ResponseBody는 생략됐다.
// 리소스를 가져오는 REST API의 HTTP 메서드는 GET을 사용한다.
public BookResource getBook(@PathVariable String bookId) {  
// @PathVariable을 사용해 경로 변수 {bookId}에서 도서 ID를 가져온다.
    
   
    Book book = bookService.find(bookId);
    // 비즈니스 로직을 호출한다. 여기서는 경로 변수를 통해 받아온 도서 ID로 도서 정보(Book)를 요청한다.
    BookResource resource = new BookResource();
    // 비즈니스 로직을 호출해서 가져온 도서 정보를 Book 리소스로 변환한다.
    resource.setBookId(book.getBookId());
    resource.setName(book.getName());
    resource.setPublishedDate(book.getPublishedDate());
    return resource;
    // Book 리소스를 반환하고 정상적으로 처리됐다는 의미로 HTTP 상태 코드 '200 OK'를 응답한다.   
}
```



실제로 사용 가능한 애플리케이션을 만들 때는 도서 정보를 관리할 테이블을 만들어서 CRUD 작업을 해야겠지만, 여기서는 편의상 **인메모리(In-memory)**로 구현한 서비스 클래스를 만들어서 동작을 확인한다. 이때 도서 정보를 표현하는 **Book** 클래스의 구조는 **BookResource** 클래스와 같다.



- **BookService 서비스 클래스를 인메모리 방식으로 구현한 예**

```java
// 생략

@Service
public class BookService {
    private final Map<String, Book> bookRepository = new ConcurrentHashMap<>();
    
    @PostConstruct
    public void loadDummyData() {
        Book book = new Book();
        book.setBookId("9791158390747");
        book.setName("슬랙으로 협업하기");
        book.setPublishedDate(LocalDate.of(2017, 8, 10));
        bookRepository.put(book.getBookId(), book);
    }
    
    public Book find(String bookId) {
        Book book = bookRepository.get(bookId); // Map에서 취득
        return book;
    }
}
```



애플리케이션 서버를 기동하고 `http://localhost:8080/books/9791158390747`과 같이 GET 메서드로 접근하면 다음과 같은 **JSON** 형식의 응답이 반환된다.



- **curl을 이용해 API를 호출하는 방법과 응답 예**

```
$ curl http://localhost:8080/books/9791158390747
{"bookId":"9791158390747", "name":"슬랙으로 협업하기", "publishedDate": [2017, 8, 10]}
```



**JSON**은 반환됐지만 **publishedDate** 값이 원하던 형태가 아니다. **publishedDate**를 원하는 형태(ISO 8601의 확장 형식)로 받으려면 **포맷을 지정**해야 한다. 포맷을 지정하는 방법은 몇 가지 있지만 여기서는 **Jackson**에서 제공하는 **@com.fasterxml.jackson.annotation.JsonFormat**을 사용해 개별적으로 포맷을 지정한다. 애플리케이션 전체의 포맷을 지정하는 방법은 뒤에 나올 6.4절 '리소스 클래스 구현'에서 소개한다.



- **@JsonFormat을 이용한 형식 지정 예**

```java
public class BookResource implements Serializable {
	// 생략
    @JsonFormat(pattern="yyyy-MM-dd") // ISO 8061 확장 형식(yyyy-MM-dd) 지정을 추가
    private LocalDate publishedDate;
    // 생략
}
```



애플리케이션 서버를 재기동하고 다시 **REST API**를 호출하면 다음과 같은 형태로 JSON이 반환된다.



- 포맷을 지정한 후의 응답 예

```json
{
	"bookId": "9791158390747",
	"name": "슬랙으로 협업하기",
	"publishedDate": "2017-08-10"
}
```



#### 리소스 생성

새로운 **Book 리소스**를 추가하는 **REST API**는 다음과 같이 구현한다.



- **리소스 작성용 REST API의 구현 예**

```java
@RequestMapping(method=RequestMethod.POST)
// @RequestMapping을 사용해 요청을 매핑한다. 리소스를 생성하는 REST API의 HTTP 메서드에는 POST를 지정한다.
public ResponseEntity<Void> createBook(
	@Validated @RequestBody BookResource newResource) {
// 리소스 클래스 타입의 인수에 @RequestBody를 붙여 요청 본문의 데이터(JSON)를 받아내고, 
// @Validated를 붙여 리소스 객체에 대한 입력값 검사를 한다.
    
    Book newBook = new Book();
    newBook.setName(newResource.getName());
    newBook.setPublishedDate(newResource.getPublishedDate());
    
    Book createBook = bookService.create(newBook);
    
    String resourceUri = "http://localhost:8080/books/" + createdBook.getBookId();
    // 작성한 도서 정보에 접근하기 위한 URI를 생성한다. 여기서 생성한 URI는 Location 헤더에 설정된다.
    
    return ResponseEntity.created(URI.create(resourceUri)).build();
    // Location 헤더를 설정하고 리소스가 정상적으로 만들어졌다는 의미로 HTTP 상태 코드인 '201 Created'를 응답한다.
    // 응답 헤더를 설정해야 할 때는 ResponseEntity를 반환하면 된다. ResponseEntity의 created 메서드를 사용하면
    // 인수에 지정한 URI가 Location 헤더에 들어가고, '201 Created'가 HTTP 상태 코드에 설정된다.
    // 만약 응답 본문이 필요없을 때는 BodyBuilder의 build 메서드를 호출해서 ResponseEntity 객체를 생성한다.
}
```



> 이 예에서는 **Location 헤더**에 설정할 **URI 정보**에 `http://localhost:8080`을 하드코딩하고 있다. 이 정보는 환경에 따라 달라지는 값이기 때문에 개발자의 로컬 환경에서만 동작하는 애플리케이션이 될 수 있다. 이렇게 특정 환경에 따라 달라지는 환경 의존적인 값을 처리할 때는 **@Value**를 활용해 프로퍼티의 URI 정보를 읽어오게 할 수 있다. 한편 스프링 프레임워크는 이보다 더 지능적인 방법을 제공하는데, 구체적인 방법은 뒤에 나올 6.3.5절 'URI 조립'에서 자세히 살펴보자.





REST API의 동작을 확인하기 위해 앞서 인메모리로 구현한 서비스 클래스에 다음 메서드를 추가하자.



-  **BookService 서비스 클래스에 create 메서드를 추가한 예**

```java
public Book create(Book book) {
	String bookId = UUID.randomUUID().toString();
	book.setBookId(bookId);
	bookRepository.put(bookId, book); // Map에 추가
	return book;
}
```



애플리케이션 서버를 재기동하고 `http://localhost:8080/books`의 **POST** 메서드를 실행해 보자. 이때 요청 콘텐츠 형식에는 **application/json**을 지정하고, 요청 본문에는 다음과 같은 JSON 정보를 지정한다.



- **요청 본문에 설정하는 JSON**

```json
{"name": "러닝! Angular 4", "publishedDate": "2017-11-23"}
```



리소스 생성에 성공하면 다음과 같은 HTTP 응답이 반환된다. HTTP 응답 코드에는 '201 Created'가 설정되고 Location 헤더에는 생성한 Book 리소스에 접근하기 위한 URI가 설정된 것을 확인할 수 있다.



- **curl을 이용해 API를 호출하는 방법과 응답 예**

```
$ curl -D - -H "Content-type: application/json" -X POST -d '{"name":"러닝! Angular 4", "publishedDate": "2017-11-23"}' http://localhost:8080/books


HTTP/1.1 201 Created
Location: http://localhost:8080/books/c1c3da32-16e9-4288-9dc9-4866f2e4407a
...
```



이후에 Location 헤더에 설정됐던 URI에 GET 메서드를 호출하면 앞서 생성된 **Book 리소스**를 가져올 수 있다.



- **응답 예**

```json
{
    "bookId": "c1c3da32-16e9-4288-9dc9-4866f2e4407a",
	"name": "러닝! Angular 4", 
	"publishedDate": "2017-11-23"
}
```





#### 리소스 갱신

특정 **Book 리소스를 갱신**하는 **REST API**는 다음과 같이 구현한다.



- **리소스 갱신용 REST API의 구현 예**

```java
@RequestMapping(path="{bookId}", method=RequestMethod.PUT)
// @RequestMapping을 사용해 요청을 매핑한다. 리소스를 갱신하는 REST API의 HTTP 메서드에는 PUT을 지정한다.
@ResponseStatus(HttpStatus.NO_CONTENT)
// 응답할 HTTP 상태 코드를 지정한다. 메서드에 @ResponseStatus를 붙여주면 임의의 HTTP 상태 코드를 응답할 수 있다.
// 이 예에서는 서버에서 클라이언트로 반환할 콘텐츠가 없다는 의미로(본문이 공백) '204 No Content'를 응답하고 있다.
// 만약 갱신된 콘텐츠를 반환하고 싶다면 갱신된 Book 리소스를 반환하며 '200 OK'로 응답하면 된다.
public void put(@PathVariable String bookId, 
               @Validated @RequestBody BookResource resource) {
    
    Book book = new Book();
    book.setBookId(bookId);
    book.setName(resource.getName());
    book.setPublishedDate(resource.getPublishedDate());
    
    bookService.update(book);
    // 비즈니스 로직을 호출하고 도서 정보를 갱신한다.
}
```



작성한 **REST API**의 동작을 확인하기 위해 앞서 만들어둔 **BookService** 서비스 클래스에 다음과 같은 메서드를 추가하자.



- **BookService 서비스 클래스에 update 메서드를 추가한 예**

```java
public Book update(Book book) {
	return bookRepository.put(book.getBookId(), book);  // Map을 갱신;
}
```



애플리케이션 서버를 재기동하고 갱신하고 싶은 Book 리소스의 URI에 대해 PUT 메서드를 호출해보자. 이때 요청 콘텐츠 형식에는 **application/json**을 지정하고 요청 본문에는 다음과 같은 **JSON** 정보를 지정한다. 이 예에서는 도서명에 '(실무 예제로 배우는 앵귤러 4 핵심 가이드)' 문구를 추가하고 출판일을 '2017-11-23'에서 '2017-11-24'로 변경했다.



- **요청 본문에 설정하는 JSON**

```json
{
	"bookId": "c1c3da32-16e9-4288-9dc9-4866f2e4407a",
	"name": "러닝! Angular 4(실무 예제로 배우는 앵귤러 4 핵심 가이드)",
	"publishedDate": "2017-11-24"
}
```



리소스 갱신이 성공하면 다음과 같은 HTTP 응답이 반환된다. HTTP 응답 코드에 '204 No Content'가 설정된 것을 확인할 수 있다.



- **curl을 이용해 API를 호출하는 방법과 응답 예**

```
curl -D - -H "Content-type: application/json" -X PUT -d '{"bookId": "c1c3da32-16e9-4288-9dc9-4866f2e4407a", "name": "러닝! Angular 4(실무 예제로 배우는 앵귤러 4 핵심 가이드)", "publishedDate": "2017-11-24"}' http://localhost:8080/books/c1c3da32-16e9-4288-9dc9-4866f2e4407a


HTTP/1.1 204 No Content
```



이것만으로는 실제로 갱신됐는지 알 수 없으므로 갱신된 데이터를 가져와서 확인해보자.



- **응답 예**

```json
{
	"bookId": "c1c3da32-16e9-4288-9dc9-4866f2e4407a",
	"name": "러닝! Angular 4",
	"publishedDate": "2017-11-24"
}
```



응답을 보면 도서명과 출판일이 갱신된 것을 확인할 수 있다.





#### 리소스 삭제

특정 **Book** 리소스를 삭제하는 **REST API**는 다음과 같이 구현한다.



- **리소스 삭제용 REST API 구현 예**

```java
@RequestMapping(path="{bookId}", method=RequestMethod.DELETE)
// @RequestMapping을 사용해 요청을 매핑한다. 리소스를 삭제하는 REST API의 HTTP 메서드에는 DELETE를 지정한다.
@ResponseStatus(HttpStatus.NO_CONTENT)
// 응답할 HTTP 상태 코드를 지정한다. 리소스를 삭제한 이후에는 응답할 리소스가 없어지기 때문에 '204 No Content'로 응답하는 것이 일반적이다. 만약 삭제된 콘텐츠를 반환하고 싶다면 삭제할 Book 리소스를 반환하며 '200 OK'로 응답하면 된다.
public void delete(@PathVariable String bookId) {
	bookService.delete(bookId);
    // 비즈니스 로직을 호출하고 도서 정보를 삭제한다.
}
```



작성한 **REST API**의 동작을 확인하기 위해 앞서 만들어둔 **BookService** 서비스 클래스에 다음 메서드를 작성하자.



- **BookService 서비스 클래스에 delete 메서드를 추가한 예**

```java
public Book delete(String bookId) {
	return bookRepository.remove(bookId);   // Map에서 삭제
}
```



애플리케이션 서버를 재기동하고 삭제하려는 **Book** 리소스의 **URI**에 대해 **DELETE** 메서드를 호출해보자. 리소스 삭제가 성공하면 다음과 같은 HTTP 응답이 반환된다. HTTP 상태 코드에 '204 No Content'가 설정된 것을 확인할 수 있다.



- **curl을 이용해 API를 호출하는 방법과 응답 예**

```
$ curl -D - -X DELETE http://localhost:8080/books/c1c3da32-16e9-4288-9dc9-4866f2e4407a


HTTP/1.1 204 No Content
...
```



이것만으로는 실제로 삭제됐는지 알 수 없으므로 삭제된 데이터를 가져와서 확인해보자. 현재 구현대로라면 삭제된 리소스를 가져올 때 **NullPointerException**이 발생하고 시스템 오류가 발생하기 때문에 '404 Not Found'를 응답하도록 코드를 수정해야 한다.



- **'404 Not Found'**를 응답하도록 예외 클래스를 구현한 예

```java
@ResponseStatus(HttpStatus.NOT_FOUND)
// @ResponseStatus는 예외 클래스에도 붙여줄 수 있다. 
// 해당 예외가 발생했을 때 반환하고 싶은 HTTP 상태 코드를 지정한다.
public class BookResourceNotFoundException extends RuntimeException {
	public BookResourceNotFoundException(String bookId) {
		super("Book is not found (bookId= " + bookId + ")");
	}
}
```





- **Book 리소스를 가져오는 REST API(getBook)의 변경 부분**

```java
Book book = bookService.find(bookId);
// 도서 정보가 없는 경우에는 예외를 던진다.
if(book == null) {
    throw new BookResourceNotFoundException(bookId);
}
```



애플리케이션 서버를 재기동한 후 '**리소스 생성 -> 리소스 삭제 -> 삭제하려는 리소스를 취득**'의 과정을 차례로 수행해보면 '404 Not Found'로 응답하는 것을 확인할 수 있다.



- **응답 헤더의 출력 예**

```
$ curl -D - http://localhost:8080/books/c1c3da32-16e9-4288-9dc9-4866f2e4407a


HTTP/1.1 404 Not Found
...
```



여기서 소개한 예외 처리 방법은 비교적 간단한 방식이었다. **REST API**의 본격적인 예외 처리 메커니즘에 대해서는 6.5절에서 자세히 설명한다.





#### 리소스 검색

이제까지는 리소스를 고유하게 식별할 수 있는 ID를 이용해 CRUD 조작을 할 수 있도록 API를 구현해봤다. 하지만 경우에 따라서는 **REST API**에서 ID가 아닌 다른 조건으로 리소스를 조작해야 할 수도 있다. 대표적인 예가 리소스를 검색하는 API다. 이때는 URI에 포함된 ID를 대신해서 검색 조건을 요청으로 전송하고 서버 측에서는 해당 조건을 받아서 처리해줘야 한다.



**Book** 리소스를 검색하는 **REST API**는 다음과 같이 구현한다. 먼저 **Book** 리소스의 검색 조건을 받기 위한 클래스를 만든다. 5장 '웹 애플리케이션 개발'에서 소개한 것처럼 검색 조건을 받는 방법에느 다음의 두 가지가 있다.

- **@RequestParam을 사용해 개별적으로 가져온다.**
- **폼 클래스와 같이 검색 조건을 담을 수 있는 자바빈즈를 만들어 요청 파라미터를 바인드한다.**



어떤 방법을 사용해도 상관은 없지만, 입력값 검사를 생각한다면 검색 조건을 담을 수 있는 클래스를 만드는 것이 나을 수 있다.



- **Book 리소스의 검색 조건을 담을 클래스의 구현 예**

```java
public class BookResourceQuery implements Serializable {
	private static final long serialVersionUID = -3946462769436598529L;
	private String name;
	@DateTimeFormat(iso=DateTimeFormat.ISO.DATE)  // ISO 날짜 형식을 지원하기 위해 @DateTimeFormat을 지정한다.
	private LocalDate publishedDate;
	// 생략
}
```



**Book** 리소스의 검색 조건을 담기 위한 클래스를 만들었다면, 이번에는 **REST API**를 구현한다.



- **Book 리소스를 검색하는 REST API 구현 예**

```java
@RequestMapping(method=RequestMethod.GET) 
// @RequestMapping을 사용해 요청을 매핑한다. 리소스를 검색할 REST API HTTP 메서드에는 GET을 지정한다.
public List<BookResource> searchBooks(@Validated BookResourceQuery query) {
// 메서드 파라미터에 Book 리소스의 검색 조건을 담을 수 있는 클래스를 지정한다.
// 검색 조건을 담을 클래스 인수에 @Validated를 붙여 입력값 검사를 하게 만든다.

	BookCriteria criteria = new BookCriteria();
    criteria.setName(query.getName());
    criteria.setPublishedDate(query.getPublishedDate());
    
    List<Book> books = bookService.findAllByCriteria(criteria);
    // 비즈니스 로직을 호출하고 검색 조건과 일치하는 도서 정보를 받아온다. 검색 조건을 비즈니스 로직에 전달하기 위해
    // POJO 형태인 BookCriteria에 요청 파라미터 정보를 옮겨 넣는다.
    
    return books.stream().map(book -> {
        BookResource resource = new BookResource();
        resource.setBookId(book.getBookId());
        resource.setName(book.getName());
        resource.setPublishedDate(book.getPublishedDate());
        return resource;
    }).collect(Collectors.toList());
    // 조건에 일치한 도서 정보 목록을 Book 리소스의 목록으로 변환한다.
    // Book 리소스의 목록을 반환하고 정상적으로 처리됐다는 의미로 HTTP 상태 코드 '200 OK'를 응답한다.
}
```



작성한 **REST API**의 동작을 확인하기 위해서 앞서 만들어둔 **BookService** 서비스 클래스에 검색 메서드를 추가한다. '이름'은 부분 일치하고 '출판일'은 완전히 일치하는 것을 추출한 다음, '출판일'을 기준으로 오름차순으로 정렬한다. 이때 도서 정보의 검색 조건을 담고 있는 **BookCriteria** 클래스는 **BookResourceQuery** 클래스와 같은 구조를 하고 있다.



- **BookService 서비스 클래스에 검색 메서드를  추가한 예**

```java
public List<Book> findAllByCriteria(BookCriteria criteria) {
	return bookRepository.values().stream().filter(book ->
			(criteria.getName() == null
            	|| book.getName().contains(criteria.getName())) && 
            (criteria.getPublishedDate() == null
            	|| book.getPublishedDate().equals(criteria.getPublishedDAte())))
        	.sorted((o1, o2) -> o1.getPublishedDate().compareTo(o2.getPublishedDate()))
        	.collect(Collectors.toList());
}
```



애플리케이션 서버를 재기동하고 **Book** 리소스를 검색해보자. 먼저 **URI**에 `http://localhost:8080/books`를 지정하고 검색 조건 없이 검색하면 모든 **Book** 리소스를 가져올 수 있다. 참고로 애플리케이션 서버를 재기동하고 나면 도서 정보가 한 건밖에 없으므로 리소스를 더 추가한 다음 검색해보자.



- **검색 조건 없이 검색한 경우의 응답 예**

```json
[
	{
		"bookId": "9791158390747",
        "name": "슬랙으로 협업하기",
        "publishedDate": "2017-08-10"
	},
    {
        "bookId": "c1c3da32-16e9-4288-9dc9-4866f2e4407a",
        "name": "러닝! Angular 4",
        "publishedDate": "2017-11-23"
    }
]
```



다음은 URI에 `http://localhost:8080/books?name=슬랙으로+협업하기`를 지정해서 **Book** 리소스를 검색해 보자.



- **검색 조건을 지정한 경우 응답 예**

```json
[
	{
		"bookId": "9791158390747",
        "name": "슬랙으로 협업하기",
        "publishedDate": "2017-08-10"
	}
]
```





마지막으로 URI에 `http://localhost:8080/books?publishedDate=1999-01-01`과 같은 실제 데이터에 없는 날짜로 검색한 후, 결과를 확인해보자.



- **검색 조건에 일치하는 리소스가 없는 경우의 응답 예**

```json
[]
```





### 6.3.4 CORS 지원

**CORS**는 **Cross-Origin Resource Sharing**의 약자로 웹 페이지에서 **AJAX(XMLHttpRequest)**를 사용할 때, 다른 도메인의 서버 리소스(**JSON** 등)에 접근하기 위한 메커니즘이다. 



여기서는 스프링 프레임워크 4.2 버전에서 추가된 **CORS** 기능에 대해 설명한다. 스프링이 제공하는 **CORS** 기능에서는 **CORS** 요청이 타당한지 확인하고, 필요에 따라 **CORS** 제어용 응답 헤더를 붙여준다.



**CORS**를 지원하는 리소스를 지정하는 방법은 다음 두 가지가 있다.

- **빈을 정의하는 방식으로 애플리케이션 단위로 설정한다.**
- **@org.springframework.web.bind.annotation.CrossOrgin을 사용해 컨트롤러와 핸들러 메서드 단위로 설정한다.**



> 스프링의 기본 구현에서는 CORS 요청이 부적절한 경우 HTTP 상태 코드를 'Forbidden 403'으로 설정해 오류를 응답한다. 그리고 스프링 웹에서 제공하는 org.springframework.web.filter.CorsFilter를 사용하면 스프링 MVC에서 관리하지 않는 리소스에 대해서도 CORS 기능을 적용할 수 있다.





#### 애플리케이션 단위로 CORS 설정

먼저 빈을 정의해서 애플리케이션 단위로 **CORS**를 설정하는 방법을 소개한다.



다음 예에서는 **/api** 이하의 리소스에 대해 **CORS**를 허용하고 있다. **CORS** 설정을 한 다음, **/api** 이하의 리소스에 대해 **AJAX**로 접근하면 응답으로 **CORS **제어용 응답 헤더가 설정되어 돌아온다.



- **리소스에 접근할 때 돌아오는 응답 헤더의 예**

```
Access-Control-Allow-Origin: http://example.com:8080
Access-Control-Allow-Credentials: true
Vary: Origin
```



- **자바 기반 설정 방식으로 빈을 정의한 예**

```java
@Configuration
@EnableWebMvc
public class WebMvcConfig extends WebMvcConfigurerAdapter {
    
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/api/**");
    }
}
```





(중략)





### 6.3.5 URI 조립

6.3.3절의 '리소스 생성'에서 조금 다뤘지만 스프링은 **URI**를 생성하는 컴포넌트(org.springframework.web.util.UriComponentsBuilder)를 제공한다. 그리고 스프링 MVC에서 핸들러 메서드의 정의 정보와 연동해서 **URI**를 생성하는 컴포넌트(org.springframework.web.servlet.mvc.method.annotation.MvcUriComponentsBuilder)도 제공한다. **UriComponentsBuilder**는 서블릿 환경과 스프링 MVC 구조에 의존하지 않는 범용적인 컴포넌트로 **MvcUriComponetsBuilder** 안에서도 이용된다.



#### UriComponentsBuilder를 이용한 URI 생성

먼저 **UriComponentsBuilder**를 이용해 **URI**를 생성하는 방법을 소개한다. **UriComponentsBuilder**를 이용하면 다음과 같은 두 가지 작업을 간단하게 할 수 있다.

- **프로토콜, 호스트명, 포트 번호, 컨텍스트 경로와 같이 환경에 의존하는 부분의 은폐**
- **URI 템플릿을 사용한 URI 조립**



6.3.3절 'REST API(핸들러 메서드) 작성'에서는 소스코드에 `http://localhost:8080/`과 같은 환경에 의존적인 값이 하드코딩돼 있었는데, 여기에 **UriComponentsBuilder**를 활용하면 환경에 의존적인 값을 소스코드에서 제거해서 환경 변화에 유연한 코드를 만들 수 있다.



- **UriComponentsBuilder를 이용한 URI 생성**

```java
@RequestMapping(method=RequestMethod.POST)
public ResponseEntity<Void> createBook(
		@Validated @RequestBody BookResource newResource,
		UriComponentsBuilder uriBuilder) {
    	// 핸들러 메서드 매개변수에 UriComponentsBuilder를 정의한다. 
    	// 서블릿 환경에서 사용하는 UriComponentsBuilder 객체가 인수에 설정된다.
    
    
    // 생략
    URI resourceURI = uriBuilder.path("books/{bookId}")  
        			  // path 메서드를 사용해 REST API를 호출하기 위한 URI 템플릿을 만든다. 
        			  // 여기서는 실행환경에 의존하는 값을 포함하지 않는 것이 중요하다.
        	.buildAndExpand(createdBook.getBookId())
        	// buildAndExpand 메서드를 사용해 URI 템플릿의 경로 변수인 '{bookId}'에 들어갈 값을 설정한다.
        	.encode()
        	// encode 메서드를 사용해 URI 인코딩을 한다.
        	.toUri();
    		// toUri 메서드를 사용해 URI를 생성한다.
    return ResponseEntity.created(resourceUri).build();
}
```





#### MvcUriComponentsBuilder를 이용한 URI 생성

다음으로 **MvcUriComponentsBuilder**를 이용해 **URI**를 생성하는 방법을 소개한다. **MvcUriComponentsBuilder**를 이용하면 핸들러 메서드의 정의 정보(요청 매핑이나 메서드 매개변수 정보)와 연동해서 **URI**를 조립할 수 있기 때문에 **UriComponentsBuilder**를 이용할 때보다 다음과 같은 점이 우수하다.

- **작성할 URI의 템플릿을 의식할 필요가 없다.**
- **타입에 안전하다.**





그럼 실제로 어떻게 구현되는지 살펴보자.



- **MvcUriComponentsBuilder를 이용한 URI 생성 예**

```java
@RequestMapping(path="{bookId}", method=RequestMethod.GET)
public BookResource getBook(@PathVariable String bookId) {
	// 생략
}

@RequestMapping(method=RequestMethod.POST)
public ResponseEntity<Void> createBook(
		@Validated @RequestBody BookResource newResource,
		UriComponentsBuilder uriBuilder) {
    // 생략
    
    URI resourceUri = MvcUriComponentsBuilder.relativeTo(uriBuilder)  
        			  // MvcUriComponentsBuilder의 relativeTo 메서드를 사용해 인수로 받은
        			  // UriComponentsBuilder를 '기본 URL'을 조립하기 위한 객체로 사용한다.
        	.withMethodCall(on(BooksRestController.class)
                           .getBook(createdBook.getBookId()))
   	    // on 메서드는 URI를 생성할 때 필요한 컨트롤러의 mock 객체를 만드는 MvcUriComponentsBuilder의 static 메서드다.         // on 메서드로 만들어진 목 객체의 @RequestMapping이 붙은 메서드를 호출하면, 그때 사용된 경로와 경로 변수가
        // 조합된 컨트롤러를 얻을 수 있다. 즉, 이 예에서는 목 객체의 getBook을 호출하면서 경로 변수 값을 인자로 받아
        // '/books/{bookId}'와 같은 경로를 만들어낼 수 있다.
        	.build().encode().toUri();
    
    
    return ResponseEntity.created(resourceUri).build();
}
```





## 6.4 리소스 클래스 구현

**리소스 클래스**는 **JSON**이나 **XML** 형식의 데이터를 자바빈즈로 표현한 클래스다. 스프링 MVC는 리소스 클래스를 통해 서버와 클라이언트 사이의 리소스 상태를 연계하는 역할을 한다. **Entity**와 같은 클래스를 리소스 클래스로 사용하는 방법도 있지만 이 책에서는 전용 클래스를 작성하는 것을 전제로 설명한다.



> 도메인 객체와는 다른 클래스를 만드는 것을 전제로 하는 이유는 클라이언트와의 입출력에서 다루는 리소스 정보와 업무 처리에서 다루는 도메인 객체의 정보가 반드시 일치하는 것은 아니기 때문이다. 리소스 클래스와 도메인 객체 클래스를 미리 분리해 두면 REST API에서 다루는 리소스 정보가 변경되거나 업무 처리에서 다루는 도메인 객체 정보가 변경되더라도 그로 인한 영향 범위를 최소화할 수 있다. 클래스를 분리하면 객체를 변환하는 처리 과정이 필요하지만 빈 변환용 오픈소스 라이브러리를 사용하면 간단히 변환할 수 있다. 리소스 클래스를 따로 만들지에 대한 여부는 상황에 따라 달라질 수 있으므로 애플리케이션의 특성을 고려해서 결정한다.





![](https://docs.google.com/drawings/d/sL8QJGUH7kVBf8lq9HedGAw/image?parent=e/2PACX-1vSgWs9-SkKDVDplyC96i2vj5TuZvbmLHvACG_NMQ3eEa1hFtJwCGbFZVRaHriS3HOeS8O0XIQ6gqqII&rev=394&h=309&w=601&ac=1)



**JSON**은 자바 클래스보다 다양한 타입의 필드를 표현하지는 못하지만, 문자열이나 숫자, 불린값(true 또는 false)의 타입을 다룰 수 있고, 리스트나 중첩된 객체와 같은 복잡한 구조도 표현할 수 있다.





### 6.4.1 리소스 클래스 작성

**리소스 클래스**는 **자바빈즈 클래스**로 작성한다.



여기서는 이번 장 초반부터 사용한 도서 정보를 예로 들어 리소스 클래스의 작성 방법을 설명한다.



- **도서 정보 리소스(JSON) 예**

```json
{
	"bookId": "9791158390747",
    "name": "슬랙으로 협업하기",
    "authors": ["노승헌"],
    "publishedDate": "2017-08-10",
    "publisher": {
        "name": "위키북스",
        "tel": "031-955-3658"
    }
}
```



이와 같은 **JSON**을 사용하기 위해 다음과 같은 **자바빈즈 클래스**를 만든다.



- **도서 정보를 표현하는 리소스 클래스의 구현 예**

```java
import java.io.Serializable;
import java.time.LocalDate;
import java.util.List;

public class BookResource implements Serializable {
	private static final long serialVersionUID = 7070361791820300580L;
    private String bookId;  
    // JSON 필드명과 같은 이름으로 프로퍼티를 만든다. 만약 이름이 다를 때는 Jackson 애너테이션(@JsonProperty)에서
    // 이름을 매핑해야 한다.
    private String name;
    private List<String> authors;
    private LocalDate publishedDate;
    private BookPublisher publisher;
    // 생략
    
    public static class Publisher implements Serializable {
        private static final long serialVersionUID = 7303861536821292739L;
        private String name;
        private String tel;
        // 생략
    }
}
```



**BookResource** 클래스의 객체를 **REST API**에서 반환하면 다음과 같은 **JSON** 형식의 응답이 만들어진다. 다만 아쉽게도 **publishedDate**가 원하는 형태로 포맷되지 않았다. 여기엔 두 가지 원인이 있다.

- **Date and Time API 클래스를 지원하는 의존 라이브러리가 추가되지 않음**
- **포맷이 지정되지 않음**



- **실제로 반환되는 JSON**

```json
{
	"bookId": "9791158390747",
    "name": "슬랙으로 협업하기",
    "authors": ["노승헌"],
    "publishedDate": {
    	"year": 2017,
        "month": "AUGUST",
        "monthValue": 8,
        "dayOfMonth": 10,
        "dayOfWeek": "THURSDAY",
        "era": "CE",
        "dayOfYear": 222,
        "leapYear": false,
        "chronology": {
            "calendarType": "iso8601"
        }
    },
    "publisher": {
        "name": "위키북스",
        "tel": "031-955-3658"
    }
}
```





### 6.4.2 Jackson을 이용한 포맷 제어

여기서는 **Jackson**을 이용해 **JSON**의 포맷을 제어하는 방법을 몇 가지 소개한다.

- **JSON에 들여쓰기를 설정하는 방법**
- **언더스코어('_')로 구분되는 JSON 필드를 다루는 방법**
- **Java SE 8에서 추가된 Date and Time API 클래스를 지원하는 방법**
- **날짜/시간 타입의 포맷을 지정하는 방법**



이 책에서는 지면 관계상 모두 소개할 수 없지만 **Jackson**에는 포맷을 제어하기 위한 애너테이션(**@JsonProperty, @JsonIgnore, @JsonInclude, @JsonIgnoreProperties, @JsonPropertyOrder, @JsonSerialize, @JsonDeserialize** 등)과 직렬화 및 역직렬화 처리를 커스터마이징할 때 필요한 추상 클래스(**JsonSerializer, JsonDeserializer**) 등이 제공된다. 이러한 애너테이션이나 클래스를 이용하면 다양한 포맷에 대응할 수 있다.





#### 스프링이 제공하는 Jackson 지원 클래스

구체적인 포맷 제어 방법을 설명하기 전에 스프링이 제공하는 **Jackson** 지원 클래스를 소개한다.



**Jackson**은 **com.fasterxml.jackson.databind.ObjectMapper**라는 클래스를 사용해 **JSON**과 **자바 객체**를 서로 변환할 수 있다. **ObjectMapper**에는 기본 동작 방식을 커스터마이징하기 위한 다양한 옵션이 준비돼 있으며 스프링이 제공하는 지원 클래스를 이용하면 **ObjectMapper**를 직접 다루는 것보다 더 간단하게 옵션을 지정할 수 있다.



스프링이 제공하는 **헬퍼 클래스(helper class)**는 다음 두 가지다.

- **org.springframework.http.converter.json.Jackson2ObjectMapperBuilder**
- **org.springframework.http.converter.json.Jackson2ObjectMapperFactoryBean**



첫 번째는 **빌더 패턴**을 사용해 **ObjectMapper**를 만드는 클래스로서 **자바 기반 설정 방식**으로 빈을 정의할 때 이용한다. 두 번째는 스프링이 제공하는 **FactoryBean**을 사용해 **ObjectMapper**를 만드는 클래스로서 주로 **XML 기반 설정 방식**으로 빈을 정의할 때 사용한다.



- **Jackson2ObjectMapperBuilder의 이용 예**

```java
@Bean
ObjectMapper objectMapper() {
	return Jackson2ObjectMapperBuilder.json()
        	// 여기에 옵션 지정
        	.build();
}
```



- **Jackson2ObjectMapperFactoryBean의 이용 예**

```xml
<bean id="objectMapper"
      class="org.springframework.http.converter.json.Jackson2ObjectMapperFactoryBean">
	<!-- 여기에 옵션을 지정 -->
</bean>
```







#### JSON에 들여쓰기를 설정하는 방법

**ObjectMapper**의 기본 동작 방식은 **JSON**에 들여쓰거나 개행을 포함하지 않기 때문에 기본 설정으로 만들어진 **JSON** 정보는 읽기 어렵다는 단점이 있다. 생성되는 **JSON**의 크기가 조금 커져도 괜찮다면 **JSON**에 들여쓰기와 개행을 포함하도록 **Jackson** 설정을 변경하자.



- **자바 기반 설정 방식을 이용한 ObjectMapper 빈 정의 예**

```java
@Bean
ObjectMapper objectMapper() {
    return Jackson2ObjectMapperBuilder.json()
        	.indentOutput(true)
        	.build();
}
```





- **XML을 이용한 ObjectMapper 빈 정의 예**

```xml
<bean id="objectMapper"
      class="org.springframework.http.converter.json.Jackson2ObjectMapperFactoryBean">
	<property name="indentOutput" value="true" />
</bean>
```





#### Date and Time API 클래스를 지원하는 방법

Java SE 8에서 추가된 **Date and Time API** 클래스를 지원하려면 **Jackson**에서 제공하는 라이브러리(**jackson-datatype-jsr310**)를 추가해야 한다.



- **pom.xml 정의 예**

```xml
<dependency>
	<groupId>com.fasterxml.jackson.datatype</groupId>
    <artifactId>jackson-datatype-jsr310</artifactId>
</dependency>
```



**jackson-datatype-jsr310**을 의존 라이브러리에 추가하면 **Date and Time API** 클래스를 다룰 수 있다. 다만 포맷을 지정하지 않으면 예상대로 결과가 나오지 않을 수 있다.



- **jackson-datatype-jsr310 적용 후(포맷을 지정하지 않은 상태) 반환된 JSON**

```json
{
	"bookId": "9791158390747",
    "name": "슬랙으로 협업하기",
    "authors": ["노승헌"],
    "publishedDate": [2017, 8, 10],
    "publisher": {
        "name": "위키북스",
        "tel": "031-955-3658"
    }
}
```





#### 날짜/시간 타입의 포맷을 지정하는 방법

날짜/시간 타입의 포맷은 **ObjectMapper**에서 지정한다.



- **자바 기반 설정 방식을 이용한 ObjectMapper의 빈 정의 예**

```java
@Bean
ObjectMapper objectMapper() {
    return Jackson2ObjectMapperBuilder.json()
        .indentOutput(true)
        .dateFormat(new StdDateFormat())
        .build();
}
```





- **XML을 이용한 ObjectMapper의 빈 정의 예**

```xml
<bean id="objectMapper"
      class="org.springframework.http.converter.json.Jackson2ObjectMapperFactoryBean">
	<property name="indentOutput" value="true" />
    <property name="dateFormat">
    	<bean class="com.fasterxml.jackson.databind.util.StdDateFormat" />
    </property>
</bean>
```



- **StdDateFormat을 적용한 후에 반환되는 JSON**

```json
{
	"bookId": "9791158390747",
    "name": "슬랙으로 협업하기",
    "authors": ["노승헌"],
    "publishedDate": "2017-08-10",
    "publisher": {
        "name": "위키북스",
        "tel": "031-955-3658"
    }
}
```



**publishedDate** 값이 **ISO 8601**의 날짜/시간 형식(yyyy-MM-dd)으로 됐다. **StdDateFormat**을 적용하면 다음과 같은 포맷을 사용할 수 있다.

- **java.time.LocalDate**은 **yyyy-MM-dd** (예: 2017-05-10)
- **java.time.LocalDateTime**은 **yyyy-MM-dd 'T'HH:mm:ss.SSS** (예: 2017-05-10T08:00:00.000)
- **java.time.ZonedDateTime**은 **yyyy-MM-dd'T'HH:mm:ss.SSS'Z'** (예: 2017-05-10T08:00:00.000+09:00)
- **java.time.LocalTime**은 **HH:mm:ss.SSS** (예: 08:00:00.000)



> @com.fasterxml.jackson.annotation.JsonFormat을 이용하면 프로퍼티 단위로 포맷을 지정할 수도 있다.
>
> ```java
> @JsonFormat(pattern = "yyyy/MM/dd")
> private LocalDate publishedDate;
> ```



## 6.5 예외 처리

이번 절에서는 **REST API**에서 발생한 **예외**를 처리하는 방법을 설명한다. **예외 처리**를 하기 위한 기본적인 메커니즘에 대해서는 5장을 참조하자.



### 6.5.1 REST API 오류 응답

**REST API**에서 오류가 발생한 경우 **REST API**에서 다룰 수 있는 리소스 형식(JSON 등)으로 응답하는 것이 일반적이다. 예를 들어, 깃허브의 **REST API**에서는 다음과 같은 **JSON**이 반환된다.



- **오류 응답용 JSON 예**

```json
{
    "message": "Not Found",
    "documentation_url": "https://developer.github.com/v3"
}
```



이 절에서는 깃허브의 **REST API**와 같은 형식으로 응답하는 방법을 소개하면서 **REST API**에서 발생하는 예외를 어떻게 처리하는지 살펴볼 것이다.



예외 처리의 구현 방법을 설명하기 전에 오류 정보를 담을 **자바빈즈**를 만들어 두자.



- **오류 정보를 담을 자바빈즈의 구현 예**

```java
import com.fasterxml.jackson.annotation.JsonProperty;
import java.io.Serializable;

public class ApiError implements Serializable {
    private static final long serialVersionUID = -8172536311626933035L;
    private String message;
    
    @JsonProperty("documentation_url")
    private String documentationUrl;
    
    // 생략
}
```





### 6.5.2 스프링 MVC의 예외 핸들러 구현

**REST API**용 **예외 처리 클래스**를 구현하는 방법을 소개한다. 스프링 프레임워크에는 **REST API**만을 위해 따로 만들어진 예외 처리 메커니즘은 없지만 **REST API용 예외 처리 클래스**를 만들 때 도움이 되는 클래스(org.springframework.web.servlet.mvc.method.annotation.ResponseEntityExceptionHandler)를 제공한다.



#### 예외 핸들러 클래스 생성

먼저 스프링 MVC를 위한 예외 핸들러 클래스를 만든다.



- **예외 핸들러 클래스의 구현 예**

```java
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.servlet.mvc.method.annotation.ResponseEntityExceptionHandler;

@ControllerAdvice
public class ApiExceptionHandler extends ResponseEntityExceptionHandler {

}
// REST API용 예외 클래스를 작성한다. ResponseEntityExceptionHandler를 상속받고 클래스에 @ControllerAdvice를 붙인다.
```



**ResponseEntityExceptionHandler**에는 스프링 MVC에서 발생하는 예외를 처리하기 위한 **@ExceptionHandler** 메서드가 구현돼 있다. 그래서 위와 같이 핸들러를 만들면 프레임워크 내에서 발생하는 예외를 가로채서 처리할 수 있다. 다만 지금과 같이 단지 **ResponseEntityExceptionHandler** 클래스를 상속받기만 한 것이라면 오류가 발생하더라도 오류 내용이 표시되지 않는 텅빈 내용이 응답된다. 그래서 원하는 내용으로 오류가 표시되게 하려면 다음과 같은 추가 작업이 필요하다.



#### 오류 정보를 응답 본문에 출력하기 위한 구현

**ResponseEntityExceptionHandler**를 상속한 클래스를 그대로 사용하면 오류가 발생해도 응답 본문이 빈 상태가 된다. 응답 본문에 오류 정보를 표시하려면 **handleExceptionInternal** 메서드를 오버라이드해야 한다.



































