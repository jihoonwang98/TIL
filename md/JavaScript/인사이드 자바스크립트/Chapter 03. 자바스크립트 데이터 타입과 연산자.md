# Chapter 03. 자바스크립트 데이터 타입과 연산자



### 목차

- 3장 자바스크립트 데이터 타입과 연산자
  - 3.1 자바스크립트 기본 타입
    - 숫자
    - 문자열
    - 불린값
    - null과 defined
  - 3.2 자바스크립트 참조 타입(객체 타입)
    - 객체 생성
    - 객체 프로퍼티 읽기/쓰기/갱신
    - for in 문과 객체 프로퍼티 출력
    - 객체 프로퍼티 삭제
  - 3.3 참조 타입의 특성
    - 객체 비교
    - 참조에 의한 함수 호출 방식
  - 3.4 프로토타입
  - 3.5 배열
    - 배열 리터럴
    - 배열의 요소 생성
    - 배열의 length 프로퍼티
    - 배열과 객체
    - 배열의 프로퍼티 동적 생성
    - 배열의 프로퍼티 열거
    - 배열 요소 삭제
    - Array() 생성자 함수
    - 유사 배열 객체
  - 3.6 기본 타입과 표준 메서드
  - 3.7 연산자
    - '+' 연산자
    - typeof 연산자
    - == (동등) 연산자와 === (일치) 연산자
    - !! 연산자

---



- 자바스크립트의 데이터 타입 종류

  - 크게 기본 타입(Primitive type)과 참조 타입(Reference Type)

  ![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=http%3A%2F%2Fcfile2.uf.tistory.com%2Fimage%2F245AF44657DF4E0125050A)

  





## 3.1 자바스크립트 기본 타입

- 자바스크립트의 기본 타입 
  - 총 5가지: 숫자 (Number), 문자열 (String), 불린 (Boolean), undefined, null
  - 이들 타입의 특징은 **<u>그 자체가 하나의 값을 나타낸다</u>**는 것이다.



- **[예제 3-1]** 기본 타입의 변수들을 생성하고, 해당 변수의 타입을 자바스크립트의 `typeof` 연산자를 이용해서 출력해 보자.

  - `typeof` 연산자는 피연산자의 타입을 리턴한다.
  - 자바스크립트는 **느슨한 타입 체크 언어**다.
    - 자바스크립트는 변수를 선언할 때 타입을 미리 정하지 않고, `var`라는 한 가지 키워드로만 변수를 선언한다. 
    - 이렇게 선언된 변수에는 어떤 타입의 데이터라도 저장하는 것이 가능하다. 따라서 **자바스크립트는 변수에 어떤 형태의 데이터를 저장하느냐에 따라 해당 변수의 타입이 결정**된다.

  

  ```javascript
  // 숫자 타입
  var intNum = 10;
  var floatNum = 0.1;
  
  // 문자열 타입
  var singleQuoteStr = 'single quote string';
  var doubleQuoteStr = "double quote string";
  var singleChar = 'a';
  
  // 불린 타입
  var boolVar = true;
  
  // undefined 타입
  var emptyVar;
  
  // null 타입
  var nullVar = null;
  
  console.log(
      typeof intNum,
      typeof floatNum,
      typeof singleQuoteStr,
      typeof doubleQuoteStr,
      typeof singleChar,
      typeof boolVar,
      typeof emptyVar,
      typeof nullVar
  );
  
  
  [출력 결과]
  number number string string string boolean undefined object
  ```

  - 주의할 점: `null` 타입 변수인 nullVar의 `typeof` 결과가 `null`이 아니라 `object`

    - 자바스크립트에서 `null` 타입 변수인지를 확인할 때 `typeof` 연산자를 사용하면 안 되고, 일치 연산자(===)를 사용해서 변수의 값을 직접 확인해야 한다.

    ```javascript
    // null 타입 변수 생성
    var nullVar = null;
    
    console.log(typeof nullVar === null);  // (출력값) false
    console.log(nullVar === null); // (출력값) true
    ```

    



- 숫자 (Number)

  - C언어의 경우 정수냐 실수냐에 따라 `int`, `long`, `float`, `double` 등과 같은 다양한 숫자 타입이 존재

  - 자바스크립트는 하나의 숫자형(`number`)만 존재한다.

    - 자바스크립트에서는 **<u>모든 숫자를 64bit 부동 소수점 형태로 저장</u>**하기 때문이다.
    - 이는 C언어의 `double` 타입과 유사하다.

    

  - **[예제 3-2]** 자바스크립트 나눗셈 연산

    - 자바스크립트에서는 정수형이 따로 없고, 모든 숫자를 실수로 처리하므로 나눗셈 연산을 할 때 주의해야 한다.
      - C언어: 5 / 2 == 2 (5와 2 모두 정수, 결과 값의 소수 부분 버림)
      - 자바스크립트: 5 / 2 == 2.5 (5와 2 모두 실수 취급)

    ```javascript
    var num = 5 / 2;
    
    console.log(num);  // 2.5
    console.log(Math.floor(num)); // 2
    ```

    

