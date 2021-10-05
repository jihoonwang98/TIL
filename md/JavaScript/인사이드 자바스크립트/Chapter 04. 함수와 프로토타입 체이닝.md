## Chapter 04. 함수와 프로토타입 체이닝



### 목차

- 4.1 함수 정의
  - 함수 리터럴
  - 함수 선언문 방식으로 함수 생성하기
  - 함수 표현식 방식으로 함수 생성하기
  - Function() 생성자 함수를 통한 함수 생성하기
  - 함수 호이스팅
- 4.2 함수 객체: 함수도 객체다
  - 자바스크립트에서는 함수도 객체다
  - 자바스크립트에서 함수는 값으로 취급된다
  - 함수 객체의 기본 프로퍼티
- 4.3 함수의 다양한 형태
  - 콜백 함수
  - 즉시 실행 함수
  - 내부 함수
  - 함수를 리턴하는 함수
- 4.4 함수 호출과 this
  - arguments 객체
  - 호출 패턴과 this 바인딩
  - 함수 리턴
- 4.5 프로토타입 체이닝
  - 프로토타입의 두 가지 의미
  - 객체 리터럴 방식으로 생성된 객체의 프로토타입 체이닝
  - 생성자 함수로 생성된 객체의 프로토타입 체이닝
  - 프로토타입 체이닝의 종점
  - 기본 데이터 타입 확장
  - 프로토타입도 자바스크립트 객체다
  - 프로토타입 메서드와 this 바인딩
  - 디폴트 프로토타입은 다른 객체로 변경이 가능하다
  - 객체의 프로퍼티 읽기나 메서드를 실행할 때만 프로토타입 체이닝이 동작한다.



## 4.1 함수 정의

- 함수를 생성하는 3가지 방법
  - 함수 선언문 (function statement)
  - 함수 표현식 (function expression)
  - Function() 생성자 함수



- 함수 리터럴

  - 자바스크립트에서는 함수도 일반 객체처럼 값으로 취급된다.

  - 때문에 객체 리터럴 방식으로 일반 객체를 생성할 수 있는 것처럼, 자바스크립트에서는 **함수 리터럴**을 이용해 함수를 생성할 수 있다.

  - 실제로 함수 선언문이나 함수 표현식 방법 모두 이런 함수 리터럴 방식으로 함수를 생성한다.

  - **[예시]** 함수 리터럴을 통한 add() 함수 정의

    ```javascript
    function add(x, y) {
    	return x + y;
    }
    ```

    - 함수명은 선택 사항이다. 자바스크립트에서는 함수명이 없는 함수를 익명 함수라고 한다.
    - 자바스크립트에서는 매개변수 타입을 기술하지 않는다.

    

- 함수 선언문 방식으로 함수 생성하기

  - 함수 선언문 방식은 앞에서 설명한 함수 리터럴 형태와 같다.

  - 주의할 점은 함수 선언문 방식으로 정의된 함수의 경우는 **반드시 함수명이 정의되어 있어야 한다**는 것이다.

  - **[예제 4-1]** add() 함수 생성 (함수 선언문 방식)

    ```javascript
    // add() 함수 선언문
    function add(x, y) {
    	return x + y;
    }
    
    console.log(add(3, 4));  // (출력값) 7
    ```

    

