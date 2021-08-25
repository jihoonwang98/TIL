# [Chapter 02] 스프링 시작하기





## 1. 스프링은 객체 컨테이너

스프링의 핵심 기능은 객체를 생성하고 초기화하는 것이다.

이와 관련된 기능은 `ApplicationContext`라는 인터페이스에 정의되어 있다.

`AnnotationConfigApplicationContext` 클래스는 이 인터페이스를 알맞게 구현한 클래스 중 하나다.

`AnnotationConfigApplicationContext` 클래스는 자바 클래스에서 정보를 읽어와 객체 생성과 초기화를 수행한다.

XML 파일이나 그루비 설정 코드를 이용해서 객체 생성/초기화를 수행하는 클래스도 존재한다.





![](https://docs.google.com/drawings/d/e/2PACX-1vQbQTBXMIFl3_fDXyLMccoLWKIHmIXUg7IcQR_BGiMF0xnPOxdmIVzvSG7NklVRwyWUUWA48YACY4Ao/pub?w=1066&h=722)

위는 `AnnotationConfigApplicationContext` 클래스의 계층도 일부이다.

계층도를 보면 가장 상위에 `BeanFactory` 인터페이스가 위치하고, 위에서 세 번째에 `ApplicationContext` 인터페이스, 

그리고 가장 하단에 `AnnotationConfigApplicationContext` 등의 구현 클래스가 위치한다.



**인터페이스**

- `BeanFactory` 인터페이스

  - 객체 생성과 검색 기능을 정의하고 있음

  - ex) `getBean()` 메서드(생성된 객체를 가져오는 메서드) 정의되어 있음
  - 싱글톤/프로토타입 빈인지 확인하는 기능 정의되어 있음

- `ApplicationContext` 인터페이스

  - 메시지, 프로필/환경 변수 등을 처리할 수 있는 기능을 추가로 정의하고 있음

  - 빈 객체의 생성, 초기화, 보관, 제거 등을 관리하고 있음 (-> 이와 같은 이유로 `ApplicationContext`를 컨테이너라고도 부름)



**구현체**

- `AnnotationConfigApplicationContext`: **자바 애너테이션**을 이용한 클래스로부터 객체 설정 정보를 가져옴
- `GenericXmlApplicationContext`: **XML**로부터 객체 설정 정보를 가져옴
- `GenericGroovyApplicationContext`: **그루비** 코드를 이용해 설정 정보를 가져옴





## 2. 싱글톤(Singleton) 객체

```java
public class Main {
	public static void main(String[] args) {
        AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext(AppContext.class);
        Greeter g1 = ctx.getBean("greeter", Greeter.class);
        Greeter g2 = ctx.getBean("greeter", Greeter.class);
        
        System.out.println("(g1 == g2) = " + (g1 == g2));    // true
        ctx.close();
    }
}
```

위 코드에서 `ctx.getBean(...)` 메서드가 같은 객체를 리턴함을 알 수 있다.

별도 설정을 하지 않을 경우 스프링은 한 개의 빈 객체 만을 생성하며, 이때 빈 객체는 '**싱글톤(singleton)** 범위를 갖는다'고 표현한다.

스프링은 기본적으로 한 개의 `@Bean` 애너테이션에 대해 한 개의 빈 객체를 생성한다.