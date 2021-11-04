# [모던 JavaScript 튜토리얼 - 코어 자바스크립트] 8. 프로토타입과 프로토타입 상속

> https://ko.javascript.info/prototypes



## 8.1 프로토타입 상속

> https://ko.javascript.info/prototype-inheritance



**도입**

- 개발을 하다 보면 기존에 있는 기능을 가져와 확장해야 하는 경우가 생깁니다.

- 사람에 관한 프로퍼티와 메서드를 가진 `user`라는 객체가 있는데, 

  `user`와 상당히 유사하지만 약간의 차이가 있는 `admin`과 `guest` 객체를 만들어야 한다고 가정해 봅시다. 

- 이때 "`user`의 메서드를 복사하거나 다시 구현하지 않고 `user`에 약간의 기능을 얹어 `admin`과 `guest` 객체를 만들 수 있지 않을까?"라는 생각이 들 겁니다.

- 자바스크립트 언어의 고유 기능인 ***프로토타입 상속(prototypal inheritance)*** 을 이용하면 위와 같은 생각을 실현할 수 있습니다.



### [[Prototype]]

- 자바스크립트의 객체는 명세서에서 명명한 `[[Prototype]]`이라는 숨김 프로퍼티를 갖습니다. 

  - 이 숨김 프로퍼티 값은 `null`이거나 다른 객체에 대한 참조가되는데, 다른 객체를 참조하는 경우 참조 대상을 **'프로토타입(prototype)'**이라 부릅니다.

  ![image-20211102082431400](./img/image-20211102082431400.png)

- 프로토타입의 동작 방식은 '신비스러운’면이 있습니다. 

- `object`에서 프로퍼티를 읽으려고 하는데 <u>해당 프로퍼티가 없으면 자바스크립트는 자동으로 프로토타입에서 프로퍼티를 찾기</u> 때문이죠. 

  - 프로그래밍에선 이런 동작 방식을 **'프로토타입 상속’**이라 부릅니다. 
  - 언어 차원에서 지원하는 편리한 기능이나 개발 테크닉 중 **프로토타입 상속**에 기반해 만들어진 것들이 많습니다.
  



- `[[Prototype]]` 프로퍼티는 내부 프로퍼티이면서 숨김 프로퍼티이지만 다양한 방법을 사용해 개발자가 값을 설정할 수 있습니다.

- 아래 예시처럼 특별한 이름인 `__proto__`을 사용하면 값을 설정할 수 있습니다.

  ```javascript
  let animal = {
    eats: true
  };
  let rabbit = {
    jumps: true
  };
  
  rabbit.__proto__ = animal;
  ```



> **`__proto__`는 `[[Prototype]]`용 getter·setter입니다.**
>
> - `__proto__`는 `[[Prototype]]`과 *다릅니다*. 
>   - **`__proto__`는 `[[Prototype]]`의 getter(획득자)이자 setter(설정자) 입니다.**
>
> - 하위 호환성 때문에 여전히 `__proto__`를 사용할 순 있지만
>
>   비교적 근래에 작성된 스크립트에선 `__proto__` 대신 함수 `Object.getPrototypeOf`나 `Object.setPrototypeOf`을 써서 프로토타입을 획득(get)하거나 설정(set)합니다. 
>
> - 근래엔 왜 `__proto__`를 쓰지 않는지와 두 함수의 자세한 설명에 대해선 이어지는 챕터에서 다룰 예정입니다.
>
>   
>
> - `__proto__`는 브라우저 환경에서만 지원하도록 자바스크립트 명세서에서 규정하였는데, 실상은 서버 사이드를 포함한 모든 호스트 환경에서 `__proto__`를 지원합니다. 
>
> - `[[Prototype]]`보다는 `__proto__`가 조금 더 직관적이어서 이해하기 쉬우므로, 본 튜토리얼의 예시에선 `__proto__`를 사용하도록 하겠습니다.



- 객체 `rabbit`에서 프로퍼티를 얻고싶은데 해당 프로퍼티가 없다면, 자바스크립트는 자동으로 `animal`이라는 객체에서 프로퍼티를 얻습니다.

  ```javascript
  let animal = {
    eats: true
  };
  let rabbit = {
    jumps: true
  };
  
  rabbit.__proto__ = animal; // (*)
  
  // 프로퍼티 eats과 jumps를 rabbit에서도 사용할 수 있게 되었습니다.
  alert( rabbit.eats ); // true (**)
  alert( rabbit.jumps ); // true
  ```

  - `(*)`로 표시한 줄에선 `animal`이 `rabbit`의 프로토타입이 되도록 설정하였습니다.

  - `(**)`로 표시한 줄에서 `alert` 함수가 `rabbit.eats` 프로퍼티를 읽으려 했는데, `rabbit`엔 `eats`라는 프로퍼티가 없습니다. 

    - 이때 자바스크립트는 `[[Prototype]]`이 참조하고 있는 객체인 `animal`에서 `eats`를 얻어냅니다. 아래 그림을 밑에서부터 위로 살펴보세요.

    ![image-20211102082828275](./img/image-20211102082828275.png)

    - 이제 “`rabbit`의 프로토타입은 `animal`입니다.” 혹은 "`rabbit`은 `animal`을 상속받는다."라고 말 할 수 있게 되었습니다.
    - 프로토타입을 설정해 준 덕분에 `rabbit`에서도 `animal`에 구현된 유용한 프로퍼티와 메서드를 사용할 수 있게 되었네요. 

- 이렇게 프로토타입에서 상속받은 프로퍼티를 **'상속 프로퍼티(inherited property)'**라고 합니다.

- 상속 프로퍼티를 사용해 `animal`에 정의된 메서드를 `rabbit`에서 호출해 봅시다.

  ```javascript
  let animal = {
    eats: true,
    walk() {
      alert("동물이 걷습니다.");
    }
  };
  
  let rabbit = {
    jumps: true,
    __proto__: animal
  };
  
  // 메서드 walk는 rabbit의 프로토타입인 animal에서 상속받았습니다.
  rabbit.walk(); // 동물이 걷습니다.
  ```

  - 아래 그림과 같이 프로토타입(`animal`)에서 `walk`를 자동으로 상속받았기 때문에 `rabbit`에서도 `walk`를 호출할 수 있게 되었습니다.

  ![image-20211102082950785](./img/image-20211102082950785.png)

