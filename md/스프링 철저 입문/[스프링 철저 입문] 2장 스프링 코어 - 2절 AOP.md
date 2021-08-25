## 2.2 AOP

소프트웨어를 개발할 때 소스코드의 규모가 커지다 보면 **로깅**이나 **캐시**와 같이 비즈니스 로직과는 크게 관련 없는 처리 내용이 소스코드의 여기저기에 산재하기 쉽다. 다음은 본래 메서드의 기능과 상관없이 메서드의 시작과 끝을 로그로 기록하는 예다.



- **메서드의 시작과 끝을 로깅한 예**

```java
public class UserServiceImpl implements UserService {
    private static final Logger log = LoggerFactory.getLogger(UserServiceImpl.class);
    
    public User findOne(String username) {
        log.debug("메서드 시작: UserServiceImpl.findOne 인수 = {}", username);
        // 생략
        log.debug("메서드 종료: UserServiceImpl.findOne 반환값 = {}", user);
        return user;
    }

}
```



이런 코드가 많아지면 나중에 로그 포맷이나 로그 레벨을 변경해야 하는 경우 모든 코드를 뒤져가며 수정해야 한다. 이런 일은 **DRY(Do not Repeat Yourself)** 원칙에 어긋나고, 향후 발생할 수 있는 변경에도 취약하며, 미처 수정 작업이 완료되지 않고 남는 부분이 있다면 시스템의 부정합을 일으키는 잠재적인 원인이 될 수 있다.



이렇게 구현하고자 하는 비즈니스 로직과는 다소 거리가 있으나 여러 모듈에 걸쳐 공통적이고 반복적으로 필요로 하는 처리 내용을 **횡단 관심사(Cross-Cutting Concern)**라고 부른다. 다음은 대표적인 **횡단 관심사**다.

- **보안**
- **로깅**
- **트랜잭션 관리**
- **모니터링**
- **캐시 처리**
- **예외 처리**



또한 프로그램 안에서 횡단 관심사에 해당하는 부분을 분리해서 한 곳으로 모으는 것을 **횡단 관심사의 분리(Separation Of Cross-Cutting Concerns)**라고 하고, 이를 실현하는 방법을 **관점 지향 프로그래밍**이라 한다.



### 2.2.1 AOP의 개요

**AOP**는 **관점 지향 프로그래밍(Aspect Oriented Programming)**을 의미하는 약자로, 여러 클래스에 흩어져 있는 횡단 관심사를 중심으로 설계와 구현을 하는 프로그래밍 기법이다.



**AOP**는 **DI**와 함께 스프링 프레임워크의 중요한 기능 중 하나다. 지금까지는 **DI**를 활용해 클래스의 인스턴스를 생성하고, 인스턴스 간의 의존 관계를 맺는 처리를 애플리케이션 코드에서 분리할 수 있었다. 여기에 **AOP**를 활용하면 이 인스턴스들이 필요로 하는 공통적인 기능을 외부에서 집어넣을 수 있게 된다. 달리 말하자면 **애플리케이션 코드에서 공통적인 기능을 분리해 내는 것**이라고 할 수 있다.



#### AOP 개념

우선 **AOP**와 관련된 대표적인 용어와 이들 간의 관계를 살펴보자.



- **애스펙트(Aspect)**: **AOP**의 단위가 되는 **횡단 관심사**에 해당한다. AOP의 예로 자주 언급되는 '로그를 출력한다', '예외를 처리한다', '트랜잭션을 관리한다'와 같은 관심사가 **Aspect**다.



- **조인 포인트(Join Point)**: **횡단 관심사**가 실행될 지점이나 시점(메서드 실행이나 예외 발생 등)을 말한다. **조인 포인트**는 **AOP**를 구현한 라이브러리에 따라 사양이 다를 수 있는데 참고로 **스프링 프레임워크의 AOP에서는 메서드 단위로 조인 포인트를 잡는다**.



- **어드바이스(Advice)**: 특정 **Join Point**에서 실행되는 코드로, **횡단 관심사**를 <u>실제로 구현</u>해서 처리하는 부분이다. **Advice**에는 **Around, Before, After** 등의 여러 유형이 있는데, 뒤에서 좀 더 자세히 다루겠다.



- **포인트컷(Pointcut)**: 수많은 **Join point** 중에서 실제로 **Advice**를 적용할 곳을 선별하기 위한 **표현식(expression)**을 말한다. 일종의 **Joint point**의 그룹이라 볼 수도 있다. 스프링 AOP에서는 **Pointcut**을 정의할 때 **XML 기반 설정 방식**으로 빈 정의 파일을 만들거나, **애너테이션 기반 설정 방식**으로 소스코드에 주석 형태로 정의한다.



- **위빙(Weaving)**: 애플리케이션 코드의 적절한 지점에 **Aspect**를 적용하는 것을 말한다. **AOP** 구현 라이브러리에 따라 **Weaving**하는 시점이 다를 수 있는데, 컴파일 시점이나 클래스 로딩 시점, 실행 시점 등의 다양한 **Weaving** 시점이 있다. 참고로 **스프링 AOP는 기본적으로 실행 시점에 weaving한다.**



- **타깃(Target)**: **AOP** 처리에 의해 처리 흐름에 변화가 생긴 객체를 말한다. **Advised Object**라고도 한다.





