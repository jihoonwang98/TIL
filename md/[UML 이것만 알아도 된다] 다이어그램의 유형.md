# [UML 이것만 알아도 된다] 다이어그램의 유형



### UML의 다이어그램 종류

- 정적 다이어그램 (static diagram)
  - 클래스, 객체, 데이터 구조와 이것들의 관계를 그림으로 표현해 소프트웨어 요소에서 변하지 않는 논리적 구조를 보여준다.
- 동적 다이어그램 (dynamic diagram)
  - 실행 흐름을 그림으로 그리거나 실체의 상태가 어떻게 바뀌는지 그림으로 표현해 소프트웨어 안의 실체가 실행 도중 어떻게 변하는지 보여준다.
- 물리적 다이어그램 (physical diagram)
  - 소스 파일, 라이브러리, 바이너리 파일, 데이터 파일 등의 물리적 실체와 이것들의 관계들을 그림으로 표현해 소프트웨어 실체의 변하지 않는 물리적 구조를 보여준다.





### 이진 트리 예시

```java
public class TreeMap {
	TreeMapNode topNode = null;
    
    public void add(Comparable key, Object value) {
        if(topNode == null) 
            topNode = new TreeMapNode(key, value);
        
        else 
            topNode.add(key, value);
    }
    
    public Object get(Comparable key) {
        return topNode == null ? null : topNode.find(key);
    }
}


class TreeMapNode {
    private final static int LESS = 0;
    private final static int GREATER = 1;
    
    private Comparable itsKey;
    private Object itsValue;
    private TreeMapNode nodes[] = new TreeMapNode[2];
    
    public TreeMapNode(Comparable key, Object value) {
        itsKey = key;
        itsValue = value;
    }
    
    public Object find(Comparable key) {
        if(key.compareTo(itsKey) == 0) {
            return itsValue;
        }
        
        return findSubNodeForKey(selectSubNode(key), key);
    }
    
    private int selectSubNode(Comparable key) {
        return (key.compareTo(itsKey) < 0) ? LESS : GREATER;
    }
    
    
    private Object findSubNodeForKey(int node, Comparable key) {
        return nodes[node] == null ? null : nodes[node].find(key);
    }
    
    
    public void add(Comparable key, Object value) {
        if(key.compareTo(itsKey) == 0) 
            itsValue = value;
        
        else 
            addSubNode(selectSubNode(key), key, value);
    }
    
    private void addSubNode(int node, Comparable key, Object value) {
        if (nodes[node] == null) 
            nodes[node] = new TreeMapNode(key, value);
        else 
            nodes[node].add(key, value);
    }
}
```







### 클래스 다이어그램

