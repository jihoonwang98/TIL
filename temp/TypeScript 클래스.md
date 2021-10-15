## TypeScript 클래스

> https://typescript-kr.github.io/pages/classes.html



### 도입

- 기존 JavaScript
  - 재사용할 수 있는 컴포넌트를 만들기 위해 <u>함수와 프로토타입 기반 상속을 사용</u>
- ECMAScript 6(ECMAScript 2015) 등장 이후
  - 객체지향적 클래스 기반의 접근 방식을 사용해 애플리케이션 생성 가능해짐
- TypeScript에서는 다음 버전의 Javascript를 기다릴 필요 없이 개발자들이 이러한 기법들을 사용할 수 있게 해줬다.



### 클래스 (Classes)

- **예제**

  ```typescript
  class Greeter {
  	greeting: string;
    constructor(message: string) {
      this.greeting = message;
    }
    greet() {
      return "Hello, " + this.greeting;
    }
  }
  
  let greeter = new Greeter('world');
  console.log(greeter.greet());  // Hello, world
  ```

  - Greeter 클래스의 구성 (멤버)
    - `greeting` 프로퍼티
    - 생성자
    - `greet` 메서드



### 상속

상속은 링크를 참고...





### Access Modifiers

- 총 3종류
  - public
    - 자바에는 default 접근 지시자가 있지만 TypeScript에서는 접근 지시자를 명시하지 않으면 기본적으로 `public`으로 적용된다.
  - private
  - protected



- ECMAScript 비공개 필드 (ECMAScript Private Fields)

  - TypeScript 3.8에서, TypeScript는 비공개 필드를 위한 JavaScript의 새로운 문법을 지원한다.

    ```typescript
    class Animal {
    	#name: string;
      constructor(theName: string) {
        this.#name = theName;
      }
    }
    
    new Animal('Cat').#name;   // 프로퍼티 '#name'은 비공개 식별자이기 때문에 'Animal' 클래스 외부에선 접근할 수 없습니다.
    ```

    - 이 문법은 JavaScript 런타임에 내장되어 있고, 각각의 비공개 필드의 격리를 더 잘 보장할 수 있다.
    - 현재 TypeScript 3.8 [릴리즈 노트](https://devblogs.microsoft.com/typescript/announcing-typescript-3-8-beta/#type-only-imports-exports)에 비공개 필드에 대해 자세히 나와있습니다.



- TypeScript의 `private` 이해하기 (Understanding TypeScript’s `private`)

  - TypeScript에는 멤버를 포함하는 클래스 외부에서 이 멤버에 접근하지 못하도록 멤버를 `private`으로 표시하는 방법이 있습니다. 

    ```typescript
    class Animal {
        private name: string;
        constructor(theName: string) { this.name = theName; }
    }
    
    new Animal("Cat").name; // 오류: 'name'은 비공개로 선언되어 있습니다;
    ```

    ... 작성중











