# [Jest] Jest 레퍼런스 (1) Expect

## Expect

> https://jestjs.io/docs/expect

더 많은 Jest matcher들을 사용하고 싶을 때, [`jest-extended`](https://github.com/jest-community/jest-extended)를 사용해보라.





## Methods

> https://jestjs.io/docs/expect#methods



**한줄정리**

- [`expect(value)`](https://jestjs.io/docs/expect#expectvalue)
  - 테스트를 시작할 때 쓰는 함수. expectation 객체를 반환. 반환된 값에 matcher를 호출해서 assert 한다.
- [`expect.extend(matchers)`](https://jestjs.io/docs/expect#expectextendmatchers)
  - `expect.extend`를 사용해서 당신의 own matcher를 정의할 수 있다.
- [`expect.anything()`](https://jestjs.io/docs/expect#expectanything)
- [`expect.any(constructor)`](https://jestjs.io/docs/expect#expectanyconstructor)
- [`expect.arrayContaining(array)`](https://jestjs.io/docs/expect#expectarraycontainingarray)
- [`expect.assertions(number)`](https://jestjs.io/docs/expect#expectassertionsnumber)
- [`expect.hasAssertions()`](https://jestjs.io/docs/expect#expecthasassertions)
- [`expect.not.arrayContaining(array)`](https://jestjs.io/docs/expect#expectnotarraycontainingarray)
- [`expect.not.objectContaining(object)`](https://jestjs.io/docs/expect#expectnotobjectcontainingobject)
- [`expect.not.stringContaining(string)`](https://jestjs.io/docs/expect#expectnotstringcontainingstring)
- [`expect.not.stringMatching(string | regexp)`](https://jestjs.io/docs/expect#expectnotstringmatchingstring--regexp)
- [`expect.objectContaining(object)`](https://jestjs.io/docs/expect#expectobjectcontainingobject)
- [`expect.stringContaining(string)`](https://jestjs.io/docs/expect#expectstringcontainingstring)
- [`expect.stringMatching(string | regexp)`](https://jestjs.io/docs/expect#expectstringmatchingstring--regexp)
- [`expect.addSnapshotSerializer(serializer)`](https://jestjs.io/docs/expect#expectaddsnapshotserializerserializer)
- [`.not`](https://jestjs.io/docs/expect#not)
- [`.resolves`](https://jestjs.io/docs/expect#resolves)
- [`.rejects`](https://jestjs.io/docs/expect#rejects)
- [`.toBe(value)`](https://jestjs.io/docs/expect#tobevalue)
- [`.toHaveBeenCalled()`](https://jestjs.io/docs/expect#tohavebeencalled)
- [`.toHaveBeenCalledTimes(number)`](https://jestjs.io/docs/expect#tohavebeencalledtimesnumber)
- [`.toHaveBeenCalledWith(arg1, arg2, ...)`](https://jestjs.io/docs/expect#tohavebeencalledwitharg1-arg2-)
- [`.toHaveBeenLastCalledWith(arg1, arg2, ...)`](https://jestjs.io/docs/expect#tohavebeenlastcalledwitharg1-arg2-)
- [`.toHaveBeenNthCalledWith(nthCall, arg1, arg2, ....)`](https://jestjs.io/docs/expect#tohavebeennthcalledwithnthcall-arg1-arg2-)
- [`.toHaveReturned()`](https://jestjs.io/docs/expect#tohavereturned)
- [`.toHaveReturnedTimes(number)`](https://jestjs.io/docs/expect#tohavereturnedtimesnumber)
- [`.toHaveReturnedWith(value)`](https://jestjs.io/docs/expect#tohavereturnedwithvalue)
- [`.toHaveLastReturnedWith(value)`](https://jestjs.io/docs/expect#tohavelastreturnedwithvalue)
- [`.toHaveNthReturnedWith(nthCall, value)`](https://jestjs.io/docs/expect#tohaventhreturnedwithnthcall-value)
- [`.toHaveLength(number)`](https://jestjs.io/docs/expect#tohavelengthnumber)
- [`.toHaveProperty(keyPath, value?)`](https://jestjs.io/docs/expect#tohavepropertykeypath-value)
- [`.toBeCloseTo(number, numDigits?)`](https://jestjs.io/docs/expect#tobeclosetonumber-numdigits)
- [`.toBeDefined()`](https://jestjs.io/docs/expect#tobedefined)
- [`.toBeFalsy()`](https://jestjs.io/docs/expect#tobefalsy)
- [`.toBeGreaterThan(number | bigint)`](https://jestjs.io/docs/expect#tobegreaterthannumber--bigint)
- [`.toBeGreaterThanOrEqual(number | bigint)`](https://jestjs.io/docs/expect#tobegreaterthanorequalnumber--bigint)
- [`.toBeLessThan(number | bigint)`](https://jestjs.io/docs/expect#tobelessthannumber--bigint)
- [`.toBeLessThanOrEqual(number | bigint)`](https://jestjs.io/docs/expect#tobelessthanorequalnumber--bigint)
- [`.toBeInstanceOf(Class)`](https://jestjs.io/docs/expect#tobeinstanceofclass)
- [`.toBeNull()`](https://jestjs.io/docs/expect#tobenull)
- [`.toBeTruthy()`](https://jestjs.io/docs/expect#tobetruthy)
- [`.toBeUndefined()`](https://jestjs.io/docs/expect#tobeundefined)
- [`.toBeNaN()`](https://jestjs.io/docs/expect#tobenan)
- [`.toContain(item)`](https://jestjs.io/docs/expect#tocontainitem)
- [`.toContainEqual(item)`](https://jestjs.io/docs/expect#tocontainequalitem)
- [`.toEqual(value)`](https://jestjs.io/docs/expect#toequalvalue)
- [`.toMatch(regexp | string)`](https://jestjs.io/docs/expect#tomatchregexp--string)
- [`.toMatchObject(object)`](https://jestjs.io/docs/expect#tomatchobjectobject)
- [`.toMatchSnapshot(propertyMatchers?, hint?)`](https://jestjs.io/docs/expect#tomatchsnapshotpropertymatchers-hint)
- [`.toMatchInlineSnapshot(propertyMatchers?, inlineSnapshot)`](https://jestjs.io/docs/expect#tomatchinlinesnapshotpropertymatchers-inlinesnapshot)
- [`.toStrictEqual(value)`](https://jestjs.io/docs/expect#tostrictequalvalue)
- [`.toThrow(error?)`](https://jestjs.io/docs/expect#tothrowerror)
- [`.toThrowErrorMatchingSnapshot(hint?)`](https://jestjs.io/docs/expect#tothrowerrormatchingsnapshothint)
- [`.toThrowErrorMatchingInlineSnapshot(inlineSnapshot)`](https://jestjs.io/docs/expect#tothrowerrormatchinginlinesnapshotinlinesnapshot)







## 1. `expect.extend(custom-matchers)` (커스텀 매처 정의하기)

- 예제 (`toBeWithinRange`라는 custom matcher를 직접 정의)

```js
expect.extend({
  toBeWithinRange(received, floor, ceiling) {
    const pass = received >= floor && received <= ceiling;
    
    if(pass) {
      return {
        message: () => `expected ${received} not to be within range ${floor} - ${ceiling}`,  // expect(머시기).not.toBeWithinRange() 실패시 나오는 메시지
        pass: true,
      }
    } else {
      return {
        message: () => `expected ${received} to be within range ${floor} - ${ceiling}`, // expect(머시기).toBeWithinRange() 실패시 나오는 메시지
        pass: false,
      }
    }
  },
});

test('numeric ranges', () => {
  expect(100).toBeWithinRange(90, 110);
  expect(101).not.toBeWithinRange(0, 100);
  expect({apples: 6, bananas: 3}).toEqual({
    apples: expect.toBeWithinRange(1, 10),
    bananas: expect.not.toBeWithinRange(11, 20),
  });
});
```



### **TypeScript**

- TS에서, `@types/jest`를 사용하고 있는 경우, 당신은 아래와 같이 `toBeWithinRange` matcher를 imported module 안에 새롭게 정의할 수 있다.

  ```ts
  declare global {
    namespace jest {
      interface Matchers<R> {
        toBeWithinRange(a: number, b: number): R;
      }
    }
  }
  ```

  





### **Async Matchers**

- `expect.extend`로 async matcher도 정의할 수 있다.

- Async matcher들은 Promise를 반환하므로 당신은 반환 값을 await 해야 할 것이다.

- 예제 (`toBeDivisibleByExternalValue` 커스텀 매쳐 정의하기)

  ```js
  expect.extend({
    async toBeDivisibleByExternalValue(received) {
      const externalValue = await getExternalValueFromRemoteSource();
      const pass = received % externalValue == 0;
      
      if (pass) {
        return {
          message: () =>
            `expected ${received} not to be divisible by ${externalValue}`,
          pass: true,
        };
      } else {
        return {
          message: () =>
            `expected ${received} to be divisible by ${externalValue}`,
          pass: false,
        };
      }
    },
  });
  
  
  test('is divisible by external value', async () => {
    await expect(100).toBeDivisibleByExternalValue();
    await expect(101).not.toBeDivisibleByExternalValue();
  });
  ```

  

### **Custom Matchers API**

- Matcher들은 **<u>반드시!!</u>** 두 개의 Key(`pass`와 `message`) 를 가진 객체(또는 객체의 Promise)를 반환해야 한다.

- `pass`: 매치된 경우 true, 안된 경우 false

- `message`: 실패했을 경우 error message를 반환하는 함수 (no arguments)

  

- 이게 좀 헷갈리긴 한데, 

  - `pass: true`랑 같이 있는 message는 `expect(x).not.yourMatcher()`가 실패했을 때 나오는 메시지고,
  - `pass: false`랑 같이 있는 message는 `expect(x).yourMatcher()`가 실패했을 때 나오는 메시지다.



- Matcher들은 `expect(x)`에 전달된 인수(`x`)와 뒤에 따라 나오는 `.yourMatcher(y, z)`의 인수들(`y`, `z`)을 통해 호출된다.

```js
expect.extend({
	yourMatcher(x, y, z) {
    return {
      pass: true,
      message: () => '',
    }
  }
})
```





### **helper functions and properties**

- 커스텀 매쳐를 정의할 때 helper function들과 property들을 사용하고 싶다면 `this`에서 찾을 수 있다.

- `this.isNot`

  - 이 matcher가 `.not` modifier와 함께 호출되었는지를 알려주는 boolean 값

- `this.promise`

  - promise와 관련된 matcher hint를 보여주는 string 값
  - 만약 matcher가 `.rejects` modifier와 함께 호출된 경우, `'rejects'` 반환
  - 만약 matcher가 `.resolves` modifier와 함께 호출된 경우, `'resolves'` 반환
  - 만약 matcher가  modifier와 함께 호출되지 않은 경우, `''` 반환

- `this.equals(a, b)`

  - Deep-equality function
  - a와 b 객체가 같은 값을 가지고 있으면 `true` 반환 (recursivley check)

- `this.expand`

  - 이 matcher가 `expand` 옵션과 함께 호출되었음을 알려주는 boolean 값
  - Jest가 `--expand` flag와 함께 호출되는 경우, `this.expand`는 Jest가 전체 diff 및 오류를 표시할 것으로 예상되는지 여부를 결정하는 데 사용할 수 있습니다.

- `this.utils`

  - `this.utils`에는 주로  [`jest-matcher-utils`](https://github.com/facebook/jest/tree/main/packages/jest-matcher-utils)의 exports로 구성된 유용한 도구가 많이 있습니다.
  - 그 중에서도 유용한 것은 오류 메시지를 보기좋게 형식화하기 위한 `matcherHint`, `printExpected` 및 `printReceived`입니다.
  - 예를 들어, 아래와 같은 `toBe` matcher 구현을 봅시다.

  ```js
  const {diff} = require('jest-diff');
  expect.extend({
    toBe(received, expected) {
      const options = {
        comment: 'Object.is equality',
        isNot: this.isNot,
        promise: this.promise,
      };
  
      const pass = Object.is(received, expected);
  
      const message = pass
        ? () =>
            this.utils.matcherHint('toBe', undefined, undefined, options) +
            '\n\n' +
            `Expected: not ${this.utils.printExpected(expected)}\n` +
            `Received: ${this.utils.printReceived(received)}`
        : () => {
            const diffString = diff(expected, received, {
              expand: this.expand,
            });
            return (
              this.utils.matcherHint('toBe', undefined, undefined, options) +
              '\n\n' +
              (diffString && diffString.includes('- Expect')
                ? `Difference:\n\n${diffString}`
                : `Expected: ${this.utils.printExpected(expected)}\n` +
                  `Received: ${this.utils.printReceived(received)}`)
            );
          };
  
      return {actual: received, message, pass};
    },
  });
  ```

  테스트하면 이렇게 나온다.

  ```js
    expect(received).toBe(expected)
  
      Expected value to be (using Object.is):
        "banana"
      Received:
        "apple"
  ```

  



### **스냅샷 테스팅??**

> https://www.daleseo.com/jest-snapshot/





## 2. `expect.anything()`

- `expect.anything()`은 `null`과 `undefined`를 제외한 모든 것과 매칭된다.

- 이 메서드는 `toEqual`이나 `toBeCalledWith` 안에서 literal value 대신 사용될 수 있다.

- 예제

  ```js
  test('map calls its argument with a non-null argument', () => {
    const mock = jest.fn();
    [1].map(x => mock(x));
    expect(mock).toBeCalledWith(expect.anything());
  });
  ```

  - mock function과 함께 호출된 인자가 `null`도 아니고 `undefined`도 아님을 assert 하는 예제



## 3. `expect.any(constructor)`

- `expect.any(constructor)`는 주어진 constructor로 생성된 것과 매칭된다.

- 이 메서드는 `toEqual`이나 `toBeCalledWith` 안에서 literal value 대신 사용될 수 있다.

- 예제

  ```js
  function randocall(fn) {
    return fn(Math.floor(Math.random() * 6 + 1));
  }
  
  test('randocall calls its callback with a number', () => {
    const mock = jest.fn();
    randocall(mock);
    expect(mock).toBeCalledWith(expect.any(Number));
  });
  ```

  

## 4. `expect.arrayContaining(array)`

- `expect.arrayContaining(array)`는 received array가 expected array의 모든 요소를 포함하고 있으면 일치합니다.

- 즉, expected array가 received array의 부분집합인 경우 일치합니다.

  

- You can use it instead of a literal value:

  - in `toEqual` or `toBeCalledWith`

  - to match a property in `objectContaining` or `toMatchObject`



```js
describe('arrayContaining', () => {
  const expected = ['Alice', 'Bob'];
  it('matches even if received contains additional elements', () => {
    expect(['Alice', 'Bob', 'Eve']).toEqual(expect.arrayContaining(expected));
  });
  it('does not match if received does not contain expected elements', () => {
    expect(['Bob', 'Eve']).not.toEqual(expect.arrayContaining(expected));
  });
});
```

```js
describe('Beware of a misunderstanding! A sequence of dice rolls', () => {
  const expected = [1, 2, 3, 4, 5, 6];
  it('matches even with an unexpected number 7', () => {
    expect([4, 1, 6, 7, 3, 5, 2, 5, 4, 6]).toEqual(
      expect.arrayContaining(expected),
    );
  });
  it('does not match without an expected number 2', () => {
    expect([4, 1, 6, 7, 3, 5, 7, 5, 4, 6]).not.toEqual(
      expect.arrayContaining(expected),
    );
  });
});
```



## 5. `expect.assertions(number)`

- `expect.assertions(number)`는 













