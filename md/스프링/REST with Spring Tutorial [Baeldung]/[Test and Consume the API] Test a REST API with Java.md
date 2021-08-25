# Test a REST API with Java :tiger:



> **[참고]**: Test a REST API with Java
>
> https://www.baeldung.com/integration-testing-a-rest-api
>
> 의역과 오역에 주의...





## 1. Overview

이번 포스팅은 **REST API에 대한 통합 테스트**를 작성하는 원칙과 메커니즘에 대해 살펴본다.



REST 자원을 테스트할 때, 

테스트가 신경써야 할 몇 가지 것들이 있다.

- **HTTP response code**
- **HTTP headers** in the response

- the **payload** (JSON, XML) 



각각의 테스트는 **<u>오직 하나의 책임</u>**에만 집중해서 작성되어야 하고, **<u>하나의 assertion</u>**만 포함되어야 한다.



일반적인 개발자들은 맨 처음부터 아주 복잡한 test scenario들을 작성하려고 하지만, clear하게separation 하는 것이 더 좋다.





## 2. Testing the Status Code



```java
@Test
public void givenUserDoesNotExists_whenUserInfoIsRetrieved_then404IsReceived()
  throws ClientProtocolException, IOException {
 
    // Given
    String name = RandomStringUtils.randomAlphabetic( 8 );
    HttpUriRequest request = new HttpGet( "https://api.github.com/users/" + name );

    // When
    HttpResponse httpResponse = HttpClientBuilder.create().build().execute( request );

    // Then
    assertThat(
      httpResponse.getStatusLine().getStatusCode(),
      equalTo(HttpStatus.SC_NOT_FOUND));
}
```





## 3. Testing the Media Type



```java
@Test
public void 
givenRequestWithNoAcceptHeader_whenRequestIsExecuted_thenDefaultResponseContentTypeIsJson()
  throws ClientProtocolException, IOException {
 
   // Given
   String jsonMimeType = "application/json";
   HttpUriRequest request = new HttpGet( "https://api.github.com/users/eugenp" );

   // When
   HttpResponse response = HttpClientBuilder.create().build().execute( request );

   // Then
   String mimeType = ContentType.getOrDefault(response.getEntity()).getMimeType();
   assertEquals( jsonMimeType, mimeType );
}
```



**Response**가 **JSON data**를 포함하고 있는지를 테스트하는 예시다.



당신도 이미 눈치챘듯이, 우리는 지금 테스트의 논리적 단계를 밟아나가고 있는 중이다.

첫번째로 먼저 request가 OK인지 **Response Status Code**를 확인했고,

그다음에 **Response의 Media Type**을 확인했다.

다음에 남은 것은 **JSON data**를 확인하는 것이다.







## 4. Testing the JSON Payload



```java
@Test
public void 
  givenUserExists_whenUserInformationIsRetrieved_thenRetrievedResourceIsCorrect()
  throws ClientProtocolException, IOException {
 
    // Given
    HttpUriRequest request = new HttpGet( "https://api.github.com/users/eugenp" );

    // When
    HttpResponse response = HttpClientBuilder.create().build().execute( request );

    // Then
    GitHubUser resource = RetrieveUtil.retrieveResourceFromResponse(
      response, GitHubUser.class);
    assertThat( "eugenp", Matchers.is( resource.getLogin() ) );
}
```



난 GitHub 자원의 default한 표현 포맷이 JSON인 것을 알지만, 보통의 경우에는

**응답**의 ***Content-Type* 헤더**가 

**요청**의 ***Accept* 헤더**와 함께 테스트되어야 한다.





## 5. Utilities for Testing

*Jackson 2* 라이브러리를 사용하면 

raw **JSON String**을 type-safe한 **Java Entity**로 변환할 수 있다.



```java
public class GitHubUser {

    private String login;

    // standard getters and setters
}
```





우리는 테스트를 clean하고 readable하고 높은 수준의 추상화 수준을 유지하기 위해서 단순한 utility만을 사용하겠다.

```java
public static <T> T retrieveResourceFromResponse(HttpResponse response, Class<T> clazz) 
  throws IOException {
 
    String jsonFromResponse = EntityUtils.toString(response.getEntity());
    ObjectMapper mapper = new ObjectMapper()
      .configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false);
    return mapper.readValue(jsonFromResponse, clazz);
}
```