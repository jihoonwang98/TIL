# Javascript 'this' 바인딩 정리

> https://medium.com/sjk5766/javascript-this-binding-%EC%A0%95%EB%A6%AC-ae84e2499962
>
> https://yuddomack.tistory.com/entry/%EC%9E%90%EB%B0%94%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8-this%EC%9D%98-4%EA%B0%80%EC%A7%80-%EB%8F%99%EC%9E%91-%EB%B0%A9%EC%8B%9D





- `this` 바인딩 종류
  - default binding
  - 함수 내부의 `this` binding
  - 즉시 실행함수 내부의 `this` binding
  - 객체의 메서드 `this` binding
  - `new` 키워드(생성자 함수)와 `this` binding



### \# 0. 파일 모드 vs node REPL(쉘)

- 파일 모드

  - Local과 Global

    ![image-20211104095726463](/Users/mojo/Library/Application Support/typora-user-images/image-20211104095726463.png)

    - Local의 `exports` === Local의 `this` === Local의 `module.exports`

    - 예제

      ![image-20211104100008496](/Users/mojo/Library/Application Support/typora-user-images/image-20211104100008496.png)

      ![image-20211104100040288](/Users/mojo/Library/Application Support/typora-user-images/image-20211104100040288.png)

      

  - 기본 바인딩

    ```js
    console.log(this);   // {}
    console.log(this === exports);   // true   
    console.log(this === module.exports);   // true
    console.log(this === global);   // false
    ```

    - 파일 모드에서 `this`는 `exports` 객체로 기본 바인딩된다.

  - 함수 내부의 `this` 바인딩

    ```js
    function checkThisInFunction() {
      console.log(this);
      console.log(this === global);
    }
    
    checkThisInFunction();
    
    [출력 결과]
    Object [global] {
      global: [Circular],
      clearInterval: [Function: clearInterval],
      clearTimeout: [Function: clearTimeout],
      setInterval: [Function: setInterval],
      setTimeout: [Function: setTimeout] {
        [Symbol(nodejs.util.promisify.custom)]: [Function]
      },
      queueMicrotask: [Function: queueMicrotask],
      clearImmediate: [Function: clearImmediate],
      setImmediate: [Function: setImmediate] {
        [Symbol(nodejs.util.promisify.custom)]: [Function]
      }
    }
    true
    ```

    - 근데 또 함수 내부에서의 `this`는 global 객체로 binding된다.

  - 즉시 실행함수 내부의 `this` 바인딩

    ```js
    (function () {
      console.log('this: ', this);
    })();
    
    [출력 결과]
    this:  Object [global] {
      global: [Circular],
      clearInterval: [Function: clearInterval],
      clearTimeout: [Function: clearTimeout],
      setInterval: [Function: setInterval],
      setTimeout: [Function: setTimeout] {
        [Symbol(nodejs.util.promisify.custom)]: [Function]
      },
      queueMicrotask: [Function: queueMicrotask],
      clearImmediate: [Function: clearImmediate],
      setImmediate: [Function: setImmediate] {
        [Symbol(nodejs.util.promisify.custom)]: [Function]
      }
    }
    ```



### \# 1. default binding