- 프로토타입 체인은 지금까지 살펴본 예시들보다 길어질 수 있습니다.

  ```javascript
  let animal = {
    eats: true,
    walk() {
      alert("동물이 걷습니다.");
    }
  };
  
  let rabbit = {
    jumps: true,
    __proto__: animal
  };
  
  let longEar = {
    earLength: 10,
    __proto__: rabbit
  };
  
  // 메서드 walk는 프로토타입 체인을 통해 상속받았습니다.
  longEar.walk(); // 동물이 걷습니다.
  alert(longEar.jumps); // true (rabbit에서 상속받음)
  ```

  ![image-20211102083053689](./img/image-20211102083053689.png)

- 프로토타입 체이닝엔 세 가지 제약사항이 있습니다.
  1. 순환 참조(circular reference)는 허용되지 않습니다. `__proto__`를 이용해 닫힌 형태로 다른 객체를 참조하면 에러가 발생합니다.
  2. `__proto__`의 값은 객체나 `null`만 가능합니다. 다른 자료형은 무시됩니다.
  3. 객체엔 오직 하나의 `[[Prototype]]`만 있을 수 있다는 당연한 제약도 있습니다. 객체는 두 개의 객체를 상속받지 못합니다.





### 쓸 때는 프로토타입을 사용하지 않습니다.

- 프로토타입은 프로퍼티를 읽을 때만 사용합니다.

  - <u>프로퍼티를 추가, 수정하거나 지우는 연산은 객체에 직접 해야 합니다.</u>

- 객체 `rabbit`에 메서드 `walk`를 직접 할당해 보겠습니다.

  ```javascript
  let animal = {
    eats: true,
    walk() {
      /* rabbit은 이제 이 메서드를 사용하지 않습니다. */
    }
  };
  
  let rabbit = {
    __proto__: animal
  };
  
  rabbit.walk = function() {
    alert("토끼가 깡충깡충 뜁니다.");
  };
  
  rabbit.walk(); // 토끼가 깡충깡충 뜁니다.
  ```

  - `rabbit.walk()`를 호출하면 프로토타입에 있는 메서드가 실행되지 않고, 객체 `rabbit`에 추가한 메서드가 실행됩니다.

    ![image-20211102083248854](./img/image-20211102083248854.png)

    

- 그런데 접근자 프로퍼티(accessor property)는 setter 함수를 통해서 프로퍼티에 값을 할당하므로 이 규칙이 적용되지 않습니다. 

  - 접근자 프로퍼티에 값을 할당하는 것은 함수를 호출하는 것과 같기 때문입니다.
  - 아래 예시에서 `admin.fullName`이 의도한 대로 잘 작동하는 것을 확인할 수 있습니다.

  ```javascript
  let user = {
    name: "John",
    surname: "Smith",
  
    set fullName(value) {
      [this.name, this.surname] = value.split(" ");
    },
  
    get fullName() {
      return `${this.name} ${this.surname}`;
    }
  };
  
  let admin = {
    __proto__: user,
    isAdmin: true
  };
  
  alert(admin.fullName); // John Smith (*)
  
  // setter 함수가 실행됩니다!
  admin.fullName = "Alice Cooper"; // (**)
  
  alert(admin.fullName); // Alice Cooper , state of admin modified
  alert(user.fullName); // John Smith , state of user protected
  ```

  - `(*)`로 표시한 줄에서 `admin.fullName`은 프로토타입(`user`)에 있는 getter 함수(`get fullName`)를 호출하고, 

    `(**)`로 표시한 줄의 할당 연산은 프로토타입에 있는 setter 함수(`set fullName`)를 호출합니다.



### 'this'가 나타내는 것

- 위 예시를 보면 "`set fullName(value)` 본문의 `this`엔 어떤 값이 들어가지?"라는 의문을 가질 수 있습니다. 

  - "프로퍼티 `this.name`과 `this.surname`에 값을 쓰면 그 값이 `user`에 저장될까, 아니면 `admin`에 저장될까?"라는 의문도 생길 수 있죠.
  - 답은 간단합니다. 
  - <u>`this`는 프로토타입에 영향을 받지 않습니다.</u>

- **메서드를 객체에서 호출했든 프로토타입에서 호출했든 상관없이 `this`는 언제나 `.` 앞에 있는 객체가 됩니다.**

  - `admin.fullName=`으로 setter 함수를 호출할 때, `this`는 `user`가 아닌 `admin`이 되죠.
  - 객체 하나를 만들고 여기에 메서드를 많이 구현해 놓은 다음, 여러 객체에서 이 커다란 객체를 상속받게 하는 경우가 많기 때문에 이런 특징을 잘 알아두셔야 합니다. 
  - **<u>상속받은 메서드를 사용하더라도 객체는 프로토타입이 아닌 자신의 상태를 수정합니다.</u>**

- 예시를 통해 좀 더 알아봅시다. ‘메서드 저장소’ 역할을 하는 객체 `animal`을 `rabbit`이 상속받게 해보겠습니다.

  - `rabbit.sleep()`을 호출하면 `this.isSleeping`이 객체 `rabbit`에 설정됩니다.

  ```javascript
  // animal엔 다양한 메서드가 있습니다.
  let animal = {
    walk() {
      if (!this.isSleeping) {
        alert(`동물이 걸어갑니다.`);
      }
    },
    sleep() {
      this.isSleeping = true;
    }
  };
  
  let rabbit = {
    name: "하얀 토끼",
    __proto__: animal
  };
  
  // rabbit의 프로퍼티 isSleeping을 true로 변경합니다.
  rabbit.sleep();
  
  alert(rabbit.isSleeping); // true
  alert(animal.isSleeping); // undefined (프로토타입에는 isSleeping이라는 프로퍼티가 없습니다.)
  ```

- 위 코드를 실행한 후, 객체의 상태를 그림으로 나타내면 다음과 같습니다.

![image-20211102083820159](./img/image-20211102083820159.png)

- `rabbit` 말고도 `bird`, `snake` 등이 `animal`을 상속받는다고 해봅시다. 
  - 이 객체들도 `rabbit`처럼 `animal`에 구현된 메서드를 사용할 수 있겠죠. 
  - 이때 상속받은 메서드의 `this`는 `animal`이 아닌 실제 메서드가 호출되는 시점의 점(`.`) 앞에 있는 객체가 됩니다. 
  - 따라서 `this`에 데이터를 쓰면 `animal`이 아닌 해당 객체의 상태가 변화합니다.
- **<u>메서드는 공유되지만, 객체의 상태는 공유되지 않는다고 결론 내릴 수 있습니다.</u>**



### for...in 반복문

- `for...in`은 상속 프로퍼티도 순회대상에 포함시킵니다.

  ```javascript
  let animal = {
    eats: true
  };
  
  let rabbit = {
    jumps: true,
    __proto__: animal
  };
  
  // Object.keys는 객체 자신의 키만 반환합니다.
  alert(Object.keys(rabbit)); // jumps
  
  // for..in은 객체 자신의 키와 상속 프로퍼티의 키 모두를 순회합니다.
  for(let prop in rabbit) alert(prop); // jumps, eats
  ```

