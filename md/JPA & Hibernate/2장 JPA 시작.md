# 2장 JPA 시작



... 생략



## 2.4 객체 매핑 시작



**회원 테이블 생성**

```sql
CREATE TABLE MEMBER (
	ID VARCHAR(255) NOT NULL,   -- 아이디(기본 키)
    NAME VARCHAR(255),          -- 이름
    AGE INTEGER,                -- 나이
    PRIMARY KEY(ID)
)
```



**회원 Entity 클래스 생성**

```java
public class Member {
	private String id;           
    private String username;
    private Integer age;
    
    // Getter, Setter들 (생략)
}
```





JPA를 사용하려면 가장 먼저 **회원 클래스**와 **회원 테이블**을 매핑해야 한다. 다음 표의 매핑 정보를 보고 매핑을 시작해보자.

| 매핑 정보       | 회원 객체 | 회원 테이블 |
| --------------- | --------- | ----------- |
| 클래스와 테이블 | Member    | MEMBER      |
| 기본 키         | id        | ID          |
| 필드와 컬럼     | username  | NAME        |
| 필드와 컬럼     | age       | AGE         |





아래처럼 회원 클래스에 JPA가 제공하는 매핑 Annotation을 추가하자.

```java
import javax.persistence.*;

@Entity
@Table(name="MEMBER")
public class Member {
    
    @Id
    @Column(name = "ID")
    private String id;           
    
    @Column(name = "NAME")
    private String username;
    
    // 매핑 정보가 없는 필드
    private Integer age;
    
    ...
}
```





- **@Entity**
  - 이 클래스를 테이블과 매핑한다고 JPA에게 알려준다.
  - 이렇게 **@Entity**가 사용된 클래스를 **엔터티 클래스**라고 한다.
- **@Table**
  - **엔터티 클래스**에 매핑할 테이블 정보를 알려준다.
- **@Id**
  - **엔터티 클래스**의 필드를 테이블의 **Primary Key(기본키)**에 매핑한다.
  - 이렇게 **@Id**가 사용된 필드를 **식별자 필드**라고 한다.

- **@Column**
  - 필드를 컬럼에 매핑한다.
- **매핑 정보가 없는 필드**
  - 매핑 어노테이션을 생략하면 **필드명**이 **컬럼명**으로 매핑된다.
  - 위의 예에서는 age 필드가 age 컬럼으로 매핑된다. (만약 DB가 대소문자를 구분하면 **@Column(name = "AGE")**로 써줘야 한다.)







## 2.5 persistence.xml 설정

JPA는 **persistence.xml**을 사용해서 필요한 **설정 정보**를 관리한다.

이 설정 파일이 **`META-INF/persistence.xml`** classpath 경로에 있으면 별도의 설정 없이 JPA가 인식할 수 있다.



```xml
<?xml version="1.0" encoding="UTF-8"?>
<persistence xmlns="http://xmlns.jcp.org/xml/ns/persistence" version="2.1">

    <persistence-unit name="jpabook">

        <properties>

            <!-- 필수 속성 -->
            <property name="javax.persistence.jdbc.driver" value="org.h2.Driver"/>
            <property name="javax.persistence.jdbc.user" value="sa"/>
            <property name="javax.persistence.jdbc.password" value=""/>
            <property name="javax.persistence.jdbc.url" value="jdbc:h2:tcp://localhost/~/test"/>
            <property name="hibernate.dialect" value="org.hibernate.dialect.H2Dialect" />

            <!-- 옵션 -->
            
            <!-- hibernate가 실행한 SQL을 출력 -->
            <property name="hibernate.show_sql" value="true" />
            
            <!-- hibernate가 실행한 SQL을 출력할 때 보기 쉽게 정렬 -->
            <property name="hibernate.format_sql" value="true" />
            
            <!-- 쿼리를 출력할 때 주석도 함께 출력 -->
            <property name="hibernate.use_sql_comments" value="true" />
            
            <!-- JPA 표준에 맞춘 새로운 키 생성 전략을 사용 (4.6절 참고)-->
            <property name="hibernate.id.new_generator_mappings" value="true" />

            <!--<property name="hibernate.hbm2ddl.auto" value="create" />-->
        </properties>
    </persistence-unit>

</persistence>
```



- 설정 파일은 *persistence*로 시작한다. 이곳에 XML 네임스페이스와 사용할 버전을 지정한다. 

- JPA 설정은 **persistence-unit(영속성 유닛)**이라는 것부터 시작한다.

  - 영속성 유닛에는 고유한 이름을 부여해야 하는데 여기서는 *jpabook*이라는 이름을 부여했다.

