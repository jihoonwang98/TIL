## 3.1 스프링 프레임워크와 데이터 소스

이번 절에서는 스프링 프레임워크에서 제공하는 데이터 접근 방법 중 **JDBC**를 통한 접근 방법에 대해 알아보겠다. 다만 그 전에 **DataSource**의 종류에 대해 먼저 살펴보고, 이후에는 **스프링 JDBC(Spring JDBC)**를 활용하는 방법에 대해 설정과 예를 보면서 살펴보겠다.





---

### 3.1.1 DataSource 개요

**DataSource**는 애플리케이션이 DB에 접근하기 위한 추상화된 연결 방식, 즉 **Connection(java.sql.Connection)**을 제공하는 역할을 한다. 그리고 스프링 프레임워크가 제공하는 **DataSource**에는 크게 <u>세 가지 종류</u>가 있다.



- **애플리케이션 모듈이 제공하는 DataSource**

  **Commons DBCP**나 **Tomcat JDBC Connection Pool**과 같이 서드파티가 제공하는 **DataSource**나 **DriverManagerDataSource** 같이 스프링 프레임워크가 테스트 용도로 제공하는 **DataSource**를 **Bean**으로 등록해서 사용하는 방식을 말한다. 이러한 방식은 DB에 접속하기 위한 사용자 ID와 패스워드, 접속 대상 URL 같은 DB 접속 정보를 애플리케이션이 직접 관리하고 **DataSource**에 설정해야 한다.



- **애플리케이션 서버가 제공하는 DataSource**

  애플리케이션 서버가 정의한 **DataSource**를 **JNDI(Java Naming and Directory Interface)**를 통해 가져와서 사용하는 방식이다. 이 방식은 DB에 접속하기 위한 각종 정보를 애플리케이션 서버에서 관리하기 때문에 애플리케이션으로부터 DB 관련 정보를 분리해낼 수 있을뿐더러 애플리케이션 서버가 제공하는 각종 관리 기능도 활용할 수 있다는 장점이 있다.



- **내장형 DB를 사용하는 DataSource**

  **HSQLDB**, **H2**, **Apache Derby** 같은 내장형 DB에 접속하는 **DataSource**를 말한다. 이 방식은 사전에 DB를 준비할 필요 없이 애플리케이션이 기동할 때 **DataSource**의 설정과 생성이 자동으로 이루어진다. DB 환경을 손쉽게 만들 수 있기 때문에 애플리케이션을 본격적으로 개발하기 전에 프로토타입을 만들거나, 업무 비즈니스 성격이 아닌 각종 지원 툴이나 관리 툴 성격의 애플리케이션을 개발할 때 많이 활용된다.





---

### 3.1.2 DataSource 설정

이제 **DataSource**를 설정하는 방법을 살펴보자. **DataSource**를 사용하려면 **Maven**의 **pom.xml**에 다음과 같이 의존 관계를 정의해야 한다.



**pom.xml 설정**

```xml
<dependency>
	<groupId>org.springframework</groupId>
    <artifactId>spring-jdbc</artifactId>
</dependency>
```

 





#### 애플리케이션 모듈이 제공하는 DataSource

**Commons DBCP** 같은 **Connection Pool**을 **DataSource**로 사용할 때의 설정 방식을 알아보자. 다음은 DB로는 **PostgreSQL**을, **DataSource**에는 **Commons DBCP**를 사용하는 예다. DB에 접속할 때 필요한 각종 정보와 **Connection Pool**의 설정 값 등은 별도의 `jdbc.properties`에 기재돼 있다고 가정한다.





- **자바 기반 설정 방식으로 DataSource 정의**

```java
@Configuration
@PropertySource("classpath:jdbc.properties")
public class PoolingDataSourceConfig {
    @Bean(destroyMethod = "close")
    public DataSource dataSource(
    	@Value("${database.driverClassName}") String driverClassName,
        @Value("${database.url}") String url,
        @Value("${database.username}") String username,
        @Value("${database.password}") String password,
        @Value("${cp.maxTotal}") int maxTotal,
        @Value("${cp.maxIdle}") int maxIdle,
        @Value("${cp.minIdle}") int minIdle,
        @Value("${cp.maxWaitMillis}") long maxWaitMillis) {
        BasicDataSource dataSource = new BasicDataSource();
        dataSource.setDriverClassName(driverClassName);
        dataSource.setUrl(url);
        dataSource.setUsername(username);
        dataSource.setPassword(password);
        dataSource.setDefaultAutoCommit(false);
        dataSource.setMaxTotal(maxTotal);
        dataSource.setMaxIdle(maxIdle);
        dataSource.setMinIdle(minIdle);
        dataSource.setMaxWaitMillis(maxWaitMillis);
        
        return dataSource;
    }  
}
```





- **데이터베이스 관련 설정 정보(jdbc.properties)**

```properties
database.url=jdbc:postgresql://localhost/sample
database.username=postgres
database.password=postgres
database.driverClassName=org.postgresql.Driver
cp.maxTotal=96
cp.maxIdle=16
cp.minIdle=0
cp.maxWaitMillis=60000
```





- **XML 기반 설정 방식으로 DataSource 정의**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="
           http://www.springframework.org/schema/beans 
           http://www.springframework.org/schema/beans/spring-beans.xsd
           http://www.springframework.org/schema/context
           http://www.springframework.org/schema/context/spring-context.xsd">
    
    <context:property-placeholder location="classpath:META-INF/jdbc.properties" />

    <bean id="dataSource"
          class="org.apache.commons.dbcp2.BasicDataSource" destroy-method="close">
    	<property name="driverClassName" value="${database.driverClassName}" />
    	<property name="url" value="${database.url}" />
        <property name="username" value="${database.username}" />
        <property name="password" value="${database.password}" />
        <property name="defaultAutoCommit" value="false" />
        <property name="maxTotal" value="${cp.maxTotal}" />
        <property name="maxIdle" value="${cp.maxIdle}" />
        <property name="minIdle" value="${cp.minIdle}" />
        <property name="maxWaitMillis" value="${cp.maxWaitMillis}" />
    </bean>
</beans>
```







#### 애플리케이션 서버가 제공하는 DataSource

다음으로 애플리케이션 서버가 제공하는 **DataSource**를 활용하는 방식을 설명한다. 이 예에서는 애플리케이션 서버에 `jdbc/mydb`라는 **JNDI**명으로 **Connection Pool**이 만들어져 있다고 가정한다.





- **자바 기반 설정 방식으로 DataSource 정의**

```java
@Configuration
public class JndiDataSourceConfig {
    @Bean
    public DataSource dataSource() {
        JndiTemplate jndiTemplate = new JndiTemplate();
        return jndiTemplate.lookup("java:comp/env/jdbc/mydb", DataSource.class);
    }
}
```





XML 기반 설정 방식에서는 **jee 네임 스페이스**를 활용해 설정을 더욱 간결하게 만들 수 있다.



- **XML 기반 설정 방식으로 DataSource 정의**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:jee="http://www.springframework.org/schema/jee"
       xsi:schemaLocation="
           http://www.springframework.org/schema/beans 
           http://www.springframework.org/schema/beans/spring-beans.xsd
           http://www.springframework.org/schema/jee
           http://www.springframework.org/schema/jee/spring-jee.xsd">
    

    <jee:jndi-lookup id="dataSource" jndi-name="java:comp/env/jdbc/mydb" />
</beans>
```









#### 내장형 DB를 사용하는 DataSource

마지막으로 내장형 DB를 사용하는 **DataSource**에 대해 알아보자. 내장형 DB를 사용할 때는 애플리케이션을 가동할 때마다 DB가 새로 구축되기 때문에 테이블과 같은 기본 구조를 만들기 위한 **DDL**과 초기 데이터를 적재하기 위한 **DML**을 함께 준비해야 한다.



다음은 내장형 DB로 **H2**를 사용하고, DB를 구축하기 위한 **DDL**로는 `schema.sql` 파일이, 초기 데이터를 적재하기 위한 **DML**로는 `insert-init-data.sql` 파일을 사용하는 예다.





- **자바 기반 설정 방식으로 DataSource 정의**

```java
import javax.sql.DataSource;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class DataSourceEmbeddedConfig {
    @Bean
    public DataSource dataSource() {
        return new EmbeddedDatabaseBuilder()
            	.setType(EmbeddedDatabaseType.H2)
            	.setScriptEncoding("UTF-8")
            	.addScripts("META-INF/sql/schema.sql", "META-INF/sql/insert-init-data.sql")
            	.build();
    }
}
```





- **XML 기반 설정 방식으로 DataSource 정의(datasource-embedded.xml)**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:jdbc="http://www.springframework.org/schema/jdbc"
       xsi:schemaLocation="
           http://www.springframework.org/schema/beans 
           http://www.springframework.org/schema/beans/spring-beans.xsd
           http://www.springframework.org/schema/jdbc
           http://www.springframework.org/schema/jdbc/spring-jdbc.xsd">

    <jdbc:embedded-database id="dataSource" type="H2">
    	<jdbc:script location="classpath:META-INF/sql/schema.sql" encoding="UTF-8" />
        <jdbc:script location="classpath:META-INF/sql/insert-init-data.sql" encoding="UTF-8" />    
    </jdbc:embedded-database>
