# [Effective TypeScript] 1장 타입스크립트 알아보기



### 목차

- 타입스크립트와 자바스크립트의 관계 이해하기
- 타입스크립트 설정 이해하기
- 코드 생성과 타입이 관계없음을 이해하기
- 구조적 타이핑에 익숙해지기
- any 타입 지양하기



## 아이템 1. 타입스크립트와 자바스크립트의 관계 이해하기

- 타입스크립트는 자바스크립트의 상위집합(superset)이다.

  - 타입스크립트는 자바스크립트의 상위집합이기 때문에 `.js` 파일에 있는 코드는 이미 타입스크립트라고 할 수 있다.

  - `main.js` 파일명을 `main.ts`로 바꾼다고 해도 달라지는 것은 없다.

  - 모든 자바스크립트 프로그램이 타입스크립트다. (참)

    - 역은 성립하지 않는다.

    - 타입스크립트 프로그램이지만 자바스크립트가 아닌 프로그램이 존재한다.

    - 이는 타입스크립트가 타입을 명시하는 추가적인 문법을 가지기 때문이다.

    - 예를 들어, 다음 코드는 유효한 타입스크립트 프로그램이다.

      ```typescript
      function greet(who: string) {
      	console.log('Hello', who);
      }
      ```

    - 그러나 자바스크립트를 구동하는 노드(node) 같은 프로그램으로 앞의 코드를 실행하면 오류를 출력한다.

    - `: string`은 타입스크립트에서 쓰이는 타입 구문이다.

    - 타입 구문을 사용하는 순간부터 자바스크립트는 타입스크립트 영역으로 들어가게 된다.

  ![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FoDk43%2FbtreLDDA85h%2FEgErghZm56t24edSLYZ3ek%2Fimg.png)

- 타입스크립트 컴파일러는 타입스크립트뿐만 아니라 일반 자바스크립트 프로그램에도 유용하다.

  - 다음 자바스크립트 코드를 예로 들어보자.

    ```js
    let city = 'new york city';
    console.log(city.toUppercase());
    ```

  - 이 코드를 실행하면 다음과 같은 오류가 발생한다.

    ```
    TypeError: city.toUppercase is not a function
    ```

  - 앞의 코드에는 타입 구문이 없지만, 타입스크립트의 타입 체커는 문제점을 찾아낸다.

    ```ts
    let city = 'new york city';
    console.log(city.toUppercase());
    // ~~~ 'toUppercase' 속성이 'string' 형식에 없습니다.
    //     'toUpperCase'을(를) 사용하시겠습니까?
    ```

  - `city` 변수가 문자열이라는 것을 알려 주지 않아도 타입스크립트는 초깃값으로부터 타입을 추론한다.



- 타입 시스템의 목표 중 하나는 런타임에 오류를 발생시킬 코드를 미리 찾아내는 것이다.

  - 타입스크립트가 '정적' 타입 시스템이라는 것은 바로 이런 특징을 말하는 것이다.

- 그러나 타입 체커가 모든 오류를 찾아내지는 않는다.

- 오류가 발생하지는 않지만 의도와 다르게 동작하는 코드도 있다.

  - 타입스크립트는 이러한 문제 중 몇 가지를 찾아내기도 한다.

  - 다음 자바스크립트 코드를 보자.

    ```js
    const states = [
      {name: 'Alabama', capital: 'Montgomery'},
      {name: 'Alaska', capital: 'Juneau'},
      {name: 'Arizona', capital: 'Phoenix'},
    	// ...  
    ];
    
    for(const state of states) {
      console.log(state.capitol);
    }
    
    [실행 결과]
    undefined
    undefined
    undefined
    ```

  - 앞의 코드는 유효한 JS(또한 TS)이며 어떠한 오류도 없이 실행된다.

  - 그러나 루프 내의 `state.capitol`은 의도한 코드가 아닌게 분명하다.

  - 이런 경우에 타입스크립트 타입 체커는 추가적인 타입 구문 없이도 오류를 찾아낸다.

    ```ts
    for(const state of states) {
      console.log(state.capitol);
                      //~~~~~~~ 'capitol' 속성이 ... 형식에 없습니다.
      								//        'capital'을(를) 사용하시겠습니까?
    }
    ```

    