- 다음으로 *properties* 값을 설정했다. 

  - javax로 시작하는 건 표준 속성이고 hibernate로 시작하는 건 하이버네이트 속성이다.

  - javax로 시작하는 속성은 JPA 표준 속성으로 특정 구현체에 종속되지 않는다. 반면에 hibernate로 시작하는 속성은 하이버네이트에서만 사용할 수 있다.
  - hibernate.dialect는 DB 방언을 설정한다.





### 2.5.1 DB 방언

JPA는 특정 DB에 종속적이지 않은 기술이다. 따라서 다른 DB로 손쉽게 교체할 수 있다. 그런데 각 DB가 제공하는 SQL 문법과 함수가 조금씩 다르다..

이처럼 <u>SQL 표준을 지키지 않거나 특정 DB만의 고유한 기능</u>을 JPA에서는 **방언(Dialect)**이라 한다.





## 2.6 애플리케이션 개발



```java
import javax.persistence.*;
import java.util.List;

public class JpaMain {
    public static void main(String[] args) {
        EntityManagerFactory emf = Persistence.createEntityManagerFactory("jpabook");
        EntityManager em = emf.createEntityManager();
        EntityTransaction tx = em.getTransaction();

        try {
            tx.begin();
            logic(em);
            tx.commit();
        } catch (Exception e) {
            tx.rollback();
        } finally {
            em.close();
            emf.close();
        }
    }
    
    private static void logic(EntityManager em) {
        // 비즈니스 로직
    }
}
```



위의 코드는 크게 3부분으로 나뉘어 진다.

- **EntityManager 설정**
- **Transaction 관리**
- **Business Logic**





### 2.6.1 EntityManager 설정



**[EntityManager의 생성 과정]**