</beans>
```











## 3.2 스프링 JDBC

3.절에서는 **스프링 JDBC**를 사용할 때 필요한 **DataSource**의 설정 방법을 알아봤다. 여기서는 <u>데이터를 다룰 때</u> 중요한 역할을 하는 **JdbcTemplate** 클래스를 살펴보자. 기본적인 CRUD 처리를 하려면 SQL을 어떻게 실행하는지, SQL에 필요한 값을 어떻게 바인딩하는지, SQL이 실행된 후, 그 결과에서 어떻게 데이터를 꺼내오는지 알아보자.







---

### 3.2.1 스프링 JDBC 개요

**스프링 JDBC**는 SQL의 실제 내용과는 상관없이 공통적이면서도 반복적으로 수행되는 **JDBC** 처리를 개발자가 직접 구현하는 대신 프레임워크가 대행하는 기능을 제공한다. 여기서 말하는 공통적이면서도 반복적으로 수행되는 처리에는 다음과 같은 것들이 있다.



- **커넥션의 연결과 종료**
- **SQL 문의 실행**
- **SQL 문 실행 결과 행에 대한 반복 처리**
- **예외 처리**



이처럼 스프링 JDBC를 활용하면 위와 같은 공통적이고 반복적인 작업을 개발자가 직접 구현하지 않아도 되므로 작업 분량이 상당히 줄어든다. 결국 이러한 내용을 제외하고 나면 개발자가 구현할 부분은 다음과 같은 내용만 남는다.



- **SQL 문 정의**
- **파라미터 설정**
- **ResultSet에서 결과를 가져온 후, 각 레코드별로 필요한 처리**







---

### 3.2.2 JdbcTemplate 클래스를 활용한 CRUD

**스프링 JDBC**는 **JdbcTemplate 클래스**(org.springframework.jdbc.core.JdbcTemplate)나 **NamedParameterJdbcTemplate 클래스**(org.springframework.jdbc.core.NamedParameterJdbcTemplate)와 같이 SQL만 가지고도 DB를 쉽게 다룰 수 있게 도와주는 클래스를 제공한다. 특히 **NamedParameterJdbcTemplate** 클래스는 사실상 데이터를 조작하는 처리를 **JdbcTemplate** 클래스에 위임하게 되는데, 



이 둘의 차이는 
<span style="color:red">JdbcTemplate 클래스</span>가 데이터 바인딩 시<span style="color:red"> '?' 문자</span>를 플레이스홀더로 사용하는 반면, 
<span style="color:blue">NamedParameterJdbcTemplate 클래스</span>는 데이터 바인딩 시 <span style="color:blue">파라미터 이름</span>을 사용할 수 있어서 '?'를 사용할 때보다 좀 더 직관적으로 데이터를 다룰 수 있게 해준다는 것이다.



한편 **JdbcTemplate** 클래스를 애플리케이션에서 사용할 때는 개발자가 직접 **JdbcTemplate**을 만들어서 쓰기보다는 DI 컨테이너가 만든 **JdbcTempalte**을 **@Autowired** 애너테이션으로 주입받아서 쓰는 것이 더 일반적이다. 예를 들어, **user** 테이블에서 **user_id**로 사용자 정보를 조회한 다음, **user_name** 정보를 결과로 받고 싶다면 다음과 같이 구현한다.



- **JdbcTemplate을 활용한 데이터 조회**

```java
@Autowired
JdbcTemplate jdbcTemplate;

public String findUserName(String userId) {
	String sql = "SELECT user_name FROM user WHERE user_id = ?";
    return jdbcTemplate.queryForObject(sql, String.class, userId);
}
```





- **자바 기반 설정 방식으로 JdbcTemplate 정의**

```java
@Configuration
public class AppConfig {
    // 생략
    @Bean
    public JdbcTemplate jdbcTemplate(DataSource dataSource) {
        return new JdbcTemplate(dataSource);
    }
}
```





#### JdbcTemplate 클래스가 제공하는 주요 메서드

JdbcTemplate 클래스에는 데이터를 다루기 위한 다양한 메서드가 준비돼 있다. 그중에서도 많이 활용되는 주요 메서드 몇 가지를 이번 절에서 설명한다.



**JdbcTemplate의 주요 메서드**

| 메서드명           | 설명                                                         |
| ------------------ | ------------------------------------------------------------ |
| **queryForObject** | 하나의 결과 레코드 중에서 하나의 칼럼 값을 가져올 때 사용함<br />**RowMapper**와 함께 사용하면 하나의 레코드 정보를 객체에 매핑할 수 있음 |
| **queryForMap**    | 하나의 결과 레코드 정보를 **Map** 형태로 매핑할 수 있음      |
| **queryForList**   | 여러 개의 결과 레코드를 다룰 수 있음<br />**List**의 한 요소가 한 레코드에 해당<br />한 레코드의 정보는 **queryForObject**나 **queryForMap**을 사용할 때와 같은 |
| **query**          | **ResultSetExtractor**, **RowCallbackHandler**와 함께 조회할 때 사용함 |
| **update**         | 데이터를 변경하는 SQL(예: **INSERT, DELETE, UPDATE**)을 실행할 때 사용함 |





#### 샘플 애플리케이션

스프링 JDBC의 기본적인 사용법을 살펴보기 위해 간단한 애플리케이션을 예로 들어 살펴보자. 이 샘플 애플리케이션은 회의실 예약 기능을 제공한다고 가정한다.



**스키마 정보**

이 예에서 사용할 테이블 스키마는 다음과 같다. 참고로 **room** 테이블과 **equipment** 테이블은 1:다 관계다.



**room(회의실) 테이블**

| 컬럼        | 타입                 | 비고 |
| ----------- | -------------------- | ---- |
| room_id(pk) | VARCHAR(10) NOT NULL |      |
| room_name   | VARCHAR(30) NOT NULL |      |
| capacity    | INT NOT NULL         |      |



**equipment(시설) 테이블**

| 컬럼              | 타입                 | 비고                 |
| ----------------- | -------------------- | -------------------- |
| equipment_id(pk)  | VARCHAR(10) NOT NULL |                      |
| room_id(pk)       | VARCHAR(10) NOT NULL | 외래키 room(room_id) |
| equipment_name    | VARCHAR(30) NOT NULL |                      |
| equipment_count   | INT NOT NULL         |                      |
| equipment_remarks | VARCHAR(100)         |                      |





**초기 적재 데이터**

테이블에 저장할 초기 적재 데이터는 다음과 같다.



**room(회의실) 테이블**

| room_id | room_name   | capacity |
| ------- | ----------- | -------- |
| A001    | 임원 회의실 | 10       |
| C001001 | 세미나 룸   | 30       |
| X9999   | 콘퍼런스 룸 | 100      |



**equipment(시설) 테이블**

| equipment_id | room_id | equipment_name   | equipment_count | equipment_remarks |
| ------------ | ------- | ---------------- | --------------- | ----------------- |
| 10-1         | A001    | 화상 회의 시스템 | 1               |                   |
| 20-1         | A001    | 프로젝터         | 1               | 고정형            |
| 40-500       | C001001 | 화상 회의용 PC   | 10              |                   |
| 20-2         | C001001 | 프로젝터         | 5               | 이동형            |
| 30-1         | C001001 | 화이트보드       | 6               | 이동형            |





#### 칼럼 값을 조회할 때 자바 표준 타입에 담아오기

우선 가장 간단한 방법으로 데이터를 조회해보면서 **JdbcTemplate**이 어떻게 사용되는지 확인해보자. 이 예에서 사용되는 SQL 문에는 바인드 변수(Bind Variable)를 사용하지 않았으며, 조회 결과로는 단 하나의 레코드 중 단 하나의 칼럼 값만 꺼내오게 했다.





**DAO 클래스 구현**

DB에 접근해서 데이터를 다루게 될 **DAO(Data Access Object)** 클래스를 정의해보자. 여기서는 **JdbcTemplate** 클래스를 사용하기 위해 스프링 프레임워크가 초기화해둔 것을 **@Autowired** 애너테이션을 주입받아 사용한다. 이후에도 특별한 경우가 아니라면 이렇게 주입받은 **JdbcTemplate** 클래스를 사용하겠다. 이어서 데이터를 다루는 처리 내용을 **DAO** 클래스의 메서드에 구현해보자.



- **JdbcTemplate을 사용하는 DAO 클래스 구현**

```java
@Component
public class JdbcRoomDao {
    @Autowired
    JdbcTemplate jdbcTemplate;
    
    public int findMaxCapacity() { 
    	String sql = "SELECT MAX(capacity) FROM room";
        return jdbcTemplate.queryForObject(sql, Integer.class);
    }
}
```







**스프링 JDBC 설정**

**JdbcTemplate** 클래스를 사용할 때는 **dataSource** 프로퍼티에 앞서 만들어둔 **DataSource**를 설정해야 한다. DI 컨테이너는 컴포넌트 스캔을 하는 과정에서 **@Component** 애너테이션이 붙은 **DAO** 클래스를 발견하게 되고, 프로퍼티에 **@Autowired**로 설정된 **JdbcTemplate** 클래스를 주입하게 된다.



- **XML 기반 설정 방식으로 JdbcTemplate 정의**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="
           http://www.springframework.org/schema/beans 
           http://www.springframework.org/schema/beans/spring-beans.xsd
           http://www.springframework.org/schema/context
           http://www.springframework.org/schema/context/spring-context.xsd">
    
    <import resource="classpath:datasource-embedded.xml" />
    
    <context:component-scan base-package="com.example" />

    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
        <property name="dataSource" ref="dataSource" />
    </bean>
</beans>
```





**DAO 클래스의 동작 확인**

이제 지금까지 만들어진 것을 확인해보자. DI 컨테이너 혹은 ApplicationContext에서 DAO 빈을 찾아온 다음, 미리 만들어둔 메서드를 실행하기만 하면 내부에서 **JdbcTemplate** 빈의 도움을 받아 데이터를 손쉽게 조회할 수 있다.



- **DAO 클래스를 이용한 데이터 조회**

```java
public static void main(String[] args) {
    ApplicationContext context = new ClassPathXmlApplicationContext("JdbcTemplateConfig.xml");
    JdbcRoomDao dao = context.getBean("jdbcRoomDao", JdbcRoomDao.class);
    int maxCapacity = dao.findMaxCapacity();
    System.out.println(maxCapacity);
}
```







#### 칼럼 값을 조회할 때 바인드 변수 사용

앞서 살펴본 예에서는 내용에 변화가 없는 **정적인** SQL 문을 사용했다. 이와 달리 데이터를 다루다 보면 SQL 문의 일부분을 반복적으로 변경해야 하는 경우가 있다. 이럴 때는 SQL 문의 **가변적**인 부분에 '?'와 같은 플레이스홀더를 두고, SQL을 실행하는 시점에 변수로 치환하는 방법을 사용할 수 있는데, 이 변수를 **바인드 변수**라고 한다. **JdbcTemplate** 클래스는 이러한 바인드 변수가 사용된 SQL도 무리 없이 실행할 수 있는 기능을 제공한다.





- **바인드 변수를 이용한 데이터 조회**

```java
public String findRoomNameById(String roomId) {
    String sql = "SELECT room_name FROM room WHERE room_id = ?";
    return jdbcTemplate.queryForObject(sql, String.class, roomId);
    
    /*
    파라미터를 배열로 전달할 때는 다음과 같은 방법을 사용한다.
    Object[] args = new Object[] { roomId };
    return jdbcTemplate.queryForObject(sql, args, String.class);
    */
}
```







#### 칼럼 값을 조회할 때 네임드 파라미터 사용

앞서 설명한 바인드 변수는 플레이스홀더에 '?' 문자를 사용한다. 그래서 실제로 파라미터를 지정할 때는 <u>바인딩할 순서가 틀리지 않도록</u> 상당한 주의가 필요한데, 심지어 바인딩할 파라미터의 개수가 많아지면 많아질수록 가독성도 떨어진다는 단점이 있다. 이러한 단점을 보완하는 파라미터 설정 방법이 있는데, 이때 사용하는 것이 **네임드 파라미터**다. 다음 예를 보면서 **네임드 파라미터**를 어떻게 활용하는지 확인해보자.