- [obj.hasOwnProperty(key)](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Object/hasOwnProperty)를 이용하면 상속 프로퍼티를 순회 대상에서 제외할 수 있습니다. 

  - 이 내장 메서드는 `key`에 대응하는 프로퍼티가 상속 프로퍼티가 아니고 `obj`에 직접 구현되어있는 프로퍼티일 때 `true`를 반환합니다.

- `obj.hasOwnProperty(key)`를 응용하면 아래 예시에서처럼 상속 프로퍼티를 걸러낼 수 있고, 상속 프로퍼티만을 대상으로 무언가를 할 수도 있습니다.

  ```javascript
  let animal = {
    eats: true
  };
  
  let rabbit = {
    jumps: true,
    __proto__: animal
  };
  
  for(let prop in rabbit) {
    let isOwn = rabbit.hasOwnProperty(prop);
  
    if (isOwn) {
      alert(`객체 자신의 프로퍼티: ${prop}`); // 객체 자신의 프로퍼티: jumps
    } else {
      alert(`상속 프로퍼티: ${prop}`); // 상속 프로퍼티: eats
    }
  }
  ```

  - 위 예시의 상속 관계를 그림으로 나타내면 다음과 같습니다. 

    ![image-20211102084050862](./img/image-20211102084050862.png)

  - `rabbit`은 `animal`을, `animal`은 `Object.prototype`을, `Object.prototype`은 `null`을 상속받고 있습니다.
  - `animal`이 `Object.prototype`를 상속받는 이유는 객체 리터럴 방식으로 선언하였기 때문입니다.



- 그림을 보면 `for..in` 안에서 사용한 메서드 `hasOwnProperty`가 `Object.prototype.hasOwnProperty`에서 왔다는 것을 확인할 수 있습니다.
- 엇? 그런데 상속 프로퍼티인 `eats`는 얼럿 창에 출력되는데, `hasOwnProperty`는 출력되지 않았습니다. 무슨 일이 있는 걸까요?
  - 이유는 간단합니다. `hasOwnProperty`는 열거 가능한(enumerable) 프로퍼티가 아니기 때문입니다. 
  - `Object.prototype`에 있는 모든 메서드의 `enumerable` 플래그는 `false`인데, `for..in`은 오직 열거 가능한 프로퍼티만 순회 대상에 포함하기 때문에 `hasOwnProperty`는 얼럿창에 출력되지 않습니다.



> **키-값을 순회하는 메서드 대부분은 상속 프로퍼티를 제외하고 동작합니다.**
>
> - `Object.keys`, `Object.values` 같이 객체의 키-값을 대상으로 무언가를 하는 메서드 대부분은 상속 프로퍼티를 제외하고 동작합니다.
> - 프로토타입에서 상속받은 프로퍼티는 *제외하고*, 해당 객체에서 정의한 프로퍼티만 연산 대상에 포함합니다.



### 요약

- 자바스크립트의 모든 객체엔 숨김 프로퍼티 `[[Prototype]]`이 있는데, 이 프로퍼티는 객체나 `null`을 가리킵니다.
- `obj.__proto__`를 사용하면 프로토타입에 접근할 수 있습니다. `__proto__`는 `[[Prototype]]`의 getter·setter로 쓰이는데, 요즘엔 잘 쓰지 않습니다. 자세한 사항은 뒤쪽 챕터에서 다룰 예정입니다.
- `[[Prototype]]`이 참조하는 객체를 '프로토타입’이라고 합니다.
- `obj`에서 프로퍼티를 읽거나 메서드를 호출하려는데 해당하는 프로퍼티나 메서드가 없으면 자바스크립트는 프로토타입에서 프로퍼티나 메서드를 찾습니다.
- 접근자 프로퍼티가 아닌 데이터 프로퍼티를 다루고 있다면, 쓰기나 지우기와 관련 연산은 프로토타입을 통하지 않고 객체에 직접 적용됩니다.
- 프로토타입에서 상속받은 `method`라도 `obj.method()`를 호출하면 `method` 안의 `this`는 호출 대상 객체인 `obj`를 가리킵니다.
- `for..in` 반복문은 객체 자체에서 정의한 프로퍼티뿐만 아니라 상속 프로퍼티도 순회 대상에 포함합니다. 반면, 키-값과 관련된 내장 메서드 대부분은 상속 프로퍼티는 제외하고 객체 자체 프로퍼티만을 대상으로 동작합니다.







## 8.2 함수의 prototype 프로퍼티

> https://ko.javascript.info/function-prototype



**도입**

- `new F()`와 같은 **생성자 함수**를 이용하면 새로운 객체를 만들 수 있다는 걸 앞서 배운 바 있습니다.
- 그런데 `F.prototype`이 객체면 `new` 연산자는 `F.prototype`을 사용해 새롭게 생성된 객체의 [[Prototype]]을 설정합니다.

> **주의:**
>
> - 자바스크립트가 만들어졌을 때는 프로토타입 기반 상속이 주요 기능 중 하나였습니다.
> - 그런데 과거엔 프로토타입에 직접 접근할 방법이 없었습니다. 
>   - 그나마 믿고 사용할 수 있었던 방법은 이번 챕터에서 설명할 **생성자 함수의 `"prototype"` 프로퍼티를 이용**하는 방법뿐이었죠. 
>   - 많은 스크립트가 아직 이 방법을 사용하는 이유가 여기에 있습니다.



- `F.prototype`에서 `"prototype"`은 `F`에 정의된 **일반 프로퍼티**라는 점에 주의해 주시기 바랍니다. 

- 앞서 배웠던 ‘프로토타입’ 객체와 같아 보이지만 `F.prototype`에서 `"prototype"`은 이름만 같은 일반 프로퍼티입니다.

  ```javascript
  let animal = {
    eats: true
  };
  
  function Rabbit(name) {
    this.name = name;
  }
  
  Rabbit.prototype = animal;
  
  let rabbit = new Rabbit("White Rabbit"); //  rabbit.__proto__ == animal
  
  alert( rabbit.eats ); // true
  ```

  - `Rabbit.prototype = animal`은 "`new Rabbit`을 호출해 만든 새로운 객체의 `[[Prototype]]`을 `animal`로 설정하라."라는 것을 의미합니다.

