# [Design Patterns] Singleton Pattern

> 참고: 
>
> https://youtu.be/VGLjQuEQgkI
>
> https://webdevtechblog.com/%EC%8B%B1%EA%B8%80%ED%84%B4-%ED%8C%A8%ED%84%B4-singleton-pattern-db75ed29c36





## 1. 특징

- 생성 패턴
- Class의 인스턴스가 오직 한 개만 존재할 수 있다.
- 다른 클래스들은 Singleton 클래스의 인스턴스에 접근할 수 있어야 한다.
- Logging, Cache, Session, Drivers에 이용된다.





## 2. 구현

- 생성자를 private으로 설정

  - 외부에서 생성자를 통해 인스턴스를 생성하지 못하게 한다.
  - Singleton 클래스의 인스턴스 생성은 오직 클래스 내부에서만 가능하다.

- getInstance() 메서드를 public static으로 설정

  - getInstance() 메서드는 Singleton 클래스의 유일한 인스턴스를 반환하는 메서드이다.

  - getInstance() 메서드는 `인스턴스.getInstance()` 꼴로 호출 불가능함.

    오로지 `Singleton_클래스명.getInstance()` 꼴로 호출 해야함. -> static 설정

    - 왜냐면 외부에서는 인스턴스에 직접 접근이 불가능하기 때문

- Instance type을 private static으로 설정

  - 인스턴스를 외부에서 직접 접근 불가능하게 설정 -> private
  - 인스턴스를 클래스 변수로 설정 -> static





### Initialization Type (초기화 방법)



#### 1) 즉시 초기화 (Eager Initialization)

```java
class SingletonEager {

    private static SingletonEager instance = new SingletonEager();

    private SingletonEager() {}

    public static SingletonEager getInstance() {
        return instance;
    }
}
```

- 클래스 로딩 시점에 인스턴스가 바로 생성됨
- getInstance()를 할 때는 이미 인스턴스가 생성된 상태임







#### 2) 지연 초기화 (Lazy Initialization)

```java
class SingletonLazy {

    private static SingletonLazy instance;
    
    private SingletonLazy() {}

    public static SingletonLazy getInstance() {
        if(instance == null) {
            instance = new SingletonLazy();
        }

        return instance;
    }
}
```

- `getInstance()` 메서드를 호출할 때까지 인스턴스 생성을 미룸

- Multi-Thread 환경에서 취약하다.
  - 두 thread가 동시에 `if(instance == null)` 문으로 들어오면 instance가 두 번 생성될 수도 있다.





#### 3) Thread safe Synchronized Method Initialization (`synchronized` 키워드 사용)

```java
class SingletonSynchronizedMethod {

    private static SingletonSynchronizedMethod instance;
    
    private SingletonSynchronizedMethod() {}

    public static synchronized SingletonSynchronizedMethod getInstance() {
        if(instance == null) {
            instance = new SingletonSynchronizedMethod();
        }
        return instance;
    }
}
```

- 2번 방법의 문제점(thread 취약)을 해결함
  - `synchronized` 키워드 덕분에 한 스레드가 `getInstance()`를 다 수행하고 난 후에야 다른 스레드가 `getInstance()`를 수행할 수 있게됨.
  - 절대 instance 두 개 안생김

- 하지만 `synchronized` 키워드는 성능 저하 현상이 생긴다...

  - 성능이 거의 100배 가량 떨어진다고 한다.

  - 왜냐면 인스턴스가 생성됐든 안됐든 무조건 모든 스레드가 동기화가 되기 때문이다.
  - 인스턴스가 생성되지 않았을 때만 동기화가 되면 더 최적화할 수 있지 않을까?





#### 4) DCL(Double Checking Locking) 방식 

```java
class SingletonDCL {

    private volatile static SingletonDCL instance;
    
    private SingletonDCL() {}

    public static SingletonDCL getInstance() {
        if(instance == null) {
            synchronized(SingletonDCL.class) {
                if(instance == null) {
                    instance = new SingletonDCL();
                }
            }
        }
        return instance;
    }
}
```

- `volatile` 키워드??
  - 변수 값 불일치 문제를 해결하기 위해 사용
  - 자세한 내용은 https://nesoy.github.io/articles/2018-06/Java-volatile 참고

- 이 방식은 인스턴스가 생성되지 않은 경우에만 동기화 블럭이 실행된다.





### <u>5) LazyHolder 방식</u>

```java
public class Singleton { 
    
    private Singleton(){} 
    
    private static class LazyHolder { 
        private static final Singleton INSTANCE = new Singleton();
    } 
    
    public static Singleton getInstance() { 
        return LazyHolder.INSTANCE; 
    } 
    
}
```

- 가장 많이 사용되는 Singleton 방식