- 타입스크립트는 타입 구문 없이도 오류를 잡을 수 있지만, 타입 구문을 추가하면 훨씬 더 많은 오류를 찾아낼 수 있다.

  - 코드의 '의도'를 타입 구문을 통해 타입스크립트에게 알려줄 수 있기 때문에 코드의 동작과 의도가 다른 부분을 찾을 수 있다.

  - 예를 들어, 다음과 같이 `capital`과 `capitol`을 맞바꾸어 보자.

    ```ts
    const states = [
      {name: 'Alabama', capitol: 'Montgomery'},
      {name: 'Alaska', capitol: 'Juneau'},
      {name: 'Arizona', capitol: 'Phoenix'},
    	// ...  
    ];
    
    for(const state of states) {
      console.log(state.capital);
                      //~~~~~~~ 'capital' 속성이 ... 형식에 없습니다.
      								//        'capitol'을(를) 사용하시겠습니까?
    }
    
    [실행 결과]
    undefined
    undefined
    undefined
    ```

  - 그런데 TS가 제시한 해결책은 잘못되었다.

  - `capital`을 고칠게 아니라 `capitol`을 고쳐야 한다. 

  - TS는 어느 쪽이 오타인지 판단하지 못한다.

  - 따라서 명시적으로 `states`를 선언하여 의도를 분명하게 하는 것이 좋다.

    ```ts
    interface State {
    	name: string;
      capital: string;
    }
    
    const states: State[] = [
      {name: 'Alabama', capitol: 'Montgomery'},
                      // ~~~~~~~~~~~~~~~~~~~
      {name: 'Alaska', capitol: 'Juneau'},
                     // ~~~~~~~~~~~~~~~~~~~
      {name: 'Arizona', capitol: 'Phoenix'},
                    // ~~~~~~~~~~~~~~~~~ 객체 리터럴은 알려진 속성만 
      							//                   지정할 수 있지만 'State' 형식에
      							//                  'capitol'이 없습니다.
        					  //                  'capital'을 쓰려고 했습니까?
    ];
    
    for(const state of states) {
      console.log(state.capital);
    }
    ```

  - 이제 오류가 어디에서 발생했는지 찾을 수 있고, 제시된 해결책도 올바르다.

    - 의도를 명확히 해서 TS가 잠재적 문제점을 찾을 수 있게 했다.

  - 예를 들어, 타입 구문 없이 배열 안에서 딱 한 번 `capitol`이라고 오타를 썼다면 오류가 되지 않는다.

  - 그런데 타입 구문을 추가하면 오류를 찾을 수 있다.

    ```ts
    const states: State[] = [
      {name: 'Alabama', capital: 'Montgomery'},
      {name: 'Alaska', capitol: 'Juneau'},
                     // ~~~~~~~~~~~~~~~~~~ 'capital'을 쓰려고 했습니까?
      {name: 'Arizona', capital: 'Phoenix'},
    ];
    ```



**요약**

- 타입스크립트는 자바스크립트의 상위집합입니다.
  - 다시 말해서, 모든 JS 프로그램은 이미 TS 프로그램입니다.
  - 반대로, TS는 별도의 문법을 가지고 있기 때문에 일반적으로는 유효한 JS 프로그램은 아닙니다.
- TS는 JS 런타임 동작을 모델링하는 타입 시스템을 가지고 있기 때문에 런타임 오류를 발생시키는 코드를 찾아내려고 합니다.
  - 그러나 모든 오류를 찾아내리라 기대하면 안됩니다.
  - 타입 체커를 통과하면서도 런타임 오류를 발생시키는 코드는 충분히 존재할 수 잇습니다.
