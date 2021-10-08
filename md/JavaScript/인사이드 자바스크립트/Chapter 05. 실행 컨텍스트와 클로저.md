Chapter 05. 실행 컨텍스트와 클로저

### 목차

- 5.1 실행 컨텍스트 개념
- 5.2 실행 컨텍스트 생성 과정
- 5.3 스코프 체인
- 5.4 클로저



## 5.1 실행 컨텍스트 개념

- 콜 스택(call stack)
  - 함수를 호출할 때 해당 함수의 호출 정보가 차곡차곡 쌓여있는 스택을 의미한다.
    - C언어를 예로 들면, 함수의 호출 정보 등으로 함수 내 지역 변수 혹은 인자값 등이 쌓이는 공간

- 실행 컨텍스트

  - 실행 컨텍스트는 앞에서 설명한 콜 스택에 들어가는 실행 정보 하나와 비슷하다.

  - ECMAScript에서는 실행 컨텍스트를 **"실행 가능한 코드를 형상화하고 구분하는 추상적인 개념"**으로 기술한다.

  - 이를 앞에서 설명한 콜 스택과 연관하여 정의하면, **"실행 가능한 자바스크립트 코드 블록이 실행되는 환경"**이라고 할 수 있고, 이 컨텍스트 안에 실행에 필요한 여러 가지 정보를 담고 있다.

    - 여기서 말하는 실행 가능한 코드 블록은 대부분의 경우 함수가 된다.

  - ECMAScript에서는 실행 컨텍스트가 형성되는 경우를 세 가지로 규정하고 있다.

    - 전역 코드
    - eval() 함수로 실행되는 코드
    - 함수 안의 코드를 실행할 경우

  - 대부분 프로그래머는 함수로 실행 컨텍스트를 만든다.

    - 그리고 이 코드 블록 안에 변수 및 객체, 실행 가능한 코드가 들어있다.
    - 이 코드가 실행되면 실행 컨텍스트가 생성되고, 실행 컨텍스트는 스택 안에 하나씩 차곡차곡 쌓이고, 제일 위(top)에 위치하는 실행 컨텍스트가 현재 실행되고 있는 컨텍스트다.

  - ECMAScript에서는 실행 컨텍스트의 생성을 다음처럼 설명한다.

    - "현재 실행되는 컨텍스트에서 이 컨텍스트와 관련 없는 실행 코드가 실행되면, 새로운 컨텍스트가 생성되어 스택에 들어가고 제어권이 그 컨텍스트로 이동한다"

  - **[예제 5-1]**

    ```javascript
    console.log('This is global context');
    
    function ExContext1() {
        console.log('This is ExContext1');
    }
    
    function ExContext2() {
        ExContext1();
        console.log('This is ExContext2');
    }
    
    ExContext2();
    ```

    ![](https://docs.google.com/drawings/d/sMU4uDNFxdjbtJWV46giuNw/image?parent=e/2PACX-1vQiuXMoUJloXB41go11kimhAxCGB0Jhf5sdczvyrPIjH533basHPtLBcGPi59BcshFWUj6e55GqbGOO&rev=200&drawingRevisionAccessToken=uvSbYVZ9qnLsFA&h=308&w=601&ac=1)

    

    - 위 그림과 같이 전역 실행 컨텍스트가 가장 먼저 실행된다.
    - 이 과정에서 새로운 함수 호출이 발생하면 새로운 컨텍스트가 만들어지고 실행되며, 종료되면 반환된다.
    - 이와 같은 과정이 반복된 후, 전역 실행 컨텍스트의 실행이 완료되면 모든 실행이 끝난다.





## 5.2 실행 컨텍스트 생성 과정

이 장에서는 실행 컨텍스트의 생성 과정을 설명할 텐데, 이 과정에서 다음과 같은 중요 개념을 설명한다.

- 활성 객체와 변수 객체
- 스코프 체인



- **[예제 5-2]**

  ```javascript
  function execute(param1, param2) {
      var a = 1, b = 2;
  
      function func() {
          return a + b;
      }
  
      return param1 + param2 + func();
  }
  
  console.log(execute(3, 4)); // 10
  ```



자바스크립트에서 함수를 실행하여 실행 컨텍스트가 생성되면 자바스크립트 엔진은 다음과 같은 일을 정해진 순서대로 실행한다.

1. 활성 객체 생성
2. arguments 객체 생성
3. 스코프 정보 생성
4. 변수 생성
5. this 바인딩
6. 코드 실행

위 예제 5-2에서 execute() 함수를 실행한 경우를 예로 들어 설명하겠다.





- 활성 객체 생성

  - 실행 컨텍스트가 생성되면 자바스크립트 엔진은 해당 컨텍스트에서 실행에 필요한 여러 가지 정보를 담을 객체를 생성하는데, 이를 **활성 객체**라고 한다.
  - 이 객체에 앞으로 매개변수나 사용자가 정의한 변수 및 객체를 저장하고, 새로 만들어진 컨텍스트로 접근 가능하게 되어 있다.
    - 이는 엔진 내부에서 접근할 수 있다는 것이지 사용자가 접근할 수 있다는 것은 아니다.

  

  ![](https://docs.google.com/drawings/d/sPPQENm8MD-oABbVBxQdIaA/image?parent=e/2PACX-1vQiuXMoUJloXB41go11kimhAxCGB0Jhf5sdczvyrPIjH533basHPtLBcGPi59BcshFWUj6e55GqbGOO&rev=80&drawingRevisionAccessToken=6zmOB4TNPusXnw&h=228&w=335&ac=1)

  



- arguments 객체 생성

  - 다음 단계에서는 arguments 객체를 생성한다.

  - 앞서 만들어진 활성 객체는 arguments 프로퍼티로 이 arguments 객체를 참조한다.

    

    ![](https://docs.google.com/drawings/d/sGrBRLNMWgi6Hg3sCAJIeCw/image?parent=e/2PACX-1vQiuXMoUJloXB41go11kimhAxCGB0Jhf5sdczvyrPIjH533basHPtLBcGPi59BcshFWUj6e55GqbGOO&rev=141&drawingRevisionAccessToken=hqmY7tMP01o81A&h=288&w=335&ac=1)

    

  - 위 그림에서는 execute() 함수의 param1과 param2가 들어왔을 경우의 활성 객체의 상태를 표현한다.



- 스코프 정보 생성

  - 현재 컨텍스트의 유효 범위를 나타내는 스코프 정보를 생성한다.
  - 이 스코프 정보는 현재 실행 중인 실행 컨텍스트 안에서 연결 리스트와 유사한 형식으로 만들어진다.
    - 현재 컨텍스트에서 특정 변수에 접근해야 할 경우, 이 리스트를 활용한다.
    - 이 리스트로 현재 컨텍스트의 변수뿐 아니라, 상위 실행 컨텍스트의 변수도 접근이 가능하다.
    - 이 리스트에서 찾지 못한 변수는 결국 정의되지 않은 변수에 접근하는 것으로 판단하여 에러를 검출한다.
    - 이 리스트를 **스코프 체인**이라고 하는데, [[scope]] 프로퍼티로 참조된다.
  - 이것이 왜 리스트의 형태를 띠고 있고, 이 체인이 어떻게 형성되며, 리스트의 구성 요소가 무엇인지는 이후에 좀 더 자세히 살펴보자.
  - 여기서는 현재 생성된 활성 객체가 스코프 체인의 제일 앞에 추가되며, execute() 함수의 인자나 지역 변수 등에 접근할 수 있다는 것만 알고 넘어가자.

  

  ![](https://docs.google.com/drawings/d/so5RMrBUkwBPAu9_pWVNWug/image?parent=e/2PACX-1vQiuXMoUJloXB41go11kimhAxCGB0Jhf5sdczvyrPIjH533basHPtLBcGPi59BcshFWUj6e55GqbGOO&rev=53&drawingRevisionAccessToken=gDw7hr7UE9AUZA&h=288&w=335&ac=1)

  

- 변수 생성

  - 현재 실행 컨텍스트 내부에서 사용되는 지역 변수의 생성이 이루어진다.

  - ECMAScript에서는 생성되는 변수를 저장하는 변수 객체를 언급하는데, 실제적으로 앞서 생성된 활성 객체가 변수 객체로 사용된다.

    - 우리가 자바스크립트 관련 문서를 읽다 보면 어떤 곳에서는 활성 객체, 어떤 곳에서는 변수 객체라고 사용되어 혼란스러운 경우가 있는데 두 객체가 같은 객체이므로, 혼동하는 일이 없도록 하자.

  - 변수 객체 안에서 호출된 함수 인자는 각각의 프로퍼티가 만들어지고 그 값이 할당된다.

    - 만약 값이 넘겨지지 않았다면 undefined가 할당된다.
    - 예제 5-2에서 execute() 함수 안에 정의된 변수 a, b와 함수 func가 생성된다.

  - 여기서 주의할 점은 이 과정에서는 변수나 내부 함수를 단지 메모리에 생성(instantiation)하고, 초기화(initialization)는 각 변수나 함수에 해당하는 표현식이 실행되기 전까지는 이루어지지 않는다는 점이다.

    - 따라서 변수 a와 b에는 먼저 undefined가 할당된다. 
    - 표현식의 실행은 변수 객체 생성이 다 이루어진 후 시작된다.

    

    

    ![](https://docs.google.com/drawings/d/sat_YEGLcdhQkC1FVYgznCw/image?parent=e/2PACX-1vQiuXMoUJloXB41go11kimhAxCGB0Jhf5sdczvyrPIjH533basHPtLBcGPi59BcshFWUj6e55GqbGOO&rev=198&drawingRevisionAccessToken=8JBQllM_TFbCTA&h=398&w=335&ac=1)

  



- this 바인딩

  - 마지막 단계에서는 this 키워드를 사용하는 값이 할당된다.
  - 여기서 this가 참조하는 객체가 없으면 전역 객체를 참조한다.

  

  ![](https://docs.google.com/drawings/d/svPQVk38Yg3ai0RougO_9AQ/image?parent=e/2PACX-1vQiuXMoUJloXB41go11kimhAxCGB0Jhf5sdczvyrPIjH533basHPtLBcGPi59BcshFWUj6e55GqbGOO&rev=27&drawingRevisionAccessToken=buq1wmyILPe6Tg&h=401&w=335&ac=1)

  

- 코드 실행
  - 이렇게 하나의 실행 컨텍스트가 생성되고, 변수 객체가 만들어진 후에, 코드에 있는 여러 가지 표현식 실행이 이루어진다.
    - 이렇게 실행되면서 변수의 초기화 및 연산, 또 다른 함수 실행 등이 이루어진다.
    - 그림에서 undefined가 할당된 변수 a와 b에도 이 과정에서 1, 2의 값이 할당된다.
  - 참고로 전역 실행 컨텍스트는 일반적인 실행 컨텍스트와는 약간 다른데, arguments 객체가 없으며, 전역 객체 하나만을 포함하는 스코프 체인이 있다.
  - ECMAScript에서 언급된 바에 의하면, 실행 컨텍스트가 형성되는 세 가지 중 하나로서 전역 코드가 있는데, 이 전역 코드가 실행될 때 생성되는 컨텍스트가 전역 실행 컨텍스트다.
  - 전역 실행 컨텍스트는 변수를 초기화하고 이것의 내부 함수는 일반적인 탑 레벨의 함수로 선언된다. 그리고 전역 실행 컨텍스트의 변수 객체가 전역 객체로 사용된다.
    - 즉, 전역 실행 컨텍스트에서는 변수 객체가 곧 전역 객체이다.
    - 따라서 전역적으로 선언된 함수와 변수가 전역 객체의 프로퍼티가 된다.
  - 전역 실행 컨텍스트 역시, this를 전역 객체의 참조로 사용한다.

  











