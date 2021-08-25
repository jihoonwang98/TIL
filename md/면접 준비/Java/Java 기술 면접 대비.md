# Java 기술 면접 대비





## OOP



### OOP의 4가지 특징

1. 추상화

   - 구체적인 사물들의 공통적인 특징을 파악해서 이를 하나의 개념(집합)으로 다루는 것
   - 인터페이스로 클래스들의 공통적인 특성(변수, 메서드)들을 묶어서 표현하는 것

2. 캡슐화

   - 정보 은닉(information hiding): 필요가 없는 정보는 외부에서 접근하지 못하도록 제한하는 것
   - 필드와 메서드를 클래스로 묶는 데이터 캡슐화
     - 높은 응집도(변경 시 해당 클래스만 변경됨)와 낮은 결합도(다른 클래스들의 간섭이 낮음)를 유지하게 해줌.

3. 상속

   - 상위 개념의 특징을 하위 개념이 물려받는 것

   - 상속은 캡슐화를 유지, 클래스의 재사용이 용이하도록 해준다.

4. 다형성

   - 서로 다른 클래스의 객체가 같은 메시지를 받았을 때 각자의 방식으로 동작하는 능력





### OOP의 5대 원칙 (SOLID)

-> 유지보수와 확장이 쉬운 소프트웨어를 만들기 위한 원칙



- **<u>S</u>**RP (Single Responsible Principle): 단일 책임 원칙
  - 객체는 단 하나의 책임만을 가져야 한다.
  - 응집도는 높고, 결합도는 낮은 프로그램을 위한 원칙



- **<u>O</u>**CP (Open Closed Principle): 개방-폐쇄 원칙

  - 기존의 코드를 변경하지 않으면서(Closed) 기능을 추가할 수 있도록(Open) 설계가 되어야 한다.

  - 자주 변경되는 내용은 수정하기 쉽게 설계하고(인터페이스 이용),

    변경되지 않아야 하는 것은 수정되는 내용에 영향을 받지 않게 설계.



- **<u>L</u>**SP (Liskov Substitution Principle): 리스코프 치환 원칙
  - 일반화 관계에 대한 이야기며, 자식 클래스는 자신의 부모 클래스에서 가능한 행위를 수행할 수 있어야 한다.



- **<u>I</u>**SP (Interface Segregation Principle): 인터페이스 분리 원칙

  - 인터페이스를 클라이언트에 특화되도록 분리시키라는 설계 원칙

  - 한 클래스는 자신이 사용하지 않는 인터페이스를 구현하지 말아야 한다.

  - 하나의 일반적인 인터페이스보다는, 여러 개의 구체적인 인터페이스가 낫다.

    - 스마트폰으로 전화, 웹 서핑, 사진 촬영 등 다양한 기능을 사용할 수 있다.

      근데 전화할 때는 웹 서핑, 사진 촬영 등 다른 기능은 사용하지 않는다.

      따라서 전화, 웹 서핑, 사진 촬영 기능은 각각 독립된 인터페이스로 구현하여, 

      서로에게 영향을 받지 않도록 설계해야 한다.



- **<u>D</u>**IP (Dependency Inversion Principle): 의존 역전 원칙

  - 의존 관계를 맺을 때 변화하기 쉬운 것 또는 자주 변화하는 것보다는 변화하기 어려운 것, 거의 변화가 없는 것에 의존하라는 것이다.

  - 변화하기 쉬운것 -> 구체적인 것(구체클래스)

    변화하기 어려운것 -> 추상적인 것(인터페이스나 추상 클래스)





## Java





### JVM



#### JVM이란?

- Java 가상 머신(Java Virtual Machine)으로 Java 바이트 코드(.class 파일)를 실행할 수 있는 주체 



#### JVM의 구성



