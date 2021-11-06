# [Typescript 핸드북] 함수

> https://typescript-kr.github.io/pages/functions.html



### 목차

- 함수 (Function)
- 함수 타입 (Function Types)
  - 함수의 타이핑 (Typing the function)
  - 함수 타입 작성하기 (Writing the function type)
  - 타입의 추론 (Inferring the types)
- 선택적 매개변수와 기본 매개변수 (Optional and Default Parameter)
- 나머지 매개변수 (Rest Parameters)
- this
  - `this`와 화살표 함수 (this and arrow functions)
  - `this` 매개변수
  - 콜백에서 `this` 매개변수
- 오버로드 (Overload)



## 소개 (Introduction)

- 함수는 JavaScript로 된 모든 애플리케이션에서의 기본적인 구성 요소입니다. 
- JavaScript 함수는 추상화 계층을 구축하거나 클래스 모방, 정보 은닉, 모듈에 대한 방법을 제공합니다. 
- TypeScript에는 클래스, 네임스페이스, 모듈이 있지만, 함수는 여전히 이 일을 어떻게 *할 것인지*를 설명하는 데 있어 핵심 역할을 수행합니다.
- TypeScript에서는 표준 JavaScript 함수가 작업을 수월하게 하도록 몇 가지 새로운 기능을 추가합니다.



## 함수 (Function)

- TypeScript 함수는 JavaScript와 마찬가지로 기명 함수(named function)와 익명 함수(anonymous function)로 만들 수 있습니다. 

  - 이를 통해 API에서 함수 목록을 작성하든 일회성 함수를 써서 다른 함수로 전달하든 애플리케이션에 가장 적합한 방법을 선택할 수 있습니다.

- JavaScript에서의 이 두 가지 방법에 대한 예시를 빠르게 다시 보겠습니다:

  ```js
  // 기명 함수
  fucntion add(x, y) {
    return x + y;
  }
  
  // 익명 함수
  let myAdd = function(x, y) { return x + y };
  ```

- JavaScript에서처럼, 함수는 함수 외부의 변수를 참조할 수 있습니다. 

  - 이런 경우를, 변수를 *캡처(capture)* 한다고 합니다. 

  - 이것이 어떻게 작동하는지 (그리고 이 기술을 사용할 때의 장단점)를 이해하는 것은 이 본문의 주제를 벗어나는 것이지만, 

    이 메커니즘이 어떻게 작동하는지에 대한 확실한 이해는 JavaScript 및 TypeScript를 사용하는 데 있어 중요합니다.

  ```js
  let z = 100;
  
  function addToZ(x, y) {
    return x + y + z;
  }
  ```



## 함수 타입 (Function Types)

### 함수의 타이핑 (Typing the function)

- 이전에 사용했던 예시에 타입을 더해보겠습니다.

  ```ts
  function add(x: number, y: number): number {
      return x + y;
  }
  
  let myAdd = function(x: number, y: number): number { return x + y };
  ```

  - 각 파라미터와 함수 자신의 반환될 타입을 정해줄 수 있습니다. TypeScript는 반환 문을 보고 반환 타입을 파악할 수 있으므로 반환 타입을 생략할 수 있습니다.















