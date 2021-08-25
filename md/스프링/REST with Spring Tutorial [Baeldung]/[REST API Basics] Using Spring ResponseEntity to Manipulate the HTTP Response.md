# Using Spring ResponseEntity to Manipulate the HTTP Response :goat:



> **[참고]**: https://www.baeldung.com/spring-response-entity
>
> 의역과 오역에 주의...





## 1. Introduction

이번 포스팅에서는 *ResponseEntity*를 이용해서 HTTP 응답의 *body, status, header*를 설정하는 방법을 알아봅니다.







## 2. ResponseEntity

*ResponseEntity*는 HTTP 응답을 표현하는 객체입니다.



따라서 이 *ResponseEntity* 객체는

- **status code**
- **headers**
- **body**

로 구성되어 있습니다.



결과적으로, 우리는 <u>HTTP 응답을 이 *ResponseEntity* 객체로 완전히 대체</u>할 수 있습니다.

만약 이 객체를 사용하고 싶다면, 우리는 endpoint에서 이 객체를 반환하기만 하면 Spring이 나머지 처리는 알아서 해줍니다. :+1:





*ResponseEntity*는 generic 타입입니다. 따라서 우리는 **응답 body**로 어떠한 type도 사용할 수 있습니다.

```java
@GetMapping("/hello")
ResponseEntity<String> hello() {
    return new ResponseEntity<>("Hello World!", HttpStatus.OK);
}
```





우리가 직접 코드로 *response status*를 지정할 수 있기 때문에, 다양한 상황에서 각 상황에 적절한 *status code*를 반환하게 하는 것도 가능합니다.

```java
@GetMapping("/age")
ResponseEntity<String> age(
  @RequestParam("yearOfBirth") int yearOfBirth) {
 
    if (isInFuture(yearOfBirth)) {
        return new ResponseEntity<>(
          "Year of birth cannot be in the future", 
          HttpStatus.BAD_REQUEST);
    }

    return new ResponseEntity<>(
      "Your age is " + calculateAge(yearOfBirth), 
      HttpStatus.OK);
}
```





추가적으로, *HTTP header*들을 설정하는 것도 가능합니다.

```java
@GetMapping("/customHeader")
ResponseEntity<String> customHeader() {
    HttpHeaders headers = new HttpHeaders();
    headers.add("Custom-Header", "foo");
        
    return new ResponseEntity<>(
      "Custom header set", headers, HttpStatus.OK);
}
```





뿐만 아니라, *ResponseEntity*는 두개의 nested builder interface를 제공합니다.

**HeadersBuilder**와 이것의 subinterface인 **BodyBuilder**를 말이죠.

우리는 이들을 *ResponesEntity*의 static method들을 통해 접근할 수 있습니다.





가장 간단한 예시는 ***body***와 ***HTTP 200 응답 코드***로 이루어진 응답을 보내는 사례입니다.

```java
@GetMapping("/hello")
ResponseEntity<String> hello() {
    return ResponseEntity.ok("Hello World!");
}

// 참고 
// package org.springframework.http.ResponseEntity를 까보면 아래처럼 되어 있다.

private static class DefaultBuilder implements ResponseEntity.BodyBuilder {
        private final Object statusCode;
        private final HttpHeaders headers = new HttpHeaders();
    	
    	... 생략
}


public static ResponseEntity.BodyBuilder status(int status) {
        return new ResponseEntity.DefaultBuilder(status);
}


public static ResponseEntity.BodyBuilder ok() {
        return status(HttpStatus.OK);
}
```





대부분의 자주 쓰이는 *HTTP status code*들에 대해서는 static method들이 구현돼있다.

```java
BodyBuilder accepted();
BodyBuilder badRequest();
BodyBuilder created(java.net.URI location);
HeadersBuilder<?> noContent();
HeadersBuilder<?> notFound();
BodyBuilder ok();
```



이 외에도, 