- 함수 표현식 방식으로 함수 생성하기

  - 자바스크립트에서는 함수도 하나의 값처럼 취급된다.

    - 따라서 함수도 숫자나 문자열처럼 변수에 할당되는 것이 가능하다.

  - 이런 방식으로 함수 리터럴로 하나의 함수를 만들고, 여기서 생성된 함수를 변수에 할당하여 함수를 생성하는 것을 **함수 표현식**이라고 말한다.

    

  - **[예제 4-2]** add() 함수 생성 (함수 표현식 방식)

    ```javascript
    // add() 함수 표현식
    var add = function (x, y) {
    	return x + y;
    };
    
    var plus = add;
    
    console.log(add(3,4));
    console.log(plus(5,6));
    ```

    - 여기서 함수 리터럴로 생성한 함수는 함수명이 없으므로 익명 함수이다.
    - 위 코드에서 알 수 있듯이 함수 표현식은 함수 선언문 문법과 거의 유사하다. 유일한 차이점은 함수 표현식 방법에서는 함수 이름이 선택 사항이며, 보통 사용하지 않는다는 것이다.

    - 위 방식은 **익명 함수를 이용한 함수 표현식 방법(익명 함수 표현식)**이다.

    - 이러한 익명 함수의 호출은 앞 예제와 같이 함수 변수에 함수 호출 연산자인 ()를 붙여서 기술하는 것으로 가능하다.

      

  - **[예제 4-3]** 기명 함수 표현식의 함수 호출 방법

    ```java
    var add = function sum(x, y) {
        return x + y;
    };
    
    console.log(add(3, 4)); // (출력값) 7
    console.log(sum(3, 4)); // ReferenceError: sum is not defined
    ```

    - 위와 같이 함수 이름이 포함된 함수 표현식을 **기명 함수 표현식**이라고 한다.

    - sum(3,4) statement는 에러가 발생하는데, 이것은 함수 표현식에서 사용된 함수명이 외부 코드에서 접근이 불가능하기 때문이다.

      - 함수 표현식에 사용된 함수명은 정의된 함수 내부에서 해당 함수를 재귀적으로 호출하거나, 디버거 등에서 함수를 구분할 때 사용된다.

        

  - 예제 4-1에서 함수 표현식이 아닌 함수 선언문으로 정의한 add() 함수는 어떻게 함수명으로 함수 외부에서 호출이 가능할까?

    - 함수 선언문 형식으로 정의된 add() 함수는 자바스크립트 엔진에 의해 다음과 같은 함수 표현식 형태로 변경되기 때문이다.

      ```javascript
      var add = function add(x, y) {
      	return x + y;
      };
      ```

    - 함수명과 함수 변수의 이름이 add로 같으므로, 함수명으로 함수가 호출되는 것처럼 보이지만, 실제로는 add 함수 변수로 함수 외부에서 호출이 가능하게 된 것이다.

    

  - **[예제 4-4]** 함수 표현식 방식으로 구현한 팩토리얼 함수

    ```javascript
    var factorialVar = function factorial(n) {
        if (n <= 1) {
            return 1;
        }
        return n * factorial(n - 1);
    };
    
    console.log(factorialVar(5)); // (출력값) 120
    console.log(factorial(3)); // ReferenceError: factorial is not defined
    ```






- 함수 선언문(function statement)와 함수 표현식(function expression)에서의 세미콜론

  - 함수 선언문 방식: 세미콜론 붙이지 않는다.
  - **<u>함수 표현식 방식: 세미콜론 붙인다.</u>**

  - 자바스크립트는 C와 같이 세미콜론 사용을 강제하지는 않는다.

    - 인터프리터가 자동으로 세미콜론을 삽입시켜 주기 때문
    - 그렇다고 해서 세미콜론 사용을 신경 쓰지 않는다면, 심각한 디버깅 상황에 직면할 수 있다.

  - **[예제]**

    ```javascript
    var func = function() {
        return 42;
    }   // 세미콜론을 사용하지 않음
    (function() {
        console.log("function called");
    })();
    ```

    - 이 예제 코드의 의도는 단순히 42를 리턴하는 func() 함수를 정의하고, 그런 다음 즉시 실행 함수 형식으로 "function called"를 출력하려고 했다.
    - 하지만 이 코드는 세미콜론을 사용하지 않았으므로, 실제로 func() 함수를 호출하면 의도와는 다르게 'number is not a function'이라는 에러가 발생한다.
    - 그 이유는 자바스크립트 파서가 func()의 함수 정의에서 세미콜론을 사용하지 않아, `return 42;` 문장을 지나 func()의 함수 정의 끝에 있는 중괄호만으로 func() 함수가 끝났다고 판단하지 않기 때문이다. 그리고 자바스크립트 파서는 이후에 괄호에 둘러싸여 정의된 즉시 실행 함수를 보고 이를 마치 func() 함수 호출 연산으로 생각해서 func() 함수를 호출해 버린다. 그렇기 때문에 func() 함수가 호출되면 42가 반환되고, 즉시 실행 함수를 실행하려고 남겨둔 마지막 () 괄호가 있으므로 `42();` 형태로 또다시 함수를 호출하려고 시도한다. 그러나 42는 숫자이지 함수가 아니므로 'number is not a function' 에러가 발생하게 되는 것이다.