- **NamedParameterJdbcTemplate을 이용한 DAO 클래스 구현**

```java
@Component
public class JdbcRoomNameDao {
    @Autowired
    NamedParameterJdbcTemplate namedParameterJdbcTemplate;
    
    public String findRoomNameById(String roomId) {
        String sql = "SELECT room_name FROM room WHERE room_id = :roomId";
        Map<String, Object> params = new HashMap<String, Object>();
        params.put("roomId", roomId);
        return namedParameterJdbcTemplate.queryForObject(sql, params, String.class);
    }
}
```





- **XML 기반 설정 방식으로 NamedParameterJdbcTemplate 정의**

```xml
<bean id="namedParameterJdbcTemplate"
      class="org.springframework.jdbc.core.namedparam.NamedParameterJdbcTemplate">
	<constructor-arg ref="dataSource" />
</bean>
```





**SqlParameterSource를 활용한 파라미터 설정**

앞에서 살펴본 예에서는 **NamedParameterJdbcTemplate** 메서드에 바인드 변수의 이름을 키로 사용하는 **Map**을 활용했다. **NamedParameterJdbcTemplate** 메서드는 **Map** 외에도 **SqlParameterSource** 인터페이스(org.springframework.jdbc.core.namedparam.SqlParameterSource)의 구현 클래스를 인수로 받을 수 있는데 **SqlParameterSource** 인터페이스의 구현 클래스를 사용하면 **Map**을 사용할 때보다 파라미터 설정을 비교적 쉽게 할 수 있다는 장점이 있다. 다음은 **SqlParameterSource** 인터페이스의 대표적인 구현체다.



- **org.springframework.jdbc.core.namedparam.MapSqlParameterSource**

  **MapSqlParameterSource** 객체의 **addValue** 메서드를 사용해 파라미터 값을 설정할 수 있다. 이때 반환값으로 다시 **MapSqlParameterSource** 객체가 나오는데, 이 객체는 앞서 추가한 파라미터가 설정된 **MapSqlParameterSource** 객체이다. 다른 파라미터를 더 추가하고 싶다면 이 **MapSqlParameterSource** 객체의 **addValue** 메서드를 사용하면 된다. 이렇게 메서드 호출 후의 반환값에 다시 메서드를 이어 호출하는 방식을 반복하면 모든 파라미터를 연속으로 추가하는 **메서드 체인(method chain)**을 구현할 수 있다. 

  

  ```java
  MapSqlParameterSource map = new MapSqlParameterSource()
  									.addValue("roomId", "A001")
  									.addValue("roomName", "임원 회의실")
  									.addValue("capacity", 10);
  ```



- **org.springframework.jdbc.core.namedparam.BeanPropertySqlParameterSource**

  SQL 문의 파라미터 값을 설정할 때 빈 객체를 활용할 수 있다. 우선 빈의 프로퍼티 값에 SQL 파라미터 값을 채운 다음, 이 빈을 **BeanPropertySqlParameterSource** 클래스의 생성자로 전달하면 된다. 이 방법은 빈의 프로퍼티명이 SQL의 파라미터명과 일치할 때 사용할 수 있다. 

  

  ```java
  Room room = new Room("A001", "임원 회의실", 10);
  BeanPropertySqlParameterSource map = new BeanPropertySqlParameterSource(room);
  ```

  





#### 조회 결과 레코드 값이 1건인 경우

기본키(PK)로 검색할 때 처럼 단 한 건의 결과가 나올 때, 결괏값을 어떻게 가져와야 하는지 살펴보자. **JdbcTemplate**으로 결괏값을 담아올 때는 사용자가 원하는 빈 형태로 가져오지 못하고 자바에서 기본적으로 제공하는 타입만 써야 한다. 그래서 조회 결과는 칼럼명을 키로 사용하는 **Map** 형태로 받게 되고, 값을 꺼낼 때는 적절한 타입으로 형변환해야 한다.



- **DAO 클래스로 1건을 조회하는 구현 예**

```java
public Room getRoomById(String roomId) {
	String sql = "SELECT room_id, room_name, capacity FROM room WHERE room_id = ?";
    Map<String, Object> result = jdbcTemplate.queryForMap(sql, roomId);
    Room room = new Room();
    room.setRoomId((String) result.get("room_id"));
    room.setRoomName((String) result.get("room_name"));
    room.setCapacity((Integer) result.get("capacity"));
    
    return room;
}
```





#### 조회 결과 레코드 값이 여러 건인 경우

다음은 조회한 결과가 여러 건인 경우를 살펴보자. 앞서 살펴본 것처럼 **JdbcTemplate**은 한 레코드의 결과에 대해 **Map** 타입으로 리턴하는데, 여러 건의 레코드가 나올 때는 **Map**을 여러 개 담은 **List** 타입의 결괏값이 나온다.



- **DAO 클래스에서 여러 건 조회를 구현한 예**

```java
public List<Room> getAllRoom() {
	String sql = "SELECT room_id, room_name, capacity FROM room";
    List<Map<String, Object>> resultList = jdbcTemplate.queryForList(sql);
    List<Room> roomList = new ArrayList<Room>();
    for(Map<String, Object> result: resultList) {
        Room room = new Room();
        room.setRoomId((String) result.get("room_id"));
        room.setRoomName((String) result.get("room_name"));
        room.setCapacity((Integer) result.get("capacity"));
        roomList.add(room);
    }
    return roomList;
}
```





#### 조회 결과 레코드 값이 0건인 경우

**JdbcTemplate**은 조회 결과가 0건이 나오면 **queryForList** 메서드는 항목의 개수가 0개인 빈 **List**를 반환하고 그 밖의 메서드에서는 찾는 데이터가 없다는 의미로 **EmptyResultDataAccessException** 예외(org.springframework.dao.EmptyResultDataAccessException)를 던지도록 돼 있다. 그래서 **JdbcTemplate**을 이용한 데이터 조회 로직을 구현할 때는 데이터가 존재한다고 가정하고 구현하면 된다. 단, **ResultSetExtractor**를 사용하는 경우는 조금 다를 수 있는데 이 부분은 뒤에서 다시 설명하겠다.





#### 테이블 내용을 변경하는 경우

테이블의 내용을 변경해야 할 때 어떻게 처리하는지 살펴보자. **JdbcTemplate**에서는 테이블을 변경하는 처리 내용이 등록이냐, 수정이냐, 삭제냐에 상관없이 모든 경우에 **update** 메서드를 사용한다. 그리고 앞서 살펴본 것처럼 조회할 때 결과가 0건이면 **EmptyResultDataAccessException** 예외가 발생한 반면, 데이터를 변경할 때는 변경 대상이 0건인 경우라도 예외를 발생시키지 않는다. 데이터를 변경한 후에는 변경된 처리 건수가 반환되므로 변경 작업 성공 여부는 이 반환값으로 판단하면 된다.



- **DAO 클래스에서 데이터 변경을 구현한 예**

```java
@Autowired
JdbcTemplate jdbcTemplate;

public int insertRoom(Room room) {
    String sql = "INSERT INTO room(room_id, room_name, capacity) VALUES (?, ?, ?)";
    return jdbcTemplate.update(sql, room.getRoomId(), room.getRoomName(), room.getCapacity());
}

public int updateRoomById(Room room) {
    String sql = "UPDATE room SET room_name=?, capacity=? WHERE room_id=?";
    return jdbcTemplate.update(sql, room.getRoomName(), room.getRoomCapacity(), room.getRoomId());
}

public int deleteRoomById(String roomId) {
    String sql = "DELETE FROM room WHERE room_id=?";
    return jdbcTemplate.update(sql, roomId);
}
```





---

### 3.2.3 SQL 질의 결과를 POJO로 변환

앞에서 설명한 것처럼 **스프링 JDBC**는 처리 결괏값을 자바가 기본적으로 제공하는 데이터 타입이나 **Map, List** 같은 컬렉션 타입으로 반환한다. 보통 애플리케이션을 개발할 때는 자바에서 제공하는 타입이 아니라 해당 비즈니스에 맞는 데이터 타입을 **POJO** 형태로 만들어 쓰는 경향이 있기 때문에 반환값을 가공해야 할 수 있다. **스프링 JDBC**에서는 기본적으로 반환된 결괏값을 원하는 형태로 변환하기 쉽도록 다음과 같은 세 가지 인터페이스를 제공한다.



- **RowMapper**

  **RowMapper** 인터페이스는 **JDBC**의 **ResultSet**을 순차적으로 읽으면서 원하는 **POJO** 형태로 매핑하고 싶을 때 사용한다. 그래서 이 인터페이스를 사용하면 **ResultSet**의 한 행(row)을 읽어 하나의 **POJO** 객체로 변환할 수 있다. 뒤에 나오는 **ResultSetExtractor**와의 차이점은 <u>**ResultSet**의 한 행이 하나의 **POJO** 객체로 변환되고, **ResultSet**이 다음 행으로 넘어가는 커서 제어를 개발자가 직접하지 않고 스프링 프레임워크가 대신 해준다</u>는 점이다.



- **ResultSetExtractor**

  **ResultSetExtractor** 인터페이스는 **JDBC**의 **ResultSet**을 자유롭게 제어하면서 원하는 **POJO** 형태로 매핑하고 싶을 때 사용한다. <u>**RowMapper**와의 차이점은 **ResultSet**의 여러 행 사이를 자유롭게 이동할 수 있다는 점</u>이다. **RowMapper**는 **ResultSet**에서 한 행씩만 참조할 수 있어서 다음 행으로 커서를 이동하는 **next** 같은 메서드를 제공하지 않는다. 그래서 **ResultSetExtractor**를 사용하면 **ResultSet**의 여러 행에서 필요한 값을 꺼내서 하나의 **POJO** 객체에 채워넣을 수 있다.



- **RowCallbackHandler**

  **RowCallbackHandler** 인터페이스의 메서드는 앞서 살펴본 **RowMapper**나 **ResultSetExtractor**와 달리 반환값이 없다. 그래서 **JDBC**의 **ResultSet** 정보를 처리 결과를 반환하기 위해 사용하는 것이 아니라 별도의 다른 처리를 하고 싶을 때 사용한다. 예를 들면, **ResultSet**에서 읽은 데이터를 파일 형태로 출력하거나, 조회된 데이터를 검증하는 용도로 활용할 수 있다.





#### RowMapper 인터페이스 구현

이제 앞서 살펴본 RowMapper 인터페이스를 구현하는 방법을 알아보자.



**RowMapper 구현 클래스 만들기**