- 그림으로 나타내면 다음과 같습니다.

  ![image-20211102084614619](./img/image-20211102084614619.png)

  - 그림에서 가로 화살표는 일반 프로퍼티인 `"prototype"`을, 세로 화살표는 `[[Prototype]]`을 나타냅니다. 
  - 세로 화살표는 `rabbit`이 `animal`을 상속받았다는 것을 의미합니다.



> **`F.prototype`은 `new F`를 호출할 때만 사용됩니다.**
>
> - `F.prototype` 프로퍼티는 `new F`가 호출될 때만 사용됩니다. 
>   - `new F`를 호출해 새롭게 만든 객체의 `[[Prototype]]`을 할당해 주죠.
> - 새로운 객체가 만들어진 후에 `F.prototype` 프로퍼티가 바뀌면(`F.prototype = <another object>`) `new F`로 만들어지는 새로운 객체는 또 다른 객체를 `[[Prototype]]`으로 갖게 됩니다. 
>   - 다만, 기존에 있던 객체의 `[[Prototype]]`은 그대로 유지됩니다.



### 함수의 prototype 프로퍼티와 constructor 프로퍼티

- 개발자가 특별히 할당하지 않더라도 모든 함수는 `"prototype"` 프로퍼티를 갖습니다.

- 기본 프로퍼티인 `"prototype"`은 `constructor` 프로퍼티 하나만 있는 객체를 가리키는데, 이 `constructor` 프로퍼티는 함수 자신을 가리킵니다.

- 이 관계를 코드와 그림으로 나타내면 다음과 같습니다.

  ```javascript
  function Rabbit() {}
  
  /* 기본 prototype
  Rabbit.prototype = { constructor: Rabbit };
  */
  ```

  ![image-20211102085015612](./img/image-20211102085015612.png)

- 아래 코드를 실행해 직접 확인해봅시다.

  ```javascript
  function Rabbit() {}
  // 기본 prototype:
  // Rabbit.prototype = { constructor: Rabbit }
  
  alert( Rabbit.prototype.constructor == Rabbit ); // true
  ```

- 특별한 조작을 가하지 않았다면 `Rabbit`을 구현한 객체 모두에서 `[[Prototype]]`을 거쳐 `constructor` 프로퍼티를 사용할 수 있습니다.

  ```javascript
  function Rabbit() {}
  // 기본 prototype:
  // Rabbit.prototype = { constructor: Rabbit }
  
  let rabbit = new Rabbit(); // {constructor: Rabbit}을 상속받음
  
  alert(rabbit.constructor == Rabbit); // true (프로토타입을 거쳐 접근함)
  ```

  ![image-20211102085205526](./img/image-20211102085205526.png)

- `constructor` 프로퍼티를 사용하면 기존에 있던 객체의 `constructor`를 사용해 새로운 객체를 만들 수 있습니다.

  ```javascript
  function Rabbit(name) {
    this.name = name;
    alert(name);
  }
  
  let rabbit = new Rabbit("White Rabbit");
  
  let rabbit2 = new rabbit.constructor("Black Rabbit");
  ```

  - 객체가 있는데 이 객체를 만들 때 어떤 생성자가 사용되었는지 알 수 없는 경우(예: 객체가 서드 파티 라이브러리에서 온 경우), 이 방법을 유용하게 쓸 수 있습니다.

- 어느 방식을 사용해 객체를 만들든 `"constructor"`에서 가장 중요한 점은 다음과 같습니다.

- **자바스크립트는 알맞은 `"constructor"` 값을 보장하지 않습니다.**

  - 함수에 기본으로 `"prototype"` 값이 설정되긴 하지만 그게 전부입니다. `"constructor"`에 벌어지는 모든 일은 전적으로 개발자에게 달려있습니다.

  - 함수의 기본 `"prototype"` 값을 다른 객체로 바꾸면 이 객체엔 `"constructor"`가 없을 겁니다.

    ```javascript
    function Rabbit() {}
    Rabbit.prototype = {
      jumps: true
    };
    
    let rabbit = new Rabbit();
    alert(rabbit.constructor === Rabbit); // false
    ```

- 이런 상황을 방지하고 알맞은 `constructor`를 유지하려면 `"prototype"` 전체를 덮어쓰지 말고 기본 `"prototype"`에 원하는 프로퍼티를 추가/제거해야 합니다.

  ```javascript
  function Rabbit() {}
  
  // Rabbit.prototype 전체를 덮어쓰지 말고
  // 원하는 프로퍼티는 그냥 추가하세요.
  Rabbit.prototype.jumps = true
  // 이렇게 하면 기본 Rabbit.prototype.constructor가 유지됩니다.
  ```

- `constructor` 프로퍼티를 수동으로 다시 만들어주는 것도 대안이 될 수 있습니다.

  ```javascript
  Rabbit.prototype = {
    jumps: true,
    constructor: Rabbit
  };
  
  // 수동으로 추가해 주었기 때문에 알맞은 constructor가 유지됩니다.
  ```



### 요약


이번 챕터에선 생성자 함수를 이용해 만든 객체에 `[[Prototype]]`을 설정해 주는 방법에 대해 간략히 알아보았습니다. 이 방법을 기반으로 하는 고급 프로그래밍 패턴에 대해선 추후 학습할 예정입니다.

몇 가지 사항만 명확하게 이해하고 있으면 지금까지 배운 것들은 복잡하지 않습니다.

- `F.prototype` 프로퍼티는 `[[Prototype]]`과는 다릅니다. `F.prototype`은 `new F()`를 호출할 때 만들어지는 새로운 객체의 `[[Prototype]]`을 설정합니다.
- `F.prototype`의 값은 객체나 null만 가능합니다. 다른 값은 무시됩니다.
- 지금까지 배운 내용은 생성자 함수에 `"prototype"`를 설정하고, 이 생성자 함수를 `new`를 사용해 호출할 때만 적용됩니다.

일반 객체에 `"prototype"` 프로퍼티를 사용하면 아무런 일이 일어나지 않습니다.

```javascript
let user = {
  name: "John",
  prototype: "Bla-bla" // 마술은 일어나지 않습니다.
};
```

모든 함수는 기본적으로 `F.prototype = { constructor : F }`를 가지고 있으므로 함수의 `"constructor"` 프로퍼티를 사용하면 객체의 생성자를 얻을 수 있습니다.



## 8.3 네이티브 프로토타입

> https://ko.javascript.info/native-prototypes



**도입**

- `prototype` 프로퍼티는 자바스크립트 내부에서도 광범위하게 사용됩니다. 
- 모든 내장 생성자 함수에서 `prototype` 프로퍼티를 사용하죠.
- 첫 번째로 자세히 살펴본 다음 어떻게 내장 객체에 새 기능을 추가하여 프로토타입 프로퍼티를 사용하는지 알아보겠습니다.