- 기본적으로 `this`는 **전역 객체**를 가리키게 된다.

  - `Node` 환경에서는 `global` 객체를,

    ![image-20211104091613018](/Users/mojo/Library/Application Support/typora-user-images/image-20211104091613018.png)

  - `Browser` 환경에서는 `Window` 객체를 가리킨다.

    ![](https://miro.medium.com/max/1178/1*7U42eKY6jWuP0g3OkKsc6w.png)

  



- 파일로 작성했을 때

  ![image-20211104091710373](/Users/mojo/Library/Application Support/typora-user-images/image-20211104091710373.png)

  - 빈 객체로 나온다.
  - 파일로 생성해서 `node` 명령으로 실행하는 경우에는 `context`가 `module.exports`를 가리킨다고 한다.





### \#2. 함수 내부의 this binding

- 일반적인 함수 내부에서 `this`를 호출하면 전역 객체를 가리킨다.

- 예제 (파일로 실행한 경우)

  ```js
  console.log('this: ', this);
  console.log('this === global:', (this === global));
  console.log();
  
  function checkThisInNormalFunc() {
    console.log('[checkThisInNormalFunc] this: ', this);
    console.log('[checkThisInNormalFunc] this === global: ', (this === global));
  }
  checkThisInNormalFunc();
  
  [출력 결과]
  this:  {}
  this === global: false
  
  
  [checkThisInNormalFunc] this:  Object [global] {
    global: [Circular],
    clearInterval: [Function: clearInterval],
    clearTimeout: [Function: clearTimeout],
    setInterval: [Function: setInterval],
    setTimeout: [Function: setTimeout] {
      [Symbol(nodejs.util.promisify.custom)]: [Function]
    },
    queueMicrotask: [Function: queueMicrotask],
    clearImmediate: [Function: clearImmediate],
    setImmediate: [Function: setImmediate] {
      [Symbol(nodejs.util.promisify.custom)]: [Function]
    }
  }
  [checkThisInNormalFunc] this === global:  true
  ```

  

- 만약 함수 내부 or 외부에서 `strict` 모드를 사용한다면 함수 내부에서 this는 전역객체를 binding하지 않습니다.

  - 함수 내부

    ```js
    function checkThisInStrict() {
      'use strict';
      console.log(this);  // undefined
    }
    
    checkThisInStrict(); 
    ```
    - `this`는 그냥 `undefined`로 나온다.

  - 함수 외부

    ```js
    'use strict';
    
    function checkThisInStrict() {
      console.log(this);
    }
    
    function checkThisInStrict2() {
      console.log(this);
    }
    
    checkThisInStrict();   // undefined
    checkThisInStrict2();   // undefined
    console.log(this);   // {} (파일로 실행)
    ```

    - 함수 외부에서 `strict` 모드를 사용하면 모든 함수 내부의 `this`는 전역 객체를 binding하지 않습니다.



### \#3. 즉시 실행함수 내부의 `this` binding

```js
(function () {
  console.log(this);
  console.log(this === global);
})();

[출력 결과]
Object [global] {
  global: [Circular],
  clearInterval: [Function: clearInterval],
  clearTimeout: [Function: clearTimeout],
  setInterval: [Function: setInterval],
  setTimeout: [Function: setTimeout] {
    [Symbol(nodejs.util.promisify.custom)]: [Function]
  },
  queueMicrotask: [Function: queueMicrotask],
  clearImmediate: [Function: clearImmediate],
  setImmediate: [Function: setImmediate] {
    [Symbol(nodejs.util.promisify.custom)]: [Function]
  }
}
true
```



- `strict` 모드로 실행

```js
'use strict';
(function () {
  console.log(this);
  console.log(this === global);
})();

[출력 결과]
undefined
false
```





### \#4. 객체의 메서드 `this` binding

- 객체 내부의 메서드에서 `this`를 binding할 경우, 그 객체 자신을 가리키게 된다.

```js
var obj = {
  name: "seo",
  print: function() {
    console.log(this.name);
    console.log(this === obj);
  }
}

obj.print();
```



- **이때, `this`가 참조되는 위치가 중요한게 아니라 <u>어디서, 어떻게</u> `this`를 포함하는 코드를 호출하느냐가 중요합니다.**

```js
var obj1 = {
  name: "seo",
  print: function() {
    console.log('this: ', this);
    console.log(this.name);
  }
}

var obj2 = {
  name: "wang",
  print: obj1.print
};


global.name = "ji";
var print = obj1.print;

obj1.print();
obj2.print();
print();


[출력 결과]
this:  { name: 'seo', print: [Function: print] }
seo
this:  { name: 'wang', print: [Function: print] }
wang
this:  Object [global] {
  global: [Circular],
  clearInterval: [Function: clearInterval],
  clearTimeout: [Function: clearTimeout],
  setInterval: [Function: setInterval],
  setTimeout: [Function: setTimeout] {
    [Symbol(nodejs.util.promisify.custom)]: [Function]
  },
  queueMicrotask: [Function: queueMicrotask],
  clearImmediate: [Function: clearImmediate],
  setImmediate: [Function: setImmediate] {
    [Symbol(nodejs.util.promisify.custom)]: [Function]
  },
  name: 'ji'
}
ji
```



> **파일 모드 vs node REPL 환경(쉘)** 
>
> - 파일 모드
>
>   ```js
>   var obj = {
>     print: function () {
>       console.log('this: ', this);
>       console.log('this.name: ', this.name);
>     }
>   }
>   
>   var name = 'ji';
>   global.name = 'global-name';
>   var print = obj.print;
>   print();
>   
>   [출력 결과]
>   this:  Object [global] {
>     global: [Circular],
>     clearInterval: [Function: clearInterval],
>     clearTimeout: [Function: clearTimeout],
>     setInterval: [Function: setInterval],
>     setTimeout: [Function: setTimeout] {
>       [Symbol(nodejs.util.promisify.custom)]: [Function]
>     },
>     queueMicrotask: [Function: queueMicrotask],
>     clearImmediate: [Function: clearImmediate],
>     setImmediate: [Function: setImmediate] {
>       [Symbol(nodejs.util.promisify.custom)]: [Function]
>     },
>     name: 'global-name'
>   }
>   this.name:  global-name
>   ```
>
>   



---

헷갈리니까 그냥 일단 Node REPL 환경에서 어떻게 나오나부터 정리



### \#1. this의 첫번째 동작 방식 - default binding (전역객체) 

- 브라우저

  ![](https://t1.daumcdn.net/cfile/tistory/99C83E4F5E31236912)

- node 쉘

  ![](https://t1.daumcdn.net/cfile/tistory/99E17F4C5E3123E90A)

- 얘네들의 정체는 바로 javascript 실행 환경의 전역 객체다.

- 그렇다. **<u>this의 첫번째 동작 방식은, this가 전역 객체(window, global)를 context 객체로 갖는 것이다.</u>**

- 예제

  ```js
  var g = 20;
  console.log(this.g); // 20
  
  function doSomething() {
    this.dummy2 = "가을";
    console.log(this); // window 객체
  }
  
  console.log(this.dummy1); // undefined
  console.log(this.dummy2); // undefined
  
  this.dummy1 = "겨울";
  
  console.log(this.dummy1); // 겨울
  console.log(this.dummy2); // undefined
  
  doSomething();
  
  console.log(this.dummy1); // 겨울
  console.log(this.dummy2); // 가을
  ```

  - 잠시 눈여겨 봐야할 사실로, 전역 스코프에서 정의한 변수들은 전역 객체에 등록된다.
  - 때문에 `var g = 20`은 `global.g = 20`과 같고 `this` 객체에 프로퍼티(dummy1, dummy2)를 등록하는 것은 사실상 전역 스코프에서 변수를 선언한 것과 같다.

- 추가로 전역 객체는 우리가 알고 있는 일반 객체(object)처럼 아무 제약 없이 동작하기 때문에 `this`를 무분별하게 사용할 경우, 전역 상태에 영향을 끼칠 수 있으므로 주의하자.



### \#2. this의 두번째 동작 방식 - 암시적(implicit) 바인딩 

- 객체의 메서드에서의 `this` 바인딩

  - 어떤 객체를 통해 함수가 호출된다면 그 객체가 바로 this의 context 객체가 된다.

- 암시적 바인딩을 간단하게 표현하면 아래와 같습니다.

  ```js
  function test() {
    console.log(this.a);
  }
  
  var obj = {
    a: 20,
    func1: test,
    func2: function() {
      console.log(this.a);
    }
  };
  
  obj.func1(); // 20
  obj.func2(); // 20
  ```

  - 위 코드의 `func1`, `func2`는 `obj`를 통해 호출되었으므로, `obj`가 `this`가 된다는 뜻

- 이 이야기는 곧 첫번째 동작 방식(기본 바인딩)과 연관지을 수 있는데, 아래 코드를 보자.

  ```js
  var b = 100;
  
  function test() {
    console.log(this.b);
  }
  
  var obj = {
    a: 20,
    func1: test,
    func2: function() {
      console.log(this.b);
    }
  };
  
  obj.func1(); // undefined
  obj.func2(); // undefined
  
  var gFunc1 = obj.func1;   // 왼쪽 statement는 this.gFunc1 = obj.func1과 같다.
  gFunc1(); // 100   // 왼쪽 statement는 this.gFunc1();과 같다.
  ```

- 그리고 헷갈림의 여지를 남기지 않기 위해 모종의 객체를 생성한 예를 하나 더 봅시다.

  ```js
  function test() {
    console.log(this.b);
  }
  
  var obj1 = {
    b: 10,
    func: test
  };
  
  var obj2 = {
    b: 40
  };
  
  obj2.func = obj1.func; // var func = obj1.func 와 같다
  obj2.func(); // 40
  ```

  



### \#3. this의 세번째 동작 방식 - 명시적 바인딩

- `call`, `apply`, `bind` 메서드를 통해 명시적으로 `this`가 가리키는 것을 정해줄 수 있음

```js
function test() {
  console.log(this);
}

var obj = { name: "yuddomack" };
test.call(obj); // { name: 'yuddomack' }
test.call("원시 네이티브 자료들은 wrapping 됩니다"); // [String: '원시 네이티브 자료들은 wrapping 됩니다']
```



### \#4. this의 네번째 동작 방식 - new 바인딩

```js
function foo(a) {
  this.a = a;
  this.qwer = 20;
}

var bar = new foo(2);
console.log(bar.a); // 2
console.log(bar.qwer); // 20
```







정리중...



























