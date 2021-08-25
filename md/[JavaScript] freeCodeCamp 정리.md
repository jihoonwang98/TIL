# [JavaScript] freeCodeCamp 정리

 



## 1. JavaScript 주석

- `// ...`: 한줄 주석
- `/* ... */`: multiline 주석



## 2. JavaScript Data Types



**8가지 자료형**

- `undefined`
- `null`
- `boolean`
- `string`
- `symbol`
- `bigint`
- `number`
- `object`



**변수 선언 -> `var` 키워드**

```javascript
var outName;
```





## 3. Uninitialized Variables

JavaScript 변수들은 초기화없이 선언됐을 경우, `undefined` 값을 default로 갖는다.

`undefined` 값에 수학적인 연산을 한다면 `NaN` ("Not a Number") 값을 얻게 될 것이다.

만약 문자열과 `undefined` 값을 concatenate 연산한다면, `undefined` 값은 "undefined" 문자열로 평가된다.



```javascript
var str = "asdf";
var a;

str + a === "asdfundefined";
```





## 4. Escaping Literal Quotes in Strings

String을 정의할 때 반드시 `""`나 `''`로 감싸줘야 한다.

그런데 만약 String 안에 `""`나 `''`를 포함하고 싶다면 어떻게 해야할까?





### 첫번째 방법

JavaScript에서는, quote 앞에 백슬래시(\\)를 붙여서 escape 할 수 있다.

```javascript
var sampleStr = "Alan said, \"Peter is learning JavaScript\".";

console.log(sampleStr);
// Alan said, "Peter is learning JavaScript".
```





### 두번째 방법

> 참고
>
> JavaScript의 String 값들은 `""`나 `''`로 감싸진다.
>
> 다른 PL과는 다르게, JavaScript에서는 두 quotes들이 동일하게 동작한다.



String 안에서 `''`를 쓰고 싶으면, `""`로 시작하고 끝내면 된다.

```javascript
var str = "Let's go!!";

console.log(str);
// Let's go!!
```



```javascript
var myStr = "<a href='http://www.example.com' target='_blank'>Link</a>";
```





## 5. JavaScript String



### Char 접근

```javascript
var str = "Charles";
str[0] === "C"

typeof(str[0]) === "string"
```

JavaScript에서는 Character가 없다. 한글자도 그냥 String 취급함.





### String Immutability

JavaScript에서는, `String` 값들은 immutable하다.

즉, 한번 생성되면 바뀔 수 없다.



```javascript
var myStr = "Bob";
myStr[0] = "J";
```

위의 코드로 `myStr`의 값을 `Job`으로 바꿀 수 없다. (`String`은 immutable이기 때문에)



`myStr`의 값을 바꾸는 방법은 새로운 string 값을 assign 하는 것이다.

```javascript
var myStr = "Bob";
myStr = "Job";
```





## 6. JavaScript Arrays

JavaScript에서는 배열에 서로 다른 타입의 값들을 담을 수 있다.

```javascript
var arr = ["string", 30];
```



String과는 다르게, Array의 entry들은 mutable하며, 자유롭게 변경 가능하다.

```javascript
var arr = [1, 2, 3];
arr[0] = 10;
```





### 메서드



**push()**

```javascript
var arr1 = [1,2,3];
arr1.push(4);

var arr2 = ["Stimpson", "J", "cat"];
arr2.push(["happy", "joy"]);
```

- Array의 **push()** 메서드는 array의 끝에 data들을 append한다.





**pop()**

```javascript
var threeArr = [1, 4, 6];
var oneDown = threeArr.pop();
console.log(oneDown);  // 6
console.log(threeArr); // [1, 4]
```

- Array의 **pop()** 메서드는 array의 끝에서 요소를 하나 뽑고 리턴한다.





**shift()**

```javascript
var ourArray = ["Stimpson", "J", ["cat"]];
var removedFromOurArray = ourArray.shift();
console.log(ourArray);    // ["J", ["cat"]]
console.log(removedFromOurArray);  // "Stimpson"
```

- Array의 **shift()** 메서드는 array의 앞에서 요소를 하나 뽑고 리턴한다.





**unshift()**

```javascript
var ourArray = ["Stimpson", "J", "cat"];
ourArray.shift();
ourArray.unshift("Happy");

console.log(ourArray);  // ["Happy", "J", "cat"]
```

- Array의 **unshift()** 메서드는 array의 앞에 요소를 하나 집어 넣을 수 있다.







## 7. JavaScript Functions

**용어**

- `parameter`: placeholder 변수
- `argument`: 실제 넘어가는 값





### 함수 선언법

```javascript
function functionName() {
  console.log("Hello World");
}

// call or invoke above function
functionName();
```



```javascript
function functionWithArgs(arg1, arg2) {
  console.log(arg1 + arg2);
}
```

- JavaScript에서는 파라미터에 타입명을 안써준다.





### Global Scope and Functions

- **Global Scope 변수**
  - function block 밖에서 선언된 변수들
  - JavaScript 코드의 어떤 곳에서나 접근 가능함
  - `var` 키워드 없이 선언된 변수들은 자동적으로 global scope를 갖게 됨



```javascript
var myGlobal = 10;    //  글로벌 변수 선언

function fun1() {
  // Assign 5 to oopsGlobal Here
  oopsGlobal = 5;     // var 키워드가 없으므로 글로벌 변수로 선언됨
}
```





### Local Scope and Functions

- **Local Scope 변수**
  - function block 안에서 선언된 변수들 
  - 함수 파라미터도 여기에 해당됨
  - 해당 function block 안에서만 접근 가능함



```javascript
function myTest() {
  var loc = "foo";
  console.log(loc);
}
myTest();
console.log(loc);  // error, 변수 loc은 function myTest 밖에서는 접근 불가능함
```





### Global vs. Local Scope in Functions

같은 이름으로 local과 global 변수 둘 다 가지는 것도 가능하다.

이렇게 할 경우, **local 변수가 global 변수보다 우선순위를 가진다.**

```javascript
var someVar = "Hat";
function myFun() {
  var someVar = "Head";
  return someVar;
}
```

위의 코드에서 함수 myFun()은 "Head" 문자열을 반환한다.







### Undefined Value returned from a Function

함수는 `return`문을 포함할 수 있지만 의무는 아니다.

함수가 `return`문을 포함하지 않은 경우 반환 값은 `undefined`가 된다.







## 8. 비교 연산자

- equality operator `==`

```javascript
1 == 1   (true)
1 == 2   (false)
1 == '1' (true)
"3" == 3 (true)
```

`==` 연산자는 양변의 값을 같은 자료형으로 맞춰서 비교한다.





- **strict** equality operator `===`

```javascript
1   ===  1   (true)
1   ===  2   (false)
1   === '1'  (false)
"3" ===  3   (false)
```

`===` 연산자는 형변환을 해주지 않는다.

양변의 데이터가 타입이 다르면 unequal하다고 평가한다.





- `>`, `>=`, `<`, `<=`

`>`(`>=`, `<`, `<=`) 연산자는 `==` 연산자처럼 양변의 자료형이 다르면 형변환을 해준다.

```javascript
5   >  3   (true)
7   > '3'  (true)
2   >  3   (false)
'1' >  9   (false)
```

















