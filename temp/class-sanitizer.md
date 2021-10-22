# class-sanitizer

> https://www.npmjs.com/package/class-sanitizer
>
> **DEPRECATION NOTICE:**
> This library is considered to be deprecated and won't be updated anymore. Please use the [class-transformer](https://github.com/typestack/class-transformer) and/or [class-validator](https://github.com/typestack/class-validator) libraries instead.



### 뭐하는 놈인가

- Decorator based class property sanitation in Typescript powered by [validator.js](https://github.com/chriso/validator.js).



### 사용법

- 이 라이브러리를 사용하려면 클래스를 생성하고 클래스의 프로퍼티에 sanitization decorator들을 붙여보자.
- `sanitize(instance)`를 호출할 때, 라이브러리가 프로퍼티에 붙여진 decorator를 적용시켜서 marked된 모든 property들의 값을 각각 업데이트시킨다.

> **NOTE:**
>
> 모든 sanitization decorator는 **<u>property</u>** decorator입니다.
>
> 다시 말해, parameters나 class definition에 붙일 수는 없습니다. 
>
> **<u>오직 클래스의 property에만 붙일 수 있습니다.</u>**

- 예제1

  ```javascript
  import { sanitize, Trim } from 'class-sanitizer';
   
  class TestClass {
    @Trim()
    label: string;
   
    constructor(label: string) {
      this.label = label;
    }
  }
   
  const instance = new TestClass(' text-with-spaces-on-both-end ');
   
  sanitize(instance);
  // -> sanitize(instance) 호출 시 label property가 trimmed 된다.
  // -> { label: 'text-with-spaces-on-both-end' }
  ```













