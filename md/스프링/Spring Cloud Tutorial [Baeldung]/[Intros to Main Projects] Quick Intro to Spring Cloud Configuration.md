# Quick Intro to Spring Cloud Configuration :zap:



> **[참고]**: Quick Intro to Spring Cloud Configuration
>
> https://www.baeldung.com/spring-cloud-configuration
>
> 의역과 오역에 주의...





## 1. Overview :mountain:

**Spring Cloud Config**는 

여러 애플리케이션과 환경에 분산된(distributed) configuration들을 

저장하고 제공하기 위한 Client/Server 접근법이다.



이 Configuration 저장소는 Git version control 하에 완벽하게 versioning 되고,

애플리케이션 runtime에 수정될 수도 있다.



또한, *Environment*, *PropertySource* 또는 *@Value*와 같은 

Configuration과 함께 지원되는 Configuration file 형식을 사용하는 Spring 애플리케이션에 매우 적합하다.



이번 포스팅에서는,

**어떻게 Git-backed config server를 구축하는지**,

**REST 애플리케이션 server에서 Spring Cloud Config를 어떻게 사용하는지,**

**마지막으로 암호화된(encrypted) property value들을 사용해 안전한 환경을 구축하는 방법**을 살펴본다.







## 2. Project Setup and Dependencies :seedling:

다음과 같은 의존성을 maven에 추가하자.

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-server</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```



client project를 만들 때는 아래와 같은 의존성이 필요하다.

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```





## 3. A Config Server Implementation

Config Server 애플리케이션의 main part는 바로 **Config class**다.

더 구체적으로 얘기하자면 **@SpringBootApplication** 어노테이션이 붙은 클래스 말이다.

이 클래스는 *auto-configure* 어노테이션인 **@EnableConfigServer**를 통해

모든 필요한 setup들을 댕겨온다.

```java
@SpringBootApplication
@EnableConfigServer
public class ConfigServer {
    
    public static void main(String[] arguments) {
        SpringApplication.run(ConfigServer.class, arguments);
    }
}
```





이제 우리는 

우리 서버가 listening 하고 있는 **server port**와

version-controlled 된 configuration 내용들을 제공해주는 **Git-url**을

설정해야 한다.



**Git-url**은 *http, ssh* 같은 protocol들을 사용할 수도 있고,

또는 local filesystem의 simple file도 사용 가능하다.



> 만약 당신이 동일한 config repository를 가리키는 다양한 config server instance들을 사용하려고 하고 있다면, 
>
> 당신은 당신의 repo를 local temporary folder에 복사함으로써 server에 대한 설정을 할 수 있다.
>
> 
>
> 하지만 two-factor authentication을 사용하는 private repository들을 조심해서 사용하라.
>
> 얘네들은 다루기가 어렵다!
>
> 이런 경우에는, 그냥 local filesystem에 복사하고 복사본 갖고 하는게 더 쉽다.



또한, *repository-url*을 이용 가능하도록 설정하는 

몇몇 placeholder 변수들과 search pattern들이 존재한다.



우리는 또한 application이 재시작될 때마다 password가 자동으로 재생성되는 것을 막기 위해서,

우리의 *application.properties* 파일에 Basic-Authentication을 위한

**username**과 **password**를 설정할 필요가 있다.

```properties
server.port=8888
spring.cloud.config.server.git.uri=ssh://localhost/config-repo
spring.cloud.config.server.git.clone-on-start=true
spring.security.user.name=root
spring.security.user.password=s3cr3t
```







## 4. A Git Repository as Configuration Storage

Server 설정을 마치기 위해, 우리는 설정된 url 아래에 **Git repository를 초기화** 해야 한다.

또한, 새로운 **properties 파일**들도 만들고 거기에 값들도 할당해야 한다.



Configuration 파일의 이름은 일반적인 Spring application.properties 처럼 구성되지만,

'application'이라는 단어 대신에 configured name이 사용되며 그 뒤에 dash(-)와 active profile이 표시됩니다.



```bash
$> git init
$> echo 'user.role=Developer' > config-client-development.properties
$> echo 'user.role=User'      > config-client-production.properties
$> git add .
$> git commit -m 'Initial config-client properties'
```





## 5. Querying the Configuration

이제 Server를 실행시킬 준비가 됐다.

우리 Server에 의해 제공되는 **Git-backed configuration API**는 

아래의 path들을 참고해서 query 될 수 있다.



```
/{application}/{profile}[/{label}]
/{application}-{profile}.yml
/{label}/{application}-{profile}.yml
/{application}-{profile}.properties
/{label}/{application}-{profile}.properties
```



**{label}** placeholder는 Git branch,

**{application}** placeholder는 Client의 애플리케이션 이름,

**{profile}** placeholder는 현재 활성 애플리케이션 profile을 나타냅니다.



따라서 우리는 





