- TS 타입 시스템은 전반적으로 JS 동작을 모델링합니다.
  - 그러나 잘못된 매개변수 개수로 함수를 호출하는 경우처럼, JS에서는 허용되지만 TS에서는 문제가 되는 경우도 있습니다.





## 아이템 2. 타입스크립트 설정 이해하기

- 다음 코드가 오류 없이 타입 체커를 통과할 수 있을지 생각해보자.

  ```ts
  function add(a, b) { 
  	return a + b;
  }
  add(10, null);
  ```

  - 설정이 어떻게 되어 있는지 모른다면 대답할 수 없다.

  - TS 컴파일러는 매우 많은 설정을 가지고 있다.

  - 이 설정들은 커맨드 라인을 이용해 할 수 있다.

    ```ts
    $ tsc --noImplicitAny program.ts
    ```

  - 또는 `tsconfig.json` 설정 파일을 통해서도 가능하다.

    ```json
    {
    	"compilerOptions": {
        "noImplicitAny": true
      }
    }
    ```

    - 가급적 설정 파일을 사용하자.
    - 설정 파일은 `tsc --init`만 실행하면 간단히 생성된다.

- 타입스크립트의 설정들은 어디서 소스 파일을 찾을지, 어떤 종류의 출력을 생성할지 제어하는 내용이 대부분이다.
- 그런데 언어 자체의 핵심 요소들을 제어하는 설정도 있다.
  - 대부분의 언어에서는 허용하지 않는 고수준 설계의 설정이다.
  - 타입스크립트는 어떻게 설정하느냐에 따라 완전히 다른 언어처럼 느껴질 수 있다.
- 설정을 제대로 사용하려면, `noImplicitAny`와 `strictNullChecks`를 이해해야 한다.



- `noImplicitAny`는 **<u>변수들이 미리 정의된 타입을 가져야 하는지 여부를 제어</u>**한다.

  - 다음 코드는 `noImplicitAny`가 해제되어 있을 때에는 유효하다.

    ```ts
    function add(a, b) {
    	return a + b;
    }
    ```

    - 편집기에서 `add` 부분에 마우스를 올려 보면, 타입스크립트가 추론한 함수의 타입을 알 수 있다.

    ```ts
    function add(a: any, b: any): any
    ```

    - `any` 타입을 매개변수에 사용하면 타입 체커는 속절없이 무력해진다.

  - 다음 예제를 보자.

    ```ts
    function add(a, b) {
    				 //  ~    'a' 매개변수에는 암시적으로 'any' 형식이 포함된다.
    				 //     ~ 'b' 매개변수에는 암시적으로 'any' 형식이 포함된다.
    	return a + b;
    }
    ```

    - `any`를 코드에 넣지 않았지만, `any` 타입으로 간주되기 때문에 이를 '암시적 `any`'라고 부른다.
    - 그런데 같은 코드임에도 `noImplicitAny`가 설정되었다면 오류가 된다.
    - 이 오류들은 명시적으로 `: any`라고 선언해 주거나 더 분명한 타입을 사용하면 해결할 수 있다.

    ```ts
    function add(a: number, b: number) {
    	return a + b;
    }
    ```

  - 타입스크립트는 타입 정보를 가질 때 가장 효과적이기 때문에, 되도록이면 `noImplicitAny`를 설정해야 한다.

  - `noImplicitAny` 설정 해제는, 자바스크립트로 되어 있는 기존 프로젝트를 타입스크립트로 전환하는 상황에만 필요합니다.



