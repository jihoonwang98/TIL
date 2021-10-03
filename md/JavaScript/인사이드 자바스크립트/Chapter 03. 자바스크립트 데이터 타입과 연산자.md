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
  // 객체 리터럴 방식으로 foo 객체 생성
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