### Object.prototype

- 빈 객체를 표현한다고 해 봅시다.

  ```javascript
  let obj = {};
  alert( obj ); // "[object Object]" ?
  ```

  - `"[object Object]"` 문자열을 생성하는 코드는 어디에 있을까요? 
  - `toString` 메서드에서 이 문자열을 생성한다는 것을 앞서 배워서 알고 있지만 도대체 코드는 어디에 있는 걸까요? 
  - `obj`는 비어 있는데 말이죠.

- `obj = new Object()`를 줄이면 `obj = {}`가 됩니다. 여기서 `Object`는 내장 객체 생성자 함수인데, 이 생성자 함수의 `prototype`은 `toString`을 비롯한 다양한 메서드가 구현되어있는 거대한 객체를 참조합니다.

  ![image-20211102085749340](./img/image-20211102085749340.png)

- `new Object()`를 호출하거나 리터럴 문법 `{...}`을 사용해 객체를 만들 때, 새롭게 생성된 객체의 `[[Prototype]]`은 이전 챕터에서 언급한 규칙에 따라 `Object.prototype`을 참조합니다.

  ![image-20211102085848706](./img/image-20211102085848706.png)

  - 따라서 `obj.toString()`을 호출하면 `Object.prototype`에서 해당 메서드를 가져오게 되죠.

- 예시를 통해 이를 확인할 수 있습니다.

  ```javascript
  let obj = {};
  
  alert(obj.__proto__ === Object.prototype); // true
  alert(obj.toString === obj.__proto__.toString); //true
  alert(obj.toString === Object.prototype.toString); //true
  ```

- 그런데 이때 `Object.prototype` 위의 체인엔 `[[Prototype]]`이 없다는 점을 주의하셔야 합니다.

  ```javascript
  alert(Object.prototype.__proto__); // null
  ```



### 다른 내장 프로토타입

- `Array`, `Date`, `Function`을 비롯한 내장 객체들 역시 프로토타입에 메서드를 저장해 놓습니다.

- 배열 `[1, 2, 3]`을 만들면 기본 `new Array()` 생성자가 내부에서 사용되기 때문에 `Array.prototype`이 배열 `[1, 2, 3]`의 프로토타입이 되죠. 

  - `Array.prototype`은 배열 메서드도 제공합니다. 
  - 이런 내부 동작은 메모리 효율을 높여주는 장점을 가져다줍니다.

- 명세서에선 모든 내장 프로토타입의 꼭대기엔 `Object.prototype`이 있어야 한다고 규정합니다. 

  - 이런 규정 때문에 몇몇 사람들은 "모든 것은 객체를 상속받는다."라는 말을 하죠.

- 세 개의 내장 객체를 이용해 전체적인 그림을 그리면 다음과 같습니다.

  ![image-20211102093126470](./img/image-20211102093126470.png)

- 각 내장 객체의 프로토타입을 확인해 보겠습니다.

  ```javascript
  let arr = [1, 2, 3];
  
  // arr은 Array.prototype을 상속받았나요?
  alert( arr.__proto__ === Array.prototype ); // true
  
  // arr은 Object.prototype을 상속받았나요?
  alert( arr.__proto__.__proto__ === Object.prototype ); // true
  
  // 체인 맨 위엔 null이 있습니다.
  alert( arr.__proto__.__proto__.__proto__ ); // null
  ```

  

- **체인 상의 프로토타입엔 중복 메서드가 있을 수 있습니다.** 

- `Array.prototype`엔 요소 사이에 쉼표를 넣어 요소 전체를 합친 문자열을 반환하는 자체 메서드 `toString`가 있습니다.

  ```javascript
  let arr = [1, 2, 3]
  alert(arr); // 1,2,3 <-- Array.prototype.toString 의 결과
  ```

- 그런데 `Object.prototype`에도 메서드 `toString`이 있습니다. 

- <u>이렇게 중복 메서드가 있을 때는 체인 상에서 가까운 곳에 있는 메서드가 사용됩니다.</u> 

- `Array.prototype`이 체인 상에서 더 가깝기 때문에 `Array.prototype`의 `toString`이 사용되죠.

  ![image-20211102093446166](./img/image-20211102093446166.png)