**RowMapper** 인터페이스를 구현하는 클래스를 만든 다음, **mapRow** 메서드 안에서 **ResultSet**의 값을 읽어 **POJO**를 생성하게 한다. 만약 검색 결과가 0건인 경우에는 **mapRow** 메서드가 호출되기도 전에 스프링 프레임워크가 **EmptyResultDataAccessException** 예외를 발생시킬 것이다. 그래서 변환 처리 부분은 조회 결과가 반드시 1건 이상 존재한다고 가정하고 구현하면 된다.



- **RowMapper 구현 클래스**

```java
public class RoomRowMapper implements RowMapper<Room> {
	@Override
    public Room mapRow(ResultSet rs, int rowNum) throws SQLException {
        Room room = new Room();
        room.setRoomId(rs.getString("room_id"));
        room.setRoomName(rs.getString("room_name"));
        room.setCapacity(rs.getInt("capacity"));
        return room;
    }
}
```





**RowMapper를 활용한 DAO 클래스 만들기**

**DAO** 클래스에서 **JdbcTemplate** 클래스의 메서드를 호출할 때는 **RowMapper** 인터페이스의 구현 클래스를 인자로 전달한다.



- **RowMapper를 활용한 DAO 클래스 구현**

```java
public Room getRoomById(String roomId) {
	String sql = "SELECT room_id, room_name, capacity FROM room WHERE room_id = ?";
    RoomRowMapper rowMapper = new RoomRowMapper();
    return jdbcTemplate.queryForObject(sql, rowMapper, roomId);
}

public List<Room> getAllRoom() {
    String sql = "SELECT room_id, room_name, capacity FROM room";
    RoomRowMapper rowMapper = new RoomRowMapper();
    return jdbcTemplate.query(sql, rowMapper);
}
```





**람다 식을 활용한 RowMapper 구현 클래스 만들기**

Java SE 8부터 지원하는 **람다 식(lambda expression)**을 활용해 DAO 클래스를 구현할 수도 있다. 람다 식을 쓰는 경우에는 **RowMapper**가 할 일을 DAO 클래스에 직접 작성할 수 있기 때문에 **RowMapper** 인터페이스를 따로 구현할 필요가 없다.



- **람다 식을 활용한 RowMapper 구현 클래스**

```java
public List<Room> getAllRoom() {
	String sql = "SELECT room_id, room_name, capacity FROM room";
    return jdbcTemplate.query(sql, (rs, rowNum) -> {
        Room room = new Room();
        room.setRoomId(rs.getString("room_id"));
        room.setRoomName(rs.getString("room_name"));
        room.setCapacity(rs.getString("capacity"));
        return room;
    });
}
```





#### BeanPropertyRowMapper를 활용한 DAO 클래스 만들기

질의 결과를 **POJO**로 변환하는 방법으로 앞서 살펴본 것처럼 **RowMapper** 인터페이스를 직접 구현하는 방법 외에도 미리 만들어져 있는 **BeanPropertyRowMapper** 클래스를 활용하는 방법이 있다. **BeanPropertyRowMapper**를 이용하면 미리 약속된 매핑 규칙에 따라 질의 결과 정보를 **POJO**로 자동 매핑할 수 있다. 다만 자동 매핑을 위해 자바의 **리플렉션(reflection)** 기능을 사용하기 때문에 처리 과정에서 약간의 성능을 희생해야 한다. 만약 자동 매핑보다 성능이 우선시되는 상황이라면 앞서 설명한 바와 같이 **RowMapper** 인터페이스를 직접 구현하는 방법을 권장한다. 한편 자동 매핑이 되기 위해 지켜야 하는 **규칙과 제약 사항**은 다음과 같다.



**매핑 규칙**

- **ResultSet의 칼럼 이름과 매핑 대상인 타깃(target) 클래스의 프로퍼티 이름을 매핑한다.**
- **칼럼 이름을 언더스코어(underscore) 문자 '_'로 구분한 다음, 낙타 표기법(camelcase)으로 조합한 것과 타깃 클래스의 프로퍼티 이름이 일치하면 칼럼 값을 매핑한다.**
- **String, boolean, Boolean, byte, Byte, short, Short, int, Integer, long, Long, float, Float, double, Double, BigDecimal, java.util.Date** 등의 기본적인 데이터 타입을 지원한다.



**제약 사항**

- **매핑될 타깃 클래스는 다른 클래스에 포함되지 않은 최상위 클래스여야 한다.**
- **매핑될 타깃 클래스에는 기본 생성자나 인수가 없는 생성자가 있어야 한다.**







- **BeanPropertyRowMapper를 활용한 DAO 클래스**

```java
public Room getRoomUseBeanPropertyById(String roomId) {
    String sql = "SELECT room_id, room_name, capacity FROM room WHERE room_id = ?";
    RowMapper<Room> rowMapper = new BeanPropertyRowMapper<Room>(Room.class);
    return jdbcTemplate.queryForObject(sql, rowMapper, roomId);
}
```





#### ResultSetExtractor 인터페이스 구현

이번에는 **ResultSetExtactor** 인터페이스를 구현하는 방법을 알아보자. 이 예는 두 개의 테이블을 왼쪽 외부 조인(left outer join)한 결과에 맞춰 두 개의 타깃 클래스를 매핑하되, 한 클래스가 다른 클래스를 포함하는 **중첩 클래스 형태**로 표현하는 방법을 보여준다. 이해를 돕기 위해 다음 그림을 살펴보자.



