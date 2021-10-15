##  Promise

> https://developer.mozilla.org/ko/docs/Web/JavaScript/Guide/Using_promises
>
> https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Promise



### Promise의 사용법

- Promise란?

  - `Promise` 객체는 비동기 작업의 최종 완료 또는 실패를 나타내는 객체이다.

  - 기본적으로 Promise는 함수에 콜백을 전달하는 대신에, 콜백을 첨부하는 방식의 객체이다.

  - **예제**

    - 비동기로 음성 파일을 생성해주는 `createAudioFileAsync()`라는 함수가 있다고 생각해보자.

    - 해당 함수는 음성 설정에 대한 정보를 받고, 두 가지 콜백 함수를 받는다.

      - 하나는 음성 파일이 성공적으로 생성되었을때 실행되는 콜백
      - 다른 하나는 에러가 발생했을 때 실행되는 콜백

    - `createAudioFileAsync()` 함수는 아래와 같이 사용된다.

      ```javascript
      function successCallback(result) {
        console.log("Audio file ready at URL: " + result); 
      }
      
      function failureCallback(error) {
        console.log("Error generating audio file: " + error);
      }
      
      createAudioFileAsync(audioSettings, successCallback, failureCallback);
      ```

    - 하지만 모던한 함수들은 위와 같이 콜백들을 전달하지 않고 콜백을 붙여 사용할 수 있게 Promise를 반환해줍니다.

    - 만약 `createAudioFileAsync()` 함수가 Promise를 반환해주도록 수정해본다면, 다음과 같이 간단하게 사용될 수 있다.

      ```javascript
      createAudioFileAsync(audioSettings)
      	.then(sucessCallback, failureCallback);
      ```

    - 우리는 이런 것들을 **비동기 함수 호출**이라고 부른다. 이런 관례는 몇 가지 장점을 가지고 있다. 각각에 대해 한번 살펴보도록 하자.



### Guarantees(굳은 약속)

- 콜백 함수를 전달해주는 고전적인 방식과는 달리, Promise는 아래와 같은 특징을 보장한다.
  - 콜백은 자바스크립트 Event Loop이 [현재 실행중인 콜 스택을 완료](https://developer.mozilla.org/en-US/docs/Web/JavaScript/EventLoop#run-to-completion)하기 이전에는 절대 호출되지 않습니다.
  - 비동기 작업이 성공하거나 실패한 뒤에 `then()`을 이용하여 추가한 콜백의 경우에도 위와 같다.
  - `then()`을 여러번 사용하여 여러 개의 콜백을 추가할 수 있다. 그리고 각각의 콜백은 주어진 순서대로 하나 하나 실행되게 된다.
- Promise의 가장 뛰어난 장점 중의 하나는 **chaining**이다.



### Chaining

- 보통 하나나 두 개 이상의 비동기 작업을 순차적으로 실행해야 하는 상황을 흔히 보게 된다.

- 순차적으로 각각의 작업이 이전 단계 비동기 작업이 성공하고 나서 그 결과값을 이용하여 다음 비동기 작업을 실행해야 하는 경우를 의미한다.

- 우리는 이런 상황에서 **promise chain**을 이용하여 해결하기도 한다.

- `then()` 함수는 새로운 promise를 반환한다. 처음에 만들었던 promise와는 다른 새로운 promise이다.

  ```javascript
  const promise = doSomething();
  const promise2 = promise.then(successCallback, failureCallback);
  ```

  또는

  ```javascript
  const promise2 = doSomething().then(successCallback, failureCallback);
  ```

- promise2는 `doSomething()`뿐만 아니라 `successCallback` 또는 `failureCallback`의 완료를 의미한다.

  - `successCallback` 또는 `failureCallback` 또한 promise를 반환하는 비동기 함수일 수도 있습니다.
  - 이 경우 `promise2`에 추가된 콜백은 `successCallback` 또는  `failureCallback`에 의해 반환된 promise 뒤에 대기한다.

- 기본적으로, 각각의 promise는 체인 안에서 서로 다른 비동기 단계의 완료를 나타낸다.



- 콜백 지옥 벗어나기

  - **전**

    ```javascript
    doSomething(function(result) {
      doSomethingElse(result, function(newResult) {
        doThirdThing(newResult, function(finalResult) {
          console.log('Got the final result: ' + finalResult);
        }, failureCallback);
      }, failureCallback);
    }, failureCallback);
    ```

  - **후**

    ```javascript
    doSomething().then(function(result) {
      return doSomethingElse(result);
    })
    .then(function(newResult) {
      return doThirdThing(newResult);
    })
    .then(function(finalResult) {
      console.log('Got the final result: ' + finalResult);
    })
    .catch(failureCallback);
    ```

  - `then` 에 넘겨지는 인자는 선택적(optional)입니다. 

  - `catch(failureCallback)` 는 `then(null, failureCallback)` 의 축약입니다. 

  - 위 표현식을 [화살표 함수](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions)로 나타내면 다음과 같습니다.

    ```javascript
    doSomething()
    .then(result => doSomethingElse(result))
    .then(newResult => doThirdThing(newResult))
    .then(finalResult => {
      console.log(`Got the final result: ${finalResult}`);
    })
    .catch(failureCallback);
    ```

    - **중요:** 반환값이 반드시 있어야 합니다, 만약 없다면 콜백 함수가 이전의 promise의 결과를 받지 못합니다.



