![](https://lh5.googleusercontent.com/-fBRQ7NIWPFpWAqRnuyF8jTHZsLZal9ncb2Ul5LcQHn3wdRh7WNehwJUTjmmkA2Y_FcB4Zdf7e2DGYg-Y6VhqBDFPQeWSkIEfbJYIN2mneys7-XdXmzMyJQSJ59KRuNTGmBV9y0y)





#### 스프링 프레임워크에서 지원하는 Advice 유형

스프링 AOP에서는 아래의 표와 같이 다섯 가지 **Advice**를 활용할 수 있다.



**스프링 프레임워크에서 이용 가능한 Advice**

| Advice              | 설명                                                         |
| ------------------- | ------------------------------------------------------------ |
| **Before**          | **Join Point 전**에 실행된다. 예외가 발생하는 경우만 제외하고 항상 실행된다. |
| **After Returning** | **Join Point가 정상적으로 종료한 후**에 실행된다. 예외가 발생하면 실행되지 않는다. |
| **After Throwing**  | **Join Point에서 예외가 발생했을 때** 실행된다. 예외가 발생하지 않고 정상적으로 종료되면 실행되지 않는다. |
| **After**           | **Join Point에서 처리가 완료된 후** 실행된다. 예외 발생이나 정상 종료 여부와 상관없이 항상 실행된다. |
| **Around**          | **Join Point 전후에** 실행된다.                              |





![](https://docs.google.com/drawings/d/szFLhddqEkU2mU3BU95p60g/image?parent=e/2PACX-1vSgWs9-SkKDVDplyC96i2vj5TuZvbmLHvACG_NMQ3eEa1hFtJwCGbFZVRaHriS3HOeS8O0XIQ6gqqII&rev=149&h=169&w=601&ac=1)



![](https://docs.google.com/drawings/d/soR2BIKpvU80Y3gITdAr01A/image?parent=e/2PACX-1vSgWs9-SkKDVDplyC96i2vj5TuZvbmLHvACG_NMQ3eEa1hFtJwCGbFZVRaHriS3HOeS8O0XIQ6gqqII&rev=66&h=181&w=601&ac=1)



![](https://docs.google.com/drawings/d/sqOW1YQbJx8acS01eI8lxYQ/image?parent=e/2PACX-1vSgWs9-SkKDVDplyC96i2vj5TuZvbmLHvACG_NMQ3eEa1hFtJwCGbFZVRaHriS3HOeS8O0XIQ6gqqII&rev=53&h=177&w=601&ac=1)



![](https://docs.google.com/drawings/d/sJlGeveA3V8ZCGpGgFfoB1Q/image?parent=e/2PACX-1vSgWs9-SkKDVDplyC96i2vj5TuZvbmLHvACG_NMQ3eEa1hFtJwCGbFZVRaHriS3HOeS8O0XIQ6gqqII&rev=1&h=168&w=601&ac=1)



![](https://docs.google.com/drawings/d/s2fCyhY5Wz8vQ_rtEFvCQ8w/image?parent=e/2PACX-1vSgWs9-SkKDVDplyC96i2vj5TuZvbmLHvACG_NMQ3eEa1hFtJwCGbFZVRaHriS3HOeS8O0XIQ6gqqII&rev=19&h=168&w=601&ac=1)





### 2.2.2 스프링 AOP

스프링 프레임워크 안에는 AOP를 지원하는 모듈로 **스프링 AOP**가 포함돼 있다. 스프링 AOP에는 DI 컨테이너에서 관리하는 빈들을 **타깃(Target)**으로 **Advice**를 적용하는 기능이 있는데, **Join Point**에 **Advice**를 적용하는 방법은 **Proxy** 객체를 만들어서 대체하는 방법을 쓴다. 그래서 **Advice**가 적용된 이후, DI 컨테이너에서 빈을 꺼내보면 원래 있던 빈 인스턴스가 아니라 **Proxy** 형태로 **Advice** 기능이 덧입혀진 빈이 나온다.



![](https://docs.google.com/drawings/d/sOzk5UK5YaN0W6T-_3M1s6w/image?parent=e/2PACX-1vSgWs9-SkKDVDplyC96i2vj5TuZvbmLHvACG_NMQ3eEa1hFtJwCGbFZVRaHriS3HOeS8O0XIQ6gqqII&rev=352&h=468&w=601&ac=1)



스프링 AOP에는 실제 개별 현장에서 폭넓게 사용돼온 **AspectJ**라는 **AOP 프레임워크**가 포함돼 있다. **AspectJ**는 **Aspect**와 **Advice**를 정의하기 위한 애너테이션이나 **Pointcut** 표현 언어(Pointcut Expression Language), **Weaving** 메커니즘 등을 제공하는 역할을 한다. 참고로 **AspectJ**에서는 컴파일할 때, 클래스를 로드할 때, 실행할 때와 같이 다양한 **Weaving** 시점을 지원하지만, **스프링 AOP**는 이 가운데 **실행 시점의 Weaving**을 기본으로 지원한다. 그래서 컴파일이나 클래스 로드 시점에 **Weaving**을 하기 위한 추가적인 설정을 따로 하지 않아도 된다.



한편, **스프링 AOP**의 기능을 활용하려면 **메이븐**의 **pom.xml**에 다음과 같이 의존 관계를 정의해야 한다.



- **pom.xml**

```xml
<dependency>
	<groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
</dependency>
<dependency>
	<groupId>org.springframework</groupId>
    <artifactId>spring-aop</artifactId>
</dependency>
<dependency>
	<groupId>org.aspectj</groupId>
    <artifactId>aspectjweaver</artifactId>
</dependency>
```



이제 간단하게 **Aspect**를 구현해보자.



- **Aspect 구현**

```java
import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.annotation.Before;
import org.aspectj.lang.annotation.Aspect;
import org.springframework.stereotype.Component;

@Aspect 
@Component
public class MethodStartLoggingAspect {
    @Before("execution(* *..*ServiceImpl.*(..))")  
    // 이름이 ServiceImpl로 끝나는 클래스의 모든 public 메서드를 대상으로 한다.
    public void startLog(JoinPoint jp) {
        System.out.println("메서드 시작: " + jp.getSignature());
    }
    /* JoinPoint 객체(org.aspectj.lang.JoinPoint)를 통해 
       실행 중인 메서드의 정보(메서드 이름, 반환값이나 타입, 인수 등)를 확인할 수 있다. */
}
```



이렇게 만든 **Aspect**를 동작시키려면 **AOP**를 활성화해야 하며, **자바 기반 설정 방식**에서는 **@EnableAspectJAutoProxy** 애너테이션을 사용한다.



- **스프링 AOP 활성화(자바 기반 설정 방식)**

```java
@Configuration
@ComponentScan("com.example")
@EnableAspectJAutoProxy
public class AppConfig {
    // 생략
}
```



**XML 기반 설정 방식**에서는 **AOP** 관련 네임스페이스와 스키마 정보를 추가한 다음, **\<aop:aspectj-autoproxy>** 요소를 설정하면 된다.



- **스프링 AOP를 활성화하는 XML 파일 정의 예**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans
	xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    	http://www.springframework.org/schema/beans/spring-beans.xsd
    	http://www.springframework.org/schema/context
    	http://www.springframework.org/schema/context/spring-context-4.3.xsd
    	http://www.springframework.org/schema/aop
    	http://www.springframework.org/schema/aop/spring-aop.xsd">
    
    <context:component-scan base-package="com.example" />
    <aop:aspectj-autoproxy />
    <!-- 생략 -->
</beans>
```



이렇게 설정한 상태에서 **UserService**의 **findOne** 메서드를 실행한다고 가정해 보자.



- **AOP가 적용된 빈의 메서드 실행**

```java
UserService userService = context.getBean(UserService.class);
userService.findOne("spring");
```



**AOP** 기능이 정상적으로 동작했다면 다음과 같은 결과를 확인할 수 있을 것이다.



- **AOP가 적용된 빈의 메서드 실행 결과**

```
메서드 시작: User com.example.demo.UserServiceImpl.findOne(String)
```





### 2.2.3 자바 기반 설정 방식에서의 Advice 정의

앞서 스프링 AOP에서 활용 가능한 **Advice**로 다섯 가지 종류가 있다고 했다. 이제 각각의 구현 방식을 예를 들어 살펴보자.



#### Before

이미 앞서 예를 들었지만 **Advice** 기능을 하는 메서드에 **@Before** 애너테이션(org.aspectj.lang.annotation.Before)을 붙인 다음, **Pointcut** 표현식을 추가하면 된다. 이때 사용되는 **Pointcut** 표현식에 대해서는 뒤에서 자세히 다루겠다.



**@Before** 애너테이션이 붙은 메서드는 **JoinPoint**를 매개변수로 선언하고 있는데, 메서드가 호출될 때 전달되는 인수를 통해 실행 중인 메서드의 정보를 구할 수 있다.



- **@Before 애너테이션을 활용한 Advice 정의**

```java
import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.annotation.Before;
import org.aspectj.lang.annotation.Aspect;
import org.springframework.stereotype.Component;

@Aspect
@Component
public class MethodStartLoggingAspect {
    @Before("execution(* *..*ServiceImpl.*(..))")
    public void startLog(JoinPoint jp) {
        System.out.println("메서드 시작: " + jp.getSignature());
    }
}
```





#### After Returning

**After Returning** Advice 작성 방법은 **Before** Advice와 거의 같다. 실행되는 시점은 대상 메서드가 정상적으로 종료한 후다.



- **@AfterReturning 애너테이션을 활용한 Advice 정의**

```java
import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.annotation.AfterReturning;
import org.aspectj.lang.annotation.Aspect;
import org.springframework.stereotype.Component;

@Aspect
@Component
public class MethodNormalEndLoggingAspect {
    @AfterReturning("execution(* *..*ServiceImpl.*(..))")
    public void startLog(JoinPoint jp) {
        System.out.println("메서드 정상 종료: " + jp.getSignature());
    }
}
```



**After Returning** Advice를 사용할 때는 정상적으로 종료한 메서드의 반환값을 구할 수 있다. **@AfterReturning** 애너테이션의 **returning **속성에 반환값을 받을 매개변수의 이름을 지정하면 된다.



- **@AfterReturning 애너테이션을 활용한 Advice 정의(반환값 정보 받기)**

```java
@Aspect
@Component
public class MethodNormalEndLoggingAspect {
    @AfterReturning(value = "execution(* *..*ServiceImpl.*(..))", returning = "user")
    public void startLog(JoinPoint jp, User user) {
        System.out.println("메서드 정상 종료: " + jp.getSignature() + "반환값=" + user);
    }
}
```





#### After Throwing

**After Throwing** Advice는 **After Returning** Advice와 반대로 예외가 발생해서 비정상적으로 종료될 때 실행된다. 또한 이때 발생한 예외가 무엇인지 알고 싶다면 **@AfterThrowing** 애너테이션의 **throwing** 속성에 예외를 받을 매개변수의 이름을 지정하면 된다.



- **@AfterThrowing 애너테이션을 활용한 Advice 정의**

```java
import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.annotation.AfterThrowing;
import org.aspectj.lang.annotation.Aspect;
import org.springframework.stereotype.Component;

@Aspect
@Component
public class MethodExceptionEndLoggingAspect {
    @AfterThrowing(value = "execution(* *..*ServiceImpl.*(..))", throwing = "e")
    public void endLog(JoinPoint jp, RuntimeException e) {
        System.out.println("메서드 비정상 종료: " + jp.getSignature());
        e.printStackTrace();
    }
}
```



**After Throwing** Advice에서는 예외를 다시 던져 **전파(propagation)**하는 것이 가능하다. 다음과 같이 특정 예외가 발생하면 일률적으로 다른 예외 형태로 변환해줘야 할 때 상당히 유용하다.



- **@AfterThrowing 애너테이션을 활용한 Advice 정의(예외 변환)**

```java
@Aspect
@Component
public class MethodExceptionPropagationAspect {
    @AfterThrowing(value = "execution(* *..*ServiceImpl.*(..))", throwing = "e")
    public void endLog(JoinPoint jp, DataAccessException e) {
        throw new ApplicationException(e);
    }
}
```



참고로 **After Throwing** Advice는 예외가 외부로 던져지는 것을 막지는 못하기 때문에 예외가 발생했을 때 꼭 필요한 동작을 수행하게 만든 다음, 예외는 Advice가 없을 때처럼 외부로 던지도록 만들어야 한다. 만약 발생한 예외가 밖으로 던져지는 것을 굳이 막아야 하는 상황이라면 **AfterThrowing** 대신 **Around** Advice를 사용하면 된다.





#### After

**After** Advice는 **After Returning** Advice나 **After Throwing** Advice와 달리 메서드가 정상 종료 여부나 예외 발생 여부와 상관없이 무조건 실행된다. 마치 **try-catch** 구문의 **finally**와 같은 역할을 한다.



- **@After 애너테이션을 활용한 Advice 정의**

```java
import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.annotation.After;
import org.aspectj.lang.annotation.Aspect;
import org.springframework.stereotype.Component;

@Aspect
@Component
public class MethodEndLoggingAspect {
    @After("execution(* *..*ServiceImpl.*(..))")
    public void endLog(JoinPoint jp) {
        System.out.println("메서드 종료: " + jp.getSignature());
    }
}
```







#### Around

**Around** Advice는 가장 강력한 Advice이며, 메서드의 실행 전과 후의 처리는 물론, Pointcut이 적용된 대상 메서드 자체도 실행할 수 있다.



- **@Around 애너테이션을 활용한 Advice 정의**

```java
import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.springframework.stereotype.Component;

@Aspect
@Component
public class MethodLoggingAspect {
    @Around("execution(* *..*ServiceImpl.*(..))")
    public Object log(ProceedingJoinPoint jp) throws Throwable {
        System.out.println("메서드 시작: " + jp.getSignature());
        try {
            // 대상 메서드 실행
            Object result = jp.proceed();
            System.out.println("메서드 정상 종료: " + jp.getSignature() + " 반환값=" + result);
            return result;
        } catch(Exception e) {
            System.out.println("메서드 비정상 종료: " + jp.getSignature());
            e.printStackTrace();
            throw e;
        }
    }
}
```





### 2.2.4 XML 기반 설정 방식에서의 Advice 정의

지금까지는 Advice를 정의할 때 **자바 기반 설정 방식**을 사용했다. 다음은 **XML 기반 설정 방식**으로 Advice를 정의하는 방법을 알아보자. 우선 애너테이션을 사용하지 않은 **Aspect** 클래스를 만든 다음, XML 파일에서 AOP를 설정하면 된다.



- **Aspect 구현**

```java
import org.aspectj.lang.Joinpoint;

public class MethodStartLoggingAspect {
    public void startLog(JoinPoint jp) {
        System.out.println("메서드 시작: " + jp.getSignature());
    }
}
```



이렇게 **Aspect**가 만들어졌다면 이번엔 **MethodStartLoggingAspect** 클래스의 **startLog** 메서드를 **Before** Advice로 정의해보자.



- **Before Advice 설정(XML 기반 설정 방식)**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans
	xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    	http://www.springframework.org/schema/beans/spring-beans.xsd
    	http://www.springframework.org/schema/aop
    	http://www.springframework.org/schema/aop/spring-aop.xsd">
    
    <!-- 생략 -->
    <aop:config>
    	<aop:aspect ref="loggingAspect">
        	<aop:before pointcut="execution(* *..*ServiceImpl.*(..))" method="startLog" />
        </aop:aspect>
    </aop:config>
    
    <bean id="loggingAspect" class="com.example.aspect.MethodStartLoggingAspect" />
</beans>
```



- **\<aop:config>** 요소 내에 여러 개의 Aspect를 정의할 수 있다.
- Aspect를 정의할 때는 **\<aop:aspect>**를 사용하고 Aspect의 **빈 ID를 ref 속성**에 지정한다. 이 과정은 자바 기반 설정 방식에서 **@Aspect** 애너테이션을 사용하는 것과 같은 역할을 한다.

- **\<aop:before>** 요소에서 **Before** Advice를 정의한다. **Pointcut** 속성에 Pointcut 표현식을 설정하고 **method** 속성에 적용할 대상 메서드의 이름을 기술한다.



**Before** Advice 외의 **After Returning, After Throwing, After, Around** Advice도 모두 같은 방법으로 정의할 수 있다.





### 2.2.5 Pointcut 표현식

지금까지 살펴본 코드에서는 Pointcut을 선택하기 위해 **'execution(* \*..* ServiceImpl.*(..))'**과 같은 표현을 사용해 왔다. 이처럼 표현식을 이용한 **JoinPoint** 선택 기능은 **AspectJ**가 제공하며, 스프링 AOP는 AspectJ가 제공하는 Pointcut 표현식을 상당수 지원한다.



**Pointcut**은 일치시킬 패턴에 따라 **지시자(designator)**의 형식이 달라지는데, 지금부터 대표적인 표현식 패턴을 하나씩 살펴보자.





#### 메서드명으로 Join Point 선택

메서드명의 패턴으로 **JoinPoint**를 선택하는 방식으로, 지금까지 봐온 **execution** 지시자를 사용한다. 특히 **execution** 지시자는 다른 지시자에 비해 상대적으로 많이 활용되는 기본적인 지시자다.



다음은 **execution 지시자**를 활용해 **Pointcut**을 표현한 것이다.



![](https://docs.google.com/drawings/d/sfbXZyClbLwZJuMzgMrd71Q/image?parent=e/2PACX-1vSgWs9-SkKDVDplyC96i2vj5TuZvbmLHvACG_NMQ3eEa1hFtJwCGbFZVRaHriS3HOeS8O0XIQ6gqqII&rev=220&h=182&w=601&ac=1)





이제 **execution 지시자**를 활용하는 예를 살펴보자.

- **execution(* com.example.user.UserService.*(..))**

  com.example.user.UserService 클래스에서 임의의 메서드를 대상으로 한다.



- **execution(* com.example.user.UserService.find*(..))**

  com.example.user.UserService 클래스에서 이름이 find로 시작하는 메서드를 대상으로 한다.



- **execution(String com.example.user.UserService.*(..))**

  com.example.user.UserService 클래스에서 반환값의 타입이 String인 메서드를 대상으로 한다.



- **execution(* com.example.user.UserService.*(String, ..))**

  com.example.user.UserService 클래스에서 첫 번째 매개변수의 타입이 String인 메서드를 대상으로 한다.



예에서 알 수 있듯이 **Pointcut 표현식**에는 **와일드카드(wildcard)**를 사용할 수 있다. 사용할 수 있는 종류에는 *, .., +의 세 가지가 있으며, 각각의 의미는 다음과 같다.



**Pointcut 표현식에 사용되는 Wildcard**

| Wildcard | 설명                                                         |
| -------- | ------------------------------------------------------------ |
| *****    | 기본적으로 임의의 문자열을 의미한다. 패키지를 표현할 때는 임의의 패키지 1개 계층을 의미한다. 메서드의 매개변수를 표현할 때는 임의의 인수 1개를 의미한다. |
| **..**   | 패키지를 표현할 때는 임의의 패키지 0개 이상 계층을 의미한다. 메서드의 매개변수를 표현할 때는 임의의 인수 0개 이상을 의미한다. |
| **+**    | 클래스명 뒤에 붙여 쓰며, 해당 클래스와 해당 클래스의 서브클래스, 혹은 구현 클래스 모두를 의미한다. |





이번에는 **Wildcard**의 의미에 유념하면서 **execution 지시자**의 사용 예를 조금 더 살펴보자.

- **execution(* com.example.service.\*.*(..))**

  임의의 클래스에 속한 임의의 메서드를 대상으로 한다. 단, 임의의 클래스는 service 패키지에 속한다.



- **execution(* com.example.service..\*.*(..))**

  임의의 클래스에 속한 임의의 메서드를 대상으로 한다. 단, 임의의 클래스는 service 패키지나 그 서브 패키지에 속한다.



- **execution(* com.example.\*.user.\*.*(..))**

  임의의 클래스에 속한 임의의 메서드를 대상으로 한다. 단, 임의의 클래스는 user 패키지에 속하고 com.example과 user 사이에는 임의의 패키지가 한 단계 더 있다.



- **execution(* com.example.user.UserService.*(\*))**

  UserService 클래스의 메서드 중 매개변수의 개수가 하나인 메서드를 대상으로 한다. 단, UserService는 com.example.user 패키지에 속한다.







#### 타입으로 Join Point 선택

타입 정보를 활용해 **Join Point**를 선택할 수도 있는데 이때는 **within 지시자**를 사용한다. **within 지시자**는 클래스명의 패턴만 사용하기 때문에 **execution 지시자**에 비해 상대적으로 표현식이 간결하다.

- **within(com.example.service..*)**

  임의의 클래스에 속한 임의의 메서드를 대상으로 한다. 단, 임의의 클래스는 service 패키지나 이 패키지의 서브 패키지에 속한다.



- **within(com.example.user.UserServiceImpl)**

  UserServiceImpl 클래스의 메서드를 대상으로 한다. 단, UserServiceImpl 클래스는 com.example.user 패키지에 속한다.



- **within(com.example.password.PasswordEncoder+)**

  PasswordEncoder 인터페이스를 구현한 클래스의 메서드를 대상으로 한다. 단, PasswordEncoder 인터페이스는 com.example.user 패키지에 속한다.



 

#### 그 밖의 기타 방법으로 Join Point 선택

스프링 AOP에는 그 밖에도 다양한 **지시자**가 준비돼 있다. 그 중에서 몇 가지 유용한 사용 형태를 소개하자면 다음과 같다.

- **bean(*Service)**

  DI 컨테이너에 관리되는 빈 가운데 이름이 'Service'로 끝나는 빈의 메서드를 대상으로 한다.



- **@annotation(com.example.annotation.TraceLog)**

  @TraceLog 애너테이션(com.example.annotation.TraceLog)이 붙은 메서드를 대상으로 한다.



- **@within(com.example.annotation.TraceLog)**

  @TraceLog 애너테이션(com.example.annotation.TraceLog)이 붙은 클래스의 메서드를 대상으로 한다.



이와 같이 특정 기능을 구현한 후, 그와 관련된 애너테이션을 만들어 둔 것이 있다면 **@annotation 지시자**나 **@within 지시자**를 활용하는 편이 표현이 간결해서 사용하기 편리할 것이다.





#### 네임드 Pointcut 활용

**Pointcut**에 이름을 붙여두면 나중에 그 이름으로 **pointcut**을 재사용할 수 있다. 이렇게 이름이 붙여진 **pointcut**을 **네임드 포인트컷(Named Pointcut)**이라 한다. **Named Pointcut**은 **@Pointcut** 애너테이션(org.aspectj.lang.annotation.Pointcut)으로 정의할 수 있는데, 이 애너테이션이 붙은 메서드 이름이 **pointcut**의 이름이 된다. 참고로 이때 메서드의 반환값은 **void**로 한다.



- **Named Pointcut 정의**

```java
@Component
@Aspect
public class NamedPointCuts {
    @Pointcut("within(com.example.web..*)")
    public void inWebLayer() {}
    
    @Pointcut("within(com.example.domain..*)")
    public void inDomainLayer() {}
    
    @Pointcut("execution(public * *(..))")
    public void anyPublicOperation() {}
}
```



이렇게 만들어진 **Named Pointcut**은 나중에 **Advice**의 **Pointcut**을 지정할 때 활용할 수 있다.





- **Named Pointcut 활용**

```java
@Aspect
@Component
public class MethodLoggingAspect {
	@Around("inDomainLayer()")
    public Object log(ProceedingJoinPoint jp) throws Throwable {
        // 생략
    }
}
```



한편 다음과 같이 **&&, ||, !**과 같은 **논리곱, 논리합, 논리부정 연산자**를 활용해 다양한 형태로 조합할 수도 있다.

```java
@Around("inDomainLayer() || inWebLayer()")
```





#### Advice 대상 객체와 인수 정보 가져오기

**JoinPoint** 타입의 인수(org.aspectj.lang.JoinPoint)를 활용하면 **Advice** 대상 객체나 메서드를 호출할 때 전달된 인수의 정보를 가져올 수 있다. 

**getTarget** 메서드를 이용하면 **Proxy가 입혀지기 전의 원본 대상 객체 정보**를, 
**getThis** 메서드를 이용하면 **Proxy**를, 
**getArgs** 메서드를 이용하면 **인수 정보**를 가져올 수 있다.



- **JoinPoint 객체를 통해 대상 객체와 인수 정보 가져오기**

```java
@Around("execution(* *..*ServiceImpl.*(..))")
public Object log(JoinPoint jp) throws Throwable {
    // 프락시가 입혀지기 전의 원본 대상 객체를 가져온다.
    Object targetObject = jp.getTarget();
    // 프락시를 가져온다.
    Object thisObject = jp.getThis();
    // 인수를 가져온다.
    Object[] args = jp.getArgs();
    // 생략
}
```



단, **JoinPoint** 인터페이스의 메서드는 반환값이 **Object** 타입이기 때문에 실제로 사용하기 전에는 형변환해야 한다. 그래서 이 과정에서 타입이 호환되지 않으면 **ClassCastException**이 발생할 수 있다. 이럴 때는 **Pointcut 지시자**인 **target**이나 **this, args** 등을 활용해 대상 객체나 인수를 **Advice** 메서드에 파라미터로 바로 바인딩하면 된다.



- **target, args 지시자를 활용해 대상 객체와 인수 정보 가져오기**

```java
@Around("execution(* com.example.CalcService.*(com.example.CalcInput)) && target(service) && args(input)")
public Object log(CalcService service, CalcInput input) throws Throwable {
    // 생략
}
```



이 방법을 사용하면 **getTarget** 메서드나 **getArgs** 메서드의 결과를 형변환할 필요가 없어지고 타입이 맞지 않는 경우는 자연스럽게 **Advice** 대상에서 제외된다. 결국 타입 불일치로 인한 오동작을 미연에 방지할 수 있는데 이런 상황을 '**타입 안전(type safe)하다**'라고 말한다. 상당히 유용한 기법이니 필요한 곳에 적절히 활용하자.





### 2.2.6 스프링 프로젝트에서 활용되는 AOP 기능

**AOP**는 스프링 프레임워크의 다양한 기능에서 활용되고 있다. 그 중 대표적인 사례를 몇 가지 소개한다.



#### 트랜잭션 관리

**트랜잭션 관리**가 필요한 메서드에 **@Transactional** 애너테이션(org.springframework.transaction.annotation.Transactional)을 지정하면 복잡한 트랜잭션 관리를 스프링 프레임워크가 대신한다. 스프링 프레임워크는 해당 메서드가 정상적으로 종료한 것이 확인되면 트랜잭션을 **커밋(commit)**하고, 처리가 실패해서 예외(exception)가 발생한 것을 감지하면 트랜잭션을 **롤백(rollback)**한다.



```java
@Transactional
public Reservation reserve(Reservation reservation) {
	// 예약 처리
}
```





#### 인가

**스프링 시큐리티**에서 제공하는 **인가** 기능을 **AOP** 형태로 적용할 수 있다. 예를 들어, 권한 제어가 필요한 메서드에 **@PreAuthorize** 애너테이션(org.springframework.security.access.prepost.PreAuthorize)을 지정하면 해당 메서드가 호출되기 전에 특정 인가 조건을 만족하는지 확인할 수 있다. 다음 예는 사용자를 생성하는 메서드를 실행하기 전에 이 메서드를 호출하는 사용자가 관리자 역할을 가졌는지 확인하는 경우다.



```java
@PreAuthorize("hasRole('ADMIN')")
public User create(User user) {
	// 사용자 등록 처리(ADMIN 역할을 가진 사용자만 실행할 수 있다.)
}
```





#### 캐싱

이 책에서는 자세히 다루지는 않지만 스프링 프레임워크에는 간단한 **캐싱** 기능도 준비돼 있다. 캐싱을 활성화하고 메서드에 **@Cacheable** 애너테이션(org.springframework.cache.annotation.Cacheable)을 지정하면 메서드의 매개변수 등을 키(key)로 사용해 메서드의 실행 결과를 **캐시**로 관리할 수 있다. 즉, 캐시에 키가 등록돼 있지 않다면 메서드를 실행하고, 반환값을 되돌려주기 전에 반환값과 키를 캐시에 함께 등록한다. 이후 같은 메서드가 호출됐는데, 캐시에 같은 키가 이미 등록돼 있다면 메서드를 실행하는 대신 앞서 캐시에 등록된 반환값을 돌려준다. 이 같은 캐싱 과정은 개발자가 직접 구현하는 대신 **AOP**가 내부적으로 처리해준다. 다음 예는 메서드 파라미터인 **email** 정보를 키로 사용하고 **User **객체를 캐싱한 경우다.



```java
@Cacheable("user")
public User findOne(String email) {
	// 사용자 취득
}
```





#### 비동기 처리

**AOP**는 **비동기 처리**에도 활용할 수 있다. **비동기 처리**를 하고 싶은 메서드에 **@Async** 애너테이션(org.springframework.scheduling.annotation.Async)을 붙여주고, 반환값으로 **CompletableFuture** 타입(java.util.concurrent.CompletableFuture)의 값이나 **DeferredResult** 타입(org.springframework.web.context.request.async.DeferredResult)의 값을 반환하게 만들면 해당 메서드는 **AOP** 방식으로 별도의 스레드에서 실행될 수 있다. 스레드 관리는 스프링 프레임워크가 처리해준다. 애플리케이션 개발자는 단지 비동기가 필요한 메서드에 애너테이션을 붙이고 반환값의 타입만 맞추면 된다.



```java
@Async
public CompletableFuture<Result> calc() {
    Result result = doSomething(); // 오랜 시간이 소요되는 작업
    return CompletableFuture.completedFuture(result);
}
```







#### 재처리

스프링 **Retry**라는 프로젝트를 활용하면 **재처리(retry)**를 **AOP**를 구현할 수 있다. 메서드에 **@Retryable** 애너테이션(org.springframework.retry.annotation.Retryable)을 붙여두면 해당 메서드가 정상적으로 처리되지 않은 경우, 원하는 조건을 만족할 때까지 재처리하게 된다. 다음 예에서는 메서드가 정상적으로 종료하지 않은 경우, 최대 3번까지 **callWebApi** 메서드를 반복하도록 설정돼 있다. 신뢰성을 보장하기 어려운 외부 시스템과 연계해야 할 때 상당히 유용하게 활용할 수 있다.



```java
@Retryable(maxAttempts = 3)
public String callWebApi() {
    // 웹 API 호출
}
```







## 2.3 데이터 바인딩과 형 변환

자바 객체의 **프로퍼티**에 외부에서 입력된 값을 설정하는 과정을 **데이터 바인딩(Data Binding)**이라 한다. 앞서 DI 컨테이너에서 관리되면서 애플리케이션을 구성하는 컴포넌트 역할의 자바 객체를 '**빈(Bean)**'이라고 불렀다. 이제부터는 **Data Binding**이나 **프로퍼티** 관점에서 다뤄지는 객체를 '**자바빈즈(JavaBeans)**'라고 좀 더 구체적으로 언급하겠다.



**프로퍼티**의 입력값으로 사용되는 값은 웹 애플리케이션의 **요청 파라미터**, **프로퍼티 파일의 설정 값**, **XML 기반 설정에서 빈을 정의할 때 지정한 프로퍼티 값**과 같은 다양한 형태가 있는데, 그 중에서도 가장 대표적인 것은 웹 애플리케이션에서 사용하는 **요청 파라미터**일 것이다.



먼저 **Data Binding**의 기능을 이해하기 위해 이 기능을 사용하지 않았을 때 **JavaBeans**에 **요청 파라미터** 값을 채워넣는 과정을 살펴보자.



- **요청 파라미터 값을 담기 위한 JavaBeans**

```java
public class EmployeeForm {
	private String name;
	private Integer joinedYear;
    // 생략
}
```



가장 오래됐지만 널리 사용돼온 방법으로는 **HttpServletRequest** 인터페이스의 **getParameter** 메서드를 사용해 **요청 파라미터**의 값을 받아낸 다음, 그 값을 **JavaBeans**의 설정자 메서드를 통해 **프로퍼티**로 지정하는 방법이 있다. **요청 파라미터**에 대응하는 설정 값은 기본적으로 **String** 타입으로 취급되기 때문에 만약 **String** 이외의 타입으로 다뤄져야 하는 **프로퍼티**라면 이 과정에서 형 변환을 해야 한다.



- **설정자 메서드를 통한 프로퍼티 설정**

```java
EmployeeForm form = new EmployeeForm();
form.setName(request.getParameter("name"));
form.setJoinedYear(Integer.valueOf(request.getParameter("joinedYear"))); // 형변환
```



위의 코드를 보면 큰 문제가 없어보인다. 하지만 **프로퍼티**의 개수가 늘어나면 그에 비례해서 **바인딩**하는 코드도 늘어난다. 또한 **프로퍼티**가 **String** 타입이 아니라면 형 변환도 직접 해야 한다. 이런 코드가 늘어나면 개발 효율을 떨어뜨릴 뿐만 아니라 복사해서 붙여넣기를 하는 과정에서 미처 수정하지 않았거나 코드를 누락할 위험도 있다. 그 밖에도 형 변환 과정에서 **null** 값이나 공백 문자가 들어오거나, 입력값 검증을 하지 않아 오동작을 일으킬 위험도 있다.



이럴 때는 **String** 값에 대한 **Data Binding**과 형 변환 기능을 활용하면 잠재된 문제 상황을 손쉽게 피할 수 있다. 과연 어떻게 사용하면 되는지 한번 살펴보자.



### 2.3.1 String 값에 대한 Data Binding

앞서 살펴본 예와 같이 **HTTP 요청 파라미터** 값을 **JavaBeans**의 **프로퍼티**로 설정하는 과정을 **Data Binding** 기능을 활용해 대신해보자. Data를 Binding하는 기능은 **DataBinder** 클래스(org.springframework.validation.DataBinder)가 제공하는데, **HTTP**를 사용하는 환경에서는 Servlet API에 맞게 커스터마이징된 **ServletRequestDataBinder** 클래스(org.springframework.web.bind.ServletRequestDataBinder)를 사용하면 된다.



- **ServletRequestDataBinder 클래스 활용**

```java
EmployeeForm form = new EmployeeForm();
ServletRequestDataBinder dataBinder = new ServletRequestDataBinder(form);
dataBinder.bind(request);
```



이렇게 **Data Binding** 기능을 활용하면 **JavaBeans**의 **프로퍼티** 개수가 100개가 넘어도 단 3줄로 표현할 수 있다. 심지어 **스프링 MVC의 Data Binding**기능을 활용하면 이 3줄마저도 필요없어진다.





### 2.3.2 스프링의 형 변환

**Data Binding**을 할 때는 **JavaBeans**의 **프로퍼티** 타입에 맞게 입력된 문자열 값을 형 변환해야 한다. 스프링 프레임워크는 형 변환을 하기 위해 다음과 같은 세 가지 방식을 제공한다.



#### 빈을 조작하거나 wrapping

스프링 프레임워크 초창기부터 제공됐던 기법으로서 빈의 **프로퍼티**를 읽고 쓰기 위해 원래 있던 **빈을 조작하고 wrapping**하는 방법을 제공한다. 단 빈을 wrapping하는 **BeanWrapper** 인터페이스(org.springframework.beans.BeanWrapper)는 개발자가 직접 다룰 일은 적고, 스프링 프레임워크 내부에서 **DataBinder** 클래스(org.springframework.validation.DataBinder)나 **BeanFactory** 인터페이스(org.springframework.beans.factory.BeanFactory)의 구현체가 활용한다. 개발자 입장에서는 **BeanWrapper** 인터페이스 대신 **PropertyEditor** 인터페이스(java.beans.PropertyEditor)의 구현 클래스를 활용해 형 변환을 하면 된다.



#### 형 변환

스프링 프레임워크 3.0.0 버전부터 지원되는 기법으로 **Converter** 인터페이스(org.springframework.core.convert.Converter) 등의 구현 클래스를 활용해 형 변환을 한다. **PropertyEditor**는 변환하기 전의 값이 **String** 타입으로 제한되는 데 반해, **Converter**는 **String** 외의 타입에 대해서도 형 변환할 수 있다.



#### 필드 값의 표현 형식 포매팅

스프링 프레임워크 3.0.0 버전부터 지원되는 기법으로 **Formatter** 인터페이스(org.springframework.format.Formatter) 등의 구현 클래스를 활용해 형 변환한다. **Formatter**는 **String**과 임의의 클래스를 상호 변환하기 위한 인터페이스로서, 형 변환할 때 **로캘(Local)**을 고려한 국제화 기능도 제공한다. 주로 숫자나 날짜와 같이 로캘에 따라 다른 포맷 형태를 가진 클래스와 형 변환이 필요할 때 사용한다.





### 2.3.3. PropertyEditor 활용

스프링 프레임워크는 다양한 형태의 **PropertyEditor** 구현 클래스를 제공한다. **PropertyEditor**는 **JavaBeans**의 **프로퍼티**를 설정할 때 **primitive type**이나 primitive type의 **wrapper type**은 물론 **java.nio.charset.Charset**이나 **java.net.URL**과 같은 다양한 타입도 무리없이 설정되도록 형 변환을 지원하는 역할을 한다. 예를 들어, 스프링 프레임워크가 제공하는 **PropertyEditor** 중에는 **Boolean** 타입을 처리할 때 '**true**'와 '**false**' 형태와 같은 문자열을 다룰 수 있을 뿐만 아니라 '**yes**'와 '**no**' 같은 형태, 그 밖에 '**on**'과 '**off**', '**1**'과 '**0**' 같은 형태도 **boolean**으로 변환할 수 있다.



다음은 프로퍼티의 기본 설정 값을 '**no**'(**false**)로 설정해 놓고, 프로퍼티 파일을 통해 재정의할 수 있게 만든 예다.



- **프로퍼티 파일로 기본 설정 값 재정의**

```properties
application.healthCheck = yes
```



- **빈에서 프로퍼티의 기본값 설정**

```java
@Component
public class ApplicationProperties {
    // 생략
    @Value("${application.healthCheck:no}")
    private boolean healthCheckEnabled; // true가 설정됨
    // 생략
}
```



만약 **PropertyEditor**가 지원하는 타입 외에도 다른 타입을 **프로퍼티**로 설정하고 싶다면 **PropertyEditor**를 사용자가 직접 개발해서 추가할 수 있다. 지면 관계상 구체적인 구현 방법은 이 책에서 다루지 않는다.





### 2.3.4 ConversionService 활용

**형 변환(Type Conversion)**을 하거나 필드 값의 표현 형식을 **포매팅**하는 기능은 **ConversionService** 인터페이스(org.springframework.core.convert.ConversionService)를 통해 제공된다. **ConversionService** 인터페이스의 구현 클래스에는 다양한 형태가 있지만 그 중에서도 **DefaultFormatingConversinoService** 클래스(org.springframework.format.support.DefaultFormattingConversionService)를 사용하는 것이 일반적이다. 스프링 프레임워크는 **PropertyEditor** 만큼이나 다양한 형태의 형 변환 구현 클래스와 필드 값의 표현 형식을 **포매팅**하는 클래스를 제공한다.

예를 들면, **Joda-Time**을 지원하는 클래스는 물론이고, **JSR 310** 사양을 위한 **Date and Time API**(java.time 패키지) 그리고 **JSR 354** 사양을 위한 **Money and Currency API**(javax.money 패키지) 등 다양한 클래스들을 지원한다.



이 같은 **ConversionService**를 이용하려면 먼저 DI 컨테이너에 등록해야 한다. 중요한 점은 빈의 ID가 '**conversionService**'가 되도록 정의해야 한다는 것인데, 이렇게 해둬야 나중에 **Conversion Service**를 활용할 컴포넌트가 빈 이름을 틀리지 않고 자동으로 주입받을 수 있다.



- **자바 기반 설정 방식으로 ConversionService 설정**

```java
@Bean
public ConversionService conversionService() {
	return new DefaultFormattingConversionService();
}
```



- **XML 기반 설정 방식으로 ConversionService 설정**

```xml
<bean id="conversionService" class="org.springframework.format.support.FormattingConversionServiceFactoryBean" />
```



다음 예는 **dateOfServiceStarting** 프로퍼티에 대해 기본값을 따로 설정하지 않고, 프로퍼티 파일에서 설정한 값을 읽도록 만들어져 있다. 참고로 로캘이 한국으로 설정된 경우 기본적으로 사용되는 날짜 포맷은 'uuuu-MM-dd'다.



- **프로퍼티 파일로 기본 설정 값 재정의**

```properties
application.dateOfServiceStarting=2017-05-10
```



- **빈에서 프로퍼티의 기본값 설정**

```java
@Component
public class ApplicationProperties {
    // 생략
    @Value("${application.dateOfServiceStarting:}")
    private java.time.LocalDate dateOfServiceStarting; // 2017년 5월 10일 날짜로 설정됨
    // 생략
}
```





### 2.3.5 포매팅용 애너테이션 활용

**DefaultFormattingConversionService**를 활용하면 형 변환이 필요한 곳에 다음과 같은 애너테이션을 붙인 다음, 형 변환시 적용할 포맷 형식을 정할 수 있다.

- **@DateTimeFormat 애너테이션**(org.springframework.format.annotation.DateTimeFormat)
- **@NumberFormat 애너테이션**(org.springframework.format.annotation.NumberFormat)



개발을 하다 보면 애플리케이션 전반에 걸쳐 일관된 포맷 형식을 사용하다가 특정 부분에서만 다른 형식의 포맷을 써야 하는 경우가 있다. 이럴 때는 다음과 같이 국소적으로 포맷 패턴을 명시하면 된다.



- **포맷을 명시적으로 지정**

```java
@Component
public class ApplicationProperties {
    // 생략
    @Value("${application.dateOfServiceStarting:}")
    @DateTimeFormat(pattern = "yyyy/MM/dd") // 포맷을 명시
    private java.time.LocalDate dateOfServiceStarting;
    // 생략
}
```





### 2.3.6 형 변환 방식의 커스터마이징

스프링 프레임워크에서는 다양한 형 변환 기능을 제공하지만 여전히 부족하다고 생각된다면 직접 변환 기능을 만들어 쓸 수도 있다. 다음은 **String** 타입의 정보를 사용자 정의 클래스의 타입으로 형 변환하기 위해 **Converter** 인터페이스(org.springframework.core.convert.converter.Converter)를 사용하는 예다.



사용자가 정의한 타입은 내부에 이메일 주소를 정보로 가지고 있으며, 클래스의 이름은 **EmailValue**다. **String** 타입을 **EmailValue** 타입으로 변환하려면 사용자가 직접 **Converter** 인터페이스를 구현한 일종의 변환기 클래스를 만들어야 하는데, 이 예에서는 **StringToEmailValueConverter**라는 이름으로 변환기 클래스를 만들었다.



- **사용자 정의 타입 구현(EmailValue)**

```java
public class EmailValue {
	@Size(max = 256)
    @Email
    private String value;
    
    public void setValue(String value) {
        this.value = value;
    }
    
    public String getValue() {
        return value;
    }
    
    public String toString() {
        return getValue();
    }
}
```





- **사용자 정의 Converter 구현(StringToEmailValueConverter)**

```java
import org.springframework.core.convert.converter.Converter;

public class StringToEmailValueConverter implements Converter<String, EmailValue> {
    @Override
    public EmailValue convert(String source) {
        EmailValue email = new EmailValue();
        email.setValue(source);
        return email;
    }
}
```



이렇게 만들어진 **Converter**는 **ConversionService**에 추가해야 나중에 스프링 프레임워크가 형 변환해야 할 때 사용할 수 있다.



- **자바 기반 설정 방식에서 사용자 정의 Converter 추가**

```java
@Bean
public ConversionService conversionService() {
	DefaultFormattingConversionService conversionService =
		new DefaultFormattingConversionService();
	// addConverter 메서드의 매개변수로 작성한 Converter를 지정
	conversionService.addConverter(new StringToEmailValueConverter());
	return conversionService;
}
```



- **XML 기반 설정 방식에서 사용자 정의 Converter 추가**

```xml
<bean id="conversionService"
      class="org.springframework.format.support.FormattingConversionServiceFactoryBean">
	<!-- converters 프로퍼티에 작성한 Converter를 설정 -->
    <property name="converters">
    	<list>
        	<bean class="com.example.StringToEmailValueConverter" />
        </list>
    </property>
</bean>
```



**Converter**가 준비되면 **String** 타입의 데이터를 프로퍼티 파일에 설정한다. 이 설정 값을 사용할 곳에서는 프로퍼티의 기본값을 설정하고 있지 않기 때문에 프로퍼티 파일의 설정 값이 사용된다. **@Value** 애너테이션으로 프로퍼티 값을 받을 빈의 프로퍼티 타입은 앞서 사용자 정의 클래스로 만들었던 **EmailValue** 타입이다. 바로 이 클래스에 **String** 타입인 '**admin@example.com**'이라는 문자열이 바인딩되며, 이때 앞서 만든 **StringToEmailValueConverter**가 사용된다.



- **프로퍼티 파일로 기본 설정 값 재정의**

```properties
application.adminEmail=admin@example.com
```



- **빈에서 프로퍼티의 설정 값을 형 변환해서 바인딩**

```java
@Component
public class ApplicationProperties {
    // 생략
    @Value("${application.adminEmail:}")
    private EmailValue adminEmail; // 'admin@example.com'이 설정됨.
    // 생략
}
```



이 책에서는 다루지 않지만 이와 반대되는 **Converter**도 꼭 한번 만들어보길 권한다. 쉽게 말해 사용자가 직접 정의한 타입에서 거꾸로 **String** 타입으로 변환하는 역방향 변환기가 필요할 수 있다. 예를 들어, HTML 입력 폼의 값을 자바빈즈의 프로퍼티에 바인딩할 때 기존의 형 변환 기능만으로는 충분하지 않을 수 있다. 만약 사용자가 직접 정의한 **Converter**를 통해 **String** 타입에서 **사용자 정의 클래스**로, 반대로 **사용자 정의 클래스**에서 **String** 타입으로 변환하는 양방향 변환이 가능하다면 HTML 입력 폼에 기존 정보를 바탕으로 HTML 입력 폼을 미리 채워넣을 필요가 있을 때 도움될 것이다.





### 2.3.7 필드 포매팅 방식의 커스터마이징

스프링 프레임워크는 필드 값을 다양한 형식으로 표현하는 포매팅 기능을 제공한다. 이 책에서는 그 중에서도 **JSR 310** 사양과 관련해서 **Date and Time API**의 **java.time.LocalDate**를 활용하는 방법을 소개한다. 이 예에서는 기본 날짜 포맷으로 '20170510'과 같은 형식(**BASIC_ISO_DATE**)을 사용한다.



- **자바 기반 설정 방식에서 Formatter 추가**

```java
@Bean
public ConversionService conversionService() {
    DefaultFormattingConversionService conversionService = 
        new DefaultFormattinConversionService();
    DateTimeFormatterRegistrar registrar = new DateTimeFormatterRegistrar();
    // BASIC_ISO_DATE 형식을 사용 (예: 20170510)
    registrar.setDateFormatter(DateTimeFormatter.BASIC_ISO_DATE);
    registrar.registerFormatters(conversionService);
    return conversionService;
}
```



- **XML 기반 설정 방식에서 Formatter 추가**

```xml
<bean id="conversionService" 
      class="org.springframework.format.support.FormattingConversionServiceFactoryBean">
	<property name="formatterRegistrars">
    	<list>
        	<bean class="org.springframework.format.datetime.standard.DateTimeFormatterRegistrar">
            	<!-- BASIC_ISO_DATE 형식을 사용 (예: 20170510) -->
                <property name="dateFormatter" value="BASIC_ISO_DATE" />
            </bean>
        </list>
    </property>
</bean>
```





다음 예는 **dateOfServiceStarting** 프로퍼티에 대해 기본값은 따로 설정하지 않고, 프로퍼티 파일에서 설정한 값을 읽도록 만들어져 있다. 참고로 로캘이 대한민국으로 설정된 경우, 기본으로 사용되는 날짜 포맷은 'uuuu-MM-dd'다. 이 예는 기본 날짜 포맷 대신 'uuuuMMdd' 같은 **BASIC_ISO_DATE **형식으로 사용하고 있다.



- **프로퍼티 파일 설정 예**

```properties
application.dateOfServiceStarting=20170510
```



- **빈에서 프로퍼티의 설정 값을 필드 포매팅을 통해 바인딩**

```java
@Component
public class ApplicationProperties {
	// 생략
    @Value("${application.dateOfServiceStarting:}")
    private java.time.LocalDate dateOfServiceStarting; // 2017년 5월 10일 날짜로 설정됨
    // 생략
}
```





## 2.4 프로퍼티 관리

이제 애플리케이션이 사용하는 각종 설정 값을 어떻게 읽어와서 사용하는지 살펴보자. 우선 다음과 같은 빈이 있다고 가정하자.



- **각종 설정 값이 소스코드 안에 기재된 예**

```java
@Bean(destroyMethod = "close")
DataSource dataSource() {
    BasicDataSource dataSource = new BasicDataSource();
    dataSource.setDriverClassName("org.postgresql.Driver");
    dataSource.setUrl("jdbc:postgresql://localhost:5432/demo");
    dataSource.setUsername("demo");
    dataSource.setPassword("pass");
    dataSource.setDefaultAutoCommit(false);
    return dataSource;
}
```



이 빈은 데이터베이스에 접근하기 위한 데이터 소스(DataSource) 역할을 하며, 빈의 이름도 **dataSource**다. 그리고 이 코드에서 볼 수 있는 각종 설정은 데이터베이스에 접속할 때 필요한 각종 정보다. 이렇게 정의된 **dataSource** 빈은 DI 컨테이너에 등록되는데, 이때 데이터베이스 URL이나 사용자명, 패스워드와 같은 환경에 의존적인 정보도 함께 들어가게 된다. 이렇게 되면 향후 데이터 접속 대상이 바뀔 때마다 빈을 다시 정의해야 하고 그로 인해 Deploy나 운영 작업이 복잡하고 번거로워질 수 있다. 스프링 프레임워크는 이러한 상황을 최소화할 수 있도록 각종 설정 정보를 효율적으로 다룰 수 있는 관리 메커니즘을 제공한다.



### 2.4.1 빈 정의 시 프로퍼티 활용

앞서 살펴본 것처럼 각종 설정 값을 소스 코드에 직접 하드 코딩하는 대신 **@Value** 애너테이션(org.springframework.beans.factory.annotation.Value)을 활용하면 별도로 분리된 프로퍼티 값을 소스 코드에 주입해서 쓸 수 있다. 다음은 자바 기반 설정 방식에서 빈을 정의할 때 **프로퍼티**를 활용한 예다.



- **자바 기반 설정 방식에서 @Value 애너테이션을 통해 프로퍼티 값 주입**

```java
@Bean(destroyMethod = "close")
DataSource dataSource(@Value("${datasource.driver-class-name}") String driverClassName,
                      @Value("${datasource.url}") String url,
                      @Value("${datasource.username}") String username,
                      @Value("${datasource.password}") String password) {
    BasicDataSource dataSource = new BasicDataSource();
    dataSource.setDriverClassName(driverClassName);
    dataSource.setUrl(url);
    dataSource.setUsername(username);
    dataSource.setPassword(password);
    dataSource.setDefaultAutoCommit(false);
    return dataSource;
}
```



- **프로퍼티 파일을 정의한 예**

```properties
datasource.driver-class-name=org.postgresql.Driver
datasource.url=postgresql://localhost:5432/demo
datasource.username=demo
datasource.password=pass
```





단 이렇게 설정된 프로퍼티 값을 스프링 프레임워크가 주입하려면 이 프로퍼티 파일의 위치가 어디인지 알고 있어야 한다. 자바 기반 설정 방식을 사용한다면 **@PropertySource** 애너테이션(org.springframework.context.annotation.PropertySource)으로 위치를 지정할 수 있다.



- **자바 기반 설정 방식에서 프로퍼티 파일 위치 지정**

```java
@Configuration
@PropertySource("classpath:application.properties")
public class AppConfig{ 
	// 생략
}
```



앞서 살펴본 자바 기반 설정 방식에서는 **@Value** 애너테이션을 통해 프로퍼티 값을 주입받았던 반면, XML 기반 설정 방식에서는 **\<property>** 요소의 **value** 속성을 통해 프로퍼티 값을 주입받는다. 이때 프로퍼티의 키 값으로 표현되는 '${프로퍼티 키}' 부분은 나중에 다른 값으로 치환될 자리라는 의미로 **플레이스홀더(placeholder)**라고 한다.



- **XML 기반 설정 방식에서 property 요소를 통해 프로퍼티 주입**

```xml
<bean id="realDataSource" class="org.apache.commons.dbcp2.BasicDataSource"
      destroy-method="close">
	<property name="driverClassName" value="${datasource.driver-class-name}" />
    <property name="url" value="${datasource.url}" />
    <property name="username" value="${datasource.username}" />
    <property name="password" value="${datasource.password}" />
    <property name="defaultAutoCommit" value="false" />
</bean>
```



한편 프로퍼티 파일의 위치는 **\<context:property-placeholder>** 요소에서 다음과 같이 지정한다.



- **XML 기반 설정 방식에서 프로퍼티 파일의 위치 지정**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans
	xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="
    	http://www.springframework.org/schema/beans
    	http://www.springframework.org/schema/beans/spring-beans.xsd
    	http://www.springframework.org/schema/context
    	http://www.springframework.org/schema/context/spring-context-4.3.xsd">
    
    <context:property-placeholder location="classpath:application.properties"/>
    <!-- 생략 -->
</beans>
```



스프링 프레임워크의 프로퍼티 관리 기능은 파일명이 '**.properties**'로 끝나는 자바 프로퍼티 파일뿐 아니라, JVM의 시스템 프로퍼티, 환경 변수의 설정 값도 모두 일관된 방법으로 프로퍼티처럼 다룰 수 있다. 만약 같은 이름의 프로퍼티가 JVM이나 환경 변수, 혹은 프로퍼티 파일 등에 중복으로 설정돼 있다면 다음과 같은 우선순위에 따라 프로퍼티가 적용된다.

1. **JVM 시스템 프로퍼티**
2. **환경 변수**
3. **프로퍼티 파일**



즉, 기본값을 프로퍼티 파일에 설정한 후, 특별한 이유가 없다면 그대로 사용하다가, 특정 환경에 맞춰 프로퍼티를 변경해야 하는 상황이 되면 환경 변수나 JVM의 시스템 프로퍼티를 사용해 다른 값으로 바꿀 수 있다.



한편 이미 앞의 예에서 선보인 적이 있지만 플레이스홀더에는 기본값을 줄 수 있다. 표기 형식은 '**${프로퍼티 키:기본값}**'과 같은데, 실제 사용된 모습은 다음과 같다.



- **XML 기반 설정 방식에서 프로퍼티의 기본값 설정**

```xml
<bean id="dataSource" class="org.apache.commons.dbcp2.BasicDataSource" destroy-method="close">
	<property name="driverClassName" value="${datasource.driver-class-name:org.postgresql.Driver}" />
    <property name="url" value="${datasource.url:jdbc:postgresql://localhost:5432/demo}" />
    <property name="username" value="${datasource.username:demo}" />
    <property name="password" value="${datasource.password:pass}" />
</bean>
```



이 방법을 활용하면 특별한 이유가 없는 한, 프로퍼티 파일을 따로 만들 필요가 없다. 일단 기본값을 설정해 두면 꼭 필요한 경우에만 재설정하면 되기 때문에 굳이 변경할 일이 거의 없는 프로퍼티에 대해서는 이 방식을 적용하는 것이 상당히 유용하다.





### 2.4.2 빈 구현 과정에서 프로퍼티 활용

앞서 살펴본 예에서는 빈을 정의할 때 빈 설정 파일에서 프로퍼티를 활용하는 방법을 살펴봤다. 이번에는 빈을 구현할 때 빈 구현 파일에서 프로퍼티를 활용하는 방법을 알아보자. 이 방식도 앞서 살펴본 바와 같이 특정 값을 하드코딩하고 싶지 않을 때 활용하면 된다.



- **빈 구현 소스코드에서 @Value 애너테이션을 통해 프로퍼티 값 주입**

```java
@Component
public class Authenticator {
    @Value("${failureCountToLock:5}")
    int failureCountToLock;
    
    /**
    *인증 처리
    **/
    public void authenticate(String username, String password) {
        // 생략
        // 연속 인증의 실패 횟수가 임곗값을 초과하면 락을 건다.
        if(failureCount >= failureCountToLock) {
            // 잠금 처리
        }
    }
}
```

 

이번에는 조금 다른 유형을 살펴보자. 아래 예는 자바 기반 설정 방식을 사용하되 빈을 정의할 때는 프로퍼티 대신 변수를 참조하게 하고, 이 변수들에게 **@Value** 애너테이션을 붙여서 프로퍼티를 주입하는 방법을 사용했다.



- **자바 기반 설정 방식에서 @Value 애너테이션을 통해 프로퍼티 값 주입**

```java
@Configuration
public class AppConfig {
    @Value("${datasource.driver-class-name}")
    String driverClassName;
    
    @Value("${datasource.url}")
    String url;
    
    @Value("${datasource.username}")
    String username;
    
    @Value("${datasource.password}")
    String password;
    
    @Bean(destroyMethod="close")
    DataSource dataSource() {
        BasicDataSource dataSource = new BasicDataSource();
        dataSource.setDriverClassName(driverClassName);
        dataSource.setUrl(url);
        dataSource.setUsername(username);
        dataSource.setPassword(password);
        dataSource.setDefaultAutoCommit(false);
        return dataSource;
    }
}
```



앞에서 살펴본 예와 비슷하게 보이지만 자세히 보면 빈을 정의할 때 **@Value** 애너테이션을 사용한 것이 아니라 자바 기반 설정의 변수에 붙였다는 것을 알 수 있다. 이러한 방법은 같은 프로퍼티를 여러 개의 빈에서 사용해야 할 경우, 각 빈 정의에서 프로퍼티를 활용하게 할 것이 아니라 자바 기반 설정의 변수 한 곳에서 프로퍼티를 주입 받은 다음, 이것을 여러 다른 빈에서 사용할 때 유용하다.







## 2.5 스프링 표현 언어

**스프링 표현 언어(SpEL: Spring Expression Language)**는 스프링 프레임워크가 제공하는 표현 언어다. **SpEL**은 스프링 프레임워크뿐 아니라 다양한 스프링 프로젝트에서도 사용되고 있으며, 이 책에서 소개하는 **스프링 시큐리티**나 **스프링 데이터 JPA, 스프링 부트** 등에서도 **SpEL**을 활용한다.





### 2.5.1 SpEL 설정

**SpEL**을 사용하려면 **spring-expression**을 의존 라이브러리로 추가한다. 



- **pom.xml 설정**

```xml
<dependency>
	<groupId>org.springframework</groupId>
    <artifactId>spring-expression</artifactId>
</dependency>
```





### 2.5.2 SpEL API 개요

개발자가 직접 **SpEL API**를 사용하는 경우는 매우 드물지만 **SpEL** 구조를 이해하기 위해 **SpEL API**를 직접 다루는 방법을 조금만 살펴보자. 참고로 여기서 소개하는 대부분의 기능은 **SpEL**을 활용하는 스프링 프로젝트 내부에서 이미 사용하고 있다.



**SpEL**과 관련된 인터페이스 가운데 사용자가 직접 사용할 만한 주요 인터페이스로는 **org.springframework.expression.ExpressionParser**와 **org.springframework.expression.Expression**의 두 가지가 있다. 우선 **ExpressionParser** 인터페이스는 문자열 형태인 표현식을 분석하고, **Expression** 객체를 생성하기 위한 메서드를 제공한다. 이어 **Expression** 인터페이스는 분석된 표현식의 내용을 실행하기 위한 메서드를 제공한다.



그럼 이제 실제로 **SpEL**을 이용해 곱셈과 덧셈 같은 간단한 수식을 풀어보자. 이때 표현식을 분석하기 위한 **ExpressionParser**로는 **SpEL** 구현 클래스인 **org.springframework.expression.spel.standard.SpelExpressionParser**를 사용하겠다.



- **SpEL을 이용한 간단한 수식 풀기**

```java
ExpressionParser parser = new SpelExpressionParser();  // SpEL 분석기 생성
Expression expression = parser.parseExpression("1 * 10 + 1");  // 표현식 분석
Integer calculationResult = expression.getValue(Integer.class)   // 표현식 실행, 결과 확인
```



이 수식의 평가 결과인 **calculationResult**의 값은 11이다. 이 구문이 이해된다면 이번에는 형태를 조금 바꿔서 자바빈즈의 프로퍼티의 값을 **SpEL**로 설정하는 예를 살펴보자.





- **SpEL을 이용한 자바빈즈의 프로퍼티 설정**

```java
ExpressionParser parser = new SpelExpressionParser();  // SpEL 분석기 생성
Expression expression = parser.parseExpression("joinedYear");  // 표현식 분석
Staff staff = new Staff();   // 자바빈즈 생성
expression.setValue(staff, "2000");  // 표현식 실행
Integer joinedYear = staff.getJoinedYear();   // 결과 확인
```



자바빈즈의 특정 프로퍼티에 대해 **접근자(getter)**나 **설정자(setter)** 메서드를 호출하고 싶다면 표현식에 해당 프로퍼티명을 명시해야 한다. 이후 자바빈즈를 생성하고 표현식을 실행한 다음, 그 결괏값을 확인해 보면 **Staff** 객체의 **joinedYear** 프로퍼티 값이 2000인 것을 확인할 수 있다. 여기서 주목해야 하는 것은 <u>표현식을 실행할 때 설정한 값은 **Integer** 타입이 아니라 **String** 타입이었다</u>는 점이다. **SpEL**은 스프링이 제공하는 **형 변환 기능(ConversionService 인터페이스)**을 활용하기 때문에 **String** 타입의 값을 임의의 다른 타입으로 형 변환할 수 있다. 이 과정에서 기본적으로 **DefaultConversionService** 클래스를 사용한다. 한편 **EvaluationContext** 인터페이스(org.springframework.expression.EvaluationContext)의 구현체를 직접 확장하면 기본 기능을 커스터마이즈할 수도 있다.



> **SpelExpressionParser** 인스턴스를 생성할 때 **org.springframework.expression.spel.SpelParserConfiguration**을 사용하면 표현식의 해석과 실행하는 기본 동작을 다른 방식으로 조정할 수 있다. 한편 스프링 프레임워크 4.1 버전부터는 표현식을 미리 컴파일해서 쓸 수 있게 됐다. 기본 설정으로는 컴파일 기능이 비활성화돼 있지만 이를 활성화할 경우, 표현식과 관련된 처리에서 성능 향상을 기대할 수 있을 것이다.





### 2.5.3 빈 정의 시 SpEL 활용

**SpEL**은 XML 기반이나 애너테이션 기반의 빈 설정 방식에서도 사용할 수 있고 **'#{표현식}'**과 같은 형태로 표기한다. 다음은 생성자의 인수에 **SpEL**을 활용하는 예다.



- **인수가 있는 생성자 구현**

```java
public class TemporaryDirectory implements Serializable {
    private static final long serialVersionUID = -687991492884005033L;
    private final File directory;
    public TemporaryDirectory(File baseDirectory, String id) {
        this.directory = new File(baseDirectory, id);
    }
    // 생략
}
```



우선 클래스를 하나 만들고 그 클래스에 대한 생성자를 만들되, 인수가 있는 생성자를 만들었다. 이제 이 인수에 **SpEL**을 활용해 값을 설정해보자.



XML 기반 설정 방식을 사용한다면 **\<constructor-arg>** 요소의 **value** 속성에 **SpEL**을 지정하면 되고, 애너테이션 기반 설정 방식을 사용한다면 생성자의 인수에 **@Value** 애너테이션을 붙여준 다음, **value** 속성에 **SpEL**을 지정하면 된다.



- **XML 기반 설정 방식에서 SpEL 활용**

```xml
<bean id="sessionScopedTemporaryDirectory"
      class="com.example.TemporaryDirectory" scope="session">
	<constructor-arg index="0" value="file://#{systemProperties['java.io.tmpdir']}/app" />
    <constructor-arg index="1" value="#{T(java.util.UUID).randomUUID().toString()}" />
    <aop:scoped-proxy />
</bean>
```





- **애너테이션 기반 설정 방식에서 SpEL 활용**

```java
@Autowired
public TemporaryDirectory(
	@Value("file://#{systemProperties['java.io.tmpdir']}/app") File baseDirectory,
	@Value("#{T(java.util.UUID).randomUUID().toString()}") String id) {
    // systemProperties는 Map 형태의 예약된 변수로 시스템의 프로퍼티 설정을 담고 있다. 
    // 여기서 임시 디렉터리 경로를 가져온 후, 생성자의 인수에 주입한다.
    // UUID.randomUUID 메서드는 무작위 값을 만드는 static 메서드다. 이 메서드를 호출해서 만들어진 임의의 문자열을 
    // 생성자 인수에 주입한다.
    
    
    this.directory = new File(baseDirectory, id);
}
```



> **SpEL**은 그 밖에도 **@EventListener, @TransactionalEventListener, @Cacheable, @CachePut, @CacheEvict** 같은 다양한 애너테이션에서 활용할 수 있다.





### 2.5.4 SpEL에서 쓸 수 있는 표현식 유형

지금까지 살펴본 표현식은 **SpEL**이 제공하는 표현식의 일부에 불과하다. 여기서는 **SpEL**이 지원하는 표현식의 주요 유형을 소개한다. 만약 좀 더 자세한 내용을 알고 싶거나 **SpEL**이 지원하는 모든 표현 방식을 확인하고 싶다면 스프링 프레임워크 공식 레퍼런스 문서를 참고하자.



#### 리터럴 값

**SpEL**은 문자열이나 수치 값(지수 표기, Hex 표기, 소수점, 음의 부호 등도 포함), 부울 값, 날짜나 시간과 같은 리터럴 값의 표현을 지원한다. 문자열의 리터럴 값을 표현하는 경우에는 작은따옴표를 사용해 '문자열'과 같은 형식으로 표현한다.





#### 객체 생성 

**SpEL**은 **List**나 **Map**을 생성하는 표현이나 **new** 연산자를 사용해 배열이나 임의의 객체를 생성하는 표현도 지원한다. 구체적으로는 다음과 같은 형식으로 표현한다. 참고로 이때 사용되는 타입은 **primitive** 타입을 제외하면 **FQCN**을 사용한다.

- **List를 생성하는 경우: "{값(, ...)}"**

  예: "{1, 2, 3}"



- **Map을 생성하는 경우: "{키:값(, ...)}"**

  예: "{name: '홍길동', joinedYear: 2017}"



- **배열을 생성하는 경우: "new 타입[인덱스]" 또는 "new타입[] {값(, ...)}" 형식**

  예: "new int[]{1, 2, 3}"



- **임의의 객체를 생성하는 경우: "new 타입(...)"**

  예: "new com.example.FileUploadHelper()"





#### 프로퍼티 참조

**SpEL**은 자바빈즈의 프로퍼티에 접근하기 위한 표현을 지원한다. 기본적으로는 프로퍼티의 이름을 명시하면 되는데, 중첩된 객체의 프로퍼티나, 컬렉션이나 배열 안에 포함된 요소, 혹은 **Map** 안에 담긴 요소 등에 대해서도 접근할 수 있다. 표현 방식은 스프링 프레임워크의 데이터 바인딩에서 사용하는 표현과 동일하다. (예: "name.first", "emails[0]")





#### 메서드 호출

**SpEL**은 자바 객체의 메서드를 호출하기 위한 표현을 지원한다. 기본적으로 일반적인 자바 메서드를 호출하는 방법과 같다. (예: "'Hello World'.substring(0, 5)")





#### 타입

**SpEL**은 타입을 의미하는 표현인 'T(타입의 FQCN)'를 지원한다. 이 표현은 상수를 나타내거나 static 메서드를 호출할 때 사용한다(상수를 표현한 예: "T(java.math.RoundingMode).CEILING"), static 메서드를 호출한 예: "T(java.util.UUID).randomUUID()")





#### 변수 참조

**SpEL**에는 변수의 개념이 있어서 변수에 접근하기 위한 표현인 '**#변수명**'을 지원한다. 이 표현은 뒤에서 소개할 스프링 시큐리티에서 메서드의 인가 기능 등에서 활용할 수 있다.





#### 빈 참조

**SpEL**은 DI 컨테이너에서 빈을 참조하기 위한 표현으로 '**@빈 이름**'을 지원한다. 이 표현은 뒤에 소개할 스프링 MVC가 제공하는 JSP 태그 라이브러리 중에서 **\<spring:eval>** 요소의 **expression** 속성에서 사용할 수 있다.





#### 연산자

**SpEL**은 표준 관계 연산자인 **<, >, <=, >=, ==, !=, !**은 물론, 인스턴스를 비교하는 **instanceof**, 정규 표현식으로 일치 여부를 확인하는 **matches**를 지원한다. 



또한 **and, or** 같은 **논리 연산자**, **if-then-else** 같은 조건 분기를 위한 **삼항 연산자**나 삼항 연산자를 간결하게 표현한 **엘비스(Elvis) 연산자**를 지원하며, **+, -, *, /, %** 같은 **산술 연산자**도 지원한다.

(삼항 연산자의 예: "name != null ? name : '-' ", 엘비스 연산자의 예: "name ?: '-' ")





#### 템플릿

**SpEL**은 텍스트 본문 안에 표현식을 집어 넣어 텍스트에 표현식의 결과를 표시하게 만드는 **템플릿(template)** 기능을 지원한다. 표현식 부분은 '**#{표현식}**'과 같은 방식으로 기재하면 된다. 예를 들어 본문에 "**Staff Name : #{name}**"과 같이 기재해두면 나중에 #{name} 부분이 자바빈즈의 **name** 속성 값으로 치환된 텍스트를 볼 수 있다. 이렇게 간단한 템플릿 기능만 필요하다면 굳이 **아파치 프리마커(Apache Freemarker)** 같은 템플릿 엔진을 도입하지 않아도 된다.





#### 컬렉션

**SpEL**은 컬렉션 안에서 조건과 일치하는 요소를 추출하기 위한 표현(**Collection Selection**)이나 컬렉션 안에 있는 요소가 가진 특정 프로퍼티의 값을 추출하기 위한 표현(**Collection Projection**)을 지원한다.







## 2.6 리소스 추상화

애플리케이션을 개발할 때는 설정 파일과 같은 다양한 리소스를 필요로 한다. 하지만 이러한 리소스들은 위치한 곳이 제각각일 수 있다. 예를 들어 파일 시스템 상의 디렉터리라거나 classpath 상의 디렉터리, 혹은 서블릿 컨테이너에 배포된 **war** 파일이나 **jar** 파일, 심지어는 또 다른 웹 서버에 있는 것과 같이 다양한 위치에 흩어져 있을 수 있다. 원래대로라면 애플리케이션을 개발할 때 이러한 리소스의 위치를 모두 알고 접근해야 하지만 스프링 프레임워크가 제공하는 **리소스 추상화** 기능을 활용하면 구체적인 위치 정보를 직접 다루지 않더라도 리소스에 접근할 수 있게 된다.



우선 스프링 프레임워크가 제공하는 인터페이스와 클래스를 살펴보자.



### 2.6.1 Resource 인터페이스와 구현 클래스

스프링 프레임워크는 리소스를 추상화하기 위해 **Resource** 인터페이스(org.springframework.core.io.Resource)와 쓰기 가능한 리소스라는 의미로 **WritableResource** 인터페이스(org.springframework.core.io.WritableResource)를 제공한다.



- **InputStreamSource 인터페이스**

```java
public interface InputStreamSource {
	InputStream getInputStream() throws IOException;
}
```



- **Resource 인터페이스**

```java
public interface Resource extends InputStreamSource {
    boolean exists();
    boolean isReadable();
    boolean isOpen();
    URL getURL() throws IOException;
    URI getURI() throws IOException;
    File getFile() throws IOException;
    long contentLength() throws IOException;
    long lastModified() throws IOException;
    Resource createRelative(String relativePath) throws IOException;
    String getFilename();
    String getDescription();
}
```





- **WritableResource 인터페이스**

```java
public interface WritableResource extends Resource {
	boolean isWritable();
	OutputStream getOutputStream() throws IOException;
}
```





스프링 프레임워크가 기본적으로 제공하는 **Resource** 인터페이스의 주요 구현 클래스는 다음과 같다.



| 클래스명                   | **설명**                                                     |
| -------------------------- | ------------------------------------------------------------ |
| **ClassPathResource**      | classpath 상에 있는 리소스를 다루기 위한 클래스              |
| **FileSystemResource**     | java.io 패키지의 클래스를 사용해 파일시스템 상의 리소스를 다루기 위한 클래스. WritableResource도 구현하고 있다. |
| **PathResource**           | Java SE 7에 추가된 java.nio.file 패키지의 클래스를 사용해 파일시스템상의 리소스를 다루기 위한 클래스. WritableResource도 구현하고 있다. |
| **UrlResource**            | URL 상의 웹 리소스를 다루기 위한 클래스. 경로에 'http://'를 쓰고 HTTP 프로토콜을 사용하는 것이 일반적이다. 단 경로에 'file://'을 쓰고 파일시스템 상의 리소스도 다룰 수 있다. |
| **ServletContextResource** | 웹 애플리케이션 상의 리소스를 다루기 위한 클래스             |



이러한 구현 클래스는 직접 골라 써도 되지만 스프링 프레임워크는 리소스의 위치를 보고 적절한 구현 클래스를 선택할 수 있는 기능을 제공한다. 이를 가능하게 하는 것이 다음에 소개할 **ResourceLoader** 인터페이스다.



> 스프링 프레임워크에서 제공하는 **WritableResource** 인터페이스의 구현 클래스는 파일 시스템에 쓰는 기능만 지원한다. 반면 스프링 클라우드 프로젝트에 속한 AWS 지원 프로젝트(Spring Cloud for Amazon Web Services)에서는 Amazon S3(Simple Storage Service)에 관리되는 데이터에 대한 업로드와 다운로드 기능을 모두 **WritableResource** 인터페이스의 구현 클래스로 제공한다. 이것은 **리소스 추상화 기능**을 잘 활용한 대표적인 예로, 애플리케이션의 소스코드를 변경하지 않아도 데이터의 저장 위치를 Amazon S3나 파일 시스템으로 바꿔 쓸 수 있다.





### 2.6.2 ResourceLoader 인터페이스

스프링 프레임워크는 **Resource** 객체를 생성하는 과정을 추상화하기 위해 **ResourceLoader** 인터페이스(org.springframework.core.io.ResourceLoader)를 제공한다. 참고로 DI 컨테이너를 구성하는 다양한 **ApplicationContext** 인터페이스의 구현 클래스는 모두 이 인터페이스를 구현하고 있다.



- **ResourceLoader 인터페이스**

```java
public interface ResourceLoader {
	String CLASSPATH_URL_PREFIX = ResourceUtils.CLASSPATH_URL_PREFIX;
    Resource getResource(String location);
    ClassLoader getClassLoader();
}
```



**Resource** 객체를 가져오려면 **getResource** 메서드의 매개변수로 리소스의 위치를 지정하면 된다. 리소스의 위치를 지정할 때는 일반적으로 파일시스템 상의 경로나 URL 상의 경로를 사용하게 되는데, 만약 클래스패스 상의 리소스를 지정해야 한다면 경로 앞에 '**classpath:**'와 같은 접두어가 붙는다. 결국 **ResourecLoader** 인터페이스의 구현 클래스는 리소스의 경로 정보를 보고 이에 맞는 **Resource** 인터페이스의 구현 클래스를 선택하게 된다. 한편 **ResourceLoader**의 서브 인터페이스로 **ResourcePatternResolver** 인터페이스(org.springframework.core.io.support.ResourcePatternResolver)도 제공되는데, 리소스의 위치를 나타내는 경로에 **앤트(Ant)** 형식의 와일드카드를 사용해 패턴에 맞는 여러 개의 리소스를 가져올 수도 있다. **ResourceLoader**와 마찬가지로 **ApplicationContext** 인터페이스의 구현 클래스는 모두 이 인터페이스를 구현하고 있다.



- **ResourcePatternResolver 인터페이스**

```java
public interface ResourcePatternResolver extends ResourceLoader {
	String CLASSPATH_ALL_URL_PREFIX = "classpath*:";
    Resource[] getResources(String locationPattern) throws IOException;
}
```



리소스의 위치를 지정할 때 주의해야 하는 것은 파일 경로 형식을 사용할 때 **ApplicationContext**의 종류에 따라 파일 경로를 읽는 방식이 달라질 수 있다는 점이다. 예를 들면, **ClassPathXmlApplicationContext** 클래스는 리소스 경로를 **클래스패스 상의 상대 경로**로 인식한다. 반면 **WebApplicationContext** 인터페이스의 구현 클래스는 리소스 경로를 **웹 애플리케이션의** **루트 디렉터리를 기준으로 한 상대 경로**로 인식한다. 만약 이 차이를 제대로 이해하고 있지 않으면 독립형 애플리케이션 환경에서 문제없이 읽을 수 있던 파일이 이상하게 애플리케이션 서버에만 디플로이하면 파일을 못 읽는다는 오류를 맞이할 수 있다. 이런 상황을 미연에 방지하고 싶다면 애초에 리소스의 경로에 명시적으로 '**classpath:**'같은 접두어를 붙이는 것을 권장한다.





### 2.6.3 Resource 인터페이스를 활용한 리소스 접근

실제로 **Resource** 인터페이스를 사용해 리소스에 접근해보자. 다음은 HTTP를 통해 웹 리소스를 가져오는 예다.



- **Resource 인터페이스의 구현 클래스를 통해 웹 리소스 가져오기**

```java
public void accessResource() throws IOException {
	// Resource 객체 생성
    Resource greetingResource =
        new UrlResource("http://localhost:8080/myApp/greeting.json");
    
    // Resource 인터페이스를 통해 웹 리소스에 접근
    try (InputStream in = greetingResource.getInputStream()) {
        String context = StreamUtils.copyToString(in, StandardCharsets.UTF_8);
        System.out.println(content);
    }
}
```



이 예는 로컬 PC에 톰캣과 같은 서블릿 컨테이너나 웹 애플리케이션 서버가 설치 돼 있고 8080번 포트로 구동돼 있으며 **myApp**이라는 웹 애플리케이션을 통해 **greeting_json**이라는 파일에 접근할 수 있다고 가정하고 있다. **greeting.json** 파일은 애플리케이션의 문서 루트(document root) 바로 아래 경로에 위치하며 다음과 같은 내용을 담고 있다.



- **greeting.json 파일의 내용**

```json
{"hello": "world"}
```



웹 애플리케이션이 성공적으로 디플로이된 후에 **accessResource** 메서드를 호출하면 표준 출력에 **greeting.json** 파일의 내용이 표시될 것이다. 이 예에서는 리소스에 접근하기 위해 **UrlResource**라고 하는 **Resource** 인터페이스의 구현 클래스를 **new**로 명시적으로 생성해서 활용하고 있다. 이번에는 이 예를 한층 더 추상화하기 위해 **ResourceLoader** 인터페이스의 구현 클래스를 **@Autowired** 애너테이션으로 자동으로 주입받는 방식으로 개선해보자.



- **ResourceLoader에서 웹 리소스를 취득하는 구현 예**

```java
@Autowired
ResourceLoader resourceLoader;

public void accessResource() throws IOException {
    // ResourceLoader를 통해 Resource 가져오기
    Resource greetingResource = 
        resourceLoader.getResource("http://localhost:8080/myApp/greeting.json");
    // 생략
}
```



일단 구현 클래스에 대한 의존성은 제거됐지만 아직 리소스 경로가 하드코딩돼 있는 상태다. 리소스를 가져올 때는 로컬 환경이나 개발 서버, 테스트 서버, 운영 서버와 같이 실제로 실행될 환경에 따라 경로가 달라질 수 있다. 결국 리소스의 경로 정보를 구하는 방법을 좀 더 유연하게 바꿀 필요가 있는데, 다음은 스프링 프레임워크의 프로퍼티 기능을 활용해 **Resource** 객체를 주입한 예다.



- **프로퍼티 기능을 활용한 리소스 경로 정보 가져오기**

```java
// 프로퍼티에서 리소스 정보를 받아 Resource 객체를 주입한다.
// 프로퍼티 값을 지정하지 않았다면 기본값을 사용한다.
// (http://localhost:8080/myApp/greeting.json)
@Value("${resource.greeting:http://localhost:8080/myApp/greeting.json}")
Resource greetingResource;

public void accessResource() throws IOException {
    try(InputStream in = greetingResource.getInputStream()) {
        String content = StreamUtils.copyToString(in, StandardCharsets.UTF_8);
        System.out.println(content);
    }
}
```



이처럼 스프링 프레임워크의 프로퍼티 기능을 활용하면 프로퍼티 값을 변경하는 것만으로 접근할 리소스를 손쉽게 바꿀 수 있다. 예를 들어, 프로퍼티 파일의 내용을 **resource.greeting=classpath:greeting.json**과 같이 변경하면 classpath 바로 아래의 **greeting.json** 파일을 리소스로 사용하게 된다.





### 2.6.4 XML 파일에서 리소스 지정

XML 기반 설정 방식에서 빈을 정의할 때 프로퍼티 파일을 참조하거나 또 다른 빈 정의 파일을 포함하는 과정에서 Resource 인터페이스의 구현 클래스가 사용된다. 정확하게는 2.1절 'DI'에서 빈 설정 파일을 분할할 때 소개한 **\<import>** 요소에서 다른 빈 설정 파일을 불러올 때 사용하고, 2.4절 '프로퍼티 관리'에서 프로퍼티 파일을 불러와 플레이스홀더를 치환해주는 **\<context:property-placeholder>** 요소에서 사용된다.



가령 다음 예는 도메인 관련 빈만 정의된 설정 파일을 임포트하는 것으로, classpath 상에서 **/META-INF/spring/domain-context.xml** 파일을 읽게 된다.

```java
<import resource="classpath:/META-INF/spring/domain-context.xml" />
```



그리고 다음 예에서는 classpath 상에 있는 모든 프로퍼티 파일을 읽어온 다음, '**${프로퍼티명}**'과 같은 형식으로 플레이스홀더를 사용할 수 있게 해준다.

```java
<context:property-placeholder location="classpath*:/**/*.properties" />
```



여기서 눈여겨봐야 할 부분은 '**classpath*:**' 접두어다. 앞서 살펴본 **classpath:**와 비슷하게 classpath 상의 리소스를 읽어오는 것은 똑같은데 **:** 앞에 *****가 더 붙으면 **jar** 파일과 같은 다른 모듈 안에 포함된 파일도 읽을 수 있다. 한편 리소스 경로를 지정할 때는 **앤트(Ant)** 형식의 와일드 카드로 ******나 *****도 사용할 수 있다.







## 2.7 메시지 관리

애플리케이션을 개발하다 보면 **문자열 형태의 메시지**를 자주 사용하게 된다. 예를 들어, 웹 애플리케이션이라고 한다면 화면에 표시하는 간단한 설명이나 제목, 입력 필드의 항목명과 같이 **내용이 잘 바뀌지 않는 문구**가 있을 것이고, 그 밖에도 어떤 처리가 끝났을 때 표시되는 결과 메시지나 오동작했을 때 표시되는 **오류 메시지**도 있을 것이다.



이러한 **메시지**를 소스코드 안에 하드코딩해도 동작은 하겠지만 대부분의 경우 프로퍼티 파일과 같은 곳에 따로 관리해서 **소스코드에서 분리**하는 경우가 많을 것이다. 이처럼 소스코드에서 메시지를 분리하는 가장 대표적인 예가 바로 다국어를 지원하는 **국제화(Internationalization)** 기능이다. 그 밖에도 굳이 국제화를 하지 않더라도 각종 메시지를 통합해서 관리할 목적으로 소스코드에서 분리하기도 한다.



이번 장에서는 이 같은 **메시지**를 외부에서 참조하는 기능인 **메시지 관리 기법**에 대해 알아보자.







### 2.7.1 MessageSource 인터페이스와 구현 클래스

스프링 프레임워크는 외부에서 메시지 정보를 가져오는 기능을 제공하는데, 그 핵심이 **MessageSource** 인터페이스(org.springframework.context.MessageSourec)다. **MessageSource**는 메시지 정보의 출처를 추상화하기 위한 것으로, 어딘가에 있을 메시지 정보를 가져오기 위해 **getMessage** 메서드를 제공한다.



- **MessageSource 인터페이스**

```java
public interface MessageSource {
    String getMessage(String code, Object[] args, Locale locale) throws NoSuchMessageException;
    String getMessage(String code, Object[] args, String defaultMessage, Locale locale);
    String getMessage(MessageSourceResolvable resolvable, Local locale) throws NoSuchMessageException;
}
```



이 같은 메서드의 기본적인 동작 방식은 **메시지 코드(code)**에 맞는 메시지를 찾아오되, 메시지 문구 안에 동적으로 변경해야 할 부분이 있다면 **인수(args)**로 받아 완성된 메시지를 만드는 것이다. 만약 요청한 코드에 대응하는 메시지가 없다면 **기본 설정한 메시지(defaultMessage)**를 사용하게 하거나 **예외(NoSuchMessageException)**를 발생시키도록 만들 수 있다.



**MessageSourceResolvable** 인터페이스(org.springframework.context.MessageSourceResolvable)는 메시지 정보를 가져오는 데 필요한 각종 정보(**code, args, defaultMessage**)를 한 덩어리로 다루는 인터페이스다. 특이한 점은 메시지 코드와 인수를 여러 벌 지정할 수 있다는 것인데, 여러 벌의 메시지 코드 후보 중, 배열 순서대로 코드를 맞춰보고 가장 먼저 찾은 메시지에 해당하는 인수를 적용해 메시지를 만들어낸다.



- **MessageSourceResolvable 인터페이스**

```java
public interface MessageSourceResolvable {
	String[] getCodes(); // 여러 개의 코드를 사용할 수 있다.
    Object[] getArguments();
    String getDefaultMessage();
}
```





스프링 프레임워크는 다양한 형태의 **MessageSource** 구현 클래스를 제공하는데 대표적인 클래스는 다음의 두 가지다.



| 클래스명                                  | 설명                                                         |
| ----------------------------------------- | ------------------------------------------------------------ |
| **ResourceBundleMessageSource**           | Java SE 표준의 java.util.ResourceBundle을 사용<br />프로퍼티 파일에서 메시지를 가져옴 |
| **ReloadableResourceBundleMessageSource** | 스프링이 제공하는 org.springframework.core.io.Resource를 사용<br />프로퍼티 파일에서 메시지를 가져옴<br />java.util.ResourceBundle의 기능을 확장 |



참고로 스프링 프레임워크 3.1.3 버전 이전에는 **ResourceBundleMessageSource**와 **ReloadableResourceBundleMessageSource** 간의 기능 차이가 상당히 컸는데, Java SE 6에 추가된 기능들이 **ResourceBundleMessageSource**에 많이 채용됐다. 그 결과 프로퍼티 파일의 인코딩(**defaultEncoding**)이나 캐시 기간(**cacheSeconds**) 설정 등이 지원되어 둘 간의 기능 차이가 많이 줄어들었다.







### 2.7.2 MessageSource 사용

이제 **ResourceBundleMessageSource**를 활용해 프로퍼티 파일에 정의된 메시지를 가져오는 방법을 소개한다. 사용할 수 있는 옵션은 조금 다르지만 **ReloadableResourceBundleMessageSource**도 같은 방법으로 사용할 수 있다.



#### MessageSource의 빈 정의

먼저 **MessageSource**를 DI 컨테이너에 등록해보자. 이때 빈 ID를 '**messageSource**'가 되도록 정의하는 것이 중요한데, 이렇게 미리 약속된 이름으로 등록해야 DI 컨테이너(**ApplicationContext**)에서 이 **MessageSource**를 사용할 수 있다.



- **자바 기반 설정 방식에서의 MessageSource 정의**

```java
@Bean
public MessageSource messageSource() {
	ResourceBundleMessageSource messageSource = new ResourceBundleMessageSource();
	// 클래스패스 상에 있는 프로퍼티 파일의 이름을 확장자를 제외하고 지정한다.
	messageSource.setBasenames("messages");
	return messageSource;
}
```





- **XML 기반 설정 방식에서의 MessageSource 정의**

```xml
<bean id="messageSource"
      class="org.springframework.context.support.ResourceBundleMessageSource">
	<!-- 클래스패스 상에 있는 프로퍼티 파일의 이름을 확장자를 제외하고 지정한다. -->
    <property name="basenames">
    	<list>
        	<value>messages</value>
        </list>
    </property>
</bean>
```





#### 메시지 정의

다음은 메시지를 프로퍼티 파일에 정의할 차례다.



- **messages.properties 정의 예**

```properties
# welcome.message={0}님, 환영합니다!
welcome.message={0}\uB2D8, \uD658\uC601\uD569\uB2C8\uB2E4!
```



- **application-messages.properties 정의 예**

```properties
# result.succeed={0} 처리가 성공했습니다.
result.succeed={0} \uCC98\uB9AC\uAC00 \uC131\uACF5\uD588\uC2B5\uB2C8\uB2E4.
```



프로퍼티 **키**에는 **메시지 코드**를, 프로퍼티 **값**에는 **MessageFormat** 클래스(java.text.MessageFormat)가 해석할 수 있는 메시지 문자열을 지정한다. 단, 영문이나 숫자와 같이 **아스키(ASCII)** 코드로 표현할 수 없는 한글과 같은 문자를 메시지에 사용하고 싶다면 **유니코드(Unicode)** 문자를 사용하되 아스키 코드에서 사용 가능한 문자로 표현해야 한다. 이러한 변환을 돕기 위해 JDK에는 **native2ascii**라는 툴이 포함돼 있다.





#### MessageSource의 API 활용

메시지를 사용할 때는 DI 컨테이너에 등록된 **MessageSource**를 주입받은 다음 **getMessage** 메서드를 호출하면 된다.



- **MessageSource의 API 사용 예**

```java
@Autowired
MessageSource messageSource; // 자동으로 주입받음

public void printWelcomeMessage() {
	// getMessage 메서드 호출
	String message = messageSource.getMessage(
        "result.succeed",
        new String[] {"사용자 등록"},
        Locale.KOREAN);
    System.out.println(message);
}
```



**printWelcomeMessage** 메시지를 호출하면 콘솔에 '사용자 등록 처리가 성공했습니다.'라는 메시지가 출력된다.





#### MessageSourceResolvable의 활용

앞서 살펴본 예에서 뭔가 이상한 점을 발견했을 것이다. 바로 '사용자 등록'이라는 문자열이 소스코드상에 하드코딩돼 있다는 점이 문제가 될 수 있다. 이처럼 메시지에 포함할 인수 값조차 프로퍼티 파일에서 읽어오고 싶은 경우에는 **DefaultMessageSourceResolvable** 클래스(org.springframework.context.support.DefaultMessageSourceResolvable)를 활용하면 된다. 다음은 '**functionName.userRegistration**'이라는 키에 해당하는 프로퍼티 값을 메시지의 인수로 전달하는 예다.



- **MessageSourceResolvable의 사용 예**

```java
MessageSourceResolvable functionName =
    new DefaultMessageSourceResolvable("functionName.userRegistration");

String message = messageSource.getMessage(
    "result.succeed",
	new MessageSourceResolvable[] {functionName},
	Locale.KOREAN);
```



이 방법을 사용하면 앞의 예에서 소스코드에 하드코딩했던 '사용자 등록'이라는 문자열을 프로퍼티 파일에서 읽는 방식으로 대체할 수 있다.



- **message.properties의 정의 예**

```properties
# functionName.userRegistration=사용자 등록
functionName.userRegistration=\uC0AC\uC6A9\uC790 \uB4F1\uB85D
```







### 2.7.3 프로퍼티 파일을 UTF-8로 인코딩

지금까지는 프로퍼티 파일에 메시지를 저장할 때 유니코드를 아스키 문자열로 변환한 문자열을 사용했다. 이것은 프로퍼티 파일이 **ISO-8859-1**로 인코딩되는 것이 사양에 규정돼 있기 때문이다. 하지만 **ResourceBundleMessageSource**는 프로퍼티 파일을 다른 인코딩 방식으로 사용할 수 있는 기능을 제공한다. 이 방법을 이용하면 더는 **native2ascii**로 변환하지 않아도 되고 만약 **ResourceBundleEditor** 같은 IDE 플러그인으로 프로퍼티 파일을 관리하지 못하는 상황이라면 아예 파일 자체를 다른 인코딩으로 저장한 후, 읽어 들일 때 해당 인코딩으로 읽어오게 명시할 수 있다.



- **기본 인코딩을 지정해 네이티브 코드 메시지를 활용**

```java
@Bean
public MessageSource messageSource() {
	ResourceBundleMessageSource messageSource = new ResourceBundleMessageSource();
    messageSource.setBasenames("messages");
    // 프로퍼티 파일이 ISO-8859-1이 아니라 UTF-8로 인코딩돼 있다.
    messageSource.setDefaultEncoding("UTF-8");
    return messageSource;
}
```



여기까지 됐다면 이제 기존 프로퍼티 파일을 **UTF-8**로 인코딩하고, 메시지를 다시 **UTF-8**로 다시 작성하면 된다.



- **메시지 정의 예**

```properties
result.succeed={0}건의 처리가 성공했습니다.
functionName.userRegistration=사용자 등록
```





### 2.7.4 다국어 지원하기

**MessageResource**의 구현 클래스는 국가별로 메시지의 언어를 다르게 적용할 수 있는 국제화 기능을 갖추고 있다. 구체적으로는 언어마다 프로퍼티 파일을 따로 만들고 Java SE의 **ResourceBundle** 사양에서 지원하는 로캘을 선택하면 된다. 자바 가상 머신의 기본 로캘이 한국(한국어)이라고 가정할 때 한국어와 영어 메시지를 둘 다 지원해야 한다면 다음과 같이 설정하면 된다.



우선 기본 로캘의 언어에 맞는 프로퍼티 파일을 만든다. 이때 파일명에는 언어나 국가 코드를 추가하지 않은 일반적인 프로퍼티 파일명을 사용한다.



- **messages.properties**

```properties
welcome.message={0}님, 환영합니다!
```







다음으로 영어 메시지를 담은 프로퍼티 파일을 만든다. 이때 파일명에 언어 정보를 접미어로 추가해야 한다.



- **messages_en.properties**

```properties
welcome.message=Welcome, {0}
```



위의 예에서는 메시지를 언어로만 구분하게 만들었지만 국가 코드와 조합하면 메시지를 더 세분화해서 관리할 수 있다. 예를 들어, 미국 영어와 영국 영어에서는 의미는 같지만 서로 다른 단어를 사용하는 경우가 있다. 이런 경우에는 **messages_en_Us.properties, messages_en_GB.properties**와 같이 파일을 구분하면 된다. 그리고 두 국가 간에 차이가 없는 공통적인 메시지는 **messages_en.properties** 파일에 두면 된다.