![](https://docs.google.com/drawings/d/sNbj37wAgW3U90dKCdjZvkA/image?parent=e/2PACX-1vRVvbIKomKLDTKGV_Zpd2jnSflB7IGvZt3_68Ps6WLyfUO3_b16JGyGje2BnfNAFQnyCXgohLQOH9QO&rev=210&h=333&w=601&ac=1)





- **EntityManagerFactory 생성**

JPA를 시작하려면 우선 persistence.xml의 설정 정보를 사용해서 **EntityManagerFactory**를 생성해야 한다. 이때 **Persistence** 클래스를 사용하는데 이 클래스는 **EntityManagerFactory**를 생성해서 JPA를 사용할 수 있게 준비한다.



`EntityManagerFactory emf = Persistence.createEntityManagerFactory("jpabook");`



이렇게 하면 **`META-INF/persistence.xml`**에서 이름이 *jpabook*인 **persistence-unit(영속성 유닛)**을 찾아서 **EntityManagerFactory**를 생성한다.



이때 persistence.xml의 설정 정보를 읽어서 JPA를 동작시키기 위한 기반 객체를 만들고 JPA 구현체에 따라서는 DB 커넥션 풀도 생성하므로 <u>**EntityManagerFacotry**를 생성하는 비용은 아주 크다</u>. 따라서 <u>**EntityManagerFactory**는 애플리케이션 전체에서 딱 한 번만 생성하고 공유해서 사용해야 한다.</u>







- **EntityManager 생성**

`EntityManager em = emf.createEntityManager();`



EntityManagerFactory에서 **EntityManager**를 생성한다. JPA의 기능 대부분은 이 **EntityManager**가 제공한다.

대표적으로 **EntityManager**를 사용해서 Entitiy를 DB에 **CRUD** 할 수 있다.



**EntityManager**는 내부에 DataSource(DB Connection)를 유지하면서 DB와 통신한다. 따라서 애플리케이션 개발자는 **EntityManager**를 가상의 DB로 생각할 수 있다.



참고로 <u>**EntityManager**는 DB Connection과 밀접한 관계가 있으므로 스레드 간에 공유하거나 재사용하면 안된다</u>.









- **종료**

사용이 끝난 **EntityManager**는 반드시 종료한다.

`em.close()`



애플리케이션을 종료할 때 **EntityMangerFactory**도 종료한다.

`emf.close();`







### 2.6.2 트랜잭션 관리

<u>JPA를 사용하면 항상 트랜잭션 안에서 데이터를 변경해야 한다</u>.

트랜잭션 없이 데이터를 변경하면 예외가 발생한다.

트랜잭션을 시작하려면 **EntityManager**에서 트랜잭션 API를 받아와야 한다.



```java
EntityTransaction tx = em.getTransaction();
tx.begin();

try {
	tx.begin();
    logic(em);
    tx.commit();
} catch (Exception e) {
    tx.rollback();
} 
```



트랜잭션 API를 사용해서 비즈니스 로직이 정상 동작하면 트랜잭션을 **commit**하고, 예외가 발생하면 **rollback**한다.









### 2.6.3 비즈니스 로직

비즈니스 로직은 단순하다. 

회원 엔터티를 하나 생성한 다음 **EntityManager**를 통해 DB에 등록, 수정, 삭제, 조회한다.



```java
public static void logic(EntityManager em) {

    String id = "id1";
    Member member = new Member();
    member.setId(id);
    member.setUsername("지한");
    member.setAge(2);
        
        
    // 등록
    em.persist(member);
        
    
    // 수정
    member.setAge(20);
        
    
    // 한 건 조회
    Member findMember = em.find(Member.class, id);
    System.out.println("findMember.getUsername() = " + findMember.getUsername());
    System.out.println("findMember.getAge() = " + findMember.getAge());
    
    
    // 목록 조회
    List<Member> members =
     	em.createQuery("select m from Member m", Member.class).getResultList();
    System.out.println("members.size() = " + members.size());
        
        
    // 삭제
    em.remove(member);
}
```



비즈니스 로직을 보면 등록, 수정, 삭제, 조회 작업이 **EntityManager**를 통해 수행되는 것을 알 수 있다. 

**EntityManager**는 객체를 저장하는 가상의 DB처럼 보인다.

먼저 등록, 수정, 삭제 코드를 분석해보자.





- **등록**

엔터티를 저장하려면 **EntityManager**의 *persist()* 메서드에 저장할 엔터티를 넘겨주면 된다. 

예제를 보면 회원 엔터티를 생성하고 *em.persist(member)*를 실행해서 엔터티를 저장했다.

JPA는 회원 엔터티의 매핑 정보(어노테이션)를 분석해서 다음과 같은 SQL을 만들어서 DB에 전달한다.



`INSERT INTO MEMBER (ID, NAME, AGE) VALUES ('id1', '지한', 2)`







- **수정**

엔터티를 수정한 후에 수정 내용을 반영하려면 *em.update()* 같은 메서드를 호출해야 할 것 같은데 <u>단순히 엔터티의 값만 변경했다</u>.

 JPA는 어떤 엔터티가 변경되었는지 추적하는 기능을 갖추고 있다. 

따라서 *member.setAge(20)*처럼 엔터티의 값만 변경하면 다음과 같은 UPDATE SQL을 생성해서 DB의 값을 변경한다.



`UPDATE MEMBER SET AGE=20, NAME='지한' WHERE ID='id1'`







- **삭제**

엔터티를 삭제하려면 **EntityManager**의 *remove()* 메서드에 삭제하려는 엔터티를 넘겨준다.

JPA는 다음 DELETE SQL을 생성해서 실행한다.



`DELETE FROM MEMBER WHERE ID = 'id1'`







- **한 건 조회**

*find()* 메서드는 조회할 **엔터티 타입**과 *@Id*로 DB 테이블의 기본 키와 매핑한 **식별자 필드 값**으로 엔터티 하나를 조회하는 가장 단순한 조회 메서드다.

이 메서드를 호출하면 다음 SELECT SQL을 생성해서 DB의 결과를 조회한다.

그리고 조회한 결과 값으로 엔터티를 생성해서 반환한다.



`SELECT * FROM MEMBER WHERE ID='id1'`







### 2.6.4 JPQL

하나 이상의 회원 목록을 조회하는 다음 코드를 자세히 살펴보자.

`em.createQuery("select m from Member m", Member.class)`



JPA를 사용하면 애플리케이션 개발자는 **엔터티 객체**를 중심으로 개발하고,

DB에 대한 처리는 JPA에 맡겨야 한다.

바로 앞에서 살펴본 등록, 수정, 삭제, 한 건 조회 예를 보면 SQL을 전혀 사용하지 않았다.



문제는 **검색 쿼리**다. 

JPA는 **엔터티 객체**를 중심으로 개발하므로 검색을 할 때도 테이블이 아닌 **엔터티 객체**를 대상으로 검색해야 한다.

그런데 테이블이 아닌 **엔터티 객체**를 대상으로 검색하려면 데이터베이스의 모든 데이터를 애플리케이션으로 불러와서

**엔터티 객체**로 변경한 다음 검색해야 하는데, 이는 사실상 불가능하다.

애플리케이션이 필요한 데이터만 DB에서 불러오려면 결국 검색 조건이 포함된 SQL을 사용해야 한다.

JPA는 **JPQL(Java Persistence Query Language)**이라는 **쿼리 언어**로 이런 문제를 해결한다.



JPA는 SQL을 추상화한 **JPQL**이라는 객체지향 쿼리 언어를 제공한다. 

**JPQL**은 SQL과 문법이 거의 유사하다.

둘의 가장 큰 차이점은 다음과 같다.

- **JPQL**은 **엔터티 객체**를 대상으로 쿼리한다. 
- **SQL**은 **DB 테이블**을 대상으로 쿼리한다.





**JPQL**을 사용하려면 먼저 `em.createQuery(JPQL, 반환 타입)` 메서드를 실행해서 쿼리 객체를 생성한 후,

쿼리 객체의 `getResultList()` 메서드를 호출하면 된다.















