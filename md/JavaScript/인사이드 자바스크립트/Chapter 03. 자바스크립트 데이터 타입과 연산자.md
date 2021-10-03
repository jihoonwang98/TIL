# Chapter 03. 자바스크립트 데이터 타입과 연산자



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





- 숫자 (Number)
  - C언어의 경우 정수냐 실수냐에 따라 `int`, `long`, `float`, `double` 등과 같은 다양한 숫자 타입이 존재
  - 자바스크립트는 하나의 숫자형(`number`)만 존재한다.
    - 자바스크립트에서는 **<u>모든 숫자를 64bit 부동 소수점 형태로 저장</u>**하기 때문이다.













