# Chapter 05. 실행 컨텍스트와 클로저



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

  



## 5.3 스코프 체인

> 설명에 앞서, 여기서부터는 활성 객체와 변수 객체를 변수 객체라는 표현으로 통일한다. 두 객체가 사실 같은 객체이므로, 혼동이 없길 바란다.



- 이 절에서는 **5.2 실행 컨텍스트 생성 과정**에서 설명한 **스코프 체인**이 어떻게 만들어지는지 살펴본다.

  - 자바스크립트 코드를 이해하려면 스코프 체인의 이해는 필수적이다.
  - 이를 알아야, 변수에 대한 인식(Identifier Resolution) 메커니즘을 알 수 있고, 현재 사용되는 변수가 어디에서 선언된 변수인지 정확히 알 수 있기 때문이다.

- 자바스크립트도 다른 언어와 마찬가지로 **스코프**, 즉 유효 범위가 있다. 

  - 이 유효 범위 안에서 변수와 함수가 존재한다.

  - 다음은 C 언어의 유효 범위를 확인할 수 있는 코드이다.

    ```c
    void example_scope() {
        int i = 0;
        int value = 1;
        for(i = 0; i < 10; i++) {
            int a = 10;
        }
        printf("a : %d", a);  // 컴파일 에러
        
        if(i == 10) {
            int b = 20;
        }
        printf("b : %d", b);  // 컴파일 에러
        printf("value : %d", value);  // 1
    }
    ```

    - C 코드를 예로 들면, `{ }`로 묶여 있는 범위 안에서 선언된 변수는 블록이 끝나는 순간 사라지므로, 밖에서는 접근할 수 없다.

      - 특히, 함수의 `{ }` 뿐만 아니라 if, for 문의 `{ }`로 한 블록으로 묶여, 그 안에서 선언된 변수가 밖에서는 접근이 불가능하다.

    - 하지만 자바스크립트에서는 함수 내의 `{ }` 블록은, 이를테면 `for() {}`, `if() {}`와 같은 구문은 유효 범위가 없다.

      - **오직 함수만이 유효 범위의 한 단위가 된다.**

      - 이 유효 범위를 나타내는 스코프가 [[scope]] 프로퍼티로 각 함수 객체 내에서 연결 리스트 형식으로 관리되는데, 이를 **스코프 체인**이라고 한다.

      - 이러한 스코프 체인은 각 실행 컨텍스트의 변수 객체가 구성 요소인 리스트와 같다.

        

        ![](https://docs.google.com/drawings/d/sdOZUCi52XKRABd-wDZB6SQ/image?parent=e/2PACX-1vQiuXMoUJloXB41go11kimhAxCGB0Jhf5sdczvyrPIjH533basHPtLBcGPi59BcshFWUj6e55GqbGOO&rev=100&drawingRevisionAccessToken=mvtijQPr_xAaTA&h=273&w=211&ac=1)

      - **각각의 함수는 [[scope]] 프로퍼티로 자신이 생성된 실행 컨텍스트의 스코프 체인을 참조한다.**

      - 함수가 실행되는 순간 실행 컨텍스트가 만들어지고, **이 실행 컨텍스트는 실행된 함수의 [[scope]] 프로퍼티를 기반으로 새로운 스코프 체인을 만든다.**



- 전역 실행 컨텍스트의 스코프 체인

  - **[예제 5-3]**

    ```javascript
    var var1 = 1;
    var var2 = 2;
    console.log(var1);  // (출력값) 1
    console.log(var2);  // (출력값) 2
    ```

    - 위 예제 5-3은 전역 코드이다.

    - 함수가 선언되지 않아 함수 호출이 없고, 실행 가능한 코드들만 나열되어 있다.

    - 이 자바스크립트 코드를 실행하면, 먼저 전역 실행 컨텍스트가 생성되고, 변수 객체가 만들어진다.

    - 이 변수 객체의 **스코프 체인**은 어떻게 될까?

      - 현재 전역 실행 컨텍스트 단 하나만 실행되고 있어 참조할 상위 컨텍스트가 없다. 자신이 최상위에 위치하는 변수 객체인 것이다.

      - 따라서, 이 변수 객체의 스코프 체인은 자기 자신만을 가진다.

      - 다시 말해서, 변수 객체의 [[scope]]는 변수 객체 자신을 가리킨다.

      - 그 후, var1, var2 변수들이 생성되고 변수 객체에 의해 참조된다.

      - **5.2 실행 컨텍스트 생성 과정**에서 언급한대로 이 변수 객체가 곧 전역 객체가 된다.

        

        ![](https://docs.google.com/drawings/d/sg3Iyjr2pFy6CeY51s-9jMg/image?parent=e/2PACX-1vQiuXMoUJloXB41go11kimhAxCGB0Jhf5sdczvyrPIjH533basHPtLBcGPi59BcshFWUj6e55GqbGOO&rev=110&drawingRevisionAccessToken=G2G5rZAq1wcAQA&h=198&w=335&ac=1)



- 함수를 호출한 경우 생성되는 실행 컨텍스트의 스코프 체인

  - 예제 5-3에서 함수를 하나 생성해보자.

  - **[예제 5-4]**

    ```javascript
    var var1 = 1;
    var var2 = 2;
    function func() {
        var var1 = 10;
        var var2 = 20;
        console.log(var1);  // (출력값) 10
        console.log(var2);  // (출력값) 20
    }
    func();
    console.log(var1);    // (출력값) 1
    console.log(var2);    // (출력값) 2
    ```

    - 이 예제를 실행하면 전역 실행 컨텍스트가 생성되고, func() 함수 객체가 만들어진다. 

    - 이 함수 객체의 [[scope]]는 어떻게 될까?

      - 함수 객체가 생성될 때, 그 함수 객체의 [[scope]]는 현재 실행되는 컨텍스트의 변수 객체에 있는 [[scope]]를 그대로 가진다. 

      - 따라서, func 함수 객체의 [[scope]]는 전역 변수 객체가 된다.

      - 그렇다면 다음과 같이 func() 함수를 실행해보자.

        `func();`

      - 함수를 실행하였으므로 새로운 컨텍스트가 만들어진다. 이 컨텍스트를 편의상 func 컨텍스트라고 하겠다.

      - func 컨텍스트의 스코프 체인은 실행된 함수의 [[scope]] 프로퍼티를 그대로 복사한 후, 현재 생성된 변수 객체를 복사한 스코프 체인의 맨 앞에 추가한다.

      - func() 함수 객체의 [[scope]] 프로퍼티가 전역 객체 하나만을 가지고 있었으므로, func 실행 컨텍스트의 스코프 체인은 다음 그림과 같이 [func 변수 객체 - 전역 객체]가 된다.

        

      ![](https://docs.google.com/drawings/d/sgJWhjnXkRQrZfWT609b3JQ/image?parent=e/2PACX-1vQiuXMoUJloXB41go11kimhAxCGB0Jhf5sdczvyrPIjH533basHPtLBcGPi59BcshFWUj6e55GqbGOO&rev=260&drawingRevisionAccessToken=LRmCEugOsRiP_g&h=324&w=513&ac=1)

      

  - 스코프 체인 정리

    - **각 함수 객체는 [[scope]] 프로퍼티로 현재 컨텍스트의 스코프 체인을 참조한다.**

    - 한 함수가 실행되면 새로운 실행 컨텍스트가 만들어지는데, 이 새로운 실행 컨텍스트는 자신이 사용할 스코프 체인을 다음과 같은 방법으로 만든다.

      - **현재 실행되는 함수 객체의 [[scope]] 프로퍼티를 복사하고, 새롭게 생성된 변수 객체를 해당 체인의 제일 앞에 추가한다.**

    - 요약하면 스코프 체인은 다음과 같이 표현할 수 있다.

      **스코프 체인 = 현재 실행 컨텍스트의 변수 객체 + 상위 컨텍스트의 스코프 체인**

      

  - **[예제 5-5]**

    ```javascript
    var value = 'value1';
    
    function printFunc() {
        var value = 'value2';
    
        function printValue() {
            return value;
        }
    
        console.log(printValue());
    }
    
    printFunc();  // (출력값) value2
    ```

    

    ![](https://docs.google.com/drawings/d/e/2PACX-1vSupQD3UQkGFjFLhR0Px0kuDvwbDF_C3sQ8smjL8UZyNxnd0lfYODrh0h_fcv_89e9PryZSDmwhAhHZ/pub?w=1320&h=903)

    

  - **[예제 5-6]**

    ```javascript
    var value = 'value1';
    
    function printValue() {
        return value;
    }
    
    function printFunc(func) {
        var value = 'value2';
        console.log(func());
    }
    
    printFunc(printValue);  // (출력값) value1
    ```

    - 이 예제는 각 함수 객체가 처음 생성될 당시 실행 컨텍스트가 무엇인지를 생각해야 한다.

    - 각 함수 객체가 처음 생성될 때 [[scope]]는 전역 객체의 [[scope]]를 참조한다.

    - 따라서 각 함수가 실행될 때 생성되는 실행 컨텍스트의 스코프 체인은 전역 객체와 그 앞에 새롭게 만들어진 변수 객체가 추가된다.

    - 그래서 printValue()가 실행될 때의 스코프 체인은 예제 5-5와 다르게 만들어진다.

      

    ![](https://docs.google.com/drawings/d/e/2PACX-1vRcDtejuz2wO-kUOi5sVs35MDLgsb_EO4qJQ1HmvhIgFHn1ndgsqqA783KSjUJ5ANuQWSS8FvP0MJJe/pub?w=1324&h=906)



- 식별자 인식 (Indentifier resolution)
  - 지금까지 실행 컨텍스트가 만들어지면서 스코프 체인이 어떻게 형성되는지 살펴보았다.
  - 이렇게 만들어진 스코프 체인으로 **식별자 인식**이 이루어진다.
    - 식별자 인식은 스코프 체인의 첫 번째 변수 객체부터 시작한다.
      - 식별자와 대응되는 이름을 가진 프로퍼티가 있는지를 확인한다.
      - 함수를 호출할 때 스코프 체인의 가장 앞에 있는 객체가 변수 객체이므로, 이 객체에 있는 공식 인자, 내부 함수, 지역 변수에 대응되는지 먼저 확인한다.
    - 첫 번째 객체에 대응되는 프로퍼티를 발견하지 못하면, 다음 객체로 이동하여 찾는다.
    - 이런 식으로 대응되는 이름의 프로퍼티를 찾을 때까지 계속된다.
    - 여기서 this는 식별자가 아닌 키워드로 분류되므로, 스코프 체인의 참조 없이 접근할 수 있음을 기억하자.



- `with` 키워드

  - 자바스크립트에는 스코프 체인을 사용자가 임의로 수정하는 `with` 키워드가 있다.

    - `with`는 `eval`과 함께, 성능을 높이고자 하는 자바스크립트 프로그래머에게는 사용하지 말아야 할 키워드다.

  - `with` 구문은 표현식을 실행하는데, 표현식이 객체면 객체는 현재 실행 컨텍스트의 스코프 체인에 추가된다. (활성화 객체의 바로 앞에)

  - `with` 구문은 다른 구문(블록 구문일 수도 있음)을 실행하고 실행 컨텍스트의 스코프 체인을 전에 있던 곳에 저장한다.

  - 함수 선언은 `with` 구문의 영향을 받지 않고 함수 객체가 생성되지만, 함수 표현식은 안에서 `with` 구문과 함께 실행될 수 있다.

  - **[예제]**

    ```javascript
    var y = {
        x: 5
    };
    
    function withExamFunc() {
        var x = 10;
        var z;
        
        with(y) {
            z = function() {
                console.log(x);  // (출력값) 5. y 객체의 x가 출력된다.
            }
        }
        
        z();
    }
    
    withExamFunc();
    ```

    - withExamFunc() 함수가 호출되면 실행 컨텍스트는 전역 변수 객체와 현재 실행 컨텍스트의 변수 객체를 포함하는 스코프 체인이 있다.

    - 여기에, `with` 구문의 실행으로 전역 변수 y에 의해 참조되는 객체를 함수 표현식이 실행되는 동안 스코프 체인의 맨 앞에 추가한다.

      

      ![](https://docs.google.com/drawings/d/e/2PACX-1vRHWtbBntAYidgA0x5p7rng_5vg6d08aj0kkdhLmQv8udjAWS85j6TxU4jyVOxXpDGS8AMm7EVvkUzI/pub?w=1351&h=909)

      

    - 따라서 앞 예제의 결과값은 y 객체 안에 정의된 x의 값인 5를 출력한다.



- 호이스팅

  - 여기까지 이해했으면 **4.1.5 함수 호이스팅**에서 소개한 호이스팅의 원인을 이해할 수 있어야 한다.

  - **[예제]**

    ```javascript
    foo();
    bar();
    
    var foo = function () {
        console.log('foo and x = ' + x);
    };
    
    function bar() {
        console.log('bar and x = ' + x);
    }
    
    var x = 1;
    
    --------------------------------------------------------------
    위 예제는 아래와 같다.
    
    var foo;
    function bar() {
        console.log('bar and x = ' + x);
    }
    
    var x;
    foo();  // TypeError
    bar();
    
    foo = function () {
        console.log('foo and x = ' + x);
    };
    
    x = 1;
    ```

    - 함수 생성 과정에서 변수 foo, 함수 객체 bar, 변수 x를 차례대로 생성한다.
    - foo와 x에는 undefined가 할당된다.
    - 실행이 시작되면 foo(), bar()를 연속해서 호출하고 foo에 함수 객체의 참조가 할당되며, 변수 x에 1이 할당된다.
    - 결국, foo()에서 'TypeError'가 발생한다. 
      - foo가 선언되어 있기는 하지만 함수가 아니기 때문이다.
    - foo()를 주석 처리한 후 실행하면 bar()에서는 'bar and x = undefined'가 출력된다.
      - x에 1이 할당되기 전에 실행했기 때문이다.

    





## 5.4 클로저

- 클로저의 개념

  - **[예제 5-7]**

    ```javascript
    function outerFunc() {
        var x = 10;
        var innerFunc = function() {
            console.log(x);
        }
        return innerFunc;
    }
    
    var inner = outerFunc();
    inner();
    ```

    ![](https://docs.google.com/drawings/d/e/2PACX-1vSneBaIzgxzVGL7sCHCi4PdQGSz2pbCFFKBF47da-ZqoUpHDdAKwqY0aTRiIHlZWkdXyn6s0gNigI-7/pub?w=1327&h=1081)

    

    - `innerFunc()`의 [[scope]]은 outerFunc 변수 객체와 전역 객체를 가진다. 
    - 그런데 여기서 잠깐 혼란스러운 부분이 있다.
      - 예제 5-7에서 `innerFunc()`은 `outerFunc()`의 실행이 끝난 후 실행된다.
      - 그렇다면 `outerFunc()` 실행 컨텍스트가 사라진 이후에 innerFunc 실행 컨텍스트가 생성되는 것인데, `innerFunc()`의 스코프 체인은 outerFunc 변수 객체를 여전히 참조할 수 있을까?
      - 예제 5-7의 결과를 보면 짐작할 수 있겠지만, outerFunc 실행 컨텍스트는 사라졌지만, outerFunc 변수 객체는 여전히 남아있고, innerFunc의 스코프 체인으로 참조되고 있다.
      - 이것이 바로 자바스크립트에서 구현한 **클로저**라는 개념이다.

  

  - 4장에서 보았듯이, 자바스크립트의 함수는 일급 객체로 취급된다.
    - 이는 함수를 다른 함수의 인자로 넘길 수도 있고,
    - return으로 함수를 통째로 반환받을 수도 있음을 의미한다.
    - 이러한 기능으로 앞 예제와 같은 코드가 가능하다.
  - 여기서 최종 반환되는 함수가 외부 함수의 지역변수에 접근하고 있다는 것이 중요하다. 
    - 이 지역변수에 접근하려면, 함수가 종료되어 외부 함수의 컨텍스트가 반환되더라도 변수 객체는 반환되는 내부 함수의 스코프 체인에 그대로 남아있어야만 접근할 수 있다.
    - 이것이 바로 **클로저**다.
    - 클로저를 조금 쉽게 풀어서 정의하면, **이미 생명 주기가 끝난 외부 함수의 변수를 참조하는 함수를 클로저라고 한다.**
    - 앞 예제에서는 outerFunc에서 선언된 x를 참조하는 innerFunc가 클로저가 된다.
    - 그리고 클로저로 참조되는 외부 변수 즉, outerFunc의 x와 같은 변수를 **자유 변수**(Free Variable)라고 한다.
    - closure라는 이름은 함수가 자유 변수에 대해 닫혀있다(closed, bound)는 의미인데, 우리말로 의역하면 '자유 변수에 엮여있는 함수'라는 표현이 맞을 듯하다.

  

  - ![](https://docs.google.com/drawings/d/szxITl2h7KPchimvKoH-C9w/image?parent=e/2PACX-1vQiuXMoUJloXB41go11kimhAxCGB0Jhf5sdczvyrPIjH533basHPtLBcGPi59BcshFWUj6e55GqbGOO&rev=310&drawingRevisionAccessToken=nTkrzY1i10PaWA&h=356&w=570&ac=1)

    - 위 그림은 자바스크립트로 클로저를 구현하는 전형적인 패턴이다.
      - 그림에서 보듯이 외부 함수의 호출이 이루어지고, 이 외부 함수에서 새로운 함수가 반환된다.
      - 반환된 함수가 클로저이고 이 클로저는 자유 변수를 묶고 있다.
      - 반환된 클로저는 새로운 함수로 사용된다.
    - 대부분의 클로저를 활용하는 코드가 이와 같은 형식을 유지한다.
    - 이러한 특성을 바탕으로 자바스크립트를 이용한 함수형 프로그래밍이 가능하다.

  - 클로저를 이해했다면 다음 예제를 보고 결과값을 예측해보자.

  - **[예제 5-8]**

    ```javascript
    function outerFunc(arg1, arg2) {
        var local = 8;
        function innerFunc(innerArg) {
            console.log((arg1 + arg2) / (innerArg + local));
        }
    
        return innerFunc;
    }
    
    var exam1 = outerFunc(2, 4);
    exam1(2);
    ```

    - 앞 예제에서는 `outerFunc()` 함수를 호출하고 반환되는 함수 객체인 `innerFunc()`가 exam1으로 참조된다. 이것은 exam1(n)의 형태로 실행될 수 있다.
    
    - 여기서 `outerFunc()`가 실행되면서 생성되는 변수 객체가 스코프 체인에 들어가게 되고, 이 스코프 체인은 innerFunc의 스코프 체인으로 참조된다.
    
      - 즉, `outerFunc()` 함수가 종료되었지만, 여전히 내부 함수(`innerFunc()`)의 [[scope]]으로 참조되므로 가비지 컬렉션의 대상이 되지 않고, 여전히 접근 가능하게 살아있다.
      - 따라서 이후에 exam1(n)을 호출하여도, `innerFunc()`에서 참조하고자 하는 변수 local에 접근할 수 있다. 
      - 클로저는 이렇게 만들어진다.
      - 이 outerFunc 변수 객체의 프로퍼티값은 여전히 (심지어 실행 컨텍스트가 끝났음에도) 읽기 및 쓰기까지 가능하다.
    
    - 앞 예제의 스코프 체인을 그림으로 나타내면 다음과 같다.
    
      ![](https://docs.google.com/drawings/d/e/2PACX-1vSDHf4OjA3w2H21viYJ9oAlXHBRX02AdXbQNLvmsvtRohfMxt3kMAfxAiaLVByrDYrWTu2fuIm6CmgQ/pub?w=1194&h=1143)
    
      - 따라서 `exam1(2)`를 호출하면, arg1, arg2, local 값은 outerFunc 변수 객체에서 찾고, innerArg는 innerFunc 변수 객체에서 찾는다.
      - 결과는 ((2 + 4) / (2 + 8))가 된다.
    
      > 위 그림을 보듯이 `innerFunc()`에서 접근하는 변수 대부분이 스코프 체인의 첫 번째 객체가 아닌 그 이후의 객체에 존재한다.
      >
      > 이는 성능 문제를 유발시킬 수 있는 여지가 있다.
      >
      > 대부분의 클로저에서는 스코프 체인에서 뒤쪽에 있는 객체에 자주 접근하므로, 성능을 저하시키는 이유로 지목되기도 한다.
      >
      > 게다가 앞에서 알아본 대로 클로저를 사용한 코드가 그렇지 않은 코드보다 메모리 부담이 많아진다.
      >
      > 그렇다고 클로저를 쓰지 않는 것은 자바스크립트의 강력한 기능 하나를 무시하고 사용하는 것과 다름이 없다.
      >
      > 따라서 클로저를 영리하게 사용하는 지혜가 필요하며, 이를 위해선 많은 프로그래밍 경험을 쌓아야 한다.



- 클로저의 활용

  - 클로저는 성능적인 면과 자원적인 면에서 약간 손해를 볼 수 있으므로 무차별적으로 사용하면 안 된다.

  - 이 장에서는 아주 전형적인 클로저의 예제 코드를 소개한다.

  - 7장 함수형 프로그래밍에서 소개할 대부분의 예제가 클로저를 활용한 것이므로 참고하길 바란다.

  - **특정 함수에 사용자가 정의한 객체의 메서드 연결하기**

    - **[예제 5-9]**

      ```javascript
      function HelloFunc(func) {
          this.greeting = "hello";
      }
      
      HelloFunc.prototype.call = function (func) {
          func ? func(this.greeting) : this.func(this.greeting);
      };
      
      var userFunc = function (greeting) {
          console.log(greeting);
      };
      
      var objHello = new HelloFunc();
      objHello.func = userFunc;
      objHello.call();
      ```

      - 함수 HelloFunc는 greeting 변수가 있고, func 프로퍼티로 참조되는 함수를 `call()` 함수로 호출한다.
      - 사용자는 func 프로퍼티에 자신이 정의한 함수를 참조시켜 호출할 수 있다.
      - 다만, `HelloFunc.prototype.call()`을 보면 알 수 있듯이 자신의 지역 변수인 greeting만을 인자로 사용자가 정의한 함수에 넘긴다.
      - 앞 예제에서 사용자는 `userFunc()` 함수를 정의하여 `HelloFunc.func()`에 참조시킨 뒤, `HelloFunc()`의 지역 변수인 greeting을 화면에 출력시킨다.

    - 위 예제에서 `HelloFunc()`는 greeting만을 인자로 넣어 사용자가 인자로 넘긴 함수를 실행시킨다.

    - 그래서 사용자가 정의한 함수도 한 개의 인자를 받는 함수를 정의할 수밖에 없다. 

    - 여기서 사용자가 원하는 인자를 더 넣어서 `HelloFunc()`를 이용하여 호출하려면 어떻게 해야 할까?

    - **[예제 5-10]**

      ```javascript
      function HelloFunc(func) {
          this.greeting = "hello";
      }
      
      HelloFunc.prototype.call = function (func) {
          func ? func(this.greeting) : this.func(this.greeting);
      };
      
      var userFunc = function (greeting) {
          console.log(greeting);
      };
      
      var objHello = new HelloFunc();
      objHello.func = userFunc;
      
      // 여기서부터 새로운 코드
      
      function saySomething(obj, methodName, name) {
          return (function (greeting) {
              return obj[methodName](greeting, name);
          });
      }
      
      function newObj(obj, name) {
          obj.func = saySomething(this, 'who', name);
          return obj;
      }
      
      newObj.prototype.who = function (greeting, name) {
          console.log(greeting + ' ' + (name || 'everyone'));
      }
      
      var obj1 = new newObj(objHello, 'zzoon');
      obj1.call();
      ```

      - 예제 5-10에서 새로운 함수 `newObj()`를 선언하였다.

        - 이 함수는 `HelloFunc()`의 객체를 좀 더 자유롭게 활용하려거 정의한 함수다.
        - 첫 번째 인자로 받는 obj는 `HelloFunc()`의 객체가 되고,
        - 두 번째 인자는 사용자가 출력을 원하는 사람 이름이 된다.

      - `var obj1 = new newObj(objHello, 'zzoon');`

        - 위 코드로 다음 코드가 실행된다.

        - ```javascript
          obj.func = saySomething(this, 'who', name);
          return obj;
          ```

        - 첫 번째 인자 obj의 func 프로퍼티에 `saySomething()` 함수에서 반환되는 함수를 참조하고, 반환한다.

        - 결국 obj1은 인자로 넘겼던 objHello 객체에서 func 프로퍼티에 참조된 함수만 바뀐 객체가 된다.

        - 따라서 다음과 같이 호출할 수 있다. 

          `obj1.call();`

        - 이 코드의 실행결과, newObj.prototype.who 함수가 호출되어 사용자가 원하는 결과인 "hello zzoon"을 출력한다.

      - 그렇다면 `saySomething()` 함수 안에서 어떤 작업이 수행되는 지 살펴보자.

        ```javascript
        function saySomething(obj, methodName, name) {
            return (function (greeting) {
                return obj[methodName](greeting, name);
            });
        }
        ```

        - 첫 번째 인자: newFunc 객체 - obj1

        - 두 번째 인자: 사용자가 정의한 메서드 이름 - 'who'

        - 세 번재 인자: 사용자가 원하는 사람 이름값 - 'zzoon'

        - 반환: 사용자가 정의한 `newFunc.prototype.who()` 함수를 반환하는 `helloFunc()`의 func 함수

          

        - 이렇게 반환되는 함수가 HelloFunc이 원하는 `function(greeting) {}` 형식의 함수가 되는데, 이것이 HelloFunc 객체의 func로 참조된다.

        - `obj1.call()`로 실행되는 것은 실질적으로 `newFunc.prototype.who()`가 된다.

        - 이와 같은 방식으로 사용자는 자신의 객체 메서드인 who 함수를 HelloFunc에 연결시킬 수 있다.

        - 여기서 클로저는 `saySomething()`에서 반환되는 `function(greeting) {}`이 되고, 이 클로저는 자유 변수 obj, methodName, name을 참조한다.

      

    - 위 예제 5-10은 정해진 형식의 함수를 콜백해주는 라이브러리가 있을 경우, 그 정해진 형식과는 다른 형식의 사용자 정의 함수를 호출할 때 유용하게 사용된다.

      - 예를 들어 브라우저에서는 onclick.onmouseover와 같은 프로퍼티에 해당 이벤트 핸들러를 사용자가 정의해 놓을 수가 있는데, 이 이벤트 핸들러의 형식은 `function(event) {}`이다. 
      - 이를 통해 브라우저는 발생한 이벤트를 event 인자로 사용자에게 넘겨주는 방식이다.
      - 여기에 event 외의 원하는 인자를 더 추가한 이벤트 핸들러를 사용하고 싶을 때, 앞과 같은 방식으로 클로저를 적절히 활용할 수 있다.



- 함수의 캡슐화

  - 다음과 같은 함수를 작성한다고 가정해보자.

    **"I am XXX. I live in XXX. I'am XX years old"라는 문장을 출력하는데, XX 부분은 사용자에게 인자로 입력 받아 값을 출력하는 함수**

  - 가장 먼저 생각할 수 있는 것은 앞 문장 템플릿을 전역 변수에 저장하고, 사용자의 입력을 받은 후, 이 전역 변수에 접근하여 완성된 문장을 출력하는 방식으로 함수를 작성하는 것이다.

  - **[예제 5-11]**

    ```javascript
    var buffAr = [
        'I am ',
        '',
        '. I live in ',
        '',
        '. I\'am ',
        '',
        ' years old.',
    ];
    
    function getCompletedStr(name, city, age) {
        buffAr[1] = name;
        buffAr[3] = city;
        buffAr[5] = age;
    
        return buffAr.join('');
    }
    
    var str = getCompletedStr('zzoon', 'seoul', 16);
    console.log(str);
    ```

    - 위 방식은 단점이 있다.

    - 바로 buffAr라는 배열은 전역 변수로서, 외부에 노출되어 있다는 점이다.

      - 이는 다른 함수에서 이 배열에 쉽게 접근하여 값을 바꿀 수도 있고, 실수로 같은 이름의 변수를 만들어 버그가 생길 수도 있다.

        

  - 앞 예제의 경우, 클로저를 활용하여 buffAr을 추가적인 스코프에 넣고 사용하게 되면, 이 문제를 해결할 수 있다.

  - **[예제 5-12]**

    ```javascript
    var getCompletedStr = (function() {
        var buffAr = [
            'I am ',
            '',
            '. I live in ',
            '',
            '. I\'am ',
            '',
            ' years old.',
        ];
    
        return (function (name, city, age) {
            buffAr[1] = name;
            buffAr[3] = city;
            buffAr[5] = age;
    
            return buffAr.join('');
        });
    })();
    
    var str = getCompletedStr('zzoon', 'seoul', 16);
    console.log(str);
    ```

    - 위 예제에서 가장 먼저 주의해서 봐야 할 점은 변수 getCompletedStr에 익명의 함수를 즉시 실행시켜 반환되는 함수를 할당하는 것이다.

    - 이 반환되는 함수가 클로저가 되고, 이 클로저는 자유 변수 buffAr을 스코프 체인에서 참조할 수 있다.

    - 클로저에 있는 스코프 체인을 그림으로 그려보면 다음과 같다.

      ![](https://docs.google.com/drawings/d/e/2PACX-1vRiSeG_FziOIw93pxMKw1o_nXDtzoc2E4sdcGQgE5YRCgYA_bRShMdHwOOX7Gn6wfw4AlP2dE32g8ir/pub?w=1225&h=1134)



- setTimeout()에 지정되는 함수의 사용자 정의



