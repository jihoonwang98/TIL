# Testing JPA Queries with Spring Boot and @DataJpaTest



> https://reflectoring.io/spring-boot-data-jpa-test/
>
> This article is accompanied by a working code example [on GitHub](https://github.com/thombergs/code-examples/tree/master/spring-boot/spring-boot-testing).



`@DataJpaTest` 애너테이션을 사용하면, Spring Boot는 DB에 날라가는 쿼리를 테스트하기 위해 embedded DB 환경을 구축해준다.



이번 튜토리얼에서는, 

첫째로 어떤 종류의 쿼리들이 테스트할 가치가 있는지 토론해본 후,

테스트를 하기 위한 DB 스키마와 DB 상태를 만드는 여러가지 방법에 대해 살펴본다.





## 1. What to Test?

우리가 스스로에게 질문해야 하는 첫 번째 질문은 '무엇'을 테스트해야 하는가?이다.



아래와 같이 `UserEntity`를 CRUD하는 Spring Data Repository를 살펴보자.

```java
interface UserRepository extends CrudRepository<UserEntity, Long> {
  // query methods
}
```



우리는 쿼리를 생성하는 다양한 방법들을 가지고 있다.

우리가 이 방법들을 모두 테스트를 통해 검증해야 하는지 결정하기 위해 이것들을 좀 더 자세히 살펴보자.





### (1) Inferred Queries

쿼리를 만드는 첫 번째 방법은 바로 *inferred query*를 생성하는 방법이다.

```java
UserEntity findByName(String name);
```



우리는 Spring Data에게 어떻게 쿼리문을 날릴지 전해주지 않아도 된다.

왜냐하면 Spring Data가 메서드명으로부터 SQL 쿼리를 자동으로 추론해주기 때문이다.



이 기능의 장점은 Spring Data가 해당 쿼리가 유효한지 애플리케이션이 시작할 때 자동으로 검사해주는 것이다.

예를 들어, 우리가 위의 메서드명을 `findByFoo()`로 바꿨는데 `UserEntity`가 `foo`라는 프로퍼티를 가지고 있지 않으면, Spring Data는 예외를 던지게 될 것이다.

```java
org.springframework.data.mapping.PropertyReferenceException: 
  No property foo found for type UserEntity!
```



<u>**그러므로 우리의 테스트 코드 중 적어도 하나라도 Spring application context를 start up하는 테스트가 존재한다면,**</u> <u>**우리는 inferred query에 대한 추가적인 테스트 코드를 작성할 필요가 없다.**</u>









### (2) Custom JPQL Queries with `@Query`

만약 inferred query로 나타내기엔 너무 복잡한 쿼리를 만들고 싶다면, custom JPQL query를 이용하는 것이 좋은 방법이다.

```java
@Query("select u from UserEntity u where u.name = :name")
UserEntity findByNameCustomQuery(@Param("name") String name);
```



inferred query와 비슷하게, Spring Data는 이러한 JPQL query들에 대해서도 유효성 검사를 해준다.

Hibernate를 JPA 구현체로 사용하는 경우 애플리케이션 startup시에 유효하지 않은 JPQL 쿼리를 발견했다면, `QuerySyntaxException`이 던져질 것이다.

```java
org.hibernate.hql.internal.ast.QuerySyntaxException: 
  unexpected token: foo near line 1, column 64 [select u from ...]
```





Custom query들은 inferred query처럼 single attribute로 entry를 찾는 것보다 훨씬 복잡할 수 있다.

얘네들은 다른 테이블과 Join 연산을 할 수도 있고 entity 대신 복잡한 DTO를 반환할 수도 있다.



그래서 결국 우리는 이러한 custom query들에 대한 테스트를 작성해야 할까?

약간은 모호한 답변일 수 있지만, **<u>우리는 해당 쿼리가 테스트가 필요할 만큼 복잡한지 스스로 판단해서 결정해야 한다.</u>**







### (3) Native Queries with `@Query`

쿼리를 생성하는 또 다른 방법은 바로 native query를 이용하는 것이다.

```java
@Query(
  value = "select * from user as u where u.name = :name",
  nativeQuery = true)
UserEntity findByNameNativeQuery(@Param("name") String name);
```



JPQL 쿼리를 쓰는 것 대신, SQL 쿼리를 직접 사용할 수도 있다.

이 경우에는 사용하고 있는 DB에 의존하는 쿼리를 사용하게 된다.



native query를 사용하는 경우 **<u>Hibernate와 Spring Data 둘 다 native query가 유효한지 startup시에 검사해주지 않는다는 것을 명심하자</u>**.

왜냐하면 query가 DB-specific한 SQL을 포함할 수도 있기 때문에, Spring Data나 Hibernate가 해당 쿼리가 유효한지 체크할 재간이 없다.



그래서, **<u>native query들은 통합 테스트시에 반드시 테스트해야 한다</u>**.

그러나, 이 native query들이 DB-specific한 SQL들을 사용하고 있다면, embedded in-memory DB에서는 잘 작동되지 않을 수 있기 때문에 우리는 실제 프로덕션용 DB를 제공해야 한다.







## 2. `@DataJpaTest` 간단히 살펴보기

Spring Data JPA repository들 또는 다른 JPA에 관련된 다른 component들을 테스트하기 위해서,

Spring Boot는 `@DataJpaTest`라는 애너테이션을 제공한다.

우리는 우리의 unit test에 이 애너테이션을 붙이면 이것이 Spring Application context를 set up해줄 것이다.



```java
@ExtendWith(SpringExtension.class)
@DataJpaTest
class UserEntityRepositoryTest {

  @Autowired private DataSource dataSource;
  @Autowired private JdbcTemplate jdbcTemplate;
  @Autowired private EntityManager entityManager;
  @Autowired private UserRepository userRepository;

  @Test
  void injectedComponentsAreNotNull(){
    assertThat(dataSource).isNotNull();
    assertThat(jdbcTemplate).isNotNull();
    assertThat(entityManager).isNotNull();
    assertThat(userRepository).isNotNull();
  }
}
```



위에서 생성된 Application Context는 우리의 Spring Boot Application에 필요한 모든 context가 포함되지 않고, JPA에 관련된 component들을 초기화하는데 필요한 것들만 부분적으로 올라간다.



위의 코드와 같이 우리는 `DataSource`, `JdbcTemplate`, `EntityManager` 들을 테스트 클래스 내에 주입시킬 수 있다.

또한, 우리는 우리의 애플리케이션 내에 있는 어떠한 Spring Data repository들도 주입시킬 수 있다.



주의할 점은 위에 있는 모든 component들은

`application.properties`나 `application.yml` 파일에 설정한 "real" DB 대신 

테스트용 embedded, in-memory DB에 대한 component들이라는 점이다.



그리고 기본적으로 이러한 모든 component들을 포함하고 있는 Application Context는

`@DataJpaTest`로 annotated된 모든 테스트 클래스의 모든 테스트 메서드들이 공유하게 된다.





이러한 이유로 각각의 테스트 메서드는 자신의 own transaction 내에서 수행된 후, 수행이 끝나면 rollback된다.(기본 설정임)

이러한 설정을 통해, DB는 테스트간 독립성을 유지하며 DB 상태를 오염되지 않게 유지할 수 있다.

> @DataJpaTest는 기본적으로 @Transactional 애너테이션을 포함하고 있다...







## 3. Creating the DB Schema

DB에 쿼리를 날리기 전에, SQL schema를 만들어야 한다.

schema를 만드는 다양한 방법들을 살펴보자.



### (1) Using Hibernate's `ddl-auto`

기본적으로, `@DataJpaTest`는 Hibernate가 DB schema를 자동적으로 생성하게 설정을 해놓는다.

이 설정값은 `application.yml`의 `spring.jpa.hibernate.ddl-auto`를 통해 설정할 수 있는데,

Spring Boot는 이 값을 기본적으로 `create-drop`으로 해놓는다.





### (2) Using `schema.sql`

Spring Boot는 application startup시에 custom한 `schema.sql` 파일을 수행하는 것을 지원한다.

만약, Spring이 `schema.sql` 파일을 classpath 내에서 발견하면, 이것을 Datasource가 수행해줄 것이다.

`schema.sql` 파일은 위에서 언급한 Hibernate의 `ddl-auto` 설정을 override 해버린다.



우리는 `spring.datasource.initialization-mode`라는 설정값을 통해 `schema.sql` 파일을 실행할 지 말지 컨트롤할 수 잇따.

이 설정값은 기본값으로 `embedded`로 되어 있다. 즉, 테스트시 embedded DB를 사용할 때만 `schema.sql`을 수행하겠다는 의미이다.

만약 이 값을 `always`로 설정하면, 언제나 수행된다.



아래의 로그를 통해 `schema.sql`이 수행되었는지 알 수 있다.

```java
Executing SQL script from URL [file:.../out/production/resources/schema.sql]
```



`schema.sql` 파일을 통해 schema를 생성할 때 Hibernate의 `ddl-auto` 설정값을 `validate`로 놓고

Hibernate가 생성된 schema가 entity 클래스와 매칭되는지 체크하게 하는 것이 좋다.

```java
@ExtendWith(SpringExtension.class)
@DataJpaTest
@TestPropertySource(properties = {
        "spring.jpa.hibernate.ddl-auto=validate"
})
class SchemaSqlTest {
  ...
}
```





### (3) Flyway, Liquibase









## 4. DB에 샘플 데이터 넣기

DB 쿼리를 테스트할 때, 우리는 보통 DB에 샘플 데이터를 넣고 우리의 쿼리가 올바른 결과값을 반환하는지를 테스트한다.



우리의 in-memory DB에 샘플 데이터를 넣는 다양한 방법에 대해 알아보자.



### (1) Using `data.sql`

`schema.sql`과 비슷하게, 우리는 insert문을 포함하고 있는 `data.sql` 파일을 사용할 수 있다.



> **이 방법의 유지보수성 한계**
>
> 우리는 모든 `insert`문을 `data.sql` 파일 한군데에 몰아넣어야 한다.
>
> 모든 테스트는 DB를 set up하기 위해 이 `data.sql` script 한 개에 의존하게 될 것이다.
>
> 테스트를 작성할수록 이 script는 점점 더 커지고 유지보수하기 어려워진다.
>
> 그리고 만약 테스트들이 서로 상충하는 DB 상태를 테스트하고 싶은 경우에는 어떻게 할 방법이 없다.
>
> 따라서 이 방법은 조심해서 사용해야 한다.





### (2) Inserting Entities Manually

테스트 각각 마다 specific한 DB state를 만들기 위한 간단한 방법으로는

그냥 테스트하기전에 엔티티를 직접 저장해주는 방법이 있다.

```java
@Test
void whenSaved_thenFindsByName() {
  userRepository.save(new UserEntity(
          "Zaphod Beeblebrox",
          "zaphod@galaxy.net"));
  assertThat(userRepository.findByName("Zaphod Beeblebrox")).isNotNull();
}
```



위처럼 간단한 케이스는 저장하기가 귀찮지 않다. 

하지만 실제 프로젝트에서는 엔티티가 훨씬 더 복잡하고 다른 엔티티와의 관계도 설정해주어야 한다.



이러한 복잡성을 줄이는 한 가지 방법은 [Objectmother and Builder](https://reflectoring.io/objectmother-fluent-builder/) pattern과 조합해서 factory method를 생성하는 것이다.



직접 Java code를 통해 DB 샘플 데이터를 넣는 방법은 다른 방법에 비해 **refactoring-safe**하다는 큰 장점이 있다.

codebase에 대한 변화는 우리의 테스트 코드에 대한 compile error를 발생하게 한다.





### (3) Using Spring DBUnit

DBUnit은 DB를 특정 state로 세팅하는 것을 도와주는 라이브러리다.

Spring DBUnit은 DBUnit과 Spring을 통합해 Spring의 트랜잭션과 함께 동작하도록 해준다.



이 라이브러리를 사용하기 위해서, 아래와 같은 종속성을 추가해주자.

```gradle
compile('com.github.springtestdbunit:spring-test-dbunit:1.3.0')
compile('org.dbunit:dbunit:2.6.0')
```



그 다음, 각각의 테스트 케이스에 대해 우리는 우리가 원하는 DB state를 나타내는

custom XML 파일을 생성할 수 있다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<dataset>
    <user
        id="1"
        name="Zaphod Beeblebrox"
        email="zaphod@galaxy.net"
    />
</dataset>
```

기본적으로, XML 파일은 테스트 클래스가 위치한 classpath 옆에 바로 위치한다.

(위의 xml 파일을 `createUser.xml`이라고 이름을 지어주자.)





테스트 클래스에서는, 우리는 두 개의 `TestExecutionListeners`를 추가해 DBUnit 서포트를 enable할 필요가 있다.

xml 파일에 설정한 DB state를 설정하기 위해 우리는 테스트 메서드 위에 `@DatabaseSetup` 애너테이션을 붙일 수 있다.

```java
@ExtendWith(SpringExtension.class)
@DataJpaTest
@TestExecutionListeners({
        DependencyInjectionTestExecutionListener.class,
        TransactionDbUnitTestExecutionListener.class
})
class SpringDbUnitTest {

  @Autowired
  private UserRepository userRepository;

  @Test
  @DatabaseSetup("createUser.xml")
  void whenInitializedByDbUnit_thenFindsByName() {
    UserEntity user = userRepository.findByName("Zaphod Beeblebrox");
    assertThat(user).isNotNull();
  }

}
```

DB state를 변화시키는 쿼리들을 테스트하고 싶다면 `@ExpectedDatabase` 애너테이션을 사용해서 테스트를 수행하고 난 후의 DB state를 정의할 수도 있다.



참고로, Spring DBUnit은 2016년 이후로 개발이 중단되었다고 한다.







### (4) Using `@Sql`

3번과 매우 유사한 방법은 Spring의 `@Sql` 애너테이션을 이용하는 방법이다.

XML을 사용해서 DB state를 나타내는 방법 대신 우리는 SQL을 직접적으로 사용할 수 잇따.

```sql
-- createUser.sql
INSERT INTO USER 
            (id, 
             NAME, 
             email) 
VALUES      (1, 
             'Zaphod Beeblebrox', 
             'zaphod@galaxy.net'); 
```



우리의 테스트 내에서, 테스트 메서드 위에 간단하게 `@Sql` 애너테이션을 붙여주면 샘플 데이터를 넣는 INSERT문이 정의된 SQL 파일을 참조할 수 있다.

```java
@ExtendWith(SpringExtension.class)
@DataJpaTest
class SqlTest {

  @Autowired
  private UserRepository userRepository;

  @Test
  @Sql("createUser.sql")
  void whenInitializedByDbUnit_thenFindsByName() {
    UserEntity user = userRepository.findByName("Zaphod Beeblebrox");
    assertThat(user).isNotNull();
  }

}
```

만약 여러 개의 script를 실행해야 할 때는 `@SqlGroup` 애너테이션을 통해 script들을 조합할 수 있다.

















