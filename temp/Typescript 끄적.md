### Typescript



### 기본 타입

- 변수 선언
  - `let 변수명: 타입명 = 초기화_값`

- 기본 타입

  - boolean

    - `let isDone: boolean = false;`

  - Number

    - ```typescript
      let decimal: number = 6;
      let hex: number = 0xf00d;
      let binary: number = 0b1010;
      let octal: number = 0o744;
      ```

    - 모든 숫자는 부동 소수 값.

  - String

    - `let color: string = "blue";`

    - **템플릿 문자열**

      - 여러 줄에 걸쳐 문자열을 작성할 수 있다.

      - 표현식을 포함시킬 수 있다.

      - 템플릿 문자열은 백틱/백쿼트( \` ) 문자로 감싸지며, `${ expr }` 형태로 표현식을 포함시킬 수 있다.

        ```typescript
        let fullName: string = `Bob Bobbington`;
        let age: number = 37;
        let sentence: string = `Hello, my name is ${ fullName }.
        I'll be ${ age + 1 } years old next month.`;
        ```

        위는 아래와 동일하다.

        ```typescript
        let sentence: string = "Hello, my name is " + fullName + ".\n\n" +
            "I'll be " + (age + 1) + " years old next month.";
        ```

  - Array

    - 배열 타입은 두 가지 방법으로 쓸 수 있다.

      1. 첫 번째 방법은, 배열 요소들을 나타내는 타입 뒤에 `[]`를 쓰는 것이빈다.

         ```typescript
         let list: number[] = [1, 2, 3];
         ```

         

      2. 두 번째 방법은 제네릭 배열 타입을 쓰는 것이빈다. (`Array<elemType>`)

         ```typescript
         let list: Array<number> = [1, 2, 3];
         ```

  - Tuple

    - 튜플 타입을 사용하면, 요소의 타입과 개수가 고정된 배열을 표현할 수 있다.

    - **예제**

      ```typescript
      // 튜플 타입으로 선언
      let x: [string, number];
      
      // 초기화
      x = ['hello', 10];
      
      // 정해진 인덱스에 위치한 요소에 접근하면 해당 타입이 나타난다.
      console.log(x[0].substring(1)); // 성공
      console.log(x[1].substring(1)); // 오류 (number 타입)
      
      // 정해진 인덱스 외에 다른 인덱스에 있는 요소에 접근하면, 오류가 발생
      x[3] = 'world';
      
      console.log(x[5].toString()); 
      ```

  - Enum

    - C# 같은 언어처럼, `enum`은 값의 집합에 더 나은 이름을 지어줄 수 있다.

      ```typescript
      enum Color {Red, Green, Blue}
      let c: Color = Color.Green;
      ```

    - 기본적으로, `enum`은 `0`부터 시작하여 멤버들의 번호를 매긴다. 멤버 중 하나의 값을 수동으로 설정하여 번호를 바꿀 수 있습니다. 예를 들어, 위 예제를 `0` 대신 `1`부터 시작해 번호를 매기도록 바꿀 수 있다.

      ```typescript
      enum Color {Red = 1, Green, Blue}
      let c: Color = Color.Green;
      ```

      또는, 모든 값을 수동으로 설정할 수 있다.

      ```typescript
      enum Color {Red = 1, Green = 2, Blue = 4} 
      let c: Color = Color.Green;
      ```

    - `enum`의 유용한 기능 중 하나는 매겨진 값을 사용해 enum 멤버의 이름을 알아낼 수 있다는 것이다.

      예를 들어, 위의 예제에서 `2`라는 값이 위의 어떤 `Color` enum 멤버와 매칭되는지 알 수 없을 때, 이에 일치하는 이름을 알아낼 수 있습니다.

      ```
      enum Color {Red = 1, Green, Blue}
      ... 숮어중
      ```

  - Any

  - Void

  - Null and Undefined

  - Never



### 인터페이스

- Our First Interface

  ```typescript
  function printLabel(labeledObj: { label: string }) {
      console.log(labeledObj.label);
  }
  // labeldObj 객체는 string 타입의 label 프로퍼티를 가지고 있기만 하면 된다.
  // 꼭 label 프로퍼티 하나만을 가질 필요는 없다.
  
  let myObj = {size: 10, label: "Size 10 Object"};
  printLabel(myObj);
  ```

  - `printLabel` 함수는 `string` 타입 `label`을 갖는 객체를 하나의 매개변수로 가집니다. 
    - 이 객체가 실제로는 더 많은 프로퍼티를 갖고 있지만, 컴파일러는 *최소한* 필요한 프로퍼티가 있는지와 타입이 잘 맞는지만 검사합니다. TypeScript가 관대하지 않은 몇 가지 경우는 나중에 다루겠습니다.

- 이번엔 같은 예제를, 문자열 타입의 프로퍼티 `label`을 가진 인터페이스로 다시 작성해보자.

  ```typescript
  interface LabeledValue {
  	label: string;
  }
  
  function printLabel(labeledObj: LabeledValue) {
    console.log(labeledObj.label);
  }
  
  let myObj = {
    size: 10, 
    label: 'Size 10 Object'
  };
  printLabel(myObj);
  ```

  - `LabeledValue` 인터페이스는 이전 예제의 요구사항을 똑같이 기술하는 이름으로 사용할 수 있다.
  - 이 인터페이스는 여전히 `문자열` 타입의 `label ` 프로퍼티 하나를 가진다는 것을 의미합니다.
    - <u>다른 언어처럼 `printLabel`에 전달한 객체가 이 인터페이스를 구현해야 한다고 명시적으로 얘기할 필요는 없습니다.</u>
    - 여기서 중요한 것은 형태뿐입니다. 함수에 전달된 객체가 나열된 요구 조건을 충족하면, 허용됩니다.
  - 타입 검사는 프로퍼티들의 순서를 요구하지 않는다.
    - 그냥 프로퍼티가 있는지, 프로퍼티가 타입이 일치하는지만을 확인한다.

  

  

  

  







