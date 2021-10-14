## Callback hell in Node.js

> https://www.geeksforgeeks.org/what-is-callback-hell-in-node-js/



### Synchronous JS와 Asynchronous JS

- 동기 JS
  - 코드를 실행하면 그 결과는 브라우저가 해당 결과를 반환할 수 있게 되자마자 반환된다.
  - 한번에 오직 한 개의 연산만이 실행될 수 있다. (싱글스레드이기 때문)
  - 다른 모든 프로세스들은 한 연산이 수행되는 동안 멈춘다.
- 비동기 Js
  - JS의 몇몇 함수들의 response는 즉시 받을 수 없기 때문에 AJAX가 필요하다.
  - 이러한 함수들은 처리하는데 시간이 좀 걸리기 때문에 다음 연산이 즉시 실행될 수가 없다.
    - 이러한 함수들이 background에서 처리되기를 기다려야 한다.
    - 이렇게 <u>결과값을 기다려야 하는 경우에 callback 함수들이 asynchronous하게 작성될 필요가 있다.</u>
  - AJAX(Asynchronous Javascript and XML)
    - AJAX를 지원하는 외부 Javascript Web API들도 존재한다.
    - AJAX를 사용하면, 많은 연산들은 동시에 수행될 수 있다.



![](https://media.vlpt.us/post-images/surim014/233bbf90-2d60-11ea-a24d-4f910e80580b/image.png)



### 콜백이 무엇인가

- 콜백들은 결과값을 반환하는 데 시간이 걸리는 함수일 뿐이다.
- asynchronous callback이 쓰이는 예시
  - DB 조회
  - 이미지 다운로드
  - 파일 읽기 등등
- 위와 같은 작업들은 수행이 끝날 때까지 시간이 많이 걸리기 때문에
  - 다음줄로 넘어갈 수도 없고, (왜냐면 error가 발생할 수도 있기 때문)
  - program을 정지시킬 수도 없다.
- 그러므로 수행이 다 끝났을 때 call back 해야 한다.





### callback hell 벗어나기

- JS는 callback hell로부터 벗어나는 두 가지 간단한 방법을 제공한다.
  - Event queue
  - Promise
- Promise
  - promise는 async 함수로부터 반환되는 객체다.
  - 이전 함수의 결과에 따라 callback method들이 promise에 추가될 수 있다.
  - `.then()`: async callback을 호출할 때 사용된다.
    - callback들을 chaining해서 사용할 수도 있다.
    - 순서는 유지된다.
  - `.fetch()`: 네트워크로부터 객체를 가져올 수 있다.
    - `.catch()` 메서드를 사용해서 어떤 block이 실패하면 exception을 catch 할 수 있다.
- 이러한 Promise들은 Event Queue에 들어가서 subsequent JS 코드들을 block하지 않는다.
  - 한번 결과들이 반환되면, event queue는 자신의 연산들을 끝마친다.
- 그 외에도 `async`, `wait`, `settimeout()`과 같은 키워드를 통해 callback들을 효율적으로 다룰 수 있다.