- Function() 생성자 함수를 통한 함수 생성하기

  - 자바스크립트의 함수도 Function()이라는 기본 내장 생성자 함수로부터 생성된 **객체**라고 볼 수 있다.

    - 앞에서 설명한 함수 선언문이나 함수 표현식 또한 Function() 생성자 함수가 아닌 함수 리터럴 방식으로 함수를 생성하지만, 결국엔 이 또한 내부적으로는 Function() 생성자 함수로 함수가 생성된다고 볼 수 있다.

  - Function() 생성자 함수로 함수를 생성하는 문법

    ```javascript
    new Function(arg1, arg2, ... argN, functionBody)
    - arg1, arg2, ..., argN: 함수의 매개변수
    - functionBody: 함수가 호출될 때 실행될 코드를 포함한 "문자열"
    ```

  - **[예제 4-5]** Function() 생성자 함수를 이용한 add() 함수 생성

    ```javascript
    var add = new Function('x', 'y', 'return x + y');
    console.log(add(3, 4));
    ```

  - 일반적으로 Function() 생성자 함수를 사용한 함수 생성 방법은 자주 사용되지 않는다. 상식 수준으로 알아두자.





- 함수 호이스팅

  - 지금까지 살펴본 함수 생성 방법 3가지는 코드가 약간씩 다르지만 모두 같은 기능의 함수를 생성함을 확인할 수 있다. 하지만 이들 사이에는 동작 방식이 약간 차이가 있다.

    - 그 중의 하나가 바로 **함수 호이스팅**(Function Hoisting)이다.

  - 자바스크립트 Guru로 알려진 더글라스 크락포드는 **함수 생성에 있어서 함수 표현식만 사용**할 것을 권하고 있다. 그 이유 중의 하나가 바로 함수 호이스팅 때문이다.

  - **[예제 4-6]** 함수 선언문 방식과 함수 호이스팅

    ```javascript
    console.log(add(2, 3));  // 5
    
    // 함수 선언문 형태로 add() 함수 정의
    function add(x, y) {
        return x + y;
    }
    
    console.log(add(3, 4));  // 7
    ```

    - `add(2, 3)`을 호출할 때 `add()` 함수가 정의되지 않았음에도 함수를 호출하는 것이 가능하다. 이것은 함수가 자신이 위치한 코드에 상관없이 **함수 선언문 형태로 정의한 함수의 유효 범위는 코드의 맨 처음부터 시작한다**는 것을 알 수 있다. 이것을 **함수 호이스팅**이라고 부른다.

  - 더글라스 크락포드는 이러한 함수 호이스팅은 함수를 사용하기 전에 반드시 선언해야 한다는 규칙을 무시하므로 코드의 구조를 엉성하게 만들 수도 있다고 지적하며, 함수 표현식 사용을 권장한다.

  - **[예제 4-7]** 함수 표현식 방식과 함수 호이스팅

    ```javascript
    console.log(add(2, 3));  // TypeError: add is not a function
    
    // 함수 표현식 형태로 add() 함수 정의
    var add = function add(x, y) {
        return x + y;
    }
    
    console.log(add(3, 4));  // 7
    ```

    - 함수 표현식 형태로 작성된 함수는 함수 호이스팅이 일어나지 않는다.

  - 이러한 함수 호이스팅이 발생하는 원인은 자바스크립트의 **변수 생성**(Instantiation)과 **초기화**(Initialization)의 작업이 분리돼서 진행되기 때문이다.

    - 자세한 내용은 5장에서 다뤄본다.