![](https://docs.google.com/drawings/d/sbDTUP5JEpXeYMpsbJkE70Q/image?parent=e/2PACX-1vSgWs9-SkKDVDplyC96i2vj5TuZvbmLHvACG_NMQ3eEa1hFtJwCGbFZVRaHriS3HOeS8O0XIQ6gqqII&rev=266&h=270&w=601&ac=1)



**ResultSetExtractor 구현 클래스 만들기**

**ResultSetExtractor** 인터페이스를 구현한 클래스를 만든 다음, **extractData** 메서드에서 **ResultSet**의 값을 읽어 타깃 객체에 매핑한다.



- **ResultSetExtractor 구현 클래스**

```java
public class RoomListResultSetExtractor implements ResultSetExtractor<List<Room>> {
    
    @Override
    public List<Room> extractData(ResultSet rs) throws SQLException, DataAccessException {
        Map<String, Room> map = new LinkedHashMap<String, Room>();
        Room room = null;
        
        while(rs.next()) {
            String roomId = rs.getString("room_id");
            room = map.get(roomId);
            if (room == null) {
                room = new Room();
                room.setRoomId(roomId);
                room.setRoomName(rs.getString("room_name"));
                room.setCapacity(rs.getInt("capacity"));
                map.put(roomId, room);
            }
            
            String equipmentId = rs.getString("equipment_id");
            
            if(equipmentId != null) {
                Equipment equipment = new Equipment();
                equipment.setEquipmentId(equipmentId);
                equipment.setRoomId(roomId);
                equipment.setEquipmentName(rs.getString("equipment_name"));
                equipment.setEquipmentCount(rs.getInt("equipment_count"));
                equipment.setEquipmentRemarks(rs.getString("equipment_remarks"));
                room.getEquipmentList().add(equipment);
            }
        }
        
        if(map.size() == 0) {
            throw new EmptyResultDataAccessException(1);
        }
        
        return new ArrayList<Room>(map.values());
    }
}
```





**ResultSetExtractor를 활용한 DAO 클래스 만들기**

**DAO** 클래스에서 **JdbcTemplate** 클래스의 메서드를 호출할 때는 **ResultSetExtractor** 인터페이스의 구현 클래스를 인자로 전달한다.



- **ResultSetExctractor를 활용한 DAO 클래스**

```java
public List<Room> getAllRoomWithEquipment() {
	String sql = "SELECT r.room_id, r.room_name, r.capacity," +
        " e.equipment_id, e.equipment_name, e.equipment_count," +
        " e.equipment_remarks FROM room r LEFT JOIN equipment e" +
        " ON r.room_id = e.room_id";
    
    RoomListResultSetExtractor extractor = new RoomListResultSetExtractor();
    return jdbcTemplate.query(sql, extractor);
}

public Room getRoomWithEquipmentById(String roomId) {
    String sql = "SELECT r.room_id, r.room_name, r.capacity," +
        " e.equipment_id, e.equipment_name, e.equipment_count," +
        " e.equipment_remarks FROM room r LEFT JOIN equipment e" +
        " ON r.room_id = e.room_id WHERE r.room_id = ?";
    
    RoomListResultSetExtractor extractor = new RoomListResultSetExtractor();
    List<Room> roomList = jdbcTemplate.query(sql, extractor, roomId);
    return roomList.get(0);
}
```



 

#### RowCallbackHandler 인터페이스 구현

마지막으로 **RowCallbackHandler** 인터페이스를 구현하는 방법을 알아보자. 다음은 **room** 테이블에 등록된 모든 데이터를 **CSV** 파일로 출력(export)하는 예다.



**RowCallbackHandler 구현 클래스 만들기**

**RowCallbackHandler** 인터페이스를 구현한 클래스를 만든 다음, **processRow** 메서드 안에서 **ResultSet**의 값을 읽고 **CSV** 파일을 만들도록 구현한다.



- **RowCallbackHandler 구현 클래스**

```java
public class RoomRowCallbackHandler implements RowCallbackHandler {
    @Override
    public void processRow(ResultSet rs) throws SQLException {
        try (BufferedWriter writer = new BufferedWriter(new OutputStreamWriter(
        	new FileOutputStream(File.createTempFile("room_", ".csv")), "UTF-8"))) {
            
            while(rs.next()) {
                Object[] array = new Object[] {
                    rs.getString("room_id"),
                    rs.getString("room_name"),
                    rs.getInt("capacity")
                };
                
                String reportRow = StringUtils.arrayToCommaDelimitedString(array);
                writer.write(reportRow);
                writer.newLine();
            }
        } catch(IOException e) {
            throw new SQLException(e);
        }
    }
}
```





**RowCallBackHandler를 활용해 DAO 클래스 만들기**

**DAO** 클래스에서 **JdbcTemplate** 클래스의 메서드를 호출할 때는 **RowCallbackHandler** 인터페이스의 구현 클래스를 인자로 전달한다.



- **RowCallbackHandler를 활용한 DAO 클래스**

```java
public void reportRoom() {
    String sql = "SELECT room_id, room_name, capacity FROM room";
    RoomRowCallbackHandler handler = new RoomRowCallbackHandler();
    jdbcTemplate.query(sql, handler);
}
```







### 3.2.4 데이터 일괄 처리

지금까지 **JdbcTemplate**과 **NamedParameterJdbcTemplate**을 활용한 기본적인 **CRUD** 연산을 구현하는 방법을 살펴봤다. 한편 엔터프라이즈 애플리케이션을 개발할 때는 단순한 **CRUD** 처리 외에도 SQL을 **배치(batch)** 방식으로 실행하거나 **저장 프로시저(stored procedure)**도 많이 활용한다. 이 책에서는 지면 관계상 깊게 다루지는 못하고 각각에 대해 간단히 소개만 한다.



#### 배치 처리

대량의 데이터를 등록하거나 갱신해야 하는 경우 SQL 문을 각각 실행하는 것이 아니라 **배치 형태**로 모아서 실행하는 방법이 있다. **스프링 JDBC**를 사용한다면 **JdbcTemplate** 클래스나 **NamedParameterJdbcTemplate** 클래스에 **batchUpdate** 메서드가 있으니 이를 활용하면 된다.



#### 저장 프로시저 호출

**저장 프로시저**와 **저장 함수(stored function)**는 **JdbcTemplate** 클래스의 **call** 메서드와 **execute** 메서드로 실행할 수 있다. 한편 **JdbcTemplate** 클래스보다 좀 더 세련된 **API**를 제공하는 **Stored Procedure** 클래스와 **SimpleJdbcCall** 클래스도 제공되니 상황에 맞게 적절한 것을 골라 쓰면 된다.





## 3.3 트랜잭션 관리

지금까지 스프링 JDBC를 활용한 기본적인 데이터 접근 방식을 알아봤는데, 업무 애플리케이션을 개발하는데 반드시 필요한 트랜잭션에 대해서는 굳이 언급하지 않았다.



이번 절에서는 스프링에서 제공하는 데이터 소스를 가지고 **트랜잭션**을 관리하는 방법을 알아보자. 우선 **애너테이션**을 활용해 트랜잭션을 관리하는 **선언적(declarative)** 방법을 설명하고, 이후 소스코드 안에 직접 **commit** 메서드나 **rollback** 메서드로 트랜잭션을 관리하는 **프로그램적인(programmactic)** 방법을 설명한다. 마지막으로 **트랜잭션의 경계와 동작 방식**을 결정 짓는 **트랜잭션 격리 수준과 전파 방식**에 대해서도 살펴보겠다.



### 3.3.1 트랜잭션 관리자

관계형 데이터베이스에 데이터에 접근할 때는 <u>트랜잭션의 경계가 어디까지이고 어떻게 관리되는지</u> 이해하고 있어야 한다. 하지만 실제로 트랜잭션을 염두에 두고 관련 코드를 구현하는 것은 상당히 힘들고 번거로운 일이 될 수 있는데, 다행히 스프링 프레임워크에서는 이러한 처리를 비교적 쉽게 구현하도록 도와주는 기능이 있다. 예를 들어, 트랜잭션 관리를 위한 코드를 비즈니스 로직에서 분리하기 위한 구조나 다른 트랜잭션을 투명하게 처리할 수 있게 하는 API 등을 들 수 있다.



스프링 트랜잭션 처리의 중심이 되는 인터페이스는 **PlatformTransactionManager**다. 이 인터페이스는 트랜잭션 처리에 필요한 API를 제공하며 개발자가 API를 호출하는 것으로 트랜잭션 조작을 수행할 수 있다(다만 다음에 설명하는 것처럼 일반적으로 **PlatformTransactionManager API**를 직접 호출하지 않고 스프링이 제공하는 편리한 API를 이용하는 사례가 더 많아지고 있다). 또한 **PlatformTransactionManager**는 트랜잭션 관리의 구현 방식을 추상화하기 위한 인터페이스이기 때문에 개발자는 서로 다른 종류의 트랜잭션을 사용하더라도 각각의 차이점을 의식할 필요없이 같은 API로 조작할 수 있다.



스프링 프레임워크는 다양한 환경과 제품에 대응하는 **PlatformTransactionManager**의 구현 클래스를 제공한다. 대표적인 구현 클래스는 다음과 같다.



**PlatformTransactionManager의 대표적인 구현 클래스**

| 클래스명                           | 설명                                                         |
| ---------------------------------- | ------------------------------------------------------------ |
| **DataSourceTransactionManager**   | JDBC 및 MyBatis 등의 JDBC 기반 라이브러리로 데이터베이스에 접근하는 경우에 이용한다. |
| **HibernateTransactionManager**    | 하이버네이트를 이용해 데이터베이스에 접근하는 경우에 이용한다. |
| **JpaTransactionManager**          | JPA로 데이터베이스에 접근하는 경우에 이용한다.               |
| **JtaTransactionManager**          | JTA에서 트랜잭션을 관리하는 경우에 이용한다.                 |
| **WebLogicJtaTransactionManager**  | 애플리케이션 서버인 웹로직(WebLogic)의 JTA에서 트랜잭션을 관리하는 경우에 이용한다. |
| **WebSphereUowTransactionManager** | 애플리케이션 서버인 웹스피어(WebSphere)의 JTA에서 트랜잭션을 관리하는 경우에 이용한다. |



#### 트랜잭션 관리자 정의

스프링 프레임워크의 트랜잭션 관리자를 사용할 때는 다음의 두 가지 작업을 해야 한다.

1. **PlatformTransactionManager의 빈을 정의한다.**
2. **트랜잭션을 관리해야 하는 메서드를 정의한다.**



여기서는 트랜잭션 관리자를 정의하는 방법을 설명한다.



#### 로컬 트랜잭션을 이용하는 경우

**기본적인 설정 방법**

로컬 트랜잭션을 사용하는 경우 JDBC API를 호출하고 트랜잭션 제어를 수행하는 **DataSourceTransactionManager**를 사용한다. 로컬 트랜잭션은 단일 데이터 저장소에 대한 트랜잭션으로 일반적으로 자주 사용되는 트랜잭션이다. 예를 들어, 단일 데이터 저장소에 대한 여러 조작을 하나의 논리적 단위로 처리하고 싶을 때 사용한다. 이 경우에는 **PlatformTransactionManager**의 구현 클래스로 **DataSourceTransactionManager**를 사용한다. 이때 **PlatformTransactionManager**의 빈 ID는 '**transactionManager**'를 사용하는 것이 좋다. 왜냐하면 스프링 프레임워크에서 기본적으로 트랜잭션 관리자의 빈 ID를 '**transactionManager**'로 가정하고 있기 때문이다. 그래서 이 관례를 따르면 각종 설정을 간단히 수행할 수 있다.



- **XML 기반 설정 방식을 이용한 빈 정의**

```xml
<bean id="transactionManager"
      class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
	<property name="dataSource" ref="dataSource" />
</bean>

<!-- @Transaction 애너테이션을 이용하는 경우 -->
<tx:annotation-driven />
```







**'transactionManager'가 아닌 다른 ID를 사용할 경우**

트랜잭션 관리자의 빈 ID에 '**transactionManager**'가 아닌 다른 ID를 사용한다면 **\<tx:annotation-driven>** 요소의 **transaction-manager** 속성에도 같은 ID를 설정해야 한다.



- **XML 기반 설정 방식을 이용한 빈 정의**

```xml
<bean id="txManager"
      class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
	<property name="dataSource" ref="dataSource" />
</bean>

<!-- @Transaction 애너테이션을 이용하는 경우 -->
<tx:annotation-driven transaction-manager="txManager" />
```





#### 글로벌 트랜잭션을 이용하는 경우

**글로벌 트랜잭션**은 여러 데이터 저장소에 걸쳐서 적용되는 트랜잭션이다. 예를 들어, 여러 데이터베이스를 사용하되, 각 데이터베이스에서 각각의 조작을 수행하고, 그 조작들을 하나의 트랜잭션으로 묶어 모두 성공하거나, 모두 실패한 것으로 처리해야 하는 경우에는 앞에서 설명한 **로컬 트랜잭션**을 사용할 수 없다. 이런 경우에는 **글로벌 트랜잭션**을 사용해야 하는데, **글로벌 트랜잭션**은 **JTA(Java Transaction API)**라는 Java EE 사양으로 표준화돼 있고 애플리케이션 서버가 **JTA**의 구현 클래스를 제공한다. **JTA**를 스프링 프레임워크에서 사용하려면 **PlatformTransactionManager**의 구현 클래스로 **JtaTransactionManager**를 선택하면 된다. 다만 애플리케이션 서버 제품에 따라 **JTA** 동작 방식이 조금씩 다를 수 있어서 제품별로 **JtaTransactionManager**의 서브클래스가 제공되기도 한다. 실제로 실행 환경의 애플리케이션 서버 제품에 따라 최적의 서브클래스를 자동으로 선택하는 기능이 제공된다.



- **XML 기반 설정 방식을 이용한 빈 정의**

```xml
<tx:jta-transaction-manager />

<!-- 
<tx:jta-transaction-manager />를 지정하면 애플리케이션 서버에서 제공하는 JtaTransactionManager를
빈 형태로 사용할 수 있다. 
-->
```





### 3.3.2 선언적 트랜잭션

**선언적 트랜잭션**은 미리 선언된 룰에 따라 트랜잭션을 제어하는 방법이다. **선언적 트랜잭션의 장점**은 정해진 룰을 준수함으로써 트랜잭션의 시작과 커밋, 롤백 등의 일반적인 처리를 비즈니스 로직 안에 기술할 필요가 없다는 점이다.



스프링 프레임워크는 선언적 트랜잭션을 이용하는 방법으로 **@Transactional** 또는 **XMl 설정**을 이용하는 두 가지 방법을 제공한다.





#### @Transactional을 이용한 선언적 트랜잭션

스프링 프레임워크가 제공하는 **@Transactional** 애너테이션을 빈의 public 메서드에 추가하는 것으로 대상 메서드의 시작 종료에 맞춰 트랜잭션을 시작, 커밋할 수 있다. 또한 스프링 프레임워크의 기본 상태에서는 메서드 안의 처리에서 데이터 접근 예외와 같은 **비검사 예외(unchecked exception)**가 발생해서 메서드 안에 처리가 중단될 때 트랜잭션이 자동으로 롤백된다.



기본 동작 방식을 변경하거나 상세한 설정을 하고 싶은 경우에는 **@Transactional** 속성을 변경할 수 있다.







#### 트랜잭션 제어에 필요한 정보

여기서는 **@Transactional** 애너테이션 속성, 즉 트랜잭션 제어에서 필요한 정보를 설명한다.



**트랜잭션 제어에서 필요한 정보**

| 속성명                     | 설명                                                         |
| -------------------------- | ------------------------------------------------------------ |
| **value**                  | 여러 트랜잭션 관리자를 이용하는 경우, 이용하는 트랜잭션 관리자의 qualifier를 지정한다. 기본 트랜잭션 관리자를 이용하는 경우에는 생략할 수 있다. |
| **transactionManager**     | value의 별칭                                                 |
| **propagation**            | 트랜잭션의 전파 방식을 지정한다.                             |
| **isolation**              | 트랜잭션의 격리 수준을 지정한다.                             |
| **timeout**                | 트랜잭션 제한 시간(초)을 지정한다. 기본값은 -1(사용하는 데이터베이스의 사양이나 설정에 따라 다름)이다. |
| **readOnly**               | 트랜잭션의 읽기 전용 플래그를 지정한다. 기본값은 false(읽기 전용이 아님)다. |
| **rollbackFor**            | 여기에 지정한 예외가 발생하면 트랜잭션을 롤백시킨다. 예외 클래스명을 여러 개 나열할 수 있으며, ','로 구분한다. 따로 지정해주지 않았다면 RuntimeException과 같은 비검사 예외가 발생할 때 트랜잭션이 롤백된다. |
| **rollbackForClassName**   | 여기에 지정한 예외가 발생하면 트랜잭션을 롤백시킨다. 예외 이름을 여러 개 나열할 수 있으며, ','로 구분한다. |
| **noRollbackFor**          | 여기에 지정한 예외가 발생하더라도 트랜잭션을 롤백시키지 않는다. 예외 클래스명을 여러 개 나열할 수 있으며, ','로 구분한다. |
| **noRollbackForClassName** | 여기에 지정한 예외가 발생하더라도 트랜잭션을 롤백시키지 않는다. 예외 이름을 여러 개 나열할 수 있으며, ','로 구분한다. |





**@Transactional** 설정 예는 다음과 같다.

| **설정 예**                                                  | **설명**                                                     |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| **@Transactional**                                           | 기본 트랜잭션 관리자를 기본 설정에서 이용한다.               |
| **@Transactional(readOnly=true, timeout=60)**                | 기본 트랜잭션 관리자를 읽기 전용 트랜잭션으로 이용한다. 제한 시간은 60초로 변경한다. |
| **@Transactional("tx1")**                                    | "tx1" 트랜잭션 관리자를 이용한다.                            |
| **@Transactional(value="tx2", propagation=Propagation.REQUIRES_NEW)** | "tx2" 트랜잭션 관리자를, 전파 방식을 REQUIRES_NEW로 사용한다. |







#### 기본 사용법

**@Transactional** 기본 사용법을 설명한다.



**트랜잭션 관리 대상에 해당하는 메서드의 인터페이스**

일반적인 자바 인터페이스로 정의한다.

```java
public interface RoomService {
	Room getRoom(String roomId);
	void insertRoom(Room room);
}
```





**트랜잭션 관리 대상에 해당하는 메서드의 구현**

**@Transactional** 애너테이션은 클래스와 메서드에 부여할 수 있다. 차이는 애너테이션이 적용되는 범위다. 이 차이를 이해하기 위해 이번에는 클래스와 메서드에 모두 **@Transactional** 애너테이션을 부여해보자.



- **트랜잭션 관리 대상에 해당하는 메서드의 구현 예**

```java
@Transactional
@Service("roomService")
// 정의한 인터페이스의 구현 클래스를 정의한다. @Transactional 애너테이션을 클래스에 부여해서 이 클래스의 메서드 전체가 트랜잭션 적용 대상이 되게 한다. 컴포넌트로서 등록하기 위해 @Service 애너테이션을 클래스에 부여한다.
public class RoomServiceImpl implements RoomService {
	@Autowired
    JdbcRoomDao jdbcRoomDao; // 프로퍼티로 이용하는 DAO 클래스를 정의하고 주입받는다.
    
    @Transactional(readOnly=true)
    /* 정의한 인터페이스의 메서드를 구현한다. 메서드에 대해 @Transactional 애너테이션을 부여하면 메서드 단위에서 트랜잭션 		 제어 방식을 개별적으로 설정할 수 있다. 앞서 클래스 애너테이션에 @Transactional이 설정된 경우라도 메서드에 지정한          @Transactional 설정이 더 우선해서 적용된다. 
       애너테이션의 readOnly 속성에 true를 설정해 읽기 전용 트랜잭션으로 만든다. */
    @Override
    public Room getRoom(String roomId) {
        return jdbcRoomDao.getRoomById(roomId);
    }
    
    @Override 
    // 이 메서드에는 @Transactional이 설정돼 있지 않지만 
    // 이 클래스에 @Transactional 애너테이션이 활성화돼 있기 때문에 트랜잭션에 적용된다.
    public void insertRoom(Room room) {
        jdbcRoomDao.insertRoom(room);
        List<Equipment> equipmentList = room.getEquipmentList();
        for(Equipment item: equipmentList) {
            jdbcRoomDao.insertEquipment(item);
        }
    }
}
```





**설정 클래스**

여기서는 빈 정의 파일이 아닌 **설정 클래스**를 이용하는 방법을 설명한다. **설정 클래스**의 포인트는 **@EnableTransactionManager** 애너테이션을 **Configuration class**에 부여하는 것이다. 이렇게 하면 **@Transactional** 애너테이션을 사용한 트랜잭션 제어가 가능해진다.



- **자바 기반 설정 방식을 이용한 빈 정의**

```java
@Configuration
@EnableTransactionManagement
public class TransactionManagerConfig {
    @Autowired
    DataSource dataSource;
    
    @Bean
    public PlatformTransactionManager transactionManager() {
        return new DataSourceTransactionManager(dataSource);
    }
}
```





#### XML 설정을 이용한 선언적 트랜잭션

**@Transactional**을 사용할 때는 트랜잭션 관리 대상 메서드를 지정하고 트랜잭션을 설정하는 두 가지 작업을 모두 **@Transactional**에서 정의했었다. XML 설정을 이용한 선언적 트랜잭션에서는 이러한 처리를 모두 XML에서 정의한다.



XML 설정에서는 **\<tx:advice>** 요소와 같은 **tx**로 시작하는 **트랜잭션 전용 XML 스키마**를 사용한다. 아울러 트랜잭션 관리 대상이 되는 메서드를 규정하기 위해 **AOP** 기능을 이용한다.



**트랜잭션 관리 대상에 해당하는 메서드 구현**

처리 로직은 **@Transactional**을 이용한 선언적 트랜잭션과 같다. 차이점은 **@Transactional** 애너테이션이 어디에도 부여되지 않는다는 것이다. 그래서 서드파티에서 제공되는 클래스를 사용하는 경우와 같이 <u>개발자가 소스코드에 애너테이션을 부여할 수 없는 경우</u>에 XML 기반의 트랜잭션 설정 방식을 사용할 수 있다.



**빈 정의 파일**

**@Transactional**과 마찬가지로 **TransactionManager**의 빈을 정의한다. 그리고 **\<tx:advice>** 요소에서 트랜잭션 설정을 위한 Advice를 설정한 후, **AOP**를 사용해서 트랜잭션 관리를 해야 하는 메서드를 Pointcut 형태로 정의한다. 최종적으로 Advice와 Pointcut을 조합해 트랜잭션 제어를 위한 Advisor를 정의한다.



- **XML 기반 설정 방식을 이용한 빈 정의**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans
	xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:aop="http://www.springframework.org/schema/aop" ----------------- (1)
    xmlns:tx="http://www.springframework.org/schema/tx"   ----------------- (2)
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
    	http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop        -----------------  (1)
        http://www.springframework.org/schema/aop/spring-aop.xsd  --------  (1)
        http://www.springframework.org/schema/tx         -----------------  (2)
        http://www.springframework.org/schema/tx/spring-tx.xsd">  --------  (2)
    <import resource="classpath:datasource-embedded.xml" />
    
    <bean id="transactionManager"    --------------------------------------------------- (3)
          class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    	<property name="dataSource" ref="dataSource"/>
    </bean>                         
    
    <tx:advice id="txAdvice"> --------- (4)
    	<tx:attributes> ------------------------------------(5)
        	<tx:method name="get*" read-only="true" />
            <tx:method name="*" />
        </tx:attributes>
    </tx:advice>
    
    <aop:config> ------------------------- (6)
    	<aop:pointcut id="txPointcut"
                      expression="execution(* com.example.RoomServiceXmlImpl.*(..))" />
        <aop:advisor advice-ref="txAdvice" pointcut-ref="txPointcut"/>
    </aop:config>
    
    <bean id="roomService" class="com.example.RoomServiceXmlImpl">      ------------ (7)
    	<property name="roomDao" ref="jdbcRoomDao" /> 
    </bean>
    
    <bean id="jdbcRoomDao" class="com.example.JdbcRoomDao">
    	<property name="dataSource" ref="dataSource" />
    </bean>
</beans>
```



**(1), (2)**: AOP 기능을 이용하기 위해 네임스페이스와 스키마에 aop, tx를 추가한다.

**(3)**: org.springframework.jdbc.datasource.DataSourceTransactionManager 클래스의 빈을 정의한다.

**(4)**: \<tx:advice> 요소를 이용해 트랜잭션 정의에 관한 Advice를 정의한다.

**(5)**: \<tx:attributes> 요소를 이용해 트랜잭션 관리 대상이 되는 메서드 정의 및 트랜잭션 설정을 수행한다. 여기서는 메서드 명이 'get'으로 시작하는 것은 읽기 전용 트랜잭션, 그 밖에는 쓰기 가능한 트랜잭션으로 정의한다. 기본적으로는 비검사 예외가 발생할 때 롤백하고, 검사 예외가 발생하면 롤백하지 않는다. 만약 검사 예외 중에 롤백시켜야 하는 예외가 있다면 rollback-for 속성에 해당 예외 클래스명을 지정하면 된다.

**(6)**: \<aop:config> 요소를 이용해 Pointcut과 Advisor를 정의한다. Pointcut은 인터페이스 구현 클래스의 메서드를 대상으로 한다. Advisor는 **(4)**에서 정의한 Advice와 여기서 정의한 Pointcut을 조합한다.

**(7)**: 인터페이스 구현 클래스의 빈을 정의한다.





### 3.3.3 명시적 트랜잭션

**명시적 트랜잭션**은 커밋이나 롤백과 같은 트랜잭션 처리를 소스코드에 직접 명시적으로 기술하는 방법이다. **메서드 단위보다도 더 작은 단위**로 트랜잭션을 제어하고 싶거나 선언적 트랜잭션으로는 표현하기 어려운 **섬세한 트랜잭션 제어**가 필요할 때 이 방법을 사용한다. JDBC API를 직접 이용하는 기술도 명시적 트랜잭션이다. 스프링 프레임워크에서는 명시적 트랜잭션을 이용하는 방법으로 **PlatformTransactionManager**와 **TransactionTemplate**을 사용하는 두 가지 방법을 제공한다.



#### PlatformTransactionManager를 이용한 명시적 트랜잭션 제어

**PlatformTransactionManager**를 이용하면 트랜잭션을 직접 제어할 수 있다. 이 경우에는 **TransactionDefinition** 및 **TransactionStatus**를 이용해 트랜잭션의 시작과 커밋, 그리고 롤백을 명시적으로 처리하게 된다.



**Service의 구현**

- **PlatformTransactionManager를 이용한 트랜잭션 제어 구현 예**

```java
@Service
public class RoomServiceImpl implements RoomService {
    
    @Autowired
    PlatformTransactionManager txManager;
    @Autowired
    JdbcRoomDao roomDao;
    
    @Override
    public void insertRoom(Room room) {
        DefaultTransactionDefinition def =
            new DefaultTransactionDefinition();
        def.setName("InsertRoomWithEquipmentTx");
        def.setReadOnly(false);
        def.setPropagationBehavior(TransactionDefinition.PROPAGATION_REQUIRED);
        TransactionStatus status = txManager.getTransaction(def);
        // getTransaction 메서드를 실행한 이후의 처리가 트랜잭션 범위가 된다. 
        // getTransaction 메서드의 반환값은 TransactionStatus 객체로서 
        // 이 TransactionStatus를 이용해 트랜잭션 커밋이나 롤백을 수행한다.
        
        try {
            roomDao.inserRoom(room);
            List<Equipment> equipmentList = room.getEquipmentList();
            for(Equipment item: equipmentList) {
                roomDao.insertEquipment(item);
            }
        } catch (Exception e) {
            txManager.rollback(status);
            throw new DataAccessException("error occured by insert room", e) { };
        }
        
        txManager.commit(status);
    }
}
```





**빈 정의**

빈 정의 설정에 특별한 점은 없다. 필요한 빈과 인젝션 설정을 하면된다.







#### TransactionTemplate을 활용한 명시적 트랜잭션 제어

**TransactionTemplate**을 이용하면 앞에서 설명한 **PlatformTransactionManager**보다도 구조적으로 트랜잭션 제어를 기술할 수 있다.

트랜잭션 제어 작업을 **TransactionCallback** 인터페이스가 제공하는 메서드에 구현하고 **TransactionTemplate**의 **execute** 메서드에 인수로 전달한다. **TransactionTemplate**은 **JdbcTemplate**과 같은 방식을 사용하고 있기 때문에 애플리케이션 개발자는 필수 처리 로직만 구현하면 된다.



**Service의 구현**

- **TransactionTemplate을 이용한 트랜잭션 제어 구현 예**

```java
@Service
public class RoomServiceImpl implements RoomService {
    @Autowired
    TransactionTemplate transactionTemplate;
    
    @Autowired
    JdbcRoomDao roomDao;
    
    @Override
    public void insertRoom(final Room room) {
        transactionTemplate.execute(new TransactionCallbackWithoutResult(){ 
        // 갱신 처리와 같이 반환값을 필요로 하지 않는 경우에는 execute 메서드를 실행한다. 
        // 만약 조회와 같이 반환값이 있는 메서드라면 TransactionTempalte의 execute 메서드를 실행할 때 
        // org.springframework.transaction.support.TransactionCallback 객체를 인수로 해서 
        // 그 결과를 반환값으로 돌려주면 된다.
        	@Override
            protected void doInTransactionWithoutResult(TransactionStatus status) {
                roomDao.insertRoom(room);
                List<Equipment> equipmentList = room.getEquipmentList();
                for(Equipment item: equipmentList) {
                    roomDao.insertEquipment(item);
                }
            }
            // 위 메서드 안의 처리 내용이 하나의 트랜잭션이 된다. 
            // 반면 참조 처리와 같이 반환값이 필요한 경우에는 doInTransaction 메서드를 구현한다.
        });
    }
}

// doInTransactionWithoutResult 메서드가 정상적으로 종료되면 TransactionTemplate이 자동으로 트랜잭션을 커밋한다.
// 또한 데이터 등록 과정에서 예외가 발생하는 경우에는 TransactionTemplate이 트랜잭션을 롤백한다.
// 만약 업무 로직에서 예외를 던지지 않고 트랜잭션을 롤백하고 싶은 경우에는 doInTransactionWithoutResult의 인수로 전달하는
// TransactionStatus의 setRollbackOnly 메서드를 실행하면 된다.
```





**빈 정의**

- **TransactionTemplate의 자바 기반 설정 방식을 이용한 빈 정의**

```java
@Configuration
public class AppConfig {
	// 생략
    
    @Bean
    public TransactionTemplate transactionTemplate(PlatformTransactionManager transactionManager) {
        TransactionTemplate transactionTemplate = new TransactionTemplate(transactionManager);
        transactionTemplate.setIsolationLevel(TransactionDefinition.ISOLATION_READ_COMMITTED);
        transactionTemplate.setTimeout(30);
        
        return transactionTemplate;
    }
}
```





### 3.3.4 트랜잭션 격리 수준과 전파 방식

이제 트랜잭션 격리 수준과 전파 방식에 대해 알아보자.





#### 트랜잭션 격리 수준

**트랜잭션 격리 수준**은 참조하는 데이터나 변경한 데이터를 다른 트랜잭션으로부터 어떻게 **격리**할 것인지를 결정한다. **격리 수준**은 여러 트랜잭션의 **동시 실행**과 데이터의 **일관성**과 깊이 관련돼 있다. 트랜잭션 격리 수준을 스프링의 기본값인 **DEFAULT**에서 다른 수준으로 변경하고 싶은 경우에는 **@Transactional**의 **isolation** 속성, **TransactionDefinition**과 **TransactionTemplate**의 **setIsolationLevel** 메서드에서 지정할 수 있다.



스프링 프레임워크에서는 데이터베이스의 기본 설정과 4개의 트랜잭션 격리 수준을 이용할 수 있다. 스프링 프레임워크에서 지원하는 격리 수준은 아래의 표와 같다. 다만 지원하는 모든 격리 수준이 실제로 사용할 수 있는지는 사용하는 데이터베이스가 어떻게 구현했느냐에 따라 달라질 수 있다.



**트랜잭션 격리 수준**

| 트랜잭션 격리 수준   | 설명                                                         |
| -------------------- | ------------------------------------------------------------ |
| **DEFAULT**          | 사용하는 데이터베이스의 기본 격리 수준을 이용한다.           |
| **READ_UNCOMMITTED** | Dirty Read, Unrepeatable Read, Phantom Read가 발생한다. 이 격리 수준은 커밋되지 않은 변경 데이터를 다른 트랜잭션에서 참조하는 것을 허용한다. 만약 변경 데이터가 롤백된 경우 다음 트랜잭션에서 무효한 데이터를 조회하게 된다. |
| **READ_COMMITTED**   | Dirty Read를 방지하지만 Unrepeatable Read, Phantom Read는 발생한다. 이 격리 수준은 커밋되지 않은 변경 데이터를 다른 트랜잭션에서 참조하는 것을 금지한다. |
| **REPEATABLE_READ**  | Dirty Read, Unrepeatable Read를 방지하지만 Phantom Read는 발생한다. |
| **SERIALIZABLE**     | Dirty Read, Unrepeatable Read, Phantom Read를 방지한다.      |





#### 트랜잭션 전파 방식

**트랜잭션 전파 방식**은 **트랜잭션 경계**에서 **트랜잭션에 참여**하는 방법을 결정한다. 지원하는 방식에 따라 '새로운 트랜잭션을 시작하는 것', '이미 시작된 트랜잭션에 참여하는 것'과 같이 몇 가지 선택지가 준비돼 있다.



**트랜잭션 경계와 전파 방식**

트랜잭션 전파 방식을 의식해야 하는 경우는 **트랜잭션 경계가 중첩될 때**다. 여러 개의 트랜잭션 경계가 중첩되지 않았다면 트랜잭션을 **TX1 시작 -> TX1 커밋 -> TX2 시작 -> TX2 커밋**과 같이 순차적으로 제어만 하면 되기 때문에 굳이 전파 방식을 의식할 필요는 없다.



트랜잭션 관리 대상이 되는 메서드 안에서 또 다른 트랜잭션 관리 대상이 되는 메서드를 호출한 경우에는 트랜잭션의 전파 방식을 고려해야 한다. 예를 들어, 두 메서드가 각각 **독립적**인 트랜잭션으로 관리될지, 혹은 **같은** 트랜잭션의 관리 범위에 들어가는지는 트랜잭션의 **전파 방식**에 따라 좌우된다.

 



![](https://docs.google.com/drawings/d/s3yf53_Z31DCVrKsc8O8ixA/image?parent=e/2PACX-1vSgWs9-SkKDVDplyC96i2vj5TuZvbmLHvACG_NMQ3eEa1hFtJwCGbFZVRaHriS3HOeS8O0XIQ6gqqII&rev=566&h=449&w=601&ac=1)





**스프링 프레임워크에서 이용 가능한 트랜잭션 전파 방식**

스프링 프레임워크에서는 다음 표와 같은 7가지 트랜잭션 전파 방식을 이용할 수 있다. 트랜잭션 전파 방식을 기본값인 **REQUIRED**에서 다른 방식으로 변경하고 싶은 경우에는 **@Transactional**의 **propagation** 속성이나 **TransactionDefinition**과 **TransactionTemplate**의 **setPropagationBehavior** 메서드에서 지정할 수 있다.



| 전파 방식         | 설명                                                         |
| ----------------- | ------------------------------------------------------------ |
| **REQUIRED**      | 이미 만들어진 트랜잭션이 존재한다면 해당 트랜잭션 관리 범위 안에 함께 들어간다. 만약 이미 만들어진 트랜잭션이 존재하지 않는다면 새로운 트랜잭션을 만든다. |
| **REQUIRES_NEW**  | 이미 만들어진 트랜잭션 범위안에 들어가지 않고 반드시 새로운 트랜잭션을 만든다. 만약 이미 만들어진 트랜잭션이 아직 종료되지 않았다면 새로운 트랜잭션은 보류 상태가 되어 이전 트랜잭션이 끝나는 것을 기다려야 한다. |
| **MANDATORY**     | 이미 만들어진 트랜잭션 범위 안에 들어가야 한다. 만약 기존에 만들어진 트랜잭션이 없다면 예외가 발생한다. |
| **NEVER**         | 트랜잭션 관리를 하지 않는다. 만약 이미 만들어진 트랜잭션이 있다면 예외가 발생한다. |
| **NOT_SUPPORTED** | 트랜잭션을 관리하지 않는다. 만약 이미 만들어진 트랜잭션이 있다면 이전 트랜잭션이 끝나는 것을 기다려야 한다. |
| **SUPPORTS**      | 이미 만들어진 트랜잭션이 있다면 그 범위 안에 들어가고, 만약 트랜잭션이 없다면 트랜잭션 관리를 하지 않는다. |
| **NESTED**        | REQUIRED와 마찬가지로 현재 트랜잭션이 존재하지 않으면 새로운  트랜잭션을 만들고 이미 존재하는 경우에는 이미 만들어진 것을 계속 이용하지만, NESTED가 적용된 구간은 중첩된 트랜잭션처럼 취급한다. NESTED 구간 안에서 롤백이 발생한 경우 NESTED 구간 안의 처리 내용은 모두 롤백되지만 NESTED 구간 밖에서 실행된 처리 내용은 롤백되지 않는다. 단, 부모 트랜잭션에서 롤백되면 NESTED 구간의 트랜잭션은 모두 롤백된다. |





**트랜잭션 전파 방식을 의식할 필요가 있는 예**

예를 들어, 애플리케이션에서 처리 내용을 **추적(trace)**하는 로그를 데이터베이스에 저장해야 한다고 가정해 보자. 만약 비즈니스 로직이 실패했을 때 업무 데이터는 롤백하고 싶지만 로그 데이터는 커밋하고 싶을 때 트랜잭션 경계가 겹치게 된다.



이때 비즈니스 로직과 로그 처리를 서로 다른 트랜잭션 경계로 정의했다고 하더라도 스프링의 기본 전파 방식인 **REQUIRED**를 사용하고 있다면 로그 데이터가 업무 데이터와 함께 **롤백**될 수 있다. 이러한 경우에는 로그 출력용 트랜잭션의 전파 방식을 **REQUIRES_NEW**로 변경하면 된다. 즉, **업무 트랜잭션**과는 다른 새로운 트랜잭션을 만들어서 **업무 트랜잭션**이 롤백되더라도 **로그 출력 트랜잭션**은 롤백되지 않도록 만들어야 한다.







## 3.4 데이터 접근 시의 예외 처리



### 3.4.1 스프링 프레임워크에서 제공하는 데이터 접근 관련 예외

스프링 프레임워크에서는 데이터 접근과 관련된 예외 처리를 일관되고 간단하게 처리하기 위해 다음과 같은 세 가지 기법을 제공한다.



#### DataAccessException을 부모 클래스로 하는 데이터 접근 예외 계층 구조

스프링 프레임워크에서는 예외 처리를 쉽게 하기 위해 데이터 접근과 관련된 모든 예외가 **DataAccessException**을 부모 클래스로 하는 계층 구조로 만들어져 있다. 이 계층 구조를 토대로 다음에 설명하는 데이터 접근 추상화를 구현한다. 



이 그림에서는 계층의 주요 클래스를 나타내기 위해 일부 클래스를 생략했다. 예를 들어, **DataIntegrityViolationException**의 부모 클래스는 **NonTransientDataAccessException**이나 이 그림에서는 생략됐고, **NonTransientDataAccessException**의 부모 클래스가 **DataAccessException**에 해당한다.



![](https://docs.google.com/drawings/d/sGNSFNkDGXNjzy2Y8DOotRA/image?parent=e/2PACX-1vSgWs9-SkKDVDplyC96i2vj5TuZvbmLHvACG_NMQ3eEa1hFtJwCGbFZVRaHriS3HOeS8O0XIQ6gqqII&rev=362&h=416&w=601&ac=1)







#### 비검사 예외를 활용한 DataAccessException 구현

계층 구조의 부모 클래스인 **DataAccessException**은 **RuntimeException**의 자식 클래스로 구현돼 있다. 이로써 **DataAccessException**은 비검사 예외가 된다. 비검사 예외는 **try ~ catch**나 **throws**에 의해 예외 처리가 강제되지 않기 때문에 이 예외를 굳이 처리할 필요가 없다면 예외 처리 코드를 생략해도 된다. 예외 처리의 자세한 내용은 다음에 설명한다.





#### 구현을 은폐한 데이터 접근 예외

일반적으로 **JPA, JDBC, Hibernate**와 같은 데이터 접근 방법의 차이와 **Oracle, PostgreSQL, H2**와 같은 데이터베이스의 차이에서 발생하는 데이터 접근 예외는 모두 각자가 정의한 모양을 하고 있어서 제각각이다. 스프링의 데이터 접근 기능에는 각 제품별로 정의된 서로 다른 예외 클래스를 데이터 접근 방법과 특정 제품에 종속되지 않는 공통적인 예외 클래스로 변환하는 기능이 있다. 이 기능을 사용하면 **DAO **구현이 바뀌더라도 예외 처리 코드에 영향을 주지 않는다.



예를 들어, **Oracle**을 사용할 때 무결성 제약 오류가 발생하면 **Oracle**의 오류 코드 'ORA-00001'이 발생한다. 이와 비슷하게 **PostgreSQL**을 사용할 때 무결성 제약 오류가 발생하면 **PostgreSQL**의 오류 코드 '23000'이 발생한다. 두 경우 모두 무결성 제약 조건에 오류가 발생한 것이기 때문에 스프링에서는 **DataIntegrityViolationException**으로 일관되게 처리한다. 결과적으로 소스코드에서는 데이터베이스 제품이 **Oracle**이냐 **PostgreSQL**이냐에 따라 오류코드를 구분하지 않아도 되며 **DataIntegrityViolationException**에 대해서만 오류 처리를 하면 된다.



이처럼 각 데이터베이스별로 오류 코드에 대응하는 내용은 **spring-jdbc-xxx.jar**의 **org/springframework/jdbc/support/sql-error-codes.xml**에 기술돼 있다. **JDBC**를 직접 이용하거나 내부적으로 **JDBC**를 사용하는 **MyBatis** 같은 **ORM**에서는 이 파일의 내용을 수정하는 방식으로 커스터마이징할 수 있다. 다만, **JPA와 Hibernate**는 이와는 다른 방식으로 구현돼 있기 때문에 **sql-error-codes.xml**을 확장하는 방법은 사용할 수 없다. 





### 3.4.2 데이터 접근 관련 예외 처리

앞에서 설명했듯이 데이터 접근 예외는 비검사 예외로 구현돼 있다. 그래서 예외 처리가 필요하다면 예외가 발생할 만한 곳에 **try~catch** 문을 사용해 원하는 예외를 잡아서 처리해야 한다. 특별히 예외 처리를 할 필요가 없다면 아무것도 해 줄 필요가 없다.



**데이터 접근 예외를 처리하는 구현 예**

데이터 접근 예외를 어떻게 처리하는지 확인하기 위해 예를 하나 들어보자. 다음은 특정 키를 검색 조건으로 데이터를 조회했으나 해당 데이터가 없어 **DataRetrievalFailureException**이 발생한 경우다. **DataRetrievalFailureException**은 스프링 프레임워크가 추상화한 예외로, 이것을 그대로 사용해도 큰 무리는 없으나 특정 프레임워크에서 제공하는 예외를 사용하기보다는 이 애플리케이션의 비즈니스 상황에 맞는 예외로 바꿔서 처리하는 것이 맞을 것이다. 그래서 아래 예에서는 **DataRetrievalFailure**를 가로채서 이 애플리케이션에서 정의한 **NotFoundRoomIdException**을 던지고 있다.



```java
public Room getRoomForUpdate(String roomId) {
    Room room = null;
    try {
        room = roomDao.getRoomForUpdate(RoomId);
    } catch(DataRetrievalFailureException e) {
        throw new NotFoundRoomIdException("roomId=" + roomId, e);
    }
    return room;
}
```





### 3.4.3 데이터 접근 관련 예외의 변환 규칙 커스터마이징

각 데이터베이스 오류 코드와 데이터 접근 예외에 관련된 내용은 **spring-jdbc-xxx.jar**에 포함된 **sql-error-codes.xml**을 사용한다. 만약 classpath 바로 아래에 **sql-error-codes.xml**을 두고 그 내용을 재정의하면 기본 설정된 내용을 커스터마이징할 수 있다.



#### spring-jdbc-xxx.jar의 org/springframework/jdbc/support/sql-error-codes.xml

**sql-error-codes.xml**은 일반적인 빈 정의 파일이다. 이 파일의 내용은 **SQLErrorCodes** 클래스의 빈을 정의하는 것으로, 빈의 ID는 데이터베이스 이름으로 돼 있다. 데이터베이스 접근 예외에 대응하는 프로퍼티가 정의돼 있는데, 각 프로퍼티 값에 대응하는 데이터베이스 고유의 오류 코드들이 쉼표로 구분되어 설정돼 있다. 예를 들어, H2 데이터베이스의 오류 코드는 다음과 같이 정의돼 있다.



- **H2 데이터베이스의 오류 코드 정의**

```xml
<bean id="H2" class="org.springframework.jdbc.support.SQLErrorCodes">
	<property name="badSqlGrammarCodes">
    	<value>42000,42001,42101,42102,42111,42112,42121,42122,42132</value>
    </property>
    <property name="duplicateKeyCodes">
    	<value>23001,23505</value>
    </property>
    <property name="dataIntegrityViolationCodes">
    	<value>22001,22003,22012,22018,22025,23000,23002,23003,23502,23503,23506,23507,23513</value>
    </property>
    <property name="dataAccessResourceFailureCodes">
    	<value>90046,90100,90117,90121,90126</value>
    </property>
    <property name="cannotAcquireLockCodes">
    	<value>50200</value>
    </property>
</bean>
```





#### Classpath 바로 아래의 sql-error-codes.xml

커스터마이징할 때는 **SQLErrorCodes**의 프로퍼티 설정을 변경하면 된다. 예를 들어, H2 데이터베이스에서 오류 코드 23001 또는 23505가 발생한 경우 **DataIntegrityViolationException**이 발생하게 만들려면 다음과 같이 설정하면 된다.



- **변환 룰을 커스터마이징하기 위한 빈 정의**

```xml
<bean id="H2" class="org.springframework.jdbc.support.SQLErrorCodes">
	<property name="badSqlGrammarCodes">
    	<value>42000,42001,42101,42102,42111,42112,42121,42122,42132</value>
    </property>
    
    <!--
    <property name="duplicateKeyCodes">
    	<value>23001,23505</value>
    </property>
	-->
    
    <property name="dataIntegrityViolationCodes">
    	<value>23001,23505,22001,22003,22012,22018,22025,23000,23002,
        23003,23502,23503,23506,23507,23513</value>
    </property>
    
    <property name="dataAccessResourceFailureCodes">
    	<value>90046,90100,90117,90121,90126</value>
    </property>
    <property name="cannotAcquireLockCodes">
    	<value>50200</value>
    </property>
</bean>
```