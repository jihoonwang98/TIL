# State Pattern



## 문제점 인식

### 예시

```java
public class Behavior {

    static final String ANGRY = "ANGRY";
    static final String HAPPY = "HAPPY";
    static final String DEPRESSION = "DEPRESSION";

    private String currState = ANGRY;

    public void behave() {
        if (this.currState.equals(ANGRY)) {
            System.out.println("나 화났다!!!!");
        } else if (this.currState.equals(HAPPY)) {
            System.out.println("나 기쁘다~~~!!");
        } else if (this.currState.equals(DEPRESSION)) {
            System.out.println("나 우울하다...");
        }
    }

    public void comfort() {
        if (this.currState.equals(ANGRY)) {
            System.out.println("화났을 땐 잠을 자자..");
        } else if (this.currState.equals(HAPPY)) {
            System.out.println("기쁠땐 술을 마시자!");
        } else if (this.currState.equals(DEPRESSION)) {
            System.out.println("우울할 땐 힙합춤을 추자..");
        }
    }

    public void work() {
        if (this.currState.equals(ANGRY)) {
            System.out.println("화났을 땐 일이 안된다...");
        } else if (this.currState.equals(HAPPY)) {
            System.out.println("기쁠땐 일도 잘된다!!");
        } else if (this.currState.equals(DEPRESSION)) {
            System.out.println("우울할 땐 일이 안된다...");
        }
    }

    public void setCurrState(String state) {
        this.currState = state;
    }
}
```







### 상태가 하나 더 추가되면?

```java
public class Behavior {

    static final String ANGRY = "ANGRY";
    static final String HAPPY = "HAPPY";
    static final String DEPRESSION = "DEPRESSION";
    
    /* 추가 */ static final String SAD = "SAD"; /* 추가 */
    

    private String currState = ANGRY;

    public void behave() {
        if (this.currState.equals(ANGRY)) {
            System.out.println("나 화났다!!!!");
        } else if (this.currState.equals(HAPPY)) {
            System.out.println("나 기쁘다~~~!!");
        } else if (this.currState.equals(DEPRESSION)) {
            System.out.println("나 우울하다...");
        }
        
        /* 추가 */ 
        else if(this.currState.equals(SAD)) {
            // 슬플때 로직
        } 
        /* 추가 */
        
    }

    public void comfort() {
        if (this.currState.equals(ANGRY)) {
            System.out.println("화났을 땐 잠을 자자..");
        } else if (this.currState.equals(HAPPY)) {
            System.out.println("기쁠땐 술을 마시자!");
        } else if (this.currState.equals(DEPRESSION)) {
            System.out.println("우울할 땐 힙합춤을 추자..");
        }
        
        /* 추가 */ 
        else if(this.currState.equals(SAD)) {
            // 슬플때 로직
        } 
        /* 추가 */
    }

    public void work() {
        if (this.currState.equals(ANGRY)) {
            System.out.println("화났을 땐 일이 안된다...");
        } else if (this.currState.equals(HAPPY)) {
            System.out.println("기쁠땐 일도 잘된다!!");
        } else if (this.currState.equals(DEPRESSION)) {
            System.out.println("우울할 땐 일이 안된다...");
        }
        
        /* 추가 */ 
        else if(this.currState.equals(SAD)) {
            // 슬플때 로직
        } 
        /* 추가 */
    }

    public void setCurrState(String state) {
        this.currState = state;
    }
}
```

- 읽기도 힘들고 유지보수하기 힘들다.
- `setCurrState(..)` 메서드의 인자로 엉뚱한 값이 들어오는 것을 막을 수 없다







## 해결책



### `State` 인터페이스

```java
public interface State {
    void behave();
    void comfort();
    void work();

    static State initState() {
        return new Happy();
    }
}
```



### `Angry`, `Happy`, `Depression` 상태 클래스

```java
public class Angry implements State{
    @Override
    public void behave() {
        System.out.println("나 화났다!!!!");
    }

    @Override
    public void comfort() {
        System.out.println("화났을 땐 잠을 자자..");
    }

    @Override
    public void work() {
        System.out.println("화났을 땐 일이 안된다...");
    }
}
```



```java
public class Happy implements State{
    @Override
    public void behave() {
        System.out.println("나 기쁘다~~~!!");
    }

    @Override
    public void comfort() {
        System.out.println("기쁠땐 술을 마시자!");
    }

    @Override
    public void work() {
        System.out.println("기쁠땐 일도 잘된다!!");
    }
}
```



```java
public class Depression implements State{
    @Override
    public void behave() {
        System.out.println("나 우울하다...");
    }

    @Override
    public void comfort() {
        System.out.println("우울할 땐 힙합춤을 추자..");
    }

    @Override
    public void work() {
        System.out.println("우울할 땐 일이 안된다...");
    }
}
```





### `Behavior` 클래스

```java
public class Behavior {

    private State currState = State.initState();

    public void behave() {
        currState.behave();
    }

    public void comfort() {
        currState.comfort();
    }

    public void work() {
        currState.work();
    }

    public void setCurrState(State state) {
        this.currState = state;
    }
}
```

- `setCurrState(..)`의 메서드 인자로 `State` 타입만 들어올 수 있으므로 엉뚱한 값이 들어올 수 없게 되었따.
- 상태에 따라 달라지는 행동은 각 상태 클래스에 정의된다.







### 상태가 하나 더 추가되면?

```java
public class NewState implements State{
    @Override
    public void behave() {
        // 새로운 behave() 로직
    }

    @Override
    public void comfort() {
        // 새로운 comfort() 로직
    }

    @Override
    public void work() {
        // 새로운 work() 로직
    }
}
```

- 위처럼 그냥 `State` 인터페이스를 구현하는 새로운 상태 클래스를 하나 더 정의하면 된다.





### 상태를 하나 삭제하고 싶으면?

- 그냥 상태 클래스를 하나 삭제해도 `Behavior` 클래스에 영향이 가지 않는다.

