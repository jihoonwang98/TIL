# [Chapter 03] 스프링 DI



## 1. 의존이란?

한 클래스가 다른 클래스의 메서드를 실행할 때 이를 '의존'한다고 표현한다.

즉, 의존은 변경에 의해 영향을 받는 관계를 말한다. 



예를 들어, MemberService 클래스가 MemberRepository 클래스의 findById() 메서드를 실행한다고 해보자.

이때 MemberRepository의 findById()가 findByName()으로 변경되면, 

이 메서드를 사용하는 MemberService 클래스의 소스 코드도 함께 변경되어야 한다.



이렇게 변경에 따른 영향이 전파되는 관계를 '의존'한다고 표현한다.





## 2. DI를 통한 의존 처리

DI(Dependency Injection)는 의존하는 객체를 직접 생성하는 대신 **의존 객체를 주입**받는 방식을 사용한다.







## 3. DI와 의존 객체 변경의 유연함



### 의존 객체를 직접 생성하는 방식의 문제점

의존 객체를 직접 생성하는 방식 -> 필드나 생성자에서 new 연산자를 이용해 객체를 생성하는 방법



**예제**

`MemberRepository` 클래스를 사용하는 두 개의 클래스 `MemberService`와 `ChangePasswordService`가 있다고 하자.

```java
public class MemberService {
	private MemberRepository memberRepository = new MemberRepository();
    ...
}

public class ChangePasswordService {
	private MemberRepository memberRepository = new MemberRepository();
    ...
}
```



이때, 회원 데이터를 좀 더 빠르게 조회하기 위해 캐시를 적용해야 하는 상황이 발생했다고 하자.

그래서 `MemberRepository` 클래스를 상속받은 `CachedMemberRepository` 클래스를 만들었다.

```java
public class CachedMemberRepository extends MemberRepository {
	...
}
```





그럼 이제 위의 `MemberService`, `ChangePasswordService` 클래스가

`MemberRepository` 대신 `CachedMemberRepository`를 사용하도록 변경해주어야 한다.

```java
public class MemberService {
	private MemberRepository memberRepository = new CachedMemberRepository;  // new MemberRepository();
    ...
}

public class ChangePasswordService {
	private MemberRepository memberRepository = new CachedMemberRepository;  // new MemberRepository();
    ...
}
```

그런데 만약 `MemberRepository`를 사용하는 클래스가 두 개가 아니라 수백개였다고 생각해보면 아찔하다...

이것이 바로 **의존 객체를 직접 생성하는 방식의 문제점**이다.





**DI(의존 객체를 주입받는 방식)**를 이용하면 수정할 코드를 줄일 수 있다.

예를 들어 애초에 `MemberService`와 `ChangePasswordService`를 만들때 DI 방식을 이용하면 아래와 같이 작성할 수 있다.

```java
public class MemberService {
	private MemberRepository memberRepository;
	
	public MemberService(MemberRepository memberRepository) {
		this.memberRepository = memberRepository;
	}
    ...
}

public class ChangePasswordService {
	private MemberRepository memberRepository;
	
	public MemberService(MemberRepository memberRepository) {
		this.memberRepository = memberRepository;
	}
    ...
}
```



두 클래스의 객체를 생성하는 코드는 다음과 같다.

```java
MemberRepository memberRepository = new MemberRepository();

MemberService memberService = new MemberService(memberRepository);
ChangePasswordService passwordService = new ChangePasswordService(memberRepository);
```





이렇게 작성한 상태에서 `MemberRepository` -> `CachedMemberRepository`로 요구사항이 바뀌었다고 해보자.

이 경우에는 `MemberService`와 `ChangePasswordService` 클래스를 수정할 필요 없이 

아래와 같이 두 클래스의 객체를 생성하는 코드만 수정해주면 된다.

```java
MemberRepository memberRepository = new CachedMemberRepository();   // 여기만 바뀜!!!!

MemberService memberService = new MemberService(memberRepository);
ChangePasswordService passwordService = new ChangePasswordService(memberRepository);
```



위와 같이 DI 방식을 사용하면 `MemberRepository`를 사용하는 클래스가 수백개여도 

변경할 곳은 의존 주입 대상이 되는 객체를 생성하는 코드 한 곳뿐이다.

앞서 의존 객체를 직접 생성했던 방식에 비해 변경할 코드가 한 곳으로 집중되는 것을 알 수 있다.









## 4. 스프링의 DI 설정



### 스프링을 이용한 객체 조립과 사용

스프링을 사용하려면 먼저 스프링이 어떤 객체를 생성하고, 의존을 어떻게 주입할지를 정의한 설정 정보를 작성해야 한다.

이 설정 정보는 자바 코드를 이용해서 작성할 수 있다. 

```java
@Configuration
public class AppCtx {

    @Bean
    public MemberRepository memberRepository() {
        return new MemberRepository();
    }
    
    @Bean
    public MemberService memberService() {
        return new MemberService(memberRepository());
    }
    
    @Bean
    public ChangePasswordService changePwdSvc() {
        ChangePasswordService pwdSvc = new ChangePasswordService();
        pwdSvc.setMemberRepository(memberRepository());
        return pwdSvc;
    }
}
```



이렇게 설정 클래스를 만들고 나면, 설정 클래스를 이용해서 컨테이너를 생성할 수 있다.

`AnnotationConfigApplicationContext` 클래스를 이용해서 스프링 컨테이너를 생성해보자.

```java
ApplicationContext ctx = new AnnotationConfigApplicationContext(AppCtx.class);
```



컨테이너를 생성하면 `getBean()` 메서드를 이용해서 사용할 객체를 구할 수 있다.

```java
MemberService svc = ctx.getBean("memberService", MemberService.class);
```

위 코드는 스프링 컨테이너(ctx)로부터 이름이 "memberService"인 빈 객체를 구한다.







### DI 방식 1: 생성자 방식

### DI 방식 2: Setter 메서드 방식

### DI 방식 3: 필드 주입 방식







## 5. 두 개 이상의 설정 파일 사용하기

스프링을 이용해서 애플리케이션을 개발하다보면 적게는 수십 개에서 많게는 수백여개 이상의 빈을 설정하게 된다.

설정하는 빈의 개수가 증가하면 한 개의 클래스 파일에 설정하는 것보다 영역별로 설정 파일을 나누면 관리하기 편해진다.



스프링은 한 개 이상의 설정 파일을 이용해서 컨테이너를 생성할 수 있다.

```java
@Configuration
public class AppConf1 {
	
	@Bean
	public MemberDao memberDao() { ... }
	
	@Bean
	public MemberPrinter memberPrinter() { ... }
}


@Configuration
public class AppConf2 {
    
    @Autowired
    private MemberDao memberDao;
    
    @Autowired
    private MemberPrinter memberPrinter;
    
    
    @Bean 
    public MemberRegisterService memberRegSvc() {
        return new MemberRegisterService(memberDao);
    }
    
    @Bean
    public ChangePasswordService changePwdSvc() {
        ChangePasswordService pwdSvc = new ChangePasswordService();
        pwdSvc.setMemberDao(memberDao);
        return pwdSvc;
    }
}
```



설정 클래스가 두 개 이상이어도 스프링 컨테이너를 생성하는 코드는 크게 다르지 않다.

```java
ctx = new AnnotationConfigApplicationContext(AppConf1.class, AppConf2.class);
```