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





