- `strictNullChecks`는 `null`과 `undefined`가 모든 타입에서 허용되는지 확인하는 설정입니다.

  - 다음은 `strictNullChecks`가 해제되었을 때 유효한 코드입니다.

    ```ts
    const x: number = null; // 정상, null은 유효한 값
    ```

  - 그러나 `strictNullChecks`를 설정하면 오류가 된다.

    ```ts
    const x: number = null;
    //    ~ 'null' 형식은 'number' 형식에 할당할 수 없다.
    ```

  - `null` 대신 `undefined`를 써도 같은 오류가 난다.

  - 만약 `null`을 허용하려고 한다면, 의도를 명시적으로 드러냄으로써 오류를 고칠 수 있다.

    ```ts
    const x: number | null = null;
    ```

  - 만약 `null`을 허용하지 않으려면, 이 값이 어디서부터 왔는지 찾아야 하고, `null`을 체크하는 코드나 단언문(assertions)을 추가해야 한다.

    ```ts
    const el = document.getElementById('status');
    el.textContent = 'Ready';
    //~~ 개체가 'null'인 것 같습니다.
    
    if(el) {
      el.textContent = 'Ready';  // 정상, null은 제외된다.
    }
    el!.textContent = 'Ready';  // 정상, el이 null이 아님을 단언한다.
    ```

- `strictNullChecks`는 `null`과 `undefined` 관련된 오류를 잡아 내는 데 많은 도움이 되지만, 코드 작성을 어렵게 합니다.

  - 새 프로젝트를 시작한다면 가급적 `strictNullChecks`를 설정하는 것이 좋다.

- `strictNullChecks`를 설정하려면 `noImplicitAny`를 먼저 설정해야 한다.

- `strcitNullChecks` 설정 없이 개발하기로 선택했다면, "`undefined`는 객체가 아닙니다"라는 끔찍한 런타임 오류를 주의하기 바란다.



- 언어에 의미적으로 영향을 미치는 설정들 (예를 들어, `noImplicitThis`와 `strictFunctionTypes`)이 많지만, `noImplicitAny`와 `strictNullChecks`만큼 중요한 것은 없습니다.
  - 이 모든 체크를 설정하고 싶다면 `strict` 설정을 하면 됩니다.
  - 타입스크립트에 `strict` 설정을 하면 대부분의 오류를 잡아냅니다.



**요약**

- 타입스크립트 컴파일러는 언어의 핵심 요소에 영향을 미치는 몇 가지 설정을 포함하고 있습니다.
- 타입스크립트 설정은 커맨드 라인을 이용하기 보다 `tsconfig.json`을 이용하는 것이 좋습니다.
- 자바스크립트 프로젝트를 타입스크립트로 전환하는게 아니라면 `noImplicitAny`를 설정하는 것이 좋습니다.
- "`undefined`는 객체가 아닙니다" 같은 런타임 오류를 방지하기 위해 `strictNullChecks`를 설정하는 것이 좋습니다.
- 타입스크립트에서 엄격한 체크를 하고 싶다면 `strict` 설정을 고려해야 합니다.



## 아이템 3. 코드 생성과 타입이 관계없음을 이해하기

- 큰 그림에서 보면, 타입스크립트 컴파일러는 두 가지 역할을 수행합니다.
  - 최신 타입스크립트/자바스크립트를 브라우저에서 동작할 수 있도록 구버전의 자바스크립트로 트랜스파일(transpile)합니다.
  - 코드의 타입 오류를 체크합니다.
- 여기서 놀라운 점은 이 두 가지가 서로 완벽히 독립적이라는 것이다.
  - 다시 말해, TS가 JS로 변환될 때 코드 내의 타입에는 영향을 주지 않는다.
  - 또한 그 JS의 실행 시점에도 타입은 영향을 미치지 않는다.
- TS 컴파일러가 수행하는 두 가지 역할을 되짚어 보면, TS가 할 수 있는 일과 없는 일을 짐작할 수 있다.



### 타입 오류가 있는 코드도 컴파일이 가능하다.

- 컴파일은 타입 체크와 독립적으로 동작하기 때문에, 타입 오류가 잇는 코드도 컴파일 가능하다.

  ```shell
  $ cat test.ts
  let x = 'hello';
  x = 1234;
  $ tsc test.ts
  test.ts:2:1 - error TS2322: '1234' 형식은 'string' 형식에 할당할 수 없습니다.
  
  2 x - 1234;
    ~
  ```

