# [Spring] `@RequestBody`, `@ModelAttribute`, `@RequestParam`의 차이



> https://mangkyu.tistory.com/72
>
> https://blog.naver.com/writer0713/221853596497



## 1. `@RequestBody`, `@ModelAttribute`, `@RequestParam`이란?



**FormHttpMessageConverter**



### `@RequestParam`

- 1개의 HTTP 요청 파라미터를 받기 위해 사용
- 필수 여부가 기본적으로 `true`로 되어 있다.
  - 따라서, 반드시 해당 파라미터가 전송되어야 함. 전송 안되면 400 Error남.
  - 필수 여부를 `@RequestParam(required=false)`로 설정해놓을 수도 있다.
  - `@RequestParam(required=false, defaultValue=)`를 통해 해당 파라미터가 전송되지 않은 경우 기본값을 설정해놓을 수도 있다.







### `@RequestBody`

- 클라이언트가 전송하는 **JSON(application/json) 형태의 HTTP Body 내용을 Java Object로 변환**해주는 역할

  - 따라서 HTTP Body가 없는 GET 요청에 대해서는 `@RequestBody`를 사용하지 못함. 에러남

    

- `@RequestBody`로 받는 데이터는 Spring의 MessageConverter 중 하나인 **<u>MappingJackson2HttpMessageConverter</u>**를 통해 Java 객체로 변환된다.







### `@ModelAttribute`

- 클라이언트가 전송하는 **multipart/form-data 형태의 HTTP Body 내용 또는 HTTP 파라미터들을 Setter를 통해 1대1로 객체에 바인딩**해주는 역할
- `@ModelAttribute`에는 매핑시키는 파라미터의 타입이 객체의 타입과 일치하는지를 포함한 다양한 <u>검증(validation)</u> 작업이 추가적으로 진행된다.
  - 예를 들어 int형 index 변수에 "1번"이라는 String을 넣으려고 하면, BindException이 발생한다.

- `@ModelAttribute`는 바인딩시키는 어떤 데이터를 set해주는 setter가 없으면 매핑이 되지 않지만,

  `@RequestBody`는 요청받은 데이터를 변환시키는 것이기 때문에, setter가 없어도 값이 매핑된다.

- 추가로 `@ModelAttribute` 애너테이션은 특정 Parameter만 받아올 수도 있다.

  - `writer=망나니&contents=안녕하세요`로 데이터를 전송했을 때,

    컨트롤러에서는 `@ModelAttribute("writer") String writer`의 형태로 writer 변수에 망나니만 바인딩시켜 받아올 수 있다.









