![diagram1](https://lh6.googleusercontent.com/7gEMUHzpwE06QYatV9tg_J_p1-CygnIipg9H1FTJsWn9zTNlThxDKj5Iqtw3oaqSZKscacMXZCDWwrIaA6hDlgZaKnX_KEmt0BFXB4djGNSZqGcoGRugS-jdtXYDiYQLVBAJhrZ9)

위의 클래스 다이어그램은 프로그램 안의 주요 클래스와 주요 관계를 보여준다.

- `TreeMap` 클래스

  - 이 클래스에는 `add(key, value)`와 `get(key)`이라는 public 메서드가 있다.

  - `TreeMap`은 `topNode`라는 변수로 `TreeMapNode` 객체를 참조한다.

- `TreeMapNode` 클래스

  - 모든 `TreeMapNode`는 `nodes`라는 컨테이너에 다른 `TreeMapNode` 인스턴스 두 개의 참조를 담아 둔다.
  - 모든 `TreeMapNode`는 `itsKey`와 `itsValue`라는 변수로 또 다른 두 인스턴스도 참조한다.
    - `itsKey` 변수는 `Comparable` 인터페이스를 구현하는 인스턴스의 참조를 담는다.
    - `itsValue` 변수는 그런 제한 없이 그냥 어떤 객체의 참조를 담는다.





**클래스 다이어그램 읽는법**

- 사각형은 클래스를 나타내고, 화살표는 관계를 나타낸다.
- 위의 다이어그램에서 모든 관계는 연관(association)이다. 
  - 연관은 한쪽 객체가 다른 쪽 객체를 참조하며, 그 참조를 통해 그 객체의 메서드를 호출하는 것을 나타내는 단순한 데이터 관계다.
  - 연관 위에 쓴 이름은 참조를 담는 변수의 이름과 대응된다.
- 화살표 옆에 쓴 숫자는 보통 이 관계를 맺음으로써 생기는 인스턴스의 개수를 나타낸다.
  - 만약 이 숫자가 1보다 크다면, 어떤 컨테이너를 사용한다는 뜻인데, 컨테이너로 대개 배열을 사용한다.

- 클래스 아이콘은 여러 구획으로 나뉠 수도 있다. 
  - 첫 번째 구획에는 언제나 클래스 이름을 쓴다.
  - 다른 구획에는 각각 함수와 변수를 쓴다.
- `<<interface>>` 표기법은 `Comparable`이 인터페이스임을 나타낸다.
- 설명한 표기법은 대부분 반드시 써야 하는 것이 아니다. 선택해서 쓸 수 있다.







### 객체 다이어그램



![diagram2](https://lh5.googleusercontent.com/bLIT3f_JgjDnUuThdCEUjdo1QFD6ox-sLixNBFxD-Htuhwr4XQ6OkA7Jnl3pGLvLhLWG8668aABOlMurSkabJJbEXHJW2I-nEBAfoHHm9IaxkRn26Tcf1fSbUcA0YrVi2BCIF9v0)



객체 다이어그램은 시스템 실행 중 어느 순간의 객체와 관계를 포착해서 보여 준다.

한 순간의 메모리 상태를 스냅샷으로 찍어둔 것이라고 생각해도 좋다.



- 객체는 사각형으로 표현되며, 이름에 밑줄이 있다.
- 콜론(:) 다음에 나오는 이름은 이 객체가 속한 클래스의 이름이다.
- 객체마다 아래 구획에 그 객체의 `itsKey` 변수의 값이 나와 있는 것에 주목하라.
- 객체 사이의 관계는 연결(link)이라 하며, 연결은 클래스 다이어그램에서의 연관에서 유도된다.







### 시퀀스 다이어그램

![diagram3](https://lh4.googleusercontent.com/OYacn_5Piev_nAe69-lMxq_dwumzGWfdZ3U1o-LnJUB9lpnjQ09JJkXvgKNPIK8jLBp2gYH7e_BwkXankMkkAnI1FVQrFv39UJSJxPxNf4JtpdNB8UGOreVWwNYEm9c0BDKfol3B)



위의 시퀀스 다이어그램은 `TreeMap.add(key, value)` 메서드가 어떻게 구현되는지 기술한다.



- 허수아비는 알려지지 않은 메서드 호출자를 나타낸다.
  - 이 호출자가 `TreeMap` 객체의 `add(key, value)` 메서드를 호출한다.
- `topNode` 변수가 `null`일 경우, `TreeMap`은 응답으로 새로운 `TreeMapNode`를 생성하고 그것을 `topNode`에 할당한다.
- `topNode` 변수가 `null`이 아닌 경우, `topNode`에 `add(key, value)` 메시지를 보낸다.



- 대괄호([]) 안의 불린 표현식은 '가드(guard)'라고 하며, 어떤 경로를 따라가야 할 지 알려준다.

- `TreeMapNode` 아이콘에 닿은 화살표는 '생성(construction)'을 나타낸다.

- 한쪽 끝에 원이 그려진 작은 화살표는 '데이터 토큰(data token)'이라고 하고,

  이 경우에 이 데이터 토큰은 생성자의 인자를 나타낸다.

- `TreeMap` 아래 홀쭉한 사각형은 '활성 상자(activation)'라고 부르는데,

  `add(key, value)` 메서드가 실행되는 데 시간이 어느정도 걸리는지 보여준다.







### 협력 다이어그램

![diagram3 (1)](https://lh4.googleusercontent.com/fFpxq9M3cmFcRF7D7RDfNbaK-yF2m0LpkFtZG8FmJxVumWodquiZRvzqwrMkYTkI6kqRPNyYMtLn4fyVacg7PZA8tIy7rJAUDwbQbyFVzV2ezMJajJZ18a7HEog17Bvhw7I8PVmu)



위 그림은 `topNode`가 `null`이 아닐 경우를 보여 주는 '협력 다이어그램(collaboration diagram)'이다.

협력 다이어그램의 정보는 시퀀스 다이어그램에 담긴 정보와 똑같다.

하지만 시퀀스 다이어그램은 메시지를 보내고 받는 순서를 명확히 하는 것이 목적인 반면,

협력 다이어그램은 <u>객체 사이의 관계를 명확히 하는 것</u>이 목적이다.



- 객체들은 연결이라고 부르는 관계로 맺어진다.
  - 어떤 객체가 다른 객체에 메시지를 보낼 수 있다면, 두 객체 사이에 연결이 있다고 말한다.
  - 이 연결 위로 지나다니는 것이 바로 메시지다.
- 메시지는 작은 화살표로 그리며, 메시지 위에는 메시지 이름과 시퀀스 숫자, 그리고 이 메시지를 보낼 때 적용하는 모든 가드를 적는다.
  - 시퀀스 숫자는 다이어그램에서 메시지가 호출되는 순서를 말한다.

- 호출의 계층 구조는 시퀀스 숫자에서 볼 수 있는 점(.)을 사용한 구조로 알 수 있다.
  - `TreeMap.add(key, value)` 함수(메시지 1번)는 `TreeMapNode.add(key, value)` 함수(메시지 1.1번)를 호출하는 식인데, 여기서 메시지 1.1번은 메시지 1번이 호출한 함수에서 처음 보내는 메시지를 나타낸다.





### 상태 다이어그램

![diagram4](https://lh5.googleusercontent.com/dPndOaUtxR9RpjLdvRPUgB8x25Le4jjRIYGbMyentNjuc56kGfR9E5sS8NT3tksHnlyitgeIdRBFLSB663k7PKMUODHEoVpPIW0bMczhc92JyZgkk9R4wl0mNMxtm7CaUbksV5_X)



UML에는 유한 상태 기계(finite state machine)를 나타내기 위한 상당히 방대한 표기법이 들어 있다.

위의 그림은 지하철 개찰구를 상태 기계로 표현한 것인데, 

이 기계에는 Locked(잠김)와 Unlocked(풀림)라는 두 가지 '상태'가 있고, 두 가지 '이벤트'를 받을 수 있다.

coin(표) 이벤트는 사용자가 개찰구에 표를 넣었음을 뜻하고,

pass(지나감) 이벤트는 사용자가 개찰구를 통해 지나감을 뜻한다.



- 화살표는 '전이(transition)'라고 부른다.
  - 이 전이 화살표에는 전이를 일으키는 이벤트와 전이가 수행하는 행동을 레이블로 단다.
  - 전이가 일어나면 시스템의 상태가 바뀐다.
- 위 그림은 다음과 같이 우리말로 변역할 수 있다.
  - Locked 상태에서 coin 이벤트를 받으면, Unlocked 상태로 가고 Unlock 함수를 호출한다.
  - Unlocked 상태에서 pass 이벤트를 받으면, Locked 상태로 가고 Lock 함수를 호출한다.
  - Unlocked 상태에서 coin 이벤트를 받으면, 그대로 Unlocked 상태에 남아 있으면서 Thankyou 함수를 호출한다.
  - Locked 상태에서 pass 이벤트를 받으면, 그대로 Locked 상태에 남아 있으면서 Alarm 함수를 호출한다.



이런 다이어그램은 시스템의 행동 방식을 파악할 때 굉장히 유용하다.

어떤 사용자가 표를 넣은 다음 아무 이유 없이 '다시 표를 넣는 것처럼' 예상하지 못한 경우에 시스템이 어떻게 행동해야 하는지 탐색할 기회를 마련해준다.







## Java 언어로 배우는 다자인 패턴 입문 中



### UML

- UML은 Unified Modeling Language의 약자로, 

  시스템을 시각화하거나 시스템의 사양이나 설계를 문서화하기 위한 표현 방법이다.





### 클래스 다이어그램

- UML의 클래스 다이어그램(Class Diagram)은 클래스나 인스턴스, 인터페이스 등의 <u>정적인 관계</u>를 표현한 것입니다.
  - 클래스 다이어그램이라는 이름으로 불리지만, 클래스만 등장하는 것은 아닙니다.





**예시 1**

```java
abstract class ParentClass {
    int field1;
    static char field2;
    abstract void methodA();
    double methodB() {
        // ...
    }
}

class ChildClass extends ParentClass {
    void methodA() {
        // ...
    }
    
    static void methodC() {
        // ...
    }
}
```



**UML 클래스 다이어그램**

![diagram1](https://lh3.googleusercontent.com/gBD0dEGMClT6XwDmcgChhTaQhcRE_VFCBmMMzcP8_CQUriv4XEHRbZxibVozUkK6KMtIS2LnHhN97k3ApfleTy3CYNxz0G4_J6kWhx9vV3bz_df4onER6SIwO94BbE5L6W6PkhgG)



- 각각의 클래스는 직사각형으로 표현한다.
  - 직사각형을 수평선으로 분할하여 위에서부터 차례대로 클래스의 이름, 필드의 이름, 메서드의 이름을 적는다.
  - 이름뿐만 아니라 부가적인 정보(액세스 제어자나 메서드의 인수나 형태 등)를 쓰는 경우도 있고, 반대로 주목할 필요가 없는 항목은 생략하는 경우도 있다.

- 실선의 화살표는 클래스의 계층관계를 표시한다.
  - 화살표는 **하위클래스에서 상위클래스**로 향하고 있다. 
- abstract 클래스와 abstract 메서드는 이탤릭체를 사용해 표현한다.
  - `ParentClass`는 추상 클래스이므로 이탤릭체로 표현되었다.
  - 마찬가지로 `ParentClass`의 `methodA()` 메서드 역시 추상 메서드이므로 이탤릭체로 표현되었다.
- static 필드와 static 메서드에는 밑줄을 사용해 표현한다.
  - `ParentClass`의 `field2`는 static 필드이므로 밑줄이 그어져 있다.
  - `ChildClass`의 `methodC()`는 static 메서드이므로 밑줄이 그어져 있다.





> **[extends 화살표의 방향]**
>
> UML에서는 하위 클래스에서 상위 클래스를 향해 화살표가 뻗어 있다.
>
> 상위 클래스를 기준으로 하여 하위 클래스를 만들기 때문에 화살표를 반대로 표시하는 것이 이해하기 쉽다고 생각할 수 있겠으나, 다음과 같이 생각해보자.
>
> 하위 클래스를 정의할 때 extends로 상위 클래스를 지목한다.
>
> 따라서 하위 클래스는 상위 클래스를 알고 있다.
>
> 그러나 상위 클래스는 하위 클래스를 알고 있다고 할 수 없다. 
>
> 상대를 지목할 수 있는 것은 상대를 알고 있을 때뿐이다.
>
> 그래서 하위 클래스에서 상위 클래스로 화살표를 표시하는 것이다.







**예시 2 (인터페이스) **

```java
interface Printable {
    abstract void print();
    abstract void newPage();
}

class PrintClass implements Printable {
    void print() {
        //...
    }
    
    void newPage() {
        //...
    }
}
```



![diagram2](https://lh3.googleusercontent.com/AhQvVhUEbwWH99LTidQ5cBhf73dl0CzmkoJV5ceLgofW1VrBOE7AC1W7n887h4Jc7qk7XvGiUqFH1Asz5SB8rvq0OvcGRJZ5r7AozsQmPwbj0kGhvlO1reY3EQhch7TohcOkLalc)



- 점선의 화살표는 인터페이스와 구현 클래스의 관계를 나타내고 있다.
  - 화살표는 **구현 클래스에서 인터페이스**로 향한다.





**예시 3 (집약 관계)**

```java
class Color {
    // ...
}

class Fruit {
    Color color;
    // ...
}

class Basket {
    Fruit[] fruits;
    // ...
}
```



![diagram3](https://lh5.googleusercontent.com/mRQg_L3zuCJpBXtF_0GkZRi9DnjX4h3KQ85rdkxA43NoGlRYDyBkrh2NbavFqbEnGFbKYisMGP0vqTca6RDnfLhaLbFoqId-fxsaRURgZt-S_X711b9vFVhrdulM7MSWuvzeXv5P)



- 바구니(Basket)에 과일(Fruit)이 여러 개 들어있고, 각 과일은 색깔(Color)를 가지고 있다.
- 이와 같이 '갖고 있는' 관계를 '**집약(aggregation)**'이라고 한다.
  - 인스턴스를 갖고 있으면 개수에 상관없이 그 관계는 집약이라고 한다.





> **[합성(composition) vs 집약(aggregation)]**
>
> > https://rightnowdo.tistory.com/entry/Association-vs-Aggregation-vs-Composition-%EC%9E%98-%EB%B9%84%EA%B5%90-%EC%84%A4%EB%AA%85%ED%95%9C-%EA%B8%80%ED%8E%8C
>
> 
>
> 사실상, Aggregation과 Composition은 Association의 특별한 Case일 뿐이다.
>
> Aggregation과 Composition은 모두 한 클래스의 객체가 다른 클래스의 객체를 "소유"한다.
>
> 그러나 이들간에는 미묘한 차이가 있다:
>
> - Aggregation은 Child Class가 Parent Class에 독립적으로 존재할 수 있음을 내포하고 있다.
>   - 교실 클래스와 학생 클래스를 생각해보자. 교실 클래스를 삭제해도 학생 클래스는 여전히 존재할 수 있다.
>
> - Composition은 Child Class가 Parent Class에 독립적으로 존재할 수 없음을 내포하고 있다.
>
>   - 집 클래스와 방 클래스를 생각해보자. 방 클래스는 집 클래스가 없이 존재할 수 없다.
>
>   
>
> **Composition 예시**
>
> ![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcLLNhk%2Fbtqvtfmm5kC%2FZko8oRJ8ij7fPx1fDc3c6K%2Fimg.png)
>
> 
>
> - 클래스 A와 클래스 B가 서로 lifecycle이 강력하게 의존적일 때 composition 관계를 사용할 수 있다.
>   - 다시 말해, 클래스 A가 삭제되면 클래스 B도 삭제되는 경우를 말한다.
>
> 
>
> **Aggregation 예시**
>
> ![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FpcF5n%2Fbtqvq98VRnL%2F0ZwHvl3lwWyJGWN6KOo31K%2Fimg.png)







**예시 4 (접근 제어자)**

```java
class Something {
    private int privateField;
    protected int protectedField;
    public int publicField;
    int packageField;
    
    private void privateMethod() { }
    protected void protectedMethod() { }
    public void publicMethod() { }
    void packageMethod() { }
}
```



![diagrm4](https://lh4.googleusercontent.com/7x1Zwxv0LfB9ki6MldcbZpGTQRhQOkUBmKBZEsaGi_A2afJ_iltY_X0O2bgfdoO5IJ4jne82mtqfGNVCv_MlUVR2aswTShr0OWsGlQ5wnPjM16OnS462cKmComiYlyGhloowrt4n)





**예시 5 (클래스의 관계)**



![diagrm4 (2)](https://lh4.googleusercontent.com/yMQ4Hy4jEobrXT8kpxdNA5rSLkZWiM1yQ8dfttCcUbVXP2KIcQ6njMXI8F2k551m_opUeWHjTM2kQoz0v0xhv7o5N_xnRnuiUQjt4PhNgKzYm7JynW7b1MvR0msK6MVkCBJnLd7v)

- Client가 Target을 사용(Uses)한다.







### 시퀀스 다이어그램

```java
class Client {
    Server server;
    void work() {
        server.open();
        server.print("Hello");
        server.close();
    }
    // ...
}

class Server {
    Device device;
    void open() {
        // ...
    }
    
    void print(String s) {
        device.write(s);
        // ...
    }
    
    void close() {
        // ...
    }
}

class Device {
    void write(String s) {
        // ...
    }
}
```



![diagrm4 (3)](https://lh4.googleusercontent.com/8pDokFnNLxH9euZo5h33fUjoOhfia_z2QGvo7IlSTWEtOhVkguZITh6E2sXCVJEx3Lg4v-GDL7KiPBSpKDUrCBwjQtjD4DpieHSFLy8XxoFnodTUj8Vs49WzbYDuLUDG6KcgK-8n)



- 위의 그림에는 세 개의 인스턴스가 등장하고 있다.
  - 인스턴스는 각각 다이어그램의 위쪽에 있는 직사각형에 대응한다.
  - 직사각형 안에는 :Client, :Server, :Device와 같이 콜론 뒤에 클래스명을 표기하고 밑줄을 긋는다.
  - 이것은 각각 Client의 인스턴스, Server의 인스턴스, Device의 인스턴스임을 나타낸다.
  - 만약 각각의 인스턴스에 이름이 필요할 경우에는 server:Server와 같이 콜론 앞에 이름을 적는다.
- 각각의 인스턴스에서 아래 방향으로 뻗어있는 점선을 Lifeline(생존선)이라고 한다.
  - 여기서 시간은 아래 방향으로 흐른다. (위쪽은 과거 아래쪽은 미래)
  - Lifeline은 인스턴스가 존재하는 동안만 존재한다.
  - Lifeline의 중간에 가늘고 긴 직사각형은 객체가 활동 중인 것을 나타낸다.
- 앞이 검은 화살표의 실선은 메서드의 호출을 나타낸다.
  - 여기서는 client가 server의 open 메서드를 호출한 것을 나타낸다.
- 점선의 화살표는 메서드에서의 리턴(반환)을 표시한다.
  - 위의 그림에서는 모든 메서드의 리턴을 그리고 있지만 생략하는 경우도 있다.
