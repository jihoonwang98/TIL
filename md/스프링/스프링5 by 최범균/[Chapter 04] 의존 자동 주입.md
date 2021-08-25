# [Chapter 04] 의존 자동 주입

앞서 3장에서 스프링의 DI 설정에 대해 살펴봤다.

설정 클래스는 주입할 의존 대상을 생성자나 메서드를 이용해서 주입했다.

하지만 이렇게 의존 대상을 설정 코드에서 직접 주입하지 않고 스프링이 자동으로 의존하는 빈 객체를 주입해주는 기능도 있다.

이를 **자동 주입**이라고 한다.

스프링 부트가 나오면서 의존 자동 주입을 사용하는 추세로 바뀌었다.





## 1. `@Autowired` 애너테이션을 이용한 의존 자동 주입

자동 주입 기능을 사용하면 스프링이 알아서 의존 객체를 찾아서 주입한다.

자동 주입 기능을 사용하는 것은 매우 간단하다.

의존을 주입할 대상에 `@Autowired` 애너테이션을 붙이기만 하면 된다.



**필드 주입**

```java
public class ChangePasswordService {

    @Autowired 
    private MemberDao memberDao;   // 필드 인젝션
    ...
}
```



**세터 주입**

```java
public class MemberInfoPrinter {

    private MemberDao memberDao;
    
    @Autowired
    public void setMemberDao(MemberDao memberDao) {
        this.memberDao = memberDao;
    }
    ...
}
```



**생성자 주입**

```java
@Component
public class Example {

    private HelloService helloService;
    
    // 자동으로 HelloService를 주입해준다.
    public Example(HelloService helloService) {
        this.helloService = helloService;
    }
	
}
```

단일 생성자인 경우에는 `@Autowired` 어노테이션을 붙이지 않아도 된다.

생성자가 2개 이상인 경우에는 생성자에 어노테이션을 붙여주어야 한다.







### 주입대상에 일치하는 빈이 없는 경우

`@Autowired` 애너테이션을 적용한 대상에 일치하는 빈이 없으면 어떻게 될까?

이때는 스프링이 주입할 빈이 존재하지 않아 에러가 발생했다고 메시지를 띄워준다.





### 주입대상에 일치하는 빈이 두 개 이상인 경우

`@Autowired` 애너테이션을 적용한 대상에 일치하는 빈이 두 개 이상이면 어떻게 될까?

이때는 스프링이 주입할 빈을 한정할 수 없어서 에러가 발생했다고 메시지를 띄워준다.

자동 주입을 하려면 해당 타입을 가진 빈이 어떤 빈인지 정확하게 한정할 수 있어야 한다.







## 2. `@Qualifier` 애너테이션을 이용한 의존 객체 선택

자동 주입 가능한 빈이 두 개 이상이면 자동 주입할 빈을 지정할 수 있는 방법이 필요하다.

이때 `@Qualifier` 애너테이션을 사용한다.

`@Qualifier` 애너테이션을 사용하면 자동 주입 대상 빈을 한정할 수 있다.

