![](http://www.tcpschool.com/lectures/img_java_programming.png)





![](https://media.vlpt.us/images/hono2030/post/21adf2f3-f155-4e50-bdb6-5e8b1675129c/image.png)



> 바이트 코드 vs 바이너리 코드(기계어)
>
> - 바이트 코드
>   - 가상 머신(JVM)이 이해할 수 있는 언어
>   - CPU가 아닌 가상 머신이 이해할 수 있는 코드를 위한 이진 표현법
>   - 바이트 코드는 바이너리 코드로 변환된다
> - 바이너리 코드
>   - 0과 1로 구성된 이진 코드
> - 기계어
>   - 기계어는 0과 1로 구성된 바이너리 코드다.





JVM은 크게 4가지로 구성된다.

-> Class Loader, Execution Engine, Garbage Collector, Rutime Data Area



- **Class Loader**

  - javac가 컴파일한 클래스 파일(.class, 바이트코드)들을 모아서

    OS로부터 할당받은 메모리 영역인 <u>Runtime Data Area로 적재</u>하는 역할을 한다.



- **Execution Engine**

  - Class Loader에 의해 메모리에 적재된 클래스(바이트코드)들을 <u>기계어로 변경</u>해

    <u>명령어 단위(operation code)로 실행</u>하는 역할을 한다.

    - 명령어 단위 실행에는 2가지 방식이 있다.

    > **인터프리터 방식**: 명령어를 하나씩 수행 (기본적인 방식. 전체 수행은 느리나 명령어 하나씩의 동작은 빠름)
    >
    > **JIT(Just In Time compiler) 방식**: 전체 바이트 코드를 네이티브 코드로 변환하고 그 이후에는 인터프리팅하지 않고 네이티브 코드 상태로 실행 (전체적인 동작은 빠르나, 컴파일하는데 상당 시간 소요)



- **Garbage Collector**
  - GC는 <u>Heap 메모리 영역</u>에 생성된 객체들 중에 참조되지 않은 객체들을 제거하는 역할을 한다.
  - GC의 동작시간은 일정하게 정해져 있지 않기 때문에 <u>언제 객체를 정리할지 알 수 없다.</u>
    - 즉, 바로 참조가 없어지자마자 작동하는 것이 아니다!!
  - GC를 수행하는 동안 GC Thread를 제외한 다른 모든 Thread는 일시정지 상태가 된다.
    - 특히 Full GC가 일어나서 수초간 모든 Thread가 정지하면 심각한 장애로 이어진다.



- **Runtime Data Area**

  - <u>JVM의 메모리 영역</u>으로 자바 애플리케이션 실행시 사용되는 데이터를 적재하는 영역이다.

  - 크게 5가지 영역으로 구분된다.

    - Method Area:

      필드 정보(클래스 멤버 변수명, 데이터 타입, 접근 제어자 정보)

      메서드 정보(메서드명, 리턴 타입, 접근 제어자 정보)

      Type 정보(Interface인지 Class인지)

      Constant Pool(상수 풀: 문자 상수, 타입, 필드, 객체 참조 저장)

      static 변수

      final 클래스 변수

    - Heap Area:

      new 키워드로 생성된 객체와 배열이 생성되는 영역

      Method Area에 로드된 class만 생성 가능하며 

      GC가 참조되지 않는 메모리를 확인하고 제거하는 영역

    - Stack Area:

      지역변수, 파라미터, 리턴 값, 연산에 사용되는 임시 값 등을 저장

      `int a = 10`을 예로 들면 

      정수 값이 할당될 수 있는 메모리 공간을 a로 잡아두고 그 메모리 영역에 10을 넣는다.

      `클래스 A a = new A()`의 경우 

      A a는 스택영역에 저장되고,

      new로 생성된 A 클래스의 인스턴스는 Heap 영역에 저장된다.

      또한 스택 영역에 생성된 a는 Heap 영역의 주소값을 가지고 있다.

      메서드 호출 시마다 개별적으로 스택이 생성된다.

    - PC Register

      Thread가 생성될 때마다 생성되는 영역으로 Program Counter

      즉, 현재 쓰레드가 실행되는 부분의 주소와 명령을 저장하고 있는 영역이다.

      이것을 이용해 쓰레드를 돌아가면서 수행한다.

    - Native Method Stack

      자바외 언어로 작성된 네이티브 코드를 위한 메모리 영역이다.

      보통 C/C++ 등의 코드를 수행하기 위한 스택



Thread 생성시 Method, Heap 영역을 모든 Thread가 공유하고,

Stack, PC Register, Native Method Stack은 각각의 스레드마다 생성되고 공유되지 않는다.





#### JVM의 수행 순서

1. 프로그램이 실행되면 OS로부터 프로그램이 필요로하는 메모리를 할당받는다.
2. 자바 컴파일러를 통해 개발자가 작성한 코드(.java)를 바이트 코드(.class)로 변환한다.
3. **Class Loader**에서 바이트 코드를 JVM에 로딩시킨다.
4. 로딩된 바이트 코드를 **Execution Engine**을 통해 기계어로 해석한다.
5. 해석된 바이트 코드들은 **Runtime Data Area**에 배치되어 실질적인 수행이 이루어진다.







### 직렬화(serialization)란?

자바에서 입출력을 할때는 `스트림`이라는 통로를 통해 데이터가 이동한다.

하지만 `객체`는 **바이트형이 아니라서** 스트림을 통해 파일에 저장하거나 네트워크로 전송할 수 없다.



따라서 객체를 스트림을 통해 입출력하려면 `바이트 배열로 변환`하는 것이 필요한데,

이를 `직렬화`라고 한다.

반대로 스트림을 통해 받은 직렬화된 객체를 원래 모양으로 만드는 과정을 `역직렬화`라고 한다.





### Boxing과 UnBoxing이란?

기본 자료형(Primitive data type)을 Wrapper class로 바꾸어 주는 것을 `Boxing`,

Wrapper class를 기본 자료형으로 바꾸어 주는 것을 `UnBoxing`이라고 한다.







### HashMap vs HashTable vs ConcurrentHashMap 차이점

- HashMap
  - 주요 메서드에 **synchronized** 키워드가 없다.
  - key, value에 null을 입력할 수 있다.



- HashTable
  - 주요 메서드에 **synchronized** 키워드가 선언되어 있다.
  - key, value에 null을 허용하지 않는다.



- ConcurrentHashMap
  - **HashMap**을 **thread-safe** 하도록 만든 클래스
  - key, value에 null을 허용하지 않는다.







### String vs StringBuffer vs StringBuilder

- String
  - immutable (불변)
  - 객체를 한 번 할당할 시 **메모리 공간에 변동이 없다.** (할당 시 Heap String Pool 영역에 생성되어 그 값을 계속 사용)
  - 동기화를 신경쓰지 않아도 됨
  - 엄청나게 많은 문자열을 선언 및 연산할 시 성능저하를 고려해야 한다.



- StringBuffer
  - mutable (가변)
  - 각 메서드별로 Synchronized Keyword가 존재한다.
  - 멀티스레드 환경에서도 **동기화를 지원**한다. (thread-safe)



- StringBuilder
  - mutable (가변)
  - 동기화 지원 X





### 자바의 메모리 영역

- 메서드 영역
  - static 변수, 전역 변수, 코드에서 사용되는 Class 정보 등이 할당된다.



- Stack
  - 지역 변수, 함수(메서드) 등이 할당되는 LIFO 방식의 메모리.



- Heap
  - new 연산자를 통한 동적할당된 객체들이 저장되며, 가비지 컬렉션에 의해 메모리가 관리됨.





### JCF (Java Collection Framework)

- 주요 인터페이스

  - List
    - 순서 있음
    - 데이터 중복 허용
    - Vector, ArrayList, LinkedList, Stack, Queue

  

  - Set
    - 순서 없음
    - 데이터 중복 허용 X
    - HashSet, TreeSet

  

  - Map
    - 순서 없음
    - key는 중복 허용 X, value는 중복 허용 O
    - HashMap, TreeMap, Hashtable, Properties





### Generic

객체의 타입을 컴파일 시에 체크하기 때문에 객체의 타입 안정성을 높







### Vector vs ArrayList

- Vector
  - 동기화된 상태 (thread-safe)
  - 상대적으로 속도가 느림 (동기화 되어있어서)



- ArrayList
  - 동기화 안된 상태
  - 상대적으로 속도가 빠름 (동기화 안되어 있어서)
  - 멀티 쓰레드 환경이 아닐 경우 사용 권장







### ArrayList와 LinkedList의 차이

공통점: 모두 Java에서 제공하는 List 인터페이스를 구현한 Collection 구현체.



차이점

- ArrayList
  - 내부적으로 데이터를 배열로 관리함
  - 데이터별 인덱스가 있어 검색(조회)에 유리 (O(1))
  - 데이터 추가/삭제의 경우 불리
- LinkedList
  - 내부적으로 노드 단위로 데이터를 관리
  - 인덱스가 따로 없기 때문에 검색(조회)할 때 전 노드를 순회하여 검색해야 함
  - 데이터 추가/삭제 시 불필요한 데이터 복사X





### Overloading vs Overriding

- Overloading

  - **같은 함수 이름**을 가지고 있으나,

    **매개변수 리스트가 다른** 메서드를 만드는 것

  

- Overriding

  - 상위 클래스에 존재하는 메서드를 하위 클래스에서 필요에 맞게 재정의하는 것







### CheckedException vs UnCheckedException

|                                    | CheckedException                     | UnCheckedException          |
| ---------------------------------- | ------------------------------------ | --------------------------- |
| 처리 여부                          | 반드시 예외를 처리                   | 예외 처리를 강제하지 않음   |
| 확인 시점                          | 컴파일 단계                          | 실행(Runtime) 단계          |
| 예외 발생시 트랜잭션 처리 (스프링) | Roll-Back 하지 않음                  | Roll-Back 함                |
| 예외 종류                          | Runtime Exception을 제외한 모든 예외 | Runtime Exception 하위 예외 |







### final 키워드

- **final class**: 다른 클래스에서 **상속 금지**
- **final method**: 하위 클래스에서 해당 메서드 **오버라이딩 금지**
- **final variable**: **상수값**





### new String() vs ""

Java에서 문자열은 Heap 영역 내의 <u>String Pool</u>이라는 곳에서 따로 관리한다.

`""`으로 선언된 String은 **String Pool에 추가**되고, 해당 값을 참조 값으로 가지게 된다.



반면, `new String()`으로 생성된 String은 

String pool이 아닌  **Heap 영역**에 새로운 객체를 등록하게 된다.





### Primitive Type Size

- boolean: 1
- char: unsigned 2
- byte: 1
- short: 2
- int: 4
- long: 8
- float: 4
- double: 8





### Identity(동일성) vs Equality(동등성)   /  == vs equals()

- **Identity(동일성, == 비교)**
  - 참조타입: 객체의 주소 비교
  - 원시타입: 값 비교
- **Equality(동등성, equals() 비교)**
  - 참조타입: 객체의 값 비교 (equals() 메서드를 오버라이딩해서 동등성 판단 기준을 정의해야 함)