**`BodyBuilder status(HttpStatus status)`**와

**`BodyBuilder status(int status) `**메서드를 통해

어떠한 *HTTP status*도 설정 가능하다.







마지막으로, **`ResponseEntity<T> BodyBuilder.body(T body)`** 메서드를 통해

우리는 *HTTP 응답 body*를 설정할 수 있다.

```java
@GetMapping("/age")
ResponseEntity<String> age(@RequestParam("yearOfBirth") int yearOfBirth) {
    if (isInFuture(yearOfBirth)) {
        return ResponseEntity.badRequest()
            .body("Year of birth cannot be in the future");
    }

    return ResponseEntity.status(HttpStatus.OK)
        .body("Your age is " + calculateAge(yearOfBirth));
}
```



또한, custom header들을 설정하는 것도 가능하다.

```java
@GetMapping("/customHeader")
ResponseEntity<String> customHeader() {
    return ResponseEntity.ok()
        .header("Custom-Header", "foo")
        .body("Custom header set");
}
```



**`BodyBuilder.body()`**가 *BodyBuilder*가 아니라 *ResponseEntity*를 반환하므로 

**`BodyBuilder.body()`**는 맨 마지막에 호출되어야 한다.



또한, <u>*HeaderBuilder*로는 **응답 body**의 어떠한 property도 설정할 수 없다</u>는 것을 명심해라.









**`ResponseEntity<T>`** 객체를 controller에서 반환할 때,

요청을 처리하다가 *exception*이나 *error*가 발생할 수도 있다.

이런 경우 client에게 error와 관련된 정보를 

**ResponseEntity** 타입이 아닌 다른 형태의 type으로 전달하고 싶을 수도 있다.



이에 따라, Spring 3.2 버전부터는 **@ControllerAdvice** 어노테이션과 함께 **@ExceptionHandler** 어노테이션을 지원한다. 



> 이 어노테이션들이 궁금하다면 아래 링크를 참조하자
>
> https://www.baeldung.com/exception-handling-for-rest-with-spring



**ResponseEntity**는 매우 강력하고 유용하지만, 남용하면 큰일난다. :scream:







## 3. Alternatives

**ResponseEntity**말고 다른 방법은 뭐가 있을까 :question::question:



### 3.1 @ResponseBody

Classic한 Spring MVC 애플리케이션에서는, endpoint들이 보통 렌더링된 HTML 페이지들을 반환했다.

때때로 우리는 페이지가 아닌 **실제 data**를 반환해야 한다. 예를 들면, AJAX와 함께 endpoint를 사용할 때처럼 말이다.



이러한 상황들에서는, *request handler method*에 **@ResponseBody**를 붙여주고, 

Spring이 method가 반환하는 값을 **HTTP 응답 body**처럼 처리할 수 있게 알려주자.





### 3.2 @ResponseStatus

endpoint가 성공적으로 return 됐을 때, Spring은 **HTTP 200 (OK) 응답**을 보낸다.

endpoint가 exception을 뱉을 때, Spring은 exception handler에 가서 어떤 *HTTP status*를 사용해야 하는지 찾는다.



우리는 이러한 메서드들에 **@ResponseStatus** 어노테이션을 붙여서 리턴할 *HTTP status*를 지정할 수 있다.





### 3.3 Manipulate the Response Directly

Spring은 **`javax.servlet.http.HttpServletResponse`** 객체를 direct로 접근할 수 있게 해준다.

접근하고 싶으면 그냥 메서드 parameter로 선언하기만 하면 된다.



```java
@GetMapping("/manual")
void manual(HttpServletResponse response) throws IOException {
    response.setHeader("Custom-Header", "foo");
    response.setStatus(200);
    response.getWriter().println("Hello World!");
}
```

하지만, Spring이 이런 기능을 추상화해주고 추가적인 기능들을 제공하기 때문에 

이렇게 직접 접근하는 것은 좋지 않다.