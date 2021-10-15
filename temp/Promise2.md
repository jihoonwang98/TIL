## async와 await

> https://ko.javascript.info/async-await



### async 함수

- `async` 키워드는 <u>function 앞</u>에 위치한다.

  ```javascript
  async function f() {
  	return 1;
  }
  ```



- **<u>function 앞에 `async`를 붙이면 해당 함수는 항상 무조건 Promise를 반환한다.</u>**

  - 만약 Promise가 아닌 값을 반환하더라도 이행 상태의 Promise(resolved promise)로 값을 감싸 resolved promise가 반환되도록 한다.

  - 아래 예시의 함수를 호출하면 `result`가 `1`인 resolved promise가 반환된다.

    ```javascript
    async function f() {
    	return 1;
    }
    
    f().then(alert); // 1
    ```

  - 명시적으로 Promise를 반환하는 것도 가능하다. 결과는 동일하다.

    ```javascript
    async function f() {
    	 return Promise.resolve(1);
    }
    f().then(alert); // 1
    ```





### await 키워드

- `await` 문법

  ```javascript
  // await는 async 함수 안에서만 동작한다.
  let value = await promise;
  ```

- <u>await는 async 함수 안에서만 동작한다.</u>

- 자바스크립트는 `await` 키워드를 만나면 Promise가 처리(settled)될 때까지 기다린다. 결과는 그 이후에 반환된다.