- 타입 체크와 컴파일이 동시에 이루어지는 C나 자바와 같은 언어를 사용하던 사람이라면 이러한 상황이 황당하게 느껴집니다.

- 타입스크립트 오류는 C나 자바 같은 언어들의 경고(warning)와 비슷합니다.

  - 문제가 될 만한 부분을 알려주지만, 그렇다고 빌드를 멈추지는 않습니다.

- 만약 오류가 있을 때 컴파일하지 않으려면, `tsconfig.json`에 `noEmitOnError`를 설정하거나 빌드 도구에 동일하게 적용하면 됩니다.



### 런타임에는 타입 체크가 불가능합니다.

- 다음처럼 코드를 작성해보자.

  ```ts
  interface Square {
  	width: number;
  }
  
  interface Rectangle extends Square {
    height: number;
  }
  
  type Shape = Square | Rectangle;
  
  function calculateArea(shape: Shape) {
    if(shape instanceof Rectangle) {
      							 // ~~~~~~~~~ 'Rectangle'은(는) 형식만 참조하지만,
      							//            여기서는 값으로 사용되고 있습니다.
      return shape.width * shape.height;
                    //     ~~~~~~~~~~~~ 'Shpae' 형식에 'height' 속성이 없습니다.
    } else {
      return shape.width * shape.width;
    }
  }
  ```

  - `instanceof` 체크는 런타임에 일어나지만, `Rectangle`은 타입이기 때문에 런타임 시점에 아무런 역할을 할 수 없다.
  - 타입스크립트의 타입은 '제거 가능(erasable)'하다. 
  - 실제로 JS로 컴파일되는 과정에서 모든 인터페이스, 타입, 타입 구문은 그냥 제거되어 버린다.

- 앞의 코드에서 다루고 있는 `shape` 타입을 명확하게 하려면, 런타임에 타입 정보를 유지하는 방법이 필요하다.

  - 하나의 방법은 `height` 속성이 존재하는지 체크해보는 것이다.

  ```ts
  function calculateArea(shape: Shape) {
  	if('height' in shape) {
      shape;  // 타입이 Rectangle
      return shape.width * shape.height;
    } else {
      shape;  // 타입이 Square
      return shape.width * shape.width;
    }
  }
  ```

  - 속성 체크는 런타임에 접근 가능한 값에만 관련되지만, 타입 체커 역시도 `shape`의 타입을 `Rectangle`로 보정해주기 때문에 오류가 사라집니다.

- 타입 정보를 유지하는 또 다른 방법으로는 런타임에 접근 가능한 타입 정보를 명시적으로 저장하는 '태그' 기법이 있다.

  ```ts
  interface Square {
    kind: 'square';
    width: number;
  }
  interface Rectangle {
    kind: 'rectangle';
    height: number;
    width: number;
  }
  type Shape = Square | Rectangle;
  
  function calculateArea(shape: Shape) {
    if(shape.kind === 'rectangle') {
      shape;  // 타입이 Rectangle
      return shape.width * shape.height;
    } else {
      shape;  // 타입이 Square
      return shape.width * shape.width;
    }
  }
  ```

  - 여기서 `Shape` 타입은 '태그된 유니온(tagged union)'의 한 사례입니다.
  - 이 기법은 런타임에 타입 정보를 손쉽게 유지할 수 있기 때문에, TS에서 흔하게 볼 수 있습니다.

- 타입(런타임 접근 불가)과 값(런타임 접근 가능)을 둘 다 사용하는 기법도 있습니다.

  - 타입을 클래스로 만들면 됩니다. 
  - `Square`와 `Rectangle`을 클래스로 만들면 오류를 해결할 수 잇습니다.

  ```ts
  class Square {
  	constructor(public width: number) {}
  }
  class Rectangle extends Square {
    constructor(public width: number, public height: number) {
      super(width);
    }
  }
  type Shape = Square | Rectangle;
  
  function calculateArea(shape: Shape) {
    if(shape instanceof Rectangle) {
      shape;  // 타입이 Rectangle
      return shape.width * shape.height;
    } else {
      shape;  // 타입이 Square
      return shape.width * shape.width; // 정상
    }
  }
  ```

  - 인터페이스는 타입으로만 사용 가능하지만, `Rectangle`을 클래스로 선언하면 타입과 값으로 모두 사용할 수 있으므로 오류가 없습니다.
  - `type Shape = Square | Rectangle` 부분에서 `Rectangle`은 타입으로 참조되지만, `shape instanceof Rectangle` 부분에서는 값으로 참조된다.
    - 어떻게 참조되는지 구분하는 건 매우 중요한데, 이는 아이템 8에서 다룬다.