## 4.2 함수 객체: 함수도 객체다

- 자바스크립트에서는 함수도 객체다.

  - JS에서는 함수의 기본 기능인 코드 실행뿐만 아니라, <u>함수 자체가 일반 객체처럼 프로퍼티들을 가질 수 있다</u>.

  - **[예제 4-8]** add() 함수도 객체처럼 프로퍼티를 가질 수 있다.

    ```javascript
    // 함수 선언 방식으로 add() 함수 정의
    function add(x, y) {
    	return x + y;
    } ---- (1)
    
    // add() 함수 객체에 result, status 프로퍼티 추가
    add.result = add(3, 2); ---- (2)
    add.status = 'OK'; 
    
    console.log(add.result);
    console.log(add.status);
    ```

    - (1): add() 함수를 생성할 때 함수 코드는 함수 객체의 **[[Code]] 내부 프로퍼티**에 자동으로 저장된다.
    - (2): add() 함수에 마치 일반 객체처럼 result 프로퍼티를 동적으로 생성하고, 여기에 add() 함수를 호출한 결과를 저장한 것을 확인할 수 있다.



- 자바스크립트에서 함수는 값으로 취급된다.
  - JS에서 함수는 객체다. 이는 **함수도 일반 객체처럼 취급될 수 있다**는 것을 말한다. 때문에 자바스크립트 함수는 다음과 같은 동작이 가능하다.
    - 리터럴에 의해 생성
    - 변수나 배열의 요소, 객체의 프로퍼티 등에 할당 가능
    - 함수의 인자로 전달 가능
    - 함수의 리턴값으로 리턴 가능
    - 동적으로 프로퍼티를 생성 및 할당 가능
  - 이와 같은 특징이 있으므로 자바스크립트에서는 함수를 **일급(First Class) 객체**라고 부른다.
    - 일급 객체란 위에서 나열한 기능이 모두 가능한 객체를 칭한다.
    - JS에서는 함수가 이러한 일급 객체의 특성을 가지므로 함수형 프로그래밍이 가능하다.
  - JS 함수의 기능은 C나 자바와 같은 다른 언어 함수의 기능과 거의 비슷하다. 입력한 값을 받아 처리한 다음 그 결과를 반환하는 구조다.
    - 하지만 이러한 기본적인 기능 외에도 JS에서 함수를 제대로 이해하려면 함수가 일급 객체이며 이는 곧 <u>함수가 일반 객체처럼 값(Value)으로 취급된다</u>는 것을 이해해야 한다.



- 변수나 프로퍼티의 값으로 할당

  - 함수는 숫자나 문자열처럼 변수나 프로퍼티의 값으로 할당될 수 있다.

  - **[예제 4-9]** 변수나 프로퍼티에 함수값을 할당하는 코드

    ```javascript
    // 변수에 함수 할당
    var foo = 100;
    var bar = function () { return 100; };
    console.log(bar());
    
    // 프로퍼티에 함수 할당
    var obj = {};
    obj.baz = function () { return 200; };
    console.log(obj.baz());
    ```

    

- 함수 인자로 전달

  - 함수는 다른 함수의 인자로도 전달이 가능하다.

  - **[예제 4-10]** 함수를 다른 함수의 인자로 넘긴 코드

    ```javascript
    // 함수 표현식으로 foo() 함수 생성
    var foo = function(func) {
    	func(); // 인자로 받은 func() 함수 호출
    };
    
    // foo() 함수 실행
    foo(function() {
        console.log('Function can be used as the argument');
    });
    ```



