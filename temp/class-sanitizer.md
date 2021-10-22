# class-sanitizer

> https://www.npmjs.com/package/class-sanitizer
>
> **DEPRECATION NOTICE:**
> This library is considered to be deprecated and won't be updated anymore. Please use the [class-transformer](https://github.com/typestack/class-transformer) and/or [class-validator](https://github.com/typestack/class-validator) libraries instead.



## 뭐하는 놈인가

- Decorator based class property sanitation in Typescript powered by [validator.js](https://github.com/chriso/validator.js).



## 사용법

- 이 라이브러리를 사용하려면 클래스를 생성하고 클래스의 프로퍼티에 sanitization decorator들을 붙여보자.
- `sanitize(instance)`를 호출할 때, 라이브러리가 프로퍼티에 붙여진 decorator를 적용시켜서 marked된 모든 property들의 값을 각각 업데이트시킨다.

> **NOTE:**
>
> 모든 sanitization decorator는 **<u>property</u>** decorator입니다.
>
> 다시 말해, parameters나 class definition에 붙일 수는 없습니다. 
>
> **<u>오직 클래스의 property에만 붙일 수 있습니다.</u>**

- 예제 (`@Trim()`)

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



### Validating arrays

- Every decorator expects a `SanitationOptions` object.

- `each` 프로퍼티가 `true`로 설정된 경우 해당 배열은 iterated 되면서 해당 배열의 모든 원소에 decorator가 적용됩니다.

- 예제

  ```javascript
  import { sanitize, Trim } from 'class-sanitizer';
   
  class TestClass {
    @Trim(undefined, { each: true })
    labels: string[];
   
    constructor(labels: string[]) {
      this.labels = labels;
    }
  }
   
  const instance = new TestClass([' labelA ', ' labelB', 'labelC ']);
   
  sanitize(instance);
  // -> Every value is trimmed in instance.labels now.
  // -> { labels: ['labelA', 'labelB', 'labelC']}
  ```

  

### Inheritance

- 클래스 상속 기능도 지원한다.
- base-class에 정의된 모든 데코레이터는 descendant-class의 동일한 이름을 가진 속성에 적용됩니다. (단, descendant-class에 속성이 있는 경우)



....



## Sanitization decorators

The following property decorators are available.

| Decorator                              | Description                                                  |
| -------------------------------------- | ------------------------------------------------------------ |
| `@Blacklist(chars: string)`            | Removes all characters that appear in the blacklist.         |
| `@Whitelist(chars: string)`            | Removes all characters that don't appear in the whitelist.   |
| `@Trim(chars?: string)`                | Trims characters (whitespace by default) from both sides of the input. You can specify chars that should be trimmed. |
| `@Ltrim(chars?: string)`               | Trims characters from the left-side of the input.            |
| `@Rtrim(chars?: string)`               | Trims characters from the right-side of the input.           |
| `@Escape()`                            | Replaces <, >, &, ', " and / with HTML entities.             |
| `@NormalizeEmail(lowercase?: boolean)` | Normalizes an email address.                                 |
| `@StripLow(keepNewLines?: boolean)`    | Removes characters with a numerical value < 32 and 127, mostly control characters. |
| `@ToBoolean(isStrict?: boolean)`       | Converts the input to a boolean. Everything except for '0', 'false' and '' returns true. In strict mode only '1' and 'true' return true. |
| `@ToDate()`                            | Converts the input to a date, or null if the input is not a date. |
| `@ToFloat()`                           | Converts the input to a float, or NaN if the input is not an integer. |
| `@ToInt(radix?: number)`               | Converts the input to an integer, or NaN if the input is not an integer. |
| `@ToString()`                          | Converts the input to a string.                              |