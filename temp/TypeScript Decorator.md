### TypeScript Decorators

> https://www.typescriptlang.org/docs/handbook/decorators.html



### 도입

- TypeScript에서 클래스가 도입되고, ES6가 등장함에 따라 현재는 

  - annotating 지원 기능
  - 클래스 자체 수정 기능
  - 클래스 멤버 수정 기능

  을 지원해야 하는 상황이 많이 생기게 되었다.

- Decorator는
  - 클래스 선언,
  - 멤

- Decorator는 JavaScript의 [stage 2 proposal](https://github.com/tc39/proposal-decorators)이고, TypeScript의 experimental feature(실험 기능? 아직 불안정한 기능인것 같다)으로 이용 가능하다.

  >  experimental feature는 future release 시에 기능이 바뀔 수 있다.



### Decorator 기능 키기

- Decorator에 대한 experimental support를 enable하려면, [`experimentalDecorators`](https://www.typescriptlang.org/tsconfig#experimentalDecorators) 컴파일러 옵션을 command line에서 키거나 `tsconfig.json`에서 켜야 한다.

  - **Command Line**

    `tsc --target ES5 --experimentalDecorators`

  - **tsconfig.json**

    ```typescript
    {
      "compilerOptions": {
        "target": "ES5",
        "experimentalDecorators": true
      }
    }
    ```



### Decorator

- Decorator를 붙일 수 있는 곳
  - [class declaration](https://www.typescriptlang.org/docs/handbook/decorators.html#class-decorators)
  - [method](https://www.typescriptlang.org/docs/handbook/decorators.html#method-decorators) 
  - [accessor](https://www.typescriptlang.org/docs/handbook/decorators.html#accessor-decorators)
  - [property](https://www.typescriptlang.org/docs/handbook/decorators.html#property-decorators) 
  - [parameter](https://www.typescriptlang.org/docs/handbook/decorators.html#parameter-decorators). 
- Decorator는 위와 같은 곳에 첨부할 수 있는 특수한 종류의 선언이다.
- Decorator 사용법
  - `@expression`: 여기서 expression은 