- 리턴값으로 활용

  - 함수는 다른 함수의 리턴값으로도 활용할 수 있다.

  - **[예제 4-11]** 함수를 다른 함수의 리턴값으로 활용한 코드

    ```javascript
    // 함수를 리턴하는 foo() 함수 정의
    var foo = function() {
    	return function () {
            console.log('this function is the return value.');
        };
    };
    
    var bar = foo();
    bar();
    ```



- 함수 객체의 기본 프로퍼티

  - 자바스크립트에서는 함수 역시 객체다.

    - 이것은 함수 역시 일반적인 객체의 기능에 추가로 호출됐을 때 정의된 코드를 실행하는 기능을 가지고 있다는 것이다.
    - 또한, 일반 객체와는 다르게 추가로 **함수 객체만의 표준 프로퍼티**가 정의되어 있다.

  - **[예제 4-12]** add() 함수 객체 프로퍼티를 출력하는 코드

    ```javascript
    function add(x, y) {
    	return x + y;
    }
    
    console.dir(add);
    ```

    ![](https://lh4.googleusercontent.com/eXxDUAej3piITOgbpkr2soRzTiAGnyicRcMiP6eYAko7bKAOawWpkyXW_cvMjySyUtDbYuxS3GNaSSLBS8Iuxg4V3gacO7DyMHhz0yb0uicLS5rmxdhBY6a8kdMv3SKaq2o9CKNv=s0)

    - ECMA5 스크립트 명세서에는 모든 함수가 **length**와 **prototype** 프로퍼티를 가져야 한다고 기술하고 있다.
    - name 프로퍼티는 함수의 이름을 나타낸다.
    - caller 프로퍼티는 자신을 호출한 함수를 나타낸다.
    - arguments 프로퍼티는 함수를 호출할 때 전달된 인자값을 나타낸다.
    - [[Prototype]] 프로퍼티는 자신의 부모 역할을 하는 프로토타입 객체를 가리킨다.
      - 함수 객체의 부모 역할을 하는 프로토타입 객체를 **Function.prototype** 객체라고 하며, 이것 역시 **함수 객체**라고 정의한다.



- Function.prototype 객체
  - 앞 설명에서 '모든 함수들의 부모 객체는 Function Prototype 객체'라고 했다. 그런데 ECMAScript 명세서에는 Function.prototype은 함수라고 정의하고 있다. 그렇다면 이런 규칙에 의해 Function.prototype 함수 객체도 결국 함수이므로 Function Prototype 객체, 즉, '자기 자신을 부모가 갖는 것인가?'라는 의문이 생길 수 있다.
    - ECMAScript 명세서에는 예외적으로 Function.prototype 함수 객체의 부모는 자바스크립트의 모든 객체의 조상격인 **Object.prototype** 객체라고 설명하고 있다.
  - Function.prototype 객체는 모든 함수들의 부모 역할을 하는 프로토타입 객체다.
    - 때문에 모든 함수는 Function Prototype 객체가 갖고 있는 프로퍼티나 메서드를 마치 자신의 것처럼 상속받아 그대로 사용할 수 있다.
  - Function.prototype 객체가 가져야 하는 프로퍼티들은 아래와 같다.
    - constructor 프로퍼티
    - toString() 메서드
    - **apply(thisArg, argArray) 메서드**
    - **call(thisArg, [, arg1 [, arg2, ]]) 메서드**
    - bind(thisArg, [, arg1 [, arg2 ,]]) 메서드



- length 프로퍼티

  - 함수 객체의 length 프로퍼티는 앞서 설명했듯 ECMAScript에서 정한 모든 함수가 가져야 하는 표준 프로퍼티로서, **<u>함수가 정상적으로 실행될 때 기대되는 인자의 개수</u>**를 나타낸다.

  - **[예제 4-13]** 함수 객체의 length 프로퍼티

    ```javascript 
    function func0() {
    }
    
    function func1(x) {
        return x;
    }
    
    function func2(x, y) {
        return x + y;
    }
    
    function func3(x, y, z) {
        return x + y + z;
    }
    
    console.log('func0.length: ' + func0.length); // (출력값) 0
    console.log('func1.length: ' + func1.length); // (출력값) 1
    console.log('func2.length: ' + func2.length); // (출력값) 2
    console.log('func3.length: ' + func3.length); // (출력값) 3
    ```

    



- prototype 프로퍼티

  - 모든 함수는 객체로서 prototype 프로퍼티를 가지고 있다.

  - 여기서 주의할 것은 함수 객체의 **prototype 프로퍼티**는 앞서 설명한 모든 객체의 부모를 나타내는 **내부 프로퍼티**인 **[[Prototype]]**과 혼동하지 말아야 한다는 것이다.

  - **prototype 프로퍼티 vs [[Prototype]] 프로퍼티**

    - 두 프로퍼티 모두 프로토타입 객체를 가리킨다는 점에서 공통점이 있지만, 관점에 차이가 있다.
    - 모든 객체에 있는 내부 프로퍼티인 [[Prototype]]는 객체 입장에서 자신의 부모 역할을 하는 프로토타입 객체를 가리키는 반면,
    - 함수 객체가 가지는 prototype 프로퍼티는 이 함수가 생성자로 사용될 때 이 함수를 통해 생성된 객체의 부모 역할을 하는 프로토타입 객체를 가리킨다.

  - prototype 프로퍼티는 함수가 생성될 때 만들어지며, 다음 그림과 같이 단지 **constructor 프로퍼티** 하나만 있는 객체를 가리킨다.

    ![](https://docs.google.com/drawings/d/sFSqbC1FLg6YYHl23CS_Y8g/image?parent=e/2PACX-1vQiuXMoUJloXB41go11kimhAxCGB0Jhf5sdczvyrPIjH533basHPtLBcGPi59BcshFWUj6e55GqbGOO&rev=81&drawingRevisionAccessToken=HYRQAKOWACbZOg&h=127&w=514&ac=1)

  - 그리고 prototype 프로퍼티가 가리키는 프로토타입 객체의 유일한 constructor 프로퍼티는 자신과 연결된 함수를 가리킨다. 

  - 즉, 자바스크립트에서는 함수를 생성할 때, 함수 자신과 연결된 프로토타입 객체를 동시에 생성하며, 이 둘은 위 그림처럼 각각 prototype과 constructor라는 프로퍼티로 서로를 참조하게 된다.

  - **[예제 4-14]** 함수 객체와 프로토타입 객체와의 관계를 보여주는 코드

    ```javascript
    // MyFunction() 함수 정의
    function myFunction() {
        return true;
    }
    
    console.dir(myFunction.prototype);
    console.dir(myFunction.prototype.constructor);
    ```

    



## 4.3 함수의 다양한 형태

- 콜백 함수

  - 함수 표현식에서 함수 이름은 꼭 붙이지 않아도 되는 선택 사항이다. 이러한 함수가 익명 함수라고 이미 설명했다.

    - 이러한 익명 함수의 대표적인 용도가 바로 **콜백 함수**이다.

  - 콜백 함수는 코드를 통해 명시적으로 호출하는 함수가 아니라, 개발자는 단지 함수를 등록하기만 하고, **어떤 이벤트가 발생했거나 특정 시점에 도달했을 때 시스템에서 호출되는 함수**를 말한다.

  - 또한, 특정 함수의 인자로 넘겨서, 코드 내부에서 호출되는 함수 또한 콜백 함수가 될 수 있다.

  - 콜백 함수의 예시: 자바스크립트에서의 이벤트 핸들러 처리

    ![](https://docs.google.com/drawings/d/src8CRnoI22eHBdRF60E_XQ/image?parent=e/2PACX-1vQiuXMoUJloXB41go11kimhAxCGB0Jhf5sdczvyrPIjH533basHPtLBcGPi59BcshFWUj6e55GqbGOO&rev=159&drawingRevisionAccessToken=vODD0sT8ggw5YQ&h=398&w=492&ac=1)

  - **[예제 4-15]** window.onload 이벤트 핸들러 예제 코드

    ```html
    <!DOCTYPE html>
    <html><body>
    	<script>
    		// 페이지 로드 시 호출될 콜백 함수
    		window.onload = function() {
    			alert('This is the callback function.');
    		};
    	</script>
    </body></html>
    ```

    - window.onload는 이벤트 핸들러로서, 웹 페이지의 로딩이 끝나는 시점에 load 이벤트가 발생하면 실행된다.
    - 예제에서는 window.onload 이벤트 핸들러를 익명 함수로 연결했다. 따라서 익명 함수가 콜백 함수로 등록된 것이다.

    

- 즉시 실행 함수

  - 함수를 정의함과 동시에 바로 실행하는 함수를 **즉시 실행 함수**(Immediate functions)라고 한다.

    - 이 함수도 익명 함수를 응용한 형태이다.

  - **[예제 4-16]** 즉시 실행 함수 예제 코드

    ```javascript
    (function (name) {
    	console.log('This is the immediate function --> ' + name);
    })('foo');
    ```

  - 즉시 실행 함수를 만드는 방법

    1. 우선 함수 리터럴을 괄호 ()로 둘러싼다. 이때 함수 이름이 있든 없든 상관없다.
    2. 그런 다음 함수가 바로 호출될 수 있게 () 괄호 쌍을 추가한다. 이때 괄호 안에 값을 추가해 즉시 실행 함수의 인자로 넘길 수 있다.

  - 이렇게 함수가 선언되자마자 실행되게 만든 즉시 실행 함수의 경우, 같은 함수를 다시 호출할 수 없다.

    - 따라서 즉시 실행 함수의 이러한 특징을 이용한다면 **최초 한 번의 실행만을 필요로 하는 초기화 코드 부분** 등에 사용할 수 있다.
    - 그 외에도 jQuery와 같은 자바스크립트 라이브러리 코드가 로드되면 실행되는 초기화 작업을 할 때 많이 사용된다.



- 내부 함수

  - 자바스크립트에서는 함수 코드 내부에서도 다시 함수 정의가 가능하다.

  - 이렇게 함수 내부에 정의된 함수를 **내부 함수**(Inner function)라고 부른다.

  - 내부 함수의 용도

    - 자바스크립트의 기능을 보다 강력하게 해주는 클로저를 생성 하는 용도
    - 부모 함수 코드에서 외부에서의 접근을 막고 독립적인 헬퍼 함수를 구현하는 용도

  - **[예제 4-18]** 내부 함수 예제 코드

    ```javascript
    // parent() 함수 정의
    function parent() {
        var a = 100;
        var b = 200;
    
        // child() 내부 함수 정의
        function child() {
            var b = 300;
    
            console.log(a);
            console.log(b);
        }
        child();
    }
    
    parent();
    [출력 결과]
    100
    300
    
    child(); // ReferenceError: child is not defined
    ```

    - 내부 함수에서는 자신을 둘러싼 부모 함수의 변수에 접근이 가능하다.
      - child() 함수 내에서 부모 함수의 변수인 a에 접근이 가능하다.
      - 이것이 가능한 이유는 자바스크립트의 **스코프 체이닝** 때문이다.
    - 내부 함수는 일반적으로 자신이 정의된 부모 함수 내부에서만 호출이 가능하다.

  - **[예제 4-19]** 함수 스코프 외부에서 내부 함수 호출하는 예제 코드

    ```javascript
    function parent() {
        var a = 100;
    
        // child() 내부 함수
        var child = function () {
            console.log(a);
        }
    
        return child;
    }
    
    var inner = parent();
    inner();
    
    [출력 결과]
    100
    ```

    - 이와 같이 실행이 끝난 parent()와 같은 부모 함수 스코프의 변수를 참조하는 innert()와 같은 함수를 **클로저**라고 한다.

    

- 함수를 리턴하는 함수

  - JS에서는 함수도 일급 객체이므로 일반 값처럼 함수 자체를 리턴할 수도 있다.

    - 이러한 점을 활용해 함수를 호출함과 동시에 다른 함수로 바꾸거나,
    - 자기 자신을 재정의하는 함수를 구현할 수도 있다.

  - **[예제 4-20]** 자신을 재정의하는 함수 예제 코드

    ```javascript
    // self() 함수
    var self = function () {
        console.log('a');
    
        return function () {
            console.log('b');
        };
    };
    
    self = self();
    self();
    
    [출력 결과]
    a
    b
    ```





## 4.4 함수 호출과 this

- arguments 객체

  - C와 같은 엄격한 언어와 달리, 자바스크립트에서는 <u>함수를 호출할 때 함수 형식에 맞춰 인자를 넘기지 않더라도 에러가 발생하지 않는다</u>.

  - **[예제 4-21]** 함수 형식에 맞춰 인자를 넘기지 않더라도 함수 호출이 가능함을 나타내는 예제 코드

    ```javascript
    function func(arg1, arg2) {
        console.log(arg1, arg2);
    }
    
    func();
    func(1);
    func(1, 2);
    func(1, 2, 3);
    
    [출력 결과]
    undefined undefined
    1 undefined
    1 2
    1 2
    ```

    - 정의된 함수의 인자보다 적게 함수를 호출할 경우, 넘겨지지 않은 인자에는 **undefined** 값이 할당된다.
    - 이와 반대로 정의된 인자 개수보다 많게 함수를 호출했을 경우는 에러가 발생하지 않고, 초과된 인수는 무시된다.

  - JS의 이러한 특성 때문에 함수 코드를 작성할 때, 런타임 시에 호출된 인자의 개수를 확인하고 이에 따라 동작을 다르게 해줘야 할 경우가 있다.

    - 이를 가능케 하는 객체가 바로 **arguments 객체**다.
    - JS에서는 함수를 호출할 때 인수들과 함께 암묵적으로 arguments 객체가 함수 내부로 전달되기 때문이다.

  - arguments 객체는 함수를 호출할 때 넘긴 인자들이 배열 형태로 저장된 객체를 의미한다.

    - 특이한 점은 이 객체는 실제 배열이 아닌 **유사 배열 객체**라는 것이다.

  - **[예제 4-22]** arguments 객체 예제 코드

    ```javascript
    // add() 함수
    function add(a, b) {
    	// arguments 객체 출력
        console.dir(arguments);
        return a + b;
    }
    
    console.log(add(1));
    console.log(add(1, 2));
    console.log(add(1, 2, 3));
    ```

    - 실행결과에서 알 수 있듯이, arguments 객체는 다음과 같이 세 부분으로 구성되어 있다.
      - 함수를 호출할 때 넘겨진 인자 (배열 형태): 함수를 호출할 때 첫 번째 인자는 0번 인덱스, 두 번째 인자는 1번 인덱스, ...
      - length 프로퍼티: 호출할 때 넘겨진 인자의 개수
      - callee 프로퍼티: 현재 실행 중인 함수의 참조값 (예제에서는 add() 함수)
    - 앞서 얘기했듯이 arguments는 객체이지 배열이 아니다.
      - 즉, length 프로퍼티가 있으므로 배열과 유사하게 동작하지만, 배열은 아니므로 배열 메서드를 사용할 경우 에러가 발생한다는 것에 주의하자.

  - **[예제]** 매개변수 개수가 정해지지 않은 sum() 함수 구현

    ```javascript
    function sum() {
        var result = 0;
        for(var i = 0 ; i < arguments.length; i++) {
            result += arguments[i];
        }
    
        return result;
    }
    
    console.log(sum(1, 2, 3));
    console.log(sum(1, 2, 3, 4, 5, 6, 7, 8));
    ```

    

- 호출 패턴과 this 바인딩

  