- 문자열 (String)

  - 문자열은 작은 따옴표(')나 큰 따옴표(")로 생성한다.

  - C언어의 `char` 타입과 같이 문자 하나만을 별도로 나타내는 데이터 타입이 존재하지 않는다.

    - 한 개의 문자를 나타내려면 길이가 1인 문자열을 사용한다.

  - 주의해야 할 점은 한 번 정의된 문자열을 변하지 않는다(immutable)는 것이다.

    

  - **[예제 3-3]** 자바스크립트 문자열 예제

  ```javascript
  // str 문자열 생성, 문자를 인덱스로 접근 가능
  var str = "test";
  console.log(str[0], str[1], str[2], str[3]); 
  
  [출력 결과]
  t e s t
  
  
  // 문자열의 첫 글자를 대문자로 변경?
  str[0] = 'T';
  console.log(str);
  
  [출력 결과]
  test
  ```

  - 문자열은 문자 배열처럼 인덱스를 이용해서 접근 가능
  - 자바스크립트에서는 **<u>한 번 생성된 문자열은 읽기만 가능하고 수정은 불가능</u>**

  

- 불린값 (boolean)
  
  - 자바스크립트는 `true`와 `false` 값을 나타내는 boolean 값을 가진다.



- null과 undefined
  - 이 두 타입은 모두 자바스크립트에서 '값이 비어있음'을 나타낸다.
  - `undefined`: 기본적으로 값이 할당되지 않음을 나타내는 타입 또는 값
    - `undefined` 타입의 변수는 변수 자체의 값 또한 undefined이다.
    - 이처럼 **자바스크립트에서 undefined는 타입이자, 값을 나타낸다**.
  - `null`: 개발자가 명시적으로 값이 비어있음을 나타낼 때 사용하는 값







## 3.2 자바스크립트 참조 타입 (객체 타입)

- 객체
  - 자바스크립트에서 기본 타입을 제외한 모든 값은 객체다.
  - 따라서 **배열, 함수, 정규표현식** 등도 모두 자바스크립트 객체로 표현된다.
  - 자바스크립트에서의 객체란?
    - 단순히 '이름(key):값(value)' 형태의 프로퍼티들을 저장하는 컨테이너로서, 해시테이블과 유사하다.
    - 기본 타입은 하나의 값만 가지는 데 비해, 참조 타입인 객체는 여러 개의 프로퍼티들을 포함할 수 있으며, 이러한 객체의 프로퍼티는 기본 타입의 값을 포함하거나, 다른 객체를 가리킬 수도 있다.
    - 이러한 프로퍼티의 성질에 따라 객체의 프로퍼티는 함수로 포함할 수 있으며, 자바스크립트에서는 이러한 프로퍼티를 **메서드**라고 부른다.



- 객체 생성

  - 자바스크립트의 객체 개념은 생성 방법이나 상속 방식 등에서 C++이나 자바와 같은 기존 OOP 언어에서의 객체 개념과는 약간 다르다.

    - 기존 OOP 언어(Java): 클래스를 정의하고, 클래스의 인스턴스를 생성하는 과정에서 객체가 만들어진다.
    - 자바스크립트: 클래스라는 개념이 없고, 객체 리터럴이나 생성자 함수 등 별도의 생성 방식이 존재한다.

  - 자바스크립트에서 객체를 생성하는 방법은 크게 세 가지가 있다.

    - 기본 제공 `Object()` 객체 생성자 함수를 이용하는 방법

    - 객체 리터럴을 이용하는 방법

    - 생성자 함수를 이용하는 방법

      

  - Object() 생성자 함수 이용해서 객체 생성하기
    
  - **[예제 3-5]** 생성자 함수를 통한 객체 생성
  
    ```javascript
    // Object()를 이용해서 foo 빈 객체 생성
    var foo = new Object();
    
    // foo 객체 프로퍼티 생성
    foo.name = 'foo';
    foo.age = 30;
    foo.gender = 'male';
    
    console.log(typeof foo);
    console.log(foo);
    
    [출력 결과]
    object
    { name: 'foo', age: 30, gender: 'male' }
    ```
  
    
  
  
  
    - 객체 리터럴 방식 이용해서 객체 생성하기
      - 리터럴이란 용어의 의미는 표기법이라고 생각하자.
      - 따라서 객체 리터럴이란 객체를 생성하는 표기법을 의미한다.
      - 객체 리터럴은 중괄호({})를 이용해서 객체를 생성한다. { } 안에 아무것도 적지 않으면 빈 객체가 생성되며, 중괄호 안에 "프로퍼티 이름"."프로퍼티 값" 형태로 표기하면, 해당 프로퍼티가 추가된 객체를 생성할 수 있다.
      - 여기서 **프로퍼티 이름은 문자열이나 숫자가 올 수 있다**.
      
    - **프로퍼티 값은 자바스크립트의 값을 나타내는 어떤 표현식도 올 수 있으며, 이 값이 함수일 경우 이러한 프로퍼티를 메서드라고 부른다**.
  
      ```javascript
      var foo = {
          name: "foo",
          age: 30,
          gender: 'male'
      };
      
      console.log(typeof foo);
      console.log(foo);
      ```
  
      
  
  - 생성자 함수 이용
  
    - 자바스크립트의 경우는 함수를 통해서도 객체를 생성할 수 있다.
    - 이렇게 객체를 생성하는 함수를 **생성자 함수**라고 부른다.
    - 생성자 함수에 대해서는 4장에서 자세히 살펴보자.





- 객체 프로퍼티 읽기/쓰기/갱신

  - 객체는 새로운 값을 가진 프로퍼티를 생성하고, 생성된 프로퍼티에 접근해서 해당 값을 읽거나 또는 원하는 값으로 프로퍼티의 값을 갱신할 수 있다.

  - 객체의 프로퍼티에 접근하는 두 가지 방법

    - 대괄호([]) 표기법
    - 마침표(.) 표기법

    - **[예제 3-7]** 객체의 프로퍼티 접근하기

  ```javascript
  // 객체 리터럴 방식으로 foo 객체 생성
  var foo = {
      name: "foo",
      major: "computer science"
  };
  
  
  // 객체 프로퍼티 읽기
  console.log(foo.name);
  console.log(foo['name']);
  console.log(foo.nickname);
  
  [출력 결과]
  foo
  foo
  undefined
  
  
  // 객체 프로퍼티 갱신
  foo.major = 'electronics engineering';
  console.log(foo.major);
  console.log(foo['major']);
  
  [출력 결과]
  electronics engineering
  electronics engineering
  
  
  // 객체 프로퍼티 동적 생성
  foo.age = 30;
  console.log(foo.age);
  
  [출력 결과]
  30
  
  
  // 대괄호 표기법만을 사용해야 하는 경우
  foo['full-name'] = 'foo bar';
  console.log(foo['full-name']);
  console.log(foo.full-name);
  
  [출력 결과]
  foo bar
  ReferenceError: name is not defined
  ```

  



- for in 문과 객체 프로퍼티 출력

  - for in 문을 사용하면, 객체에 포함된 모든 프로퍼티에 대해 루프를 수행할 수 있다.
  - **[예제 3-8]** for in 문을 통한 객체 프로퍼티 출력

  ```javascript
  // 객체 리터럴 방식으로 foo 객체 생성
  var foo = {
      name: 'foo',
      age: 30,
      major: 'computer science'
  };
  
  // for in 문을 이용한 객체 프로퍼티 출력
  var prop;
  for(prop in foo) {
      console.log(prop, foo[prop]);
  }
  
  [출력 결과]
  name foo
  age 30
  major computer science
  ```

  - for in 문이 수행되면서 prop 변수에 foo 객체의 프로퍼티가 하나씩 할당된다. 따라서 prop에 할당된 프로퍼티명을 이용해서 `foo[prop]`과 같이 대괄호 표기법을 이용해서 프로퍼티값을 출력한 것이다.

  

- 객체 프로퍼티 삭제

  - 자바스크립트에서는 객체의 프로퍼티를 `delete` 연산자를 이용해 즉시 삭제할 수 있다.
  - 여기서 주의할 점은 `delete` 연산자는 객체의 프로퍼티를 삭제할 뿐, 객체 자체를 삭제하지는 못한다는 것이다.

  ```javascript
  // 객체 리터럴 방식으로 foo 객체 생성
  var foo = {
      name: 'foo',
      nickname: 'babo'
  };
  
  
  console.log(foo.nickname); // babo
  delete foo.nickname;
  console.log(foo.nickname); // undefined
  
  delete foo;
  console.log(foo.name);  // foo
  ```

  - `delete` 연산자는 프로퍼티만을 삭제할 수 있다.

  



## 3.3 참조 타입의 특성

- 자바스크립트에서의 객체
  - 기본 타입인 숫자, 문자열, 불린값, null, undefined 5가지를 제외한 모든 값은 객체다.
  - 배열이나 함수 역시 객체로 취급된다.
  - 이러한 객체는 자바스크립트에서 **참조 타입**이라고 부른다.
    - 이것은 객체의 모든 연산이 실제 값이 아닌 참조값으로 처리되기 때문이다.



- **[예제 3-9]** 동일한 객체를 참조하는 두 변수 objA와 objB

  ```javascript
  var objA = {
      val: 40
  };
  
  var objB = objA;
  
  console.log(objA.val);
  console.log(objB.val);
  
  objB.val = 50;
  console.log(objA.val);
  console.log(objB.val);
  
  [출력 결과] 
  40
  40
  50
  50
  ```

  - ![](https://docs.google.com/drawings/d/sXYzmy7cRgHMvhcMJs5mmlg/image?parent=e/2PACX-1vQiuXMoUJloXB41go11kimhAxCGB0Jhf5sdczvyrPIjH533basHPtLBcGPi59BcshFWUj6e55GqbGOO&rev=31&drawingRevisionAccessToken=3Dj4qKeY8MtI8Q&h=165&w=500&ac=1)





- 객체 비교

  - 동등 연산자(==)를 사용하여 두 객체를 비교할 때도 **<u>객체의 프로퍼티값이 아닌 참조값을 비교한다</u>**는 것에 주의해야 한다.
  - **[예제 3-10]** 기본 타입과 참조 타입의 비교 연산

  ```javascript
  var a = 100;
  var b = 100;
  
  var objA = {value: 100};
  var objB = {value: 100};
  var objC = objB;
  
  console.log(a == b);
  console.log(objA == objB);
  console.log(objB == objC);
  
  [출력 결과]
  true
  false
  true
  ```

  

- 참조에 의한 함수 호출 방식

  - 기본 타입과 참조 타입의 경우는 함수 호출 방식도 다르다.

    - 기본 타입의 경우는 **값에 의한 호출**(Call By Value) 방식으로 동작한다.
      - 즉, 함수를 호출할 때 인자로 기본 타입의 값을 넘길 경우, 호출된 함수의 매개변수로 복사된 값이 전달된다.
      - 때문에 함수 내부에서 매개변수를 이용해 값을 변경해도, 실제로 호출된 변수의 값이 변경되지는 않는다.
    - 객체와 같은 참조 타입의 경우는 **참조에 의한 호출**(Call by Reference) 방식으로 동작한다.
      - 즉, 함수를 호출할 때 인자로 참조 타입인 객체를 전달할 경우, 객체의 프로퍼티값이 함수의 매개변수에 복사되지 않고, 인자로 넘긴 객체의 참조값이 그대로 함수 내부로 전달된다.
      - 때문에 함수 내부에서 참조값을 이용해서 인자로 넘긴 실제 객체의 값을 변경할 수 있는 것이다.

  - **[예제 3-11]** Call by Value와 Call by Reference의 차이

    ```javascript
    var a = 100;
    var objA = {value: 100};
    
    function changeArg(num, obj) {
        num = 200;
        obj.value = 200;
        console.log(num);
        console.log(obj);
    }
    
    changeArg(a, objA);
    
    console.log(a);
    console.log(objA);
    
    [출력 결과]
    200
    { value: 200 }
    100
    { value: 200 }
    ```





## 3.4 프로토타입

- 프로토타입

  - 자바스크립트의 **모든 객체는 자신의 부모 역할을 하는 객체와 연결되어 있다**.

    - 이것은 마치 객체지향의 상속 개념과 같이 부모 객체의 프로퍼티를 마치 자신의 것처럼 쓸 수 있는 것 같은 특징이 있다.

  - 자바스크립트에서는 이러한 부모 객체를 **프로토타입 객체**라고 부른다.

  - **[예제 3-12]** 객체 생성 및 출력

    ```javascript
    var foo = {
        name: 'foo',
        age: 30
    };
    
    foo.toString(); ---- (1)
    
    console.dir(foo); ---- (2)
    
    [출력 결과]
    { name: 'foo', age: 30 }
    ```

    - (1): foo 객체는 `toString()` 메서드가 없으므로 에러가 발생해야 하지만, 정상적으로 실행된다. 에러가 발생하지 않고 예제가 실행될 수 있는 이유는 바로 **foo 객체의 프로토타입**에 `toString()` 메서드가 이미 정의되어 있고, foo 객체가 상속처럼 `toString()` 메서드를 호출했기 때문이다.

    - (2): foo 객체를 출력해보자.

      ![](https://lh6.googleusercontent.com/CG2nQsdAn_mKAHCLhoWlnRU6Qq5P6ml4aw8lcJ9mSUojqWQDUCzZJ8T218iJ-L4vrlMV5D9_CU2Hh1EK26LWKr2dWLVYOgOodIBW7SuWFkiIZXIpd-6lXVbrJF5NL-AsxEnNaWN5=s0)

  

  - ECMAScript 명세서에는 자바스크립트의 **모든 객체는 자신의 프로토타입을 가리키는 [[Prototype]]라는 숨겨진 프로퍼티**를 가진다고 설명하고 있다.

  - 모든 객체의 프로토타입은 자바스크립트의 룰에 따라 객체를 생성할 때 결정된다.

    - 위의 예제 3-12와 같이 객체 리터럴 방식으로 생성된 객체의 경우 **Object.prototype 객체**가 프로토타입 객체가 된다는 것만 기억하고 넘어가자.

      ![](https://docs.google.com/drawings/d/sErN42DsAb65VmZgfH788iQ/image?parent=e/2PACX-1vQiuXMoUJloXB41go11kimhAxCGB0Jhf5sdczvyrPIjH533basHPtLBcGPi59BcshFWUj6e55GqbGOO&rev=164&drawingRevisionAccessToken=OcpLCMqrnwHrNQ&h=382&w=575&ac=1)

  - 또한, 객체를 생성할 때 결정된 프로토타입 객체는 임의의 다른 객체로 변경하는 것도 가능하다.

    - 즉, 부모 객체를 동적으로 바꿀 수도 있는 것이다.
    - 자바스크립트에서는 이러한 특징을 활용해서 객체 상속 등의 기능을 구현한다.





## 3.5 배열

- 자바스크립트에서의 배열
  - 배열은 자바스크립트 객체의 특별한 형태다.
  - C나 자바의 배열과 같은 기능을 하는 객체지만, 이들과는 다르게 굳이 크기를 지정하지 않아도 되며, 어떤 위치에 어느 타입의 데이터를 저장하더라도 에러가 발생하지 않는다.



- 배열 리터럴

  - **배열 리터럴**은 자바스크립트에서 새로운 배열을 만드는 데 사용하는 표기법이다.

    - 배열 리터럴은 대괄호([])를 사용한다.

  - **[예제 3-13]** 배열 리터럴을 통한 배열 생성

    ```javascript
    // 배열 리터럴을 통한 5개 원소를 가진 배열 생성
    var colorArr = ['orange', 'yellow', 'blue', 'green', 'red'];
    console.log(colorArr[0]);
    console.log(colorArr[1]);
    console.log(colorArr[4]);
    ```

    

- 배열의 요소 생성

  - 객체가 동적으로 프로퍼티를 추가할 수 있듯이, 배열도 동적으로 배열 원소를 추가할 수 있다.

  - 특히, 자바스크립트 배열의 경우는 값을 순차적으로 넣을 필요 없이 아무 인덱스 위치에나 값을 동적으로 추가할 수 있다.

  - **[예제 3-14]** 배열 요소의 동적 생성

    ```javascript
    // 빈 배열
    var emptyArr = [];
    console.log(emptyArr[0]);
    
    // 배열 요소 동적 생성
    emptyArr[0] = 100;
    emptyArr[3] = 'eight';
    emptyArr[7] = true;
    console.log(emptyArr);
    console.log(emptyArr.length);
    
    [출력 결과]
    undefined
    [ 100, <2 empty items>, 'eight', <3 empty items>, true ]
    8
    ```

    - 자바스크립트는 배열의 크기를 현재 배열의 인덱스 중 가장 큰 값을 기준으로 정한다.
    - 값이 할당되지 않은 인덱스의 요소는 `undefined` 값을 기본으로 가진다. 



- 배열의 length 프로퍼티

  - 자바스크립트의 모든 배열은 length 프로퍼티가 있다.

  - length 프로퍼티는 배열의 원소 개수를 나타내지만, 실제로 배열에 존재하는 원소 개수와 일치하지는 않는다.

    - 위의 예제에서도 살펴봤듯이 emptyArr에 3개의 배열 요소를 생성했지만, emptyArr.length 값은 8이라는 것을 확인할 수 있다.
    - 즉, length 프로퍼티는 **배열 내에 가장 큰 인덱스에 1을 더한 값**이다.
    - 때문에 배열의 가장 큰 인덱스 값이 변하면, length 값 또한 자동으로 그에 맞춰 변경된다.

    

  - **[예제 3-15]** 배열의 length 프로퍼티 변경

    ```javascript
    var arr = [];
    console.log(arr.length);
    
    arr[0] = 0;
    arr[1] = 1;
    arr[2] = 2;
    arr[100] = 100;
    
    console.log(arr.length);
    
    [출력 결과]
    0
    101
    ```

    - 실제 메모리는 length 크기처럼 할당되지 않는다.

    

  - **[예제 3-16]** 배열 length 프로퍼티의 명시적 변경

    ```javascript
    var arr = [0, 1, 2];
    console.log(arr.length);
    
    arr.length = 5;
    console.log(arr);
    
    arr.length = 2;
    console.log(arr);
    console.log(arr[2]);
    
    [출력 결과]
    3
    [ 0, 1, 2, <2 empty items> ]
    [ 0, 1 ]
    undefined
    ```

    - ![](https://docs.google.com/drawings/d/s8zTxA1nNscr8GRW6mZcdCQ/image?parent=e/2PACX-1vQiuXMoUJloXB41go11kimhAxCGB0Jhf5sdczvyrPIjH533basHPtLBcGPi59BcshFWUj6e55GqbGOO&rev=28&drawingRevisionAccessToken=90OQvlTnBqqy8A&h=74&w=225&ac=1)

      

    

    - ![](https://docs.google.com/drawings/d/suF0gO2uqDCMVpz0xo4qP7g/image?parent=e/2PACX-1vQiuXMoUJloXB41go11kimhAxCGB0Jhf5sdczvyrPIjH533basHPtLBcGPi59BcshFWUj6e55GqbGOO&rev=141&drawingRevisionAccessToken=VLrmhX6rTHiNNg&h=373&w=427&ac=1)

      

    - ![](https://docs.google.com/drawings/d/swSiYDPLXkBTT_UzOvUx_xw/image?parent=e/2PACX-1vQiuXMoUJloXB41go11kimhAxCGB0Jhf5sdczvyrPIjH533basHPtLBcGPi59BcshFWUj6e55GqbGOO&rev=90&drawingRevisionAccessToken=dlt1NmNWT7_LiQ&h=422&w=427&ac=1)

    

- 배열 표준 메서드와 length 프로퍼티

  - 자바스크립트는 배열에서 사용 가능한 다양한 표준 메서드를 제공한다.

  - 이러한 배열 메서드는 **length 프로퍼티**를 기반으로 동작하고 있다.

  - 예시: `push()` 메서드

    - `push()` 메서드는 인수로 넘어온 항목을 배열의 끝에 추가하는 자바스크립트 표준 배열 메서드다.
    - 이 메서드는 배열의 현재 length 값의 위치에 새로운 원소값을 추가한다.
    - 배열의 length 프로퍼티는 '현재 배열의 맨 마지막 원소의 인덱스 + 1'을 의미하므로 결국 length 프로퍼티가 가리키는 인덱스에 값을 저장하는 것은 배열의 끝에 값을 추가하는 것과 같은 결과가 되기 때문이다.

  - **[예제 3-17]** push() 메서드와 length 프로퍼티

    ```javascript
    // arr 배열 생성
    var arr = ['zero', 'one', 'two'];
    
    // push() 메서드 호출
    arr.push('three');
    console.log(arr);
    
    // length 값 변경 후, push() 메서드 호출
    arr.length = 5;
    arr.push('four');
    console.log(arr);
    
    [출력 결과]
    [ 'zero', 'one', 'two', 'three' ]
    [ 'zero', 'one', 'two', 'three', <1 empty item>, 'four' ]
    ```

    - ![](https://docs.google.com/drawings/d/s2XrSr7LIGY7Mz7r-lJtt7A/image?parent=e/2PACX-1vQiuXMoUJloXB41go11kimhAxCGB0Jhf5sdczvyrPIjH533basHPtLBcGPi59BcshFWUj6e55GqbGOO&rev=77&drawingRevisionAccessToken=9iMtPQ0tcYKcPQ&h=205&w=427&ac=1)

      

    - ![](https://docs.google.com/drawings/d/syA630ERChgmAS1QBsInNJA/image?parent=e/2PACX-1vQiuXMoUJloXB41go11kimhAxCGB0Jhf5sdczvyrPIjH533basHPtLBcGPi59BcshFWUj6e55GqbGOO&rev=58&drawingRevisionAccessToken=62OTdTqEx1ykUA&h=179&w=427&ac=1)

      

    - ![](https://docs.google.com/drawings/d/sP-MDp4Yqm2_AGacbj28GiQ/image?parent=e/2PACX-1vQiuXMoUJloXB41go11kimhAxCGB0Jhf5sdczvyrPIjH533basHPtLBcGPi59BcshFWUj6e55GqbGOO&rev=41&drawingRevisionAccessToken=R0yXUydmf_Vk3w&h=173&w=427&ac=1)

      

    - ![](https://docs.google.com/drawings/d/s10WYmLyYrK5IGMVgKiV1mA/image?parent=e/2PACX-1vQiuXMoUJloXB41go11kimhAxCGB0Jhf5sdczvyrPIjH533basHPtLBcGPi59BcshFWUj6e55GqbGOO&rev=52&drawingRevisionAccessToken=SuAjL3m_CuW1WQ&h=165&w=427&ac=1)





- 배열과 객체

  - 자바스크립트에서는 배열 역시 객체다.

    - 하지만 배열은 일반 객체와는 약간 차이가 있다.
    - 이번 절에서는 배열과 객체의 차이점에 대해서 살펴보자.

  - **[예제 3-18]** 배열과 객체의 유사점과 차이점

    ```javascript
    // colorsArray 배열
    var colorsArray = ['orange', 'yellow', 'green'];
    console.log(colorsArray[0]); // (출력값) orange
    console.log(colorsArray[1]); // (출력값) yellow
    console.log(colorsArray[2]); // (출력값) green
    
    // colorsObj 객체
    var colorsObj = {
        '0': 'orange',
        '1': 'yellow',
        '2': 'green'
    };
    console.log(colorsObj[0]); // (출력값) orange
    console.log(colorsObj[1]); // (출력값) yellow
    console.log(colorsObj[2]); // (출력값) green
    
    // typeof 연산자 비교
    console.log(typeof colorsArray); // (출력값) object (not array)
    console.log(typeof colorsObj); // (출력값) object
    
    // length 프로퍼티
    console.log(colorsArray.length); // (출력값) 3
    console.log(colorsObj.length); // (출력값) undefined
    
    // 배열 표준 메서드
    colorsArray.push('red'); // ['orange', 'yellow', 'green', 'red']
    colorsObj.push('red'); // TypeError: colorsObj.push is not a function
    ```

    - 앞서 객체의 프로퍼티 접근을 설명할 때, 대괄호 안에는 접근하려는 프로퍼티의 속성을 **문자열 형태**로 적어야 한다고 했다.

      - 때문에 `colorsObj[0]`이 아니라 `colorsObj['0']`의 형태로 기입하는 것이 맞다.
      - 하지만 다음 예제에서는 `colorsObj[0]`의 결과가 'orange'로 정상적인 결과가 출력됐다.
      - 이것은 자바스크립트 엔진이 `[]` 연산자 내에 숫자가 사용될 경우, 해당 숫자를 자동으로 문자열 형태로 바꿔주기 때문이다.

    - `typeof` 연산 결과는 배열과 객체가 모두 **object**라는 것을 기억하자.

    - 배열 `colorsArray`는 length 프로퍼티가 존재하지만, 객체 `colorsObj`는 length 프로퍼티가 존재하지 않는다.

    - 배열과 객체의 또 하나의 차이점은 `colorsObj`는 배열이 아니므로 앞서 설명한 `push()`와 같은 **표준 배열 메서드**를 사용할 수 없다는 것이다.

      - 앞서 설명한 것처럼 객체 리터럴 방식으로 생성한 객체의 경우, 객체 표준 메서드를 저장하고 있는 **Object.prototype 객체**가 프로토타입이다.
      - 반면에 배열의 경우 **Array.prototype 객체**가 부모 객체인 프로토타입이 된다. 그리고 Array.prototype 객체의 프로토타입은 Object.prototype 객체가 된다.

      

      ![](https://docs.google.com/drawings/d/sLncV0X96cHfLFxKJEEuUfA/image?parent=e/2PACX-1vQiuXMoUJloXB41go11kimhAxCGB0Jhf5sdczvyrPIjH533basHPtLBcGPi59BcshFWUj6e55GqbGOO&rev=96&drawingRevisionAccessToken=spH9orLDBgxY1A&h=369&w=553&ac=1)

    



- 배열의 프로퍼티 동적 생성

  - 배열도 자바스크립트 객체이므로, 인덱스가 숫자인 배열 원소 이외에도 객체처럼 동적으로 프로퍼티를 추가할 수 있다.

  - **[예제 3-20]** 배열의 동적 프로퍼티 생성

    ```javascript
    // 배열 생성
    var arr = ['zero', 'one', 'two'];
    console.log(arr.length);  // (출력값) 3
    
    // 프로퍼티 동적 추가
    arr.color = 'blue';
    arr.name = 'number_array';
    console.log(arr.length);  // (출력값) 3
    
    // 배열 원소 추가
    arr[3] = 'red';
    console.log(arr.length);  // (출력값) 4
    
    // 배열 객체 출력
    console.dir(arr);  // (출력값) [ 'zero', 'one', 'two', 'red', color: 'blue', name: 'number_array' ]
    ```

    - **배열의 length 프로퍼티는 배열 원소의 가장 큰 인덱스가 변했을 경우만 변경된다.**
    - 배열도 객체처럼 'key: value' 형태로 배열 원소 및 프로퍼티 등이 있음을 알 수 있다.



- 배열의 프로퍼티 열거

  - 객체는 for in 문으로 프로퍼티를 열거한다고 설명했다.

  - 배열도 객체이므로 for in 문을 사용해서 배열 내의 모든 프로퍼티를 열거할 수 있지만, 이렇게 되면 불필요한 프로퍼티가 출력될 수 있으므로 되도록 for 문을 사용하는 것이 좋다.

  - **[예제 3-21]** 배열의 프로퍼티 열거

    ```javascript
    var arr = ['zero', 'one', 'two'];
    arr.color = 'blue';
    arr.name = 'number_array';
    arr[3] = 'red';
    
    
    for(var prop in arr) {
        console.log(prop, arr[prop]);
    }
    
    [출력 결과]
    0 zero
    1 one
    2 two
    3 red
    color blue
    name number_array
    
    
    for (var i = 0; i < arr.length; i++) {
        console.log(i, arr[i])
    }
    
    [출력 결과]
    0 zero
    1 one
    2 two
    3 red
    ```



- 배열 요소 삭제

  - 배열도 객체이므로, 배열 요소나 프로퍼티를 삭제하는 데 `delete` 연산자를 사용할 수 있다.

  - **[예제 3-22]** delete 연산자를 이용한 배열 프로퍼티 삭제

    ```javascript
    var arr = ['zero', 'one', 'two', 'three'];
    delete arr[2];
    
    console.log(arr);
    console.log(arr.length);
    
    [출력 결과]
    [ 'zero', 'one', <1 empty item>, 'three' ]
    4
    ```

    - 앞 예제에서 `delete arr[2]`로 배열의 요소를 삭제하면, arr[2]에 undefined가 할당되게 된다. 
    - 그러나 delete 연산자로 배열 요소 삭제 후에도 배열의 length 값은 변하지 않은 것을 확인할 수 있다.
    - 즉, delete 연산자는 해당 요소의 값을 undefined로 설정할 뿐 원소 자체를 삭제하지는 않는다.
    - 때문에 보통 배열에서 요소들을 완전히 삭제할 경우 자바스크립트에서는 **splice() 배열 메서드**를 사용한다.

  - `splice()` 배열 메서드

    - `splice(start, deleteCount, item...)`
    - start: 배열에서 시작 인덱스
    - deleteCount: start 인덱스에서부터 삭제할 요소의 수
    - item: 삭제할 위치에 추가할 요소

  - **[예제 3-23]** splice() 메서드를 사용한 배열 프로퍼티 삭제

    ```javascript
    var arr = ['zero', 'one', 'two', 'three'];
    
    arr.splice(2, 1);
    console.log(arr);
    console.log(arr.length);
    
    [출력 결과]
    [ 'zero', 'one', 'three' ]
    3
    ```

    - delete 연산자와는 다르게 배열 요소를 완전히 없애게 된다.



- Array() 생성자 함수

  - 배열은 일반적으로 배열 리터럴로 생성하지만, 배열 리터럴도 결국 자바스크립트 기본 제공 **Array() 생성자 함수**로 배열을 생성하는 과정을 단순화시킨 것이다.

  - 생성자 함수로 배열과 같은 객체를 생성할 때는 반드시 new 연산자를 같이 써야 한다는 것을 기억하자.

  - Array() 생성자 함수는 호출할 때 인자 개수에 따라 동작이 다르므로 주의해야 한다.

    - 호출할 때 인자가 1개이고, 숫자인 경우: 호출된 인자를 length로 갖는 빈 배열 생성
    - 그외의 경우: 호출된 인자를 요소로 갖는 배열 생성

  - **[예제 3-24]** Array() 생성자 함수를 통한 배열 생성

    ```javascript
    var foo = new Array(3);
    console.log(foo);
    console.log(foo.length);
    
    [출력 결과]
    [ <3 empty items> ]
    3
    
    var bar = new Array(1, 2, 3);
    console.log(bar);
    console.log(bar.length);
    
    [출력 결과]
    [ 1, 2, 3 ]
    3
    ```



- 유사 배열 객체

  - 배열의 length 프로퍼티는 배열의 동작에 있어서 중요한 프로퍼티이다.

  - 그렇다면 만약 일반 객체에 length라는 프로퍼티가 있으면 어떻게 될까?

  - 자바스크립트에서는 이렇게 length라는 프로퍼티를 가진 객체를 **유사 배열 객체**(array-like objects)라고 부른다.

  - **[예제 3-25]** 유사 배열 객체의 배열 메서드 호출

    ```javascript
    var arr = ['bar'];
    var obj = {
        name: 'foo',
        length: 1
    };
    
    arr.push('baz');
    console.log(arr);  // (출력값) [ 'bar', 'baz' ]
    
    obj.push('baz');  // TypeError: obj.push is not a function
    ```

    - 위의 예제에서 객체 obj는 length 프로퍼티가 있으므로 유사 배열 객체다.
      - 이러한 유사 배열 객체의 가장 큰 특징은 객체임에도 불구하고, 자바스크립트의 표준 배열 메서드를 사용하는 게 가능하다는 것이다.
    - 배열 arr는 `push()` 표준 배열 메서드를 활용해서 'baz' 원소를 추가하는 것이 가능한 반면에, 유사 배열 객체 obj는 당연히 배열이 아니므로 바로 `push()` 메서드를 호출할 경우에 에러가 발생한다.
    - 하지만 유사 배열 객체의 경우 4장에서 배울 `apply()` 메서드를 사용하면 객체지만 표준 배열 메서드를 활용하는 것이 가능하다.

  - **[예제 3-26]** 유사 배열 객체에서 apply()를 활용한 배열 메서드 호출

    ```javascript
    var obj = {
        name: 'foo',
        length: 1
    };
    
    Array.prototype.push.apply(obj, ['baz']);
    console.log(obj);
    
    [출력 결과]
    { '1': 'baz', name: 'foo', length: 2 }
    ```

    - 실제로 앞으로 우리가 살펴볼 **arguments** 객체나 **jQuery** 객체가 바로 유사 배열 객체 형태로 되어 있으므로 이러한 성질을 잘 기억해두자.





## 3.6 기본 타입과 표준 메서드

- 기본 타입의 표준 메서드

  - 자바스크립트는 숫자, 문자열, 불린값에 대해 각 타입별로 호출 가능한 **표준 메서드**를 정의하고 있다.

  - 하지만 기본 타입의 경우는 객체가 아닌데 어떻게 메서드를 호출할 수 있을까?

  - 기본 타입의 값들에 대해서 객체 형태로 메서드를 호출할 경우, 이들 기본값은 **<u>메서드 처리 순간에 객체로 변환된 다음 각 타입별 표준 메서드를 호출하게 된다</u>**. 그리고 메서드 호출이 끝나면 다시 기본값으로 복귀하게 된다.

  - **[예제 3-27]** 기본 타입 변수에서의 메서드 호출

    ```javascript
    // 숫자 메서드 호출
    var num = 0.5;
    console.log(num.toExponential(1));
    
    // 문자열 메서드 호출
    console.log("test".charAt(2));
    
    [출력 결과]
    5.0e-1
    s
    ```

  - 이렇듯 숫자와 문자열 등은 기본 타입이지만, 이들을 위해 정의된 표준 메서드들을 **객체처럼 호출할 수 있다**는 것을 기억하자.





## 3.7 연산자

- '+' 연산자

  - '+' 연산자는 **더하기 연산**과 **문자열 연결 연산**을 수행한다.

    - 두 피연산자가 모두 숫자일 경우: 더하기 연산
    - 그 외의 경우: 문자열 연결 연산
      - 숫자 + 문자열은 숫자를 문자열로 casting한 후 문자열 연결 연산

  - **[예제 3-28]** + 연산자 예제

    ```javascript
    var add1 = 1 + 2;
    var add2 = 'my' + 'string';
    var add3 = 1 + 'string';
    var add4 = 'string' + 2;
    
    console.log(add1);
    console.log(add2);
    console.log(add3);
    console.log(add4);
    
    [출력 결과]
    3
    mystring
    1string
    string2
    ```

    

- typeof 연산자

  - typeof 연산자는 피연산자의 타입을 문자열 형태로 리턴한다.

  - 여기서 null과 배열이 'object'라는 점, 함수는 'function'이라는 점에 유의하자.

    | 타입 종류 | 타입명    | typeof 반환값 |
    | --------- | --------- | ------------- |
    | 기본 타입 | 숫자      | 'number'      |
    | 기본 타입 | 문자열    | 'string'      |
    | 기본 타입 | 불린값    | 'boolean'     |
    | 기본 타입 | null      | 'object'      |
    | 기본 타입 | undefined | 'undefined'   |
    | 참조 타입 | 객체      | 'object'      |
    | 참조 타입 | 배열      | 'object'      |
    | 참조 타입 | 함수      | 'function'    |



- == (동등) 연산자와 === (일치) 연산자

  - == (동등) 연산자: 비교하려는 피연산자의 타입이 다를 경우에 타입 변환을 거친 다음 비교한다.

  - === (일치) 연산자: 피연산자의 타입이 다를 경우에 타입을 변경하지 않고 비교한다.

  - **[예제 3-29]** == 연산자 vs === 연산자

    ```javascript
    console.log(1 == '1'); // (출력값) true
    console.log(1 === '1'); // (출력값) false
    ```

    - 숫자 1과 문자열 '1'을 ==로 비교하면 타입이 다르므로 같은 타입(이 경우는 숫자)으로 변환해서 두 값이 같다고 판단해 true가 됨
    - 반면 === 연산자는 두 피연산자가 타입이 다르므로 바로 false가 출력된다.
    - == 연산자의 비교는 타입 변환에 따른 잘못된 결과를 얻을 수 있으므로 대부분의 JS 코딩 가이드에서는 === 연산자로 비교하기를 권하고 있다.



- !! 연산자

  - !!의 역할은 피연산자를 불린값으로 변환하는 것이다.

  - **[예제 3-30]** !! 연산자 활용을 통한 불린값 변환

    ```javascript
    console.log(!!0); // (출력값) false
    console.log(!!1); // (출력값) true
    console.log(!!'string'); // (출력값) true
    console.log(!!''); // (출력값) false
    console.log(!!true); // (출력값) true
    console.log(!!false); // (출력값) false
    console.log(!!null); // (출력값) false
    console.log(!!undefined); // (출력값) false
    console.log(!!{}); // (출력값) true
    console.log(!![1, 2, 3]); // (출력값) true
    ```

    - 객체는 빈 객체({})라도 true로 변환되는 것을 주의하자.

