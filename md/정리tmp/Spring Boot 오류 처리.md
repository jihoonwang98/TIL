# Spring Boot 오류 처리



> 출처: https://supawer0728.github.io/2019/04/04/spring-error-handling/



## ErrorController

Spring Boot에서 기본적으로 오류 처리를 어떻게 해주는지 살펴보자.

아래는 **404 Not Found**에 대한 html, json 응답 예제이다.



`GET /123` - html

![](https://supawer0728.github.io/images/spring-error-handling/notfound.png)



`GET /123` - json

```json
{
  "timestamp": "2019-02-15T21:48:44.447+0000",
  "status": 404,
  "error": "Not Found",
  "message": "No message available",
  "path": "/123"
}
```





위에서 살펴본 것과 같이 별다른 설정 없이 spring boot에서 웹 애플리케이션을 실행하면

기본적으로 오류 처리가 되고 있음을 알 수 있다.



그렇다면 어떠한 설정으로 spring boot에서 오류를 처리하는지 

먼저 **spring boot의 오류처리에 대한 properties**를 살펴보자.





### Spring Boot의 기본 오류 처리 properties

```properties
# spring boot의 기본 properties
server.error:
  include-exception: false
  include-stacktrace: never # 오류 응답에 stacktrace 내용을 포함할 지 여부
  path: '/error' # 오류 응답을 처리할 Handler의 경로
  whitelabel.enabled: true # 서버 오류 발생시 브라우저에 보여줄 기본 페이지 생성 여부
```



- `server.error.include-exception` : 응답에 exception의 내용을 포함할지 여부
- `server.error.include-stacktrace` : 응답에 stacktrace 내용을 포함할지 여부
- `server.error.path` : 오류 응답을 처리할 핸들러(ErrorController)의 path
- `server.error.whitelabel.enabled` : 브라우저 요청에 대해 서버 오류시 기본으로 노출할 페이지를 사용할지 여부



`server.error.whitelabel.enabled`의 기본값이 `true`이기 때문에 위에서와 같이 오류 페이지가 노출되고 있었다.

아래 스크린샷은 `include-exception`과 `include-stacktrace`를 활성화하면 아래와 같이 응답을 받을 수 있다.





**HTML 응답**

![](https://supawer0728.github.io/images/spring-error-handling/error-html.png)





**JSON 응답**

```json
{
  "timestamp": "2019-04-04T09:31:27.931+0000",
  "status": 500,
  "error": "Internal Server Error",
  "exception": "java.lang.IllegalStateException",
  "message": "test",
  "trace": "java.lang.IllegalStateException: test ...(길어서 줄임)",
  "path": "/rest-test"
}
```





## Spring Boot의 기본 오류 처리 - BasicErrorController

그렇다면 Spring Boot에서는 어떻게 이런 기본 처리를 하고 있는 것일까.

Spring Boot는 오류가 발생하면 `server.error.path`에 설정된 경로에서 요청을 처리하게 한다.

Spring Boot에서는 기본적으로 BasicErrorController가 등록이 되어 해당 요청을 처리하게 된다.

BasicErrorController는 대략적으로 아래와 같이 구현되어 있다.



```java
@Controller
@RequestMapping("${server.error.path:${error.path:/error}}") // 1)
public class BasicErrorController extends AbstractErrorController {

  @Override
  public String getErrorPath() {
    return this.errorProperties.getPath();
  }

  @RequestMapping(produces = MediaType.TEXT_HTML_VALUE) // 2)
  public ModelAndView errorHtml(HttpServletRequest request, HttpServletResponse response) {
            
    HttpStatus status = getStatus(request);
    Map<String, Object> model =
      getErrorAttributes(request, isIncludeStackTrace(request, MediaType.TEXT_HTML)));
		
    response.setStatus(status.value());
    ModelAndView modelAndView = resolveErrorView(request, response, status, model);
    return (modelAndView != null) ? modelAndView : new ModelAndView("error", model);
  }

  @RequestMapping // 3)
  public ResponseEntity<Map<String, Object>> error(HttpServletRequest request) {
    
    // 4)
    Map<String, Object> body =
      getErrorAttributes(request, isIncludeStackTrace(request, MediaType.ALL));
    HttpStatus status = getStatus(request);
    return new ResponseEntity<>(body, status);
  }    
}
```



1. Spring 환경 내에 `server.error.path` 혹은 `error.path`로 등록된 property의 값을 넣거나, 없는 경우 `/error`를 사용한다.
2. HTML로 응답을 주는 경우 `errorHtml`에서 응답을 처리한다.
3. HTML 외의 응답이 필요한 경우 `error`에서 처리한다.
4. 실질적으로 view에 보낼 model을 생성한다



> **BasicErrorController 정리**
> `BasicErrorController`에서는 HTML 요청, 그 외의 요청을 나누어서 처리할 핸들러를 등록하고 `getErrorAttributes`를 통해 응답을 위한 모델을 생성한다.