- Chrome 개발자 콘솔과 같은 도구를 사용하면 상속 관계를 확인할 수 있습니다. `console.dir`를 사용하면 내장 객체의 상속 관계를 확인하는 데 도움이 됩니다.

  ![](https://ko.javascript.info/article/native-prototypes/console_dir_array.png)

- 배열이 아닌 다른 내장 객체들 또한 같은 방법으로 동작합니다. 

  - 함수도 마찬가지이죠. 
  - 함수는 내장 객체 `Function`의 생성자를 사용해 만들어지는데 `call`, `apply`를 비롯한 함수에서 사용할 수 있는 메서드는 `Fuction.prototype`에서 가져옵니다. 
  - 함수에도 `toString`이 구현되어 있습니다.

  ```javascript
  function f() {}
  
  alert(f.__proto__ == Function.prototype); // true
  alert(f.__proto__.__proto__ == Object.prototype); // true, 객체에서 상속받음
  ```



### 원시값

- 문자열과 숫자 불린값을 다루는 것은 엄청 까다롭습니다.

- 문자열과 숫자 불린값은 객체가 아닙니다. 

  - 그런데 이런 <u>원시값들의 프로퍼티</u>에 접근하려고 하면 

    내장 생성자 `String`, `Number`, `Boolean`을 사용하는 임시 래퍼(wrapper) 객체가 생성됩니다. 

  - 임시 래퍼 객체는 이런 메서드를 제공하고 난 후에 사라집니다.

- 래퍼 객체는 보이지 않는 곳에서 만들어집니다. 최적화는 엔진이 담당하죠. 

- 그런데 명세서에선 각 자료형에 해당하는 래퍼 객체의 메서드를 프로토타입 안에 구현해 놓고 `String.prototype`, `Number.prototype`, `Boolean.prototype`을 사용해 쓸 수 있도록 규정합니다.



> **`null`과 `undefined`에 대응하는 래퍼 객체는 없습니다.**
>
> - 특수 값인 `null`과 `undefined`는 문자열과 숫자 불린값과는 거리가 있습니다. 
> - `null`과 `undefined`에 대응하는 래퍼 객체는 없죠. 
>   - 따라서 `null`과 `undefined`에선 메서드와 프로퍼티를 이용할 수 없습니다. 
>   - 프로토타입도 물론 사용할 수 없습니다.



### 네이티브 프로토타입 변경하기

- 네이티브 프로토타입을 수정할 수 있습니다. `String.prototype`에 메서드를 하나 추가하면 모든 문자열에서 해당 메서드를 사용할 수 있죠.

  ```javascript
  String.prototype.show = function() {
    alert(this);
  };
  
  "BOOM!".show(); // BOOM!
  ```

- 개발을 하다 보면 "새로운 내장 메서드를 만드는 게 좋지 않을까?"라는 생각이 들 때가 있습니다. 네이티브 프로토타입에 새 내장 메서드를 추가하고 싶은 유혹이 자꾸 생기죠. 그런데 이는 좋지 않은 방법입니다.

> **중요:**
>
> - 프로토타입은 전역으로 영향을 미치기 때문에 프로토타입을 조작하면 충돌이 날 가능성이 높습니다. 
> - 두 라이브러리에서 동시에 `String.prototype.show` 메서드를 추가하면 한 라이브러리의 메서드가 다른 라이브러리의 메서드를 덮어쓰죠.
> - 이런 이유로 네이티브 프로토타입을 수정하는 것을 추천하지 않습니다.

- **모던 프로그래밍에서 네이티브 프로토타입 변경을 허용하는 경우는 딱 하나뿐입니다. 바로 폴리필을 만들 때입니다.**

  - 폴리필은 자바스크립트 명세서에 있는 메서드와 동일한 기능을 하는 메서드 구현체를 의미합니다. 
  - 명세서에는 정의되어 있으나 특정 자바스크립트 엔진에서는 해당 기능이 구현되어있지 않을 때 폴리필을 사용합니다.
  - 폴리필을 직접 구현하고 난 후 폴리필을 내장 프로토타입에 추가할 때만 네이티브 프로토타입을 변경합시다.

- 예시

  ```javascript
  if (!String.prototype.repeat) { // repeat이라는 메서드가 없다고 가정합시다
    // 프로토타입에 repeat를 추가
  
    String.prototype.repeat = function(n) {
      // string을 n회 반복(repeat)합니다.
  
      // 실제 이 메서드를 구현하려면 코드는 더 복잡해질겁니다.
      // 전체 알고리즘은 명세서에서 확인할 수 있겠죠.
      // 그런데 완벽하지 않은 폴리필이라도 충분히 쓸만합니다.
      return new Array(n + 1).join(this);
    };
  }
  
  alert( "La".repeat(3) ); // LaLaLa
  ```



### 프로토타입에서 빌려오기

- [call/apply와 데코레이터, 포워딩](https://ko.javascript.info/call-apply-decorators#method-borrowing)에서 메서드 빌리기에 대한 내용을 학습한 바 있습니다.

  - 한 객체의 메서드를 다른 객체로 복사할 때 이 기법이 사용됩니다.

- 개발을 하다 보면 네이티브 프로토타입에 구현된 메서드를 빌려야 하는 경우가 종종 생깁니다.

- 유사 배열 객체를 만들고 여기에 `Array` 메서드를 복사해봅시다.

  ```javascript
  let obj = {
    0: "Hello",
    1: "world!",
    length: 2,
  };
  
  obj.join = Array.prototype.join;
  
  alert( obj.join(',') ); // Hello,world!
  ```

  - 예시를 실행하면 에러 없이 의도한 대로 동작합니다. 
  - 내장 메서드 `join`의 내부 알고리즘은 제대로 된 인덱스가 있는지와 `length` 프로퍼티가 있는지만 확인하기 때문입니다. 
    - 호출 대상이 진짜 배열인지는 확인하지 않죠. 다수의 내장 메서드가 이런 식으로 동작합니다.
  - 메서드 빌리기 말고도 `obj.__proto__`를 `Array.prototype`로 설정해 배열 메서드를 상속받는 방법이 있습니다. 
    - 이렇게 하면 `obj`에서 모든 `Array`메서드를 사용할 수 있습니다.
    - 그런데 이 방법은 `obj`가 다른 객체를 상속받고 있을 때는 사용할 수 없습니다. 자바스크립트는 단일 상속만을 허용한다는 점을 기억하시기 바랍니다.
  - 메서드 빌리기는 유연합니다. 여러 객체에서 필요한 기능을 가져와 섞는 것을 가능하게 해줍니다.



### 요약

- 모든 내장 객체는 같은 패턴을 따릅니다.
  - 메서드는 프로토타입에 저장됩니다(`Array.prototype`, `Object.prototype`, `Date.prototype` 등).
  - 객체 자체엔 데이터만 저장합니다(배열의 요소, 객체의 프로퍼티, 날짜 등).
- 원시값 또한 래퍼 객체의 프로토타입에 `Number.prototype`, `String.prototype`, `Boolean.prototype` 같은 메서드를 저장합니다. `undefined`와 `null` 값은 래퍼 객체를 가지지 않습니다.
- 내장 프로토타입을 수정할 수 있습니다. 내장 프로토타입의 메서드를 빌려와 새로운 메서드를 만드는 것 역시 가능합니다. 그러나 내장 프로토타입 변경은 되도록 하지 않아야 합니다. 내장 프로토타입은 새로 명세서에 등록된 기능을 사용하고 싶은데 자바스크립트 엔진엔 이 기능이 구현되어있지 않을 때만 변경하는 게 좋습니다.







## 8.4 프로토타입 메서드와 `__proto__`가 없는 객체

> https://ko.javascript.info/prototype-methods



**도입**

- 이 절의 첫 번째 챕터에서 프로토타입을 설정하기 위한 모던한 방법이 있다고 언급했습니다.

- `__proto__`는 브라우저를 대상으로 개발하고 있다면 다소 구식이기 때문에 더는 사용하지 않는 것이 좋습니다. 표준에도 관련 내용이 명시되어있습니다.

  대신 아래와 같은 모던한 메서드들을 사용하는 것이 좋죠.

  - [Object.create(proto, \[descriptors\])](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Object/create)– `[[Prototype]]`이 `proto`를 참조하는 빈 객체를 만듭니다. 이때 프로퍼티 설명자를 추가로 넘길 수 있습니다.
  - [Object.getPrototypeOf(obj)](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Object/getPrototypeOf) – `obj`의 `[[Prototype]]`을 반환합니다.
  - [Object.setPrototypeOf(obj, proto)](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Object/setPrototypeOf) – `obj`의 `[[Prototype]]`이 `proto`가 되도록 설정합니다.

- 앞으론 아래 예시처럼 `__proto__` 대신 이 메서드들을 사용하도록 합시다.

  ```javascript
  let animal = {
    eats: true
  };
  
  // 프로토타입이 animal인 새로운 객체를 생성합니다.
  let rabbit = Object.create(animal);
  
  alert(rabbit.eats); // true
  
  alert(Object.getPrototypeOf(rabbit) === animal); // true
  
  Object.setPrototypeOf(rabbit, {}); // rabbit의 프로토타입을 {}으로 바꿉니다.
  ```

- 메서드를 소개할 때 잠시 언급한 것처럼 `Object.create`에는 프로퍼티 설명자를 선택적으로 전달할 수 있습니다. 

  - 설명자를 이용해 새 객체에 프로퍼티를 추가해 보겠습니다.

  ```javascript
  let animal = {
    eats: true
  };
  
  let rabbit = Object.create(animal, {
    jumps: {
      value: true
    }
  });
  
  alert(rabbit.jumps); // true
  ```

  - 설명자는 [프로퍼티 플래그와 설명자](https://ko.javascript.info/property-descriptors)에서 배운 것과 같은 형태로 사용하면 됩니다.

- `Object.create`를 사용하면 `for..in`을 사용해 프로퍼티를 복사하는 것보다 더 효과적으로 객체를 복제할 수 있습니다.

  ```javascript
  let clone = Object.create(Object.getPrototypeOf(obj), Object.getOwnPropertyDescriptors(obj));
  ```

  - `Object.create`를 호출하면 `obj`의 모든 프로퍼티를 포함한 완벽한 사본이 만들어집니다. 
  - 사본엔 열거 가능한 프로퍼티와 불가능한 프로퍼티, 데이터 프로퍼티, getter, setter 등 모든 프로퍼티가 복제됩니다.
    - `[[Prototype]]`도 복제되죠.



### 비하인드 스토리

- `[[Prototype]]`을 다룰 수 있는 방법은 다양합니다. 목표는 하나인데 목표를 이루기 위한 수단은 여러 가지이네요!
  - 왜 그럴까요? 역사적인 이유가 있습니다.
  - 생성자 함수의 `"prototype"` 프로퍼티는 아주 오래전부터 그 기능을 수행하고 있었습니다.
  - 그런데 2012년, 표준에 `Object.create`가 추가되었죠. `Object.create`를 사용하면 주어진 프로토타입을 사용해 객체를 만들 수 있긴 하지만, 프로토타입을 얻거나 설정하는것은 불가능했습니다. 그래서 브라우저는 비표준 접근자인 `__proto__`를 구현해 언제나 프로토타입을 얻거나 설정할 수 있도록 하였죠.
  - 이후 2015년에 `Object.setPrototypeOf`와 `Object.getPrototypeOf`가 표준에 추가되면서 `__proto__`와 동일한 기능을 수행할 수 있게 되었습니다. 그런데 이 시점엔 `__proto__`가 모든 곳에 구현되어 있어서 사실상 표준(de-facto standard)이 되어버렸죠. 표준의 부록 B(Annex B)에 추가되기도 하였습니다. 이 부록에 추가되면 브라우저가 아닌 환경에선 선택사항이라는것을 의미합니다
  - 이런 이유 때문에 지금은 여러 방식을 원하는 대로 쓸 수 있게 된 것입니다.
- 그런데 "왜 `__proto__`가 함수 `getPrototypeOf/setPrototypeOf`로 대체되었을까?"라는 의문이 떠오를 수 있습니다. 
  - 흥미로운 질문이죠. 
  - 답은 `__proto__`가 왜 나쁜지 이해하면 얻을 수 있습니다. 아래 내용을 계속 읽으면서 답을 찾아봅시다. 



> **속도가 중요하다면 기존 객체의 `[[Prototype]]`을 변경하지 마세요.**
>
> - 원한다면 언제나 `[[Prototype]]`을 얻거나 설정할 수 있습니다. 기술적인 제약이 있는 건 아니죠. 
> - 하지만 대개는 객체를 생성할 때만 `[[Prototype]]`을 설정하고 이후엔 수정하지 않습니다. 
>   - `rabbit`이 `animal`을 상속받도록 설정하고 난 이후엔 이를 변경하지 않죠.
> - 자바스크립트 엔진은 이런 시나리오를 토대로 최적화되어 있습니다. 
>   - `Object.setPrototypeOf`나 `obj.__proto__=`를 써서 프로토타입을 그때그때 바꾸는 연산은 객체 프로퍼티 접근 관련 최적화를 망치기 때문에 매우 느립니다. 
>   - 그러므로 `[[Prototype]]`을 바꾸는 것이 어떤 결과를 초래할지 확실히 알거나 속도가 전혀 중요하지 않은 경우가 아니라면 `[[Prototype]]`을 바꾸지 마세요.



### '아주 단순한' 객체

- 알다시피 객체는 키-값 쌍을 저장할 수 있는 연관 배열입니다.

- 그런데 커스텀 사전을 만드는 것과 같이 사용자가 직접 입력한 키를 가지고 객체를 만들다 보면 사소한 결함이 발견됩니다. 

  - 다른 문자열은 괜찮지만 `"__proto__"`는 키로 사용할 수 없다는 결함이죠.

- 예시를 살펴봅시다.

  ```javascript
  let obj = {};
  
  let key = prompt("입력하고자 하는 key는 무엇인가요?", "__proto__");
  obj[key] = "...값...";
  
  alert(obj[key]); // "...값..."이 아닌 [object Object]가 출력됩니다.
  ```

  - 사용자가 프롬프트 창에 `__proto__`를 입력하면 값이 제대로 할당되지 않네요.
  - `__proto__` 프로퍼티는 특별한 프로퍼티라는 것을 이미 알고 있기 때문에 그렇게 놀랄만한 일은 아니긴 합니다. 
  - `__proto__`는 항상 객체이거나 `null`이어야 하죠. 문자열은 프로토타입이 될 수 없습니다.
  - 그런데 사실 이런 결과를 의도하면서 구현한 건 아닐 겁니다. 키가 무엇이 되었든, 키-값 쌍을 저장하려고 하는데 키가 `__proto__`일 때 값이 제대로 저장되지 않는 건 명백한 버그이죠.

- 위 예시에선 이 버그가 그리 치명적이진 않습니다. 

- 그런데 할당 값이 객체일 때는 프로토타입이 바뀔 수 있다는 치명적인 버그가 발생할 수 있습니다. 

  - 프로토타입이 바뀌면 예상치 못한 일이 발생할 수 있기 때문입니다.

- 개발자들은 대개 프로토타입이 중간에 바뀌는 시나리오는 배제한 채 개발을 진행합니다. 

  - 이런 고정관념 때문에 버그의 원인을 찾는 게 힘들어지죠. 
  - 프로토타입이 바뀐 것을 눈치채지 못하기 때문입니다. 
  - 서버 사이드에서 자바스크립트를 사용 중일 땐 이런 버그가 취약점이 되기도 합니다.
  - `toString`을 비롯한 내장 메서드에 할당을 할 때도 같은 이유 때문에 예상치 못한 일이 일어날 수 있습니다.

- 그렇다면 이런 문제는 어떻게 예방할 수 있을까요?

  - 객체 대신 `맵`을 사용하면 모든 것이 해결됩니다.

  - 그런데 자바스크립트를 만든 사람들이 아주 오래전부터 이런 문제를 고려했기 때문에 `객체`를 써도 문제를 피할 수 있습니다. 

- 어떤 방법이 있는지 알아봅시다.

- 아시다시피 `__proto__`는 객체의 프로퍼티가 아니라 `Object.prototype`의 접근자 프로퍼티입니다.

  ![image-20211102095729878](./img/image-20211102095729878.png)

  - 그렇기 때문에 `obj.__proto__`를 읽거나 쓸때는 이에 대응하는 getter·setter가 프로토타입에서 호출되고 `[[Prototype]]`을 가져오거나 설정합니다.
  - 이 절을 시작할 때 언급한 것처럼 `__proto__`는 `[[Prototype]]`에 접근하기 위한 방법이지 `[[Prototype]]` 그 자체가 아닌 것이죠.

- 이제 간단한 트릭을 써 객체가 연관 배열의 역할을 다 할 수 있도록 해보겠습니다.

  ```javascript
  let obj = Object.create(null);
  
  let key = prompt("입력하고자 하는 key는 무엇인가요?", "__proto__");
  obj[key] = "...값...";
  
  alert(obj[key]); // "...값..."이 제대로 출력됩니다.
  ```

  - `Object.create(null)`을 사용해 프로토타입이 없는 빈 객체를 만들어 보았습니다. 
    - `[[Prototype]]`이 `null`인 객체를 만든 것이죠.

  ![image-20211102100018843](./img/image-20211102100018843.png)

- `Object.create(null)`로 객체를 만들면 `__proto__` getter와 setter를 상속받지 않습니다. 이제 `__proto__`는 평범한 데이터 프로퍼티처럼 처리되므로 버그 없이 예시가 잘 동작하게 됩니다.

  - 이런 객체는 ‘아주 단순한(very plain)’ 혹은 ‘순수 사전식(pure dictionary)’ 객체라고 부릅니다. 일반 객체 `{...}` 보다 훨씬 단순하기 때문이죠.
  - 아주 단순한 객체는 내장 메서드가 없다는 단점이 있습니다. `toString`같은 메서드를 사용할 수 없죠.

  ```javascript
  let obj = Object.create(null);
  
  alert(obj); // Error: Cannot convert object to primitive value (toString이 없음)
  ```

- 연관 배열로 쓸 때는 이런 단점이 문제가 되진 않습니다.

  - 객체 관련 메서드 대부분은 `Object.keys(obj)` 같이 `Object.something(...)` 형태입니다. 
  - 이 메서드들은 프로토타입에 있는 게 아니기 때문에 '아주 단순한 객체’에도 사용할 수 있습니다.

  ```javascript
  let chineseDictionary = Object.create(null);
  chineseDictionary.hello = "你好";
  chineseDictionary.bye = "再见";
  
  alert(Object.keys(chineseDictionary)); // hello,bye
  ```





### 요약

프로토타입에 직접 접근할 땐 다음과 같은 모던 메서드를 사용할 수 있습니다.

- [Object.create(proto, [descriptors\])](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Object/create) – `[[Prototype]]`이 `proto`인 객체를 만듭니다. 참조 값은 `null`일 수 있고 프로퍼티 설명자를 넘기는 것도 가능합니다.
- [Object.getPrototypeOf(obj)](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Object.getPrototypeOf) – `obj`의 `[[Prototype]]`을 반환합니다(`__proto__` getter와 같습니다).
- [Object.setPrototypeOf(obj, proto)](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Object.setPrototypeOf) – `obj`의 `[[Prototype]]`을 `proto`로 설정합니다(`__proto__` setter와 같습니다).

사용자가 키를 직접 만들 수 있게 허용하면, 내장 `__proto__` getter·setter는 안전하지 않습니다. 키가 `"__proto__"`일 때 에러가 발생할 수 있죠. 단순한 에러면 좋겠지만 보통 예측 불가능한 결과가 생깁니다.

이를 방지하려면 `Object.create(null)`을 사용해 `__proto__`가 없는 '아주 단순한 객체’를 만들거나, `맵`을 일관되게 사용하는 것이 좋습니다.

한편, `Object.create`를 사용하면 객체의 얕은 복사본(shallow-copy)을 만들 수 있습니다.

```javascript
let clone = Object.create(Object.getPrototypeOf(obj), Object.getOwnPropertyDescriptors(obj));
```

`__proto__`는 `[[Prototype]]`의 getter·setter라는 점과 다른 메서드처럼 `Object.prototype`에 정의되어 있다는 것도 확인해 보았습니다.

`Object.create(null)`을 사용하면 프로토타입이 없는 객체를 만들 수 있습니다. 이런 객체는 '순수 사전’처럼 사용됩니다. `"__proto__"`를 키로 사용해도 문제를 일으키지 않죠.

이런 내용과 더불어 아래 메서드들을 같이 살펴보면 좋습니다.

- [Object.keys(obj)](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Object/keys) / [Object.values(obj)](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Object/values) / [Object.entries(obj)](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Object/entries) – `obj` 내 열거 가능한 프로퍼티 키, 값, 키-값 쌍을 담은 배열을 반환합니다.
- [Object.getOwnPropertySymbols(obj)](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Object/getOwnPropertySymbols) – `obj` 내 심볼형 키를 담은 배열을 반환합니다.
- [Object.getOwnPropertyNames(obj)](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Object/getOwnPropertyNames) – `obj` 내 문자형 키를 담은 배열을 반환합니다.
- [Reflect.ownKeys(obj)](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Reflect/ownKeys) – `obj`내 키 전체를 담은 배열을 반환합니다.
- [obj.hasOwnProperty(key)](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Object/hasOwnProperty) – 상속받지 않고 `obj` 자체에 구현된 키 중 이름이 `key`인 것이 있으면 `true`를 반환합니다.

`Object.keys`를 비롯하여 객체의 프로퍼티를 반환하는 메서드들은 객체가 ‘직접 소유한’ 프로퍼티만 반환합니다. 상속 프로퍼티는 `for..in`을 사용해 얻을 수 있습니다.

