# [TypeScript] Decorator 만들기



### 1. 데코레이터를 쓰기 위한 tsconfig.json 설정

```json
// tsconfig.json
{
    "compilerOptions": {
      	...
        "experimentalDecorators": true  // 요놈 true
    }
}
```



### 2. 데코레이터란?

- 자바의 annotation(`@`)과 비슷한 놈
- 데코레이터는 `@함수` 형식을 사용한다.
- 즉, 데코레이터를 선언할 때는 함수를 선언해주면 된다.



### 3. 데코레이터가 붙을 수 있는 곳

- 클래스 
- 메서드
- 프로퍼티
- 매개변수
- 접근자



### 4. 클래스 데코레이터 만들기

- 클래스 레벨에 붙여준다.
- 클래스 데코레이터는 **<u>클래스 생성자에 적용</u>**된다.
  - 클래스 정의를 관찰, 수정 또는 교체하는 데 사용할 수 있다.
- 클래스 데코레이터 표현식(함수)의 인수 1가지
  - 클래스의 생성자
- 클래스 데코레이터가 값을 반환하는 경우
  - 클래스가 선언을 제공하는 생성자 함수로 바꾼다.



- 예제1

  ```ts
  function ClassDecorator(constructor: Function) {
    console.log('constructor: ', constructor);
  }
  
  @ClassDecorator
  class Greeter { 
  	greeting: string;
    constructor(message: string) {
      this.greeting = message;
    }
    greet() {
      return "Hello, " + this.greeting;
    }
  }
  
  new Greeter('John');
  
  
  // 출력 결과
  constructor:  [class Greeter]
  ```



- 예제2 (생성자 재정의)

  ```ts
  function ClassDecorator<T extends {new(...args:any[]):{}}>(constructor:T) {
      return class extends constructor {
          newProperty = "new property";
          hello = "override";
      }
  }
  
  @ClassDecorator
  class Greeter {
      property = "property";
      hello: string;
      constructor(m: string) {
          this.hello = m;
      }
  }
  
  console.log(new Greeter("world"));
  ```





### 5. 메서드 데코레이터 만들기

- 메서드 레벨에 붙여준다.
- 메서드 데코레이터는 **<u>메서드의 프로퍼티 설명자(Property Descriptor)에 적용</u>**된다.
  - 메서드 정의를 관찰, 수정 또는 대체하는 데 사용할 수 있다.
- 메서드 데코레이터 표현식(함수)의 인수 3가지
  - 정적 멤버에 대한 클래스의 생성자 함수 또는 인스턴스 멤버에 대한 클래스의 프로토타입 
    - `target: any`
    - Static 메서드인 경우 target === 해당 클래스의 생성자 함수
    - 아닌 경우 target === 해당 클래스의 프로토타입
  - 멤버의 이름 
    - `propertyKey: string`
  - 멤버의 PropertyDescriptor 
    - `descriptor: PropertyDescriptor`



- 예제

  ```ts
  function Enumerable(value: boolean) {
    return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
      descriptor.enumerable = value;
    }
  }
  
  class Greeter {
    greeting: string;
    constructor(message: string) {
      this.greeting = message;
    }
    
    @Enumerable(false)
    greet() {
      return "Hello, " + this.greeting;
    }
  }
  ```

  

### 6. 프로퍼티 데코레이터

- 프로퍼티 레벨에 붙여준다.
- 프로퍼티 데코레이터의 표현식의 인수 2가지
  - 정적 멤버에 대한 클래스의 생성자 함수 또는 인스턴스 멤버에 대한 클래스의 프로토타입
    - `target: any`
    - Static 메서드인 경우 target === 해당 클래스의 생성자 함수
    - 아닌 경우 target === 해당 클래스의 프로토타입
  - 멤버의 이름
    - `propertyKey: string`













