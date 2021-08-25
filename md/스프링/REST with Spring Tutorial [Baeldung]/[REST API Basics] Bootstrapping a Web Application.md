# Bootstrap a Web Application With Spring 5 :dog:



> **[참고]**: https://www.baeldung.com/bootstraping-a-web-application-with-spring-and-java-based-configuration
>
> 의역과 오역에 주의...





## 1. Overview :cat:

이번 포스팅은 어떻게 Web Application을 Spring으로 Bootstrap(부팅) 하는지 그 방법에 대해 설명한다.

어떻게든 부팅이나 한 번 해보자



> **Bootstrap**이란? 일반적으로 한 번 시작되면 알아서 진행되는 일련의 과정을 뜻한다.
>
> 부팅 과정을 Bootstrap이라고 하기도 한다. (Bootstrap을 줄여서 부팅이라는 말이 생겼다고 한다 :bulb:)





## 2. Bootstrapping Using Spring Boot :elephant:



### 2.1 Maven 의존성 추가

먼저, `spring-boot-starter-web` 의존성을 추가하자.

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <version>2.4.0</version>
</dependency>
```



이 starter에는 아래와 같은 것들을 포함하고 있다.

- `spring-web`과 `spring-webmvc` 모듈
- `tomcat starter`: 직접 server를 설치하지 않아도 웹 애플리케이션을 실행할 때 내장된 톰캣 서버를 사용하기 위해 사용되는 starter





### 2.2 Spring Boot 애플리케이션 만들기

**Spring Boot**를 이용해서 애플리케이션을 시작하는 가장 간단한 방법은

**main class**를 생성하고 그 위에다가 **@SpringBootApplication** 어노테이션을 붙여주는 것이다.



```java
@SpringBootApplication
public class SpringBootRestApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringBootRestApplication.class, args);
    }
}
```



**@SpringBootApplication** 어노테이션은 **@Configuration**, **@EnableAutoConfiguration**, **@ComponentScan** 세 개를 합친 것과 같은 효과를 낸다.



**@SpringBootApplication**이 붙어있으면, <u>해당 어노테이션이 붙은 클래스가 속한 패키지와 그 하위의 패키지 모두에서 component들을 스캔</u>한다.









다음으로, **자바 기반**으로 **Spring bean을 설정**하기 위해, **config class**를 하나 만들고 그 위에다가 **@Configuration** 어노테이션을 붙여준다.

```java
@Configuration
public class WebConfig {

}
```



**@Configuration** 어노테이션은 **@Component** 어노테이션을 포함하고 있다.

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface Configuration {
    @AliasFor(
        annotation = Component.class
    )
    String value() default "";

    boolean proxyBeanMethods() default true;
}
```

**@Component** 어노테이션이 붙어있으면 **component-scan**의 대상이 되어서 spring bean으로 등록된다.



**@Configuration** 클래스의 메인 목적은 **Spring IoC** 컨테이너에게 **Bean** 정의들을 알려주는 역할이다.







## 3. Bootstrapping Using spring-webmvc :rabbit:

Spring Boot를 사용하지 않고 부팅하는 방법을 알아보자. (~~귀찮다~~ :sob:) 



### 3.1 Maven Dependencies



### 3.2 The Java-based Web Configuration



### 3.3 The Initializer Class





## 4. XML Configuration





## 5. Conclusion



