### 타입 연산은 런타임에 영향을 주지 않는다.

- `string` 또는 `number` 타입인 값을 항상 `number`로 정제하는 경우를 가정해보자.

- 다음 코드는 타입 체커를 통과하지만 잘못된 방법을 썼다.

  ```ts
  function asNumber(val: number | string): number {
  	return val as number;
  }
  ```

  - `as number`는 '타입 단언문'이다.
    - 이런 문법이 언제 적절히 사용될 수 있는지는 아이템 9에서 다룬다.

- 변환된 JS 코드를 보면 이 함수가 실제로 어떻게 동작하는지 알 수 있따.

  ```ts
  function asNumber(val) {
  	return val;
  }
  ```

  - 코드에 아무런 정제 과정이 없다.
  - `as number`는 타입 연산이고 런타임 동작에는 아무런 영향이 없다.

- 값을 정제하기 위해서는 런타임의 타입을 체크해야 하고 JS 연산을 통해 변환해줘야 한다.

  ```ts
  function asNumber(val: number | string) number {
  	return typeof(val) === 'string' ? Number(val) : val;
  }
  ```



### 런타임 타입은 선언된 타입과 다를 수 있다.

- 다음 함수를 보고 마지막의 `console.log`까지 실행될 수 있을지 생각해보자.

  ```ts
  function setLightSwitch(value: boolean) {
  	switch (value) {
      case true: 
        turnLightOn();
        break;
      case false:
        turnLightOff();
        break;
      default:
        console.log('실행되지 않을까봐 걱정됩니다...');
    }
  }
  ```

  - TS는 일반적으로 실행되지 못하는 죽은 코드를 찾아내지만, 여기서는 `strict`를 설정해도 못 찾아낸다.
  - 그러면 마지막 부분을 실행할 수 있는 경우는 무엇일까?
    - `: boolean`이 타입 선언문이라는 것에 주목하자.
    - TS의 타입이기 때문에 `: boolean`은 런타임에 제거된다.
    - JS였다면 실수로 `setLightSwitch`를 "ON"으로 호출할 수도 있었을 것이다.

- 순수 TS에서도 마지막 코드를 실행하는 방법이 존재한다.

  - 예를 들어, 네트워크 호출로부터 받아온 값으로 함수를 실행하는 경우가 있다.

  ```ts
  interface LightApiResponse {
  	lightSwitchValue: boolean;
  }
  async function setLight() {
    const response = await fetch('/light');
    const result: LightApiResponse = await response.json();
    setLightSwitch(result.lightSwitchValue);
  }
  ```

  - `/light`를 요청하면 그 결과로 `LightApiResponse`를 반환하라고 선언했지만, 실제로 그렇게 되리라는 보장은 없다.
  - API를 잘못 파악해서 `lightSwitchValue`가 실제로는 문자열이었다면, 런타임에는 `setLightSwitch` 함수까지 전달될 것이다.
  - 또는 배포된 후에 API가 변경되어 `lightSwitchValue`가 문자열이 되는 경우도 있을 것이다.

- TS에서는 런타임 타입과 선언된 타입이 맞지 않을 수 있다.

  - 선언된 타입이 언제든지 달라질 수 있다는 것을 명심하자.



### 타입스크립트 타입으로는 함수를 오버로드 할 수 없다.

- C++ 같은 언어는 동일한 이름에 매개변수만 다른 여러 버전의 함수를 허용한다.

  - 이를 '함수 오버로딩'이라고 한다.

- 그러나 TS에서는 타입과 런타임의 동작이 무관하기 때문에, 함수 오버로딩은 불가능하다.

  ```ts
  function add(a: number, b: number) { return a + b; }
        // ~~~ 중복된 함수 구현입니다.
  function add(a: string, b: string) { return a + b; } 
        // ~~~ 중복된 함수 구현입니다.
  ```

- TS가 함수 오버로딩 기능을 지원하기는 하지만, 온전히 타입 수준에서만 동작합니다.

  - 하나의 함수에 대해 여러 개의 선언문을 작성할 수 있지만, 구현체(implementation)은 오직 하나뿐입니다.

  ```ts
  function add(a: number, b: number): number;
  function add(a: string, b: string): string;
  
  function add(a, b) {
    return a + b;
  }
  
  const three = add(1, 2);
  const twelve = add('1', '2');
  ```

  - `add`에 대한 처음 두 개의 선언문은 타입 정보를 제공할 뿐입니다.
    - 이 두 선언문은 TS가 JS로 변환되면서 제거되며, 구현체만 남게 됩니다.
    - 이런 스타일의 오버로딩을 사용하려면, 아이템 50을 먼저 살펴봅시다. (꼭 알아야 할 주의 사항이 있습니다)



### TS 타입은 런타임 성능에 영향을 주지 않습니다.

- 타입과 타입 연산자는 JS 변환 시점에 제거되기 때문에, 런타임의 성능에 아무런 영향을 주지 않습니다.
- TS의 정적 타입은 실제로 비용이 전혀 들지 않습니다.
- TS를 쓰는 대신 런타임 오버헤드를 감수하며 타입 체크를 해본다면, TS 팀이 다음 주의사항들을 얼마나 잘 테스트해 왓는지 몸소 느끼게 될 것이빈다.
  - '런타임' 오버헤드가 없는 대신, TS 컴파일러는 '빌드타임' 오버헤드가 잇습니다.
    - TS 팀은 컴파일러 성능을 매우 중요하게 생각합니다.
    - 따라서 컴파일은 일반적으로 상당히 빠르며, 특히 증분(incremental) 빌드 시에 더욱 체감됩니다.
    - 오버헤드가 커지면, 빌드 도구에서 'transpile only'을 설정하여 타입 체크를 건너뛸 수 잇습ㄴ니다.
  - 타입스크립트가 컴파일하는 코드는 오래된 런타임 환경을 지원하기 위해 호환성을 높이고 성능 오버헤드를 감안할지, 호환성을 포기하고 성능 중심의 네이티브 구현체를 선택할지의 문제에 맞닥뜨릴 수 있습니다.
    - 예를 들어 제네레이터 함수가 ES5 타깃으로 컴파일되려면, TS 컴파일러는 호환성을 위한 특정 헬퍼 코드를 추가할 것입니다.
    - 이런 경우가 제네레이터와의 호환성을 위한 오버헤드 또는 성능을 위한 네이티브 구현체 선택의 문제입니다.
    - 어떤 경우든지 호환성과 성능 사이의 선택은 컴파일 타깃과 언어 레벨의 문제이며 여전히 타입과는 무관하빈다.



**요약**

- 코드 생성은 타입 시스템과 무관합니다. 타입스크립트 타입은 런타임 동작이나 성능에 영향을 주지 않습니다.
- 타입 오류가 존재하더라도 코드 생성(컴파일)은 가능합니다.
- 타입스크립트 타입은 런타임에 사용할 수 없습니다.
  - 런타임에 타입을 지정하려면, 타입 정보 유지를 위한 별도의 방법이 필요합니다.
    - 일반적으로는 태그된 유니온과 속성 체크 방법을 사용합니다.
    - 또는 클래스 같이 타입스크립트 타입과 런타입 값, 둘 다 제공하는 방법이 있습니다.































