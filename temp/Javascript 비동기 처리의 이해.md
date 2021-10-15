## Javascript 비동기 처리의 이해

> https://realrain.net/post/async-await/



### 브라우저에서의 자바스크립트 런타임 (runtime)

- 런타임

  - 프로그램이 실행되고 있는 시간, 혹은 실행되고 있는 공간. 구현체라고도 함

  - Java의 JVM이 대표적인 런타임이라고 할 수 있다.

  - JVM도 라이선스에 따라 OracleJDK, OpenJDK, AdoptOpenJDK 등이 있는 것처럼

    Python도 IronPython, PyPy 등 다양한 구현체가 있습니다.

  - 자바스크립트의 경우 node.js가 나오기 전까지는 IE, Firefox, Chrome 같은 **<u>웹 브라우저</u>**가 런타임으로서 유일했다.



- 브라우저 내의 자바스크립트 런타임 환경
  - 아래 4가지 요소로 구성되어 있다.
  - 자바스크립트 엔진: 힙 메모리, 콜 스택을 포함하는 자바스크립트 해석 엔진
  - WebAPI: DOM 조작, 네트워크 요청/응답 등의 브라우저 고유 기능
  - Callback Queue: WebAPI로부터 전달받은 콜백 함수 저장
  - Event Loop: 콜 스택이 빌 때마다, 콜백 큐에서 콜백 함수를 콜 스택으로 하나씩 옮김

![](https://realrain.net/images/runtime_1.png)







- 예제

```javascript
console.log('Hello');
setTimeout(() => console.log('Javascript'), 1000)
```

위 코드를 실행하고 나면 브라우저에서 아래와 같은 일이 일어난다.

![](https://realrain.net/images/runtime_2.png)

- 먼저 `console.log('Hello');`가 콜 스택에 올라가고 실행된다.







![](https://realrain.net/images/runtime_3.png)

- 콘솔에 `hello`가 출력되고 나면, 다음으로 `setTimeout`이 콜 스택에 올라간다.





![](https://realrain.net/images/runtime_4.png)

- `setTimeout`은 WebAPI의 기능이므로 이를 WebAPI가 실행하도록 넘겨주고 콜 스택을 비운다.
- WebAPI는 1초 후 같이 넘겨받은 callback 함수를 콜백 큐에 넣어 줍니다.





![](https://realrain.net/images/runtime_5.png)

- 이벤트 루프는 자바스크립트 엔진의 콜 스택을 감시하고 있다가 콜 스택이 비게 되면 콜백 큐에서 콜백 함수 `console.log('Javascript')`를 꺼내 콜 스택으로 옮긴다.
- 자바스크립트 엔진은 최종적으로 콜 스택에서 이를 실행해 콘솔에 `JavaScript`를 출력한다.
- `setTimeout` 외에도 DOM에 부착되는 이벤트 핸들러, AJAX 요청도 이와 동일하게 동작하게 된다.





### 왜 이벤트 루프인가요?

- 브라우저의 자바스크립트 런타임에서 이러한 방식을 사용하는 이유는 자바스크립트 엔진이 **싱글 스레드**로 작동하기 때문이다.
- 싱글 스레드 환경에서 이벤트 루프를 사용하지 않고 WebAPI를 동기식으로 요청하게 되면, 해당 동작이 끝날 때까지 blocking되어 다른 동작을 처리할 수 없기 때문이다. (자바스크립트 엔진이 싱글 스레드인 것이지, 브라우저가 싱글 스레드로 동작하는 것은 아니다.)
- 싱글 스레드로 동작함에도 불구하고 사용자 입장에서 여러 작업이 동시에 일어나는 것처럼 보이는 이유가 바로 **이벤트 루프**를 통해 동시성을 구현했기 때문이다.





### 다시 비동기로 돌아와서...

- 위에서 언급한 이유로 WebAPI 함수들은 자바스크립트 엔진에서 모두 비동기로 동작한다.

- `setTimeout`과 같이 특정 시간 이후에 실행되기만 하면 되는 함수는 그다지 큰 고민을 할 여지가 없지만 결과값을 사용해야 하는 AJAX 요청은 비동기로 동작하는 환경에서 여러가지 어려움을 야기한다.

  

  ```javascript
  var httpRequest = new XMLHttpRequest();
  httpRequest.onreadystatechange = function() {
    if(httpRequest.readyState === XMLHttpRequest.DONE) {
      if(httpRequest.status === 200) {
        // 요청 응답을 alert
        alert(httpRequest.responseText);
      } else {
        alert('error!');
      }
    }
  };
  httpRequest.open('GET', 'http://www.example.org', true);
  httpRequest.send(null);
  ```

  - WebAPI에 `fetch()`가 추가되기 전에는 위와 같은 식으로 AJAX 요청을 처리했습니다.
  - 여러가지 요청 상태값에 대한 처리를 포함하고 있지만 결국 `httpRequest.onreadystatechange`에 콜백 함수를 부착하는 형태이다.



### 비동기 함수의 문제점 및 해결책

```javascript
var userData;

function setUserData(data) {
	userData = someProcess(data);
};

$.get('url주소', function(response) {
	setUserData(response);
})

console.log(userdata);
```

- AJAX 요청을 jQuery로 깔끔하게 처리하고, `setUserData`라는 요청 결과 데이터를 가공해 `userData`에 넣어주는 콜백을 달았습니다.
- 위 코드는 `undefined`가 출력된다.
  - `$.get(...)` 함수가 비동기로 처리되는 동안 바로 `console.log(userData)`가 실행되기 때문



#### 해결책

```javascript
var userData;

function setUserData(data, callback) {
  userData = someProcess(data);
  callback(userData);
};

$.get('http://www.example.org', function(response) {
  setUserData(response, function(result) {
    console.log(result);
  });
})
```





### Promise의 탄생

- 콜백 함수를 이용한 비동기 처리의 문제점을 해결하기 위해 생겨난 것이 바로 ES6에서 새로 제안된 `Promise`입니다.

- `Promise`는 비동기 연산을 감싸는 일종의 프록시 객체입니다.

- 프로미스에 대한 자세한 설명은 [이곳](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Promise)을 참조하세요.

- **예제**

  ```javascript
  function getData() {
    return new Promise((resolve, reject) => {
      $.get('http://www.example.org', function(response) {
        if(response) {
          resolve(response);
        }
        reject(new Error('error'));
      });
    });
  }
  
  getData()
  	.then(task)
  	.then(someTask)
  	.then(someNextTask);
  ```

  - Promise 덕분에 콜백을 중첩해서 호출하는 대신, Promise의 Chaining을 통해 좀 더 깔끔한 코드를 만들 수 있게 되었습니다.
  - 에러 처리도 상당히 편리해진다.
  - 그러나 <u>아직도 일련의 종속적인 작업들이 하나의 Promise Chain에 묶여있다는 점이 조금 불편하게 느껴진다.</u>



### 다시 async - await

- `async`, `await`은 ES8부터 자바스크립트에 포함되었고, `Promise`를 동기식 코드로 표현할 수 있게 해주는 문법이다.
- `await`을 Promise 앞에 붙이게 되면 Promise가 성공했을 때의 결과값이 반환된다.

```javascript
async () => {
  const response = await getData();
  const result = await someTask(response);
  const result_2 = await someNextTask(result);
  const result_3 = await someNextNextTask(result_2);
}
```

- 이제 비동기 작업을 감싼 Promise API는 감춰지고 온전히 동기적으로 이해할 수 있는 코드가 만들어진다.

















