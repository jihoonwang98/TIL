### Typescript

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

      

  - Any

  - Void

  - Null and Undefined

  - Never

