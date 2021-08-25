## 10.1 JPA란?

**JPA(Java Persistence API)**는 자바 표준 **ORM(Object-Relational Mapping)**으로, 2006년에 발표된 Java EE 5 사양에서 채택됐다. 이 책에서는 지면 관계상 **ORM**이나 **JPA**에 대해서 **스프링 데이터 JPA**를 사용하는데 꼭 필요한 정도로만 다룰 것이다.



### 10.1.1 ORM과 JPA의 개념

**ORM**의 개념을 한마디로 설명하면 **관계형 데이터베이스**(이하 데이터베이스)에 데이터를 읽고 쓰는 처리를 **객체**에 데이터를 읽고 쓰는 방식으로 구현하는 기술이다. 상용 시스템을 개발할 때는 애플리케이션에서 다루는 데이터를 **DB**에 저장하는 것이 일반적이다. 자바에서 SQL을 실행하는 프로그램을 한번이라도 직접 만들어 본 적이 있다면 다음과 같은 불만을 가진 적이 있을 것이다.

- **DB에 표현된 테이블 간의 관계를 자바 객체 간의 참조 관계로 표현하는 것이 어렵다.**
- **SQL을 사용할 때, 입출력할 값을 담을 객체(Data Transfer Object라는 클래스)를 너무 많이 만든다.**
- **테이블 이름만 다른 비슷비슷한 SQL을 너무 많이 만든다.**
- **SQL에서 읽고 쓰는 테이블 칼럼과 자바 객체의 프로퍼티를 매핑하는 것은 기계적이고 소모적인 작업이다.**



프로그래밍 언어는 **객체지향** 언어인 반면에 DB나 SQL은 객체지향이 아니기 때문에 이러한 문제는 언제든지 발생할 수 있다. 이러한 **객체지향** 언어와 **관계형 DB** 사이의 간극을 메우려는 사상이 **ORM**이다. **ORM**은 객체와 DB를 매핑하는 역할을 대신해 준다. 그래서 프로그램 개발자가 객체에 접근하기 위한 코드를 개발하면 결과적으로 DB의 테이블에도 접근할 수 있게 되고, 이제까지 DB를 의식하면서 짜야 했던 번거로운 코드에서 해방되어 보다 비즈니스 로직에 집중할 수 있게된다.



앞에서 설명한 것처럼 **JPA**는 자바 표준의 **ORM**으로 현재는 Java EE의 핵심적인 구성 요소다. 표준화가 진행될 당시 자바용 **ORM**은 **하이버네이트**가 인기를 끌고 있었기 때문에 **JPA**는 **하이버네이트**의 기본 사상을 이어받아 사양이 책정됐다. **JPA 1.0**이 Java EE 5에서 책정된 후 **JPA 2.0**(Java EE 6), **JPA 2.1**(Java EE 7)과 사양 및 구현과 함께 거듭 업데이트되고 있다.



**JPA**는 다른 Java EE의 구성 요소와 결합도가 낮은 부분이 많은데, 예를 들어 **EJB(Enterprise JavaBeans)** 컨테이너를 지원하지 않는 톰캣에서도 **JPA** 구현만 추가하면 이용할 수 있다. 일부 **JPA** 구현이 **OSS**로 제공되기 때문에 이용하기도 쉽다. **JPA** 구현을 일반적으로 '**영속화 프로바이더**'라고 부르기도 한다.



**OSS로 제공되는 주요 JPA 구현**

| 제품명             | 특징                                                         |
| ------------------ | ------------------------------------------------------------ |
| **EclipseLink**    | JPA 참조 구현체. GlassFish에서 사용된다.                     |
| **Hibernate ORM**  | JPA의 기반이 된 하이버네이트의 JPA 구현. JBoss/WildFly에서 사용된다. |
| **Apache OpenJPA** | 아파치 소프트웨어 재단에서 개발된 JPA 구현. Apache TomEE에서 사용된다. |
| **DataNucleus**    | Google AppEngine에서 사용된다.                               |



자세한 내용은 나중에 설명하겠지만 **JPA**의 매력을 잘 알 수 있는 예를 소개한다. 다음 코드에서는 DB에서 한 건의 레코드를 조회한다. 첫 번째는 기존 **스프링 JDBC**를 사용한 예고, 두 번째는 **JPA**를 사용한 예다.



- **스프링 JDBC를 사용한 코드**

```java
public Room getRoomById(String roomId) {
    String sql = "SELECT room_id, room_name, capacity FROM room WHERE room_id = ?";
    RowMapper<Room> rowMapper = new BeanPropertyRowMapper<Room>(Room.class);
    return getJdbcTemplate().queryForObject(sql, rowMapper, roomId);
}
```



- **JPA를 사용한 코드 예**

```java
@PersistenceContext
EntityManager entityManager;

public Room getRoomById(String roomId) {
	return entityManager.find(Room.class, roomId);
}
```



**JPA**를 사용한 경우 **스프링 JDBC**에서는 필요했던 **SQL**이 없어진 데다 **DB의 칼럼과 자바 객체와의 매핑**이 감춰진 것을 알 수 있다. 그럼 **JPA**가 어떤 구조로 **SQL**이나 **DB 정의**를 은폐하고 **JPA**가 언제 어떻게 DB에 접근하는지 살펴보자. 따라서 핵심은 **Entity**와 **EntityManager**다. 이 예에서는 **Room** 클래스와 **Entity**가 여기에 해당한다.







### 10.1.2 Entity

DB에서 영속적으로 저장된 데이터를 자바 객체로 매핑한 것을 **Entity**라고 한다. **Entity**는 메모리 상에 자바 객체의 인스턴스 형태로 존재하며, 이후에 설명할 **EntityManager**에 의해 DB의 데이터와 동기화된다. **Entity**는 **POJO**를 이용한 클래스로 기술할 수 있지만 **Entity**임을 **JPA** 구현에 인식시키거나 매핑에 필요한 정보를 추가하기 위해 **JPA**가 제공하는 애너테이션을 사용해야 한다.



> 애너테이션을 사용하지 않고 XML 파일에서 Entity 매핑을 정의할 수도 있다. 이 책에서는 애너테이션을 사용한 경우만 소개한다.



**JPA**에서는 객체를 고유하게 식별하기 위한 기본키를 **Entity** 프로퍼티와 테이블에 갖게 할 필요가 있다. 이 기본키에 의해 **Entity**와 **DB에 영속화된 데이터**가 연결된다. **Entity**의 구현 예로 **room** 테이블에 대한 **Entity** 클래스 정의를 다음과 같이 나타낸다.



**room 테이블**

| 칼럼          | 타입         | PK   | FK   |
| ------------- | ------------ | ---- | ---- |
| **room_id**   | INT NOT NULL | O    |      |
| **room_name** | VARCHAR(10)  |      |      |
| **capacity**  | INT          |      |      |



- **Entity 클래스의 구현 예**

```java
import javax.persistence.*;

@Entity 
@Table(name = "room") 
// name 속성에 매핑할 테이블명을 지정한다. 
// name 속성을 생략한 경우 클래스명을 대문자로 한 이름의 테이블(아래의 예에서는 'ROOM')에 매핑된다.
public class Room implements Serializable {
    // JPA 측에서는 Entity가 Serializable일 필요는 없지만 확장성을 고려해서 Serializable로 지정하고 있다.
    
    @Id 
    // 기본키임을 나타낸다. 기본키가 복합키로 된 경우에는 @javax.persistence.EmbeddedId를 사용해 대응할 수 있다.
    @GeneratedValue 
    // @javax.persistence.GeneratedValue 애너테이션을 부여해 기본키 생성을 JPA에 맡길 수 있다. 
    /* strategy 속성에 GenerationType을 지정해 생성 방법(시퀀스를 사용한다, 키 생성용 테이블을 사용한다 등)을 지정할 수 
       있지만 기본적으로는 GenerationType.AUTO 즉, 사용하는 DB의 최적의 키 생성 방법이 자동으로 선택된다. */
    @Column(name = "room_id")
    // @javax.persistence.Column 애너테이션을 부여하고 매핑되는 칼럼명을 지정한다. 
    // 생략한 경우 프로퍼티명을 대문자로 한 이름의 칼럼(이번 경우에는 'ROOMID')에 매핑된다.
    private Integer roomId;
    
    @Column(name = "room_name")
    private String roomName;
    
    @Column(name = "capacity")
    private Integer capacity;
    
    // 생략    
}
```



설명에서 알 수 있듯이 **JPA**에서는 **Entity**와 영속화된 데이터 간의 매핑에는 기본 규칙이 있어 기본값과 다른 경우에 **@Table**이나 **@Column** 등의 애너테이션에 매핑 룰을 지정한다. 그래서 **JPA **기본 매핑 규칙에 따라 테이블의 물리 설계를 하면 매핑 규칙 설정을 줄일 수 있다. 이러한 설정 사상을 **'Configuration by Exception(예외적인 상황에서만 명시적으로 설정한다)'**이라 한다. 또한 자바와 데이터베이스의 형변환도 **JDBC** 표준 매핑 규칙에 따라 수행하기 때문에 기본적으로는 설정이 필요없다. 또한 **JPA**에는 이 책에서 다 소개할 수 없을 만큼 많은 애너테이션을 통해 **Entity** 설정을 할 수 있다. 



### 10.1.3 EntityManager

**Entity**를 필요에 따라 DB와 동기화하는 역할을 담당하는 것이 **EntityManager**다. **EntityManager**에는 **영속성 컨텍스트(Persistence Context)**라는 **Entity**를 관리하기 위한 영역이 있다. 애플리케이션이 데이터베이스의 데이터에 접근하는 경우에는 반드시 **EntityManager**를 통해 **영속성 컨텍스트**의 **Entity**를 취득하거나 새로 생성한 **Entity**를 **영속성 컨텍스트**에 등록해야 한다. 이를 통해 **EntityManager**가 **Entity** 변경을 추적할 수 있어 적절한 타이밍에 데이터베이스와 동기화한다.







- **Entity와 EntityManager의 관계**

![](https://docs.google.com/drawings/d/sBBx_TEULSF6_35Qp6cDsgg/image?parent=e/2PACX-1vSgWs9-SkKDVDplyC96i2vj5TuZvbmLHvACG_NMQ3eEa1hFtJwCGbFZVRaHriS3HOeS8O0XIQ6gqqII&rev=525&h=336&w=601&ac=1)





**EntityManager**는 **Entity**의 상태를 변경하거나 데이터베이스와의 동기화를 위한 **API**(메서드)를 제공한다. 대표적인 **API**를 소개한다.



**EntityManager가 제공하는 대표적인 API**

| 메서드명                                                     | 설명                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| **\<T> T find(java.lang.Class\<T> entityClass, java.lang.Object primaryKey)** | 기본키를 지정해서 Entity를 검색하고 반환한다. 영속성 컨텍스트에 해당하는 Entity가 존재하지 않는 경우 데이터베이스에 SQL(SELECT 문)을 발행해 해당 데이터를 취득하고 Entity를 생성해서 반환한다. |
| **void persist(java.lang.Object entity)**                    | 애플리케이션에서 생성한 인스턴스를 Entity로 영속성 컨텍스트에서 관리한다. SQL의 INSERT 문에 해당하지만 persist 메서드를 실행한 시점에는 데이터베이스의 SQL은 실행되지 않고 영속성 컨텍스트에 축적된다. |
| **\<T> T merge(T entity)**                                   | 영속성 컨텍스트에서 관리되고 있지만 **분리 상태**가 된 Entity를 영속성 컨텍스트에서 다시 관리한다. **관리 상태**의 경우에는 차이점을 추적할 수 없기 때문에 데이터베이스에 변경을 반영하기 위한 SQL(UPDATE 문)이 영속성 컨텍스트에 축적된다. |
| **void remove(java.lang.Object entity)**                     | Entity를 영속성 컨텍스트 및 데이터베이스에서 삭제한다. persist 메서드나 merge 메서드와 마찬가지로 데이터베이스에 SQL(DELETE 문)이 즉시 발행되지 않고 영속성 컨텍스트에 축적된다. |
| **void flush()**                                             | 영속성 컨텍스트에 축적된 모든 Entity의 변경 정보를 데이터베이스에 강제적으로 동기화한다. 일반적으로 데이터베이스에 반영하는 작업은 트랜잭션을 커밋할 때 하지만 커밋 이전에 반영할 필요가 있는 경우에 사용한다. |
| **void refresh(java.lang.Object entity)**                    | Entity의 상태를 데이터베이스의 데이터로 강제로 변경한다. 데이터베이스에 반영되지 않는 Entity에 대해 변경된 사항은 덮어쓰게 된다. |
| **\<T> TypedQuery\<T> createQuery(java.lang.String qlString, java.lang.class\<T> resultClass)** | 기본키 이외의 것으로 데이터베이스에 접근하는 경우에는 JPA용 쿼리를 실행해 Entity를 취득하거나 변경할 수 있다. 이 API는 쿼리를 작성하기 위한 API 중 하나로서 비슷한 API가 여러 개 제공된다. |
| **void detach(java.lang.Object entity)**                     | Entity를 영속성 컨텍스트에서 삭제하고 **분리 상태**로 만든다. 이 Entity에 대해 변경된 모든 사항은 merge 메서드를 실행하지 않는 한 데이터베이스에 반영되지 않는다. |
| **void clear()**                                             | 영속성 컨텍스트에서 관리되는 모든 Entity를 **분리 상태**로 만든다. |
| **boolean contains(java.lang.Object entity)**                | Entity가 영속성 컨텍스트에서 관리되는지를 반환한다.          |



여기서 핵심은 **영속성 컨텍스트**가 데이터베이스의 **캐시**와 같은 역할을 하고 있으며 **EntityManager**에 대한 작업이 수행되더라도 즉시 데이터베이스에 반영되지 않는다는 것이다. 다시 말해 SQL이 발행되지 않는다는 점이다. 트랜잭션이 커밋되어 종료되거나 애플리케이션에서 강제적으로 **flush** 메서드를 호출한 타이밍에 영속성 컨텍스트에 축적된 **Entity**의 변경 사항이 데이터베이스에 반영된다.



**EntityManager**는 **JPA** 구현이 제공하는 **EntityManagerFactory**에 의해 생성된다. 그러나 **Java EE**의 **EJB** 컨테이너를 사용하는 경우에는 **@javax.persistence.PersistenceContext** 애너테이션을 필드에 추가하면 **EJB** 컨테이너가 생성한 **EntityManager**를 취득할 수 있기 때문에 애플리케이션이 직접 **EntityManagerFactory**를 사용할 필요는 없다. 스프링을 사용하는 경우에도 **EJB** 컨테이너와 마찬가지로 **EntityManager**를 주입하는 기능이 제공된다.



**영속성 컨텍스트**는 트랜잭션마다 준비되기 때문에 **Entity**는 같은 트랜잭션에서만 공유되고 다른 트랜잭션에서 처리 중인 **Entity**는 보여지지 않는다. 당연히 여러 트랜잭션에 걸쳐 **Entity**를 관리할 수도 없다.



나중에 소개할 **스프링 데이터 JPA**에서는 **EntityManager**의 존재가 감춰져서 사용자가 **EntityManager**를 직접 조작할 필요가 없다. 그러나 **EntityManager**가 배후에 존재한다는 점이나 다음에 소개할 **Entity**의 상태를 제대로 이해하지 않으면 예기치 않은 동작을 일으키는 경우가 있기 때문에 주의할 필요가 있다.



### 10.1.4 Entity 상태

**JPA**에서는 **Entity**의 상태를 다음과 같이 정의한다.



| 상태                     | 설명                                                         |
| ------------------------ | ------------------------------------------------------------ |
| **new 상태**             | 새로운 Entity의 인스턴스가 생성되고 영속성 컨텍스트에 등록되지 않은 상태. 이 시점에서 Entity는 단지 자바 객체이며 EntityManager는 아무것도 하지 않음 |
| **관리 상태**            | 영속성 컨텍스트에 Entity가 등록된 상태. EntityManager에 의해 데이터베이스와의 동기화가 활성화된다. |
| **분리 상태**            | 관리 상태였던 Entity가 영속성 컨텍스트에서 분리된 상태. new 상태와 마찬가지로 이 상태에서는 데이터베이스와 동기화되지 않지만 관리 상태로 되돌릴 수단이 제공된다. |
| **삭제된 상태         ** | 데이터베이스에서 삭제되는 것이 예정된 상태. EntityManager가 데이터베이스의 데이터를 삭제하고 종료될 때까지 이 상태가 계속된다. |



각 상태의 변화와 전환을 일으키는 주요 트리거는 다음 그림과 같다. **EntityManager**가 제공하는 각 **API**는 **Entity** 상태에 따라 실행 여부가 나뉘기 때문에 **JPA** 사용자는 **Entity**가 어떤 상태인지 제대로 파악하고 있어야 한다. 특히 중요한 것은 트랜잭션이 종료되면 영속성 컨텍스트의 모든 **Entity**가 **분리 상태**로 전환된다는 점이다. 여러 트랜잭션을 횡단하거나 트랜잭션 외부에서 **Entity**에 접근하는 경우에는 **Entity**가 분리 상태로 돼 있고 데이터베이스와 동기화되지 않은 상태라는 점에 주의한다.



![](https://docs.google.com/drawings/d/swkrj1hyi9naHYD5F2TONJg/image?parent=e/2PACX-1vSgWs9-SkKDVDplyC96i2vj5TuZvbmLHvACG_NMQ3eEa1hFtJwCGbFZVRaHriS3HOeS8O0XIQ6gqqII&rev=293&h=393&w=601&ac=1)



### 10.1.5 연관관계

관계형 데이터베이스에서는 연관된 테이블 간의 관계를 정의할 수 있는데, 이때 테이블 간의 연결 고리 역할을 하는 칼럼을 **외부 키**로 정의하게 된다. **JPA**에서는 **데이터베이스의 연관관계**를 **Entity 간의 참조 관계**로 매핑한다. 이 같은 연관관계가 제대로 매핑되게 하려면 **EntityManager**가 각 **Entity** 간의 관계를 사전에 제대로 인식할 필요가 있는데, 이를 위해 연관관계를 명시적으로 정의하기 위한 몇 가지 기법이 준비돼 있다. 연관관계는 두 객체 간의 **카디널리티**(일대일, 일대다, 다대다)와 **방향**에 따라 다음과 같은 유형으로 분류될 수 있다.

- **단방향 일대일**
- **양방향 일대일**
- **단방향 일대다**
- **단방향 다대일**
- **양방향 일대다/다대일**
- **단방향 다대다**
- **양방향 다대다**



예를 들어, **하나의 주문**이 **상품**마다 명세서를 **여러 주문 명세서**로 관리하는 경우에는 **주문**과 **주문 명세서**의 관계는 **양방향 일대다/다대일**, **주문 명세서**와 **상품**의 관계는 **단방향 다대일**이다. 관계도 **Configuration  by Exception** 사상에 따라 **JPA**가 프로퍼티 타입 등에서 관계를 추측해서 인식하지만 명시적으로 관계를 정의하는 것도 가능하다. 이 책에서는 앞에서 설명한 **room** 테이블과 새로 등장하는 **equipment** 테이블을 통해 애너테이션을 이용해 **Entity** 간의 관계를 정의하는 방법을 소개한다.



**equipment 테이블**

| 칼럼                  | 타입         | PK   | FK   |
| --------------------- | ------------ | ---- | ---- |
| **equipment_id**      | INT NOT NULL | O    |      |
| **room_id**           | INT          |      | O    |
| **equipment_name**    | VARCHAR(30)  |      |      |
| **equipment_count**   | INT          |      |      |
| **equipment_remarks** | VARCHAR(100) |      |      |



**equipment** 테이블에 **room**에 대한 관계를 연결하기 위한 외부 키 **room_id**를 준비하고 있다. 같은 **room**에는 여러 **equipment**가 놓여져 있지만 같은 비품이 여러 방에 배치되는 것은 불가능하기 때문에 **room**과 **equipment**는 **일대다/다대일**의 관계다. 어떤 방에 어떤 비품이 놓여있는지 궁금하지만 반대로 비품이 어느 방에 놓여있는지 궁금한 경우도 있다. 이러한 경우에는 **양방향 일대다/다대일**의 관계를 양쪽 **Entity**에 설정한다.



- **다대일 관계를 맺는 Entity 클래스의 구현 예**

```java
@Entity
@Table(name="equipment")
public class Equipment implements Serializable {
    @Id
    @GeneratedValue
    @Column(name="equipment_id")
    private Integer equipment;
    
    @Column(name="equipment_name")
    private String equipmentName;
    
    @ManyToOne 
    // 양방향 관계이기 때문에 equipment에서 room에 접근 가능하며 다대일의 관계다.
    // 이 경우 room에 참조를 나타내는 프로퍼티를 준비하고 @javax.persistence.ManyToOne 애너테이션을 부여한다.
    @Column(name="room_id")
    // 연결된 room을 특정하기 위한 외부 키의 칼럼을 @javax.persistence.MayToOne 애너테이션에
    // 연결하고 @javax.persistence.JoinColumn으로 설정한다.
    private Room room;
    // equipment에서 보면 room은 일에 해당하기 때문에 Room 클래스로 room 프로퍼티를 준비한다.
    
    @Column(name="equipment_count")
    private Integer equipmentCount;
    
    @Column(name="equipment_remarks")
    private String equipmentRemarks;
}
```



- **일대다 관계를 맺는 Entity 클래스의 구현 예**

```java
@Entity
@Table(name="room")
public class Room implements Serializable {
	@Id
    @GeneratedValue
    @Column(name="room_id")
    private Integer roomId;
    
    @Column(name="room_name")
    private String roomName;
    
    @Column(name="capacity")
    private Integer capacity;
    
    @OneToMany(mappedBy="room", cascade=CascadeType.ALL)
    // room에서 본 equipment는 일대다이기 때문에 @javax.persistence.OneToMany 애너테이션을 
    // 부여하고 equipment에 일대다 관계를 설정한다. 양방향 관계의 경우 외부 키 정보, 즉 room과
    // equipment 간의 관계를 맺고 있는 Equipment 프로퍼티명(이 예제의 경우는 room)을
    // mappedBy 요소에 지정할 수 있다. 또한 cascade 속성을 지정하면 자신에 대한 조작을
    // 관련 Entity에도 전파할 수 있다.
    private List<Equipment> equipments;
    // room에서 본 equipment는 다이기 때문에 List 타입으로 equipments 프로퍼티를 준비해야 한다.
}
```



> 일대다의 다 관계를 저장하는 프로퍼티 타입에는 컬렉션 타입을 사용할 수 있지만 **List** 타입이 아닌 **Set** 타입을 사용한 경우에는 주의할 필요가 있다. **JPA** 구현으로 **Hibernate**를 사용하고 **Set** 타입을 사용한 경우 **분리 상태**에 있는 **Entity**의 동일성 평가를 제대로 하려면 **Entity**의 **equals** 메서드와 **hashCode** 메서드를 재정의해야 한다.





다음으로 애플리케이션이 연관관계 정보를 이용해 관련 Entity를 가져오는 방법을 살펴보자.



- **연관관계가 있는 Entity를 가져오는 예**

```java
@PersistenceContext
EntityManager entityManager;

@Transactional(readOnly=true)
public List<Equipment> getEquipmentsInRoom(Integer roomId) {
    Room room = entityManager.find(Room.class, roomId);
    // (1) 기본키를 검색 조건으로 사용해 데이터베이스에서 room을 가져온다.
    return room.getEquipments();
    // (2) room과 관련된 equipment 목록을 접근자 메서드를 통해 가져온다. equipments 프로퍼티에는 연관관계가 정의돼 있기 때문에 외부 키의 room_id와 연결된 equipment의 모든 레코드가 데이터베이스에서 조회된다. 데이터베이스에 접근하거나 프로퍼티에 데이터를 저장하는 것은 JPA가 알아서 처리하기 때문에 사용자는 단지 접근자 메서드만 호출하면 된다.
}

@Transactional(readOnly=true)
public Room getRoomOfEquipment(Integer equipmentId) {
    Equipment equipment = entityManager.find(Equipment.class, equipmentId);
    // (3) 기본키를 검색 조건으로 사용해 equipment를 데이터베이스에서 가져온다.
    return equipment.getRoom();
    // (4) equipment에서 볼 때 room은 최대 1개만 연관관계를 맺을 수 있기 때문에 가져온 결과는
    // 1개의 room이나 null이 된다.
}
```



이처럼 연관관계를 가진 **Entity** 본체에서 프로퍼티에 접근하는 것만으로도 연관관계에 있는 다른 **Entity**에 접근할 수 있는 것이 **JPA**의 큰 강점이다. 실제로는 내부적으로 다음 중 한 시점에서 연관관계에 있는 **Entity** 데이터를 가져오기 위한 **SQL**이 실행된다.

- **(2), (4)**와 같이 **Entity**와 매핑된 프로퍼티에 처음 접근할 때 SQL을 실행할 수 있으며, 이를 **Lazy 페치**라고 한다.

- **(1), (3)**과 같이 **본체 Entity**에서 **EntityManager**의 **API**가 사용될 때 SQL을 실행할 수 있으며, 이를 **Eager 페치**라고 한다.



이처럼 SQL을 실행해서 데이터를 가져오는 과정을 **JPA**에서는 '**페치(fetch)**'라고 한다. 기본적으로 **@OneToOne, @ManyToOne**은 **Eager 페치**를 사용하고, 그 밖의 경우에는 **Lazy 페치**를 사용한다. 만약 연관된 **Entity**를 페치하는 방법을 명시적으로 지정하고 싶다면 연관관계를 정의하는 애너테이션에서 **fetch** 속성을 사용하면 된다. 다음은 연관관계를 가진 **equipments**를 **Eager 페치**로 읽어오는 설정이다.



- **페치 방법의 지정**

```java
@OneToMany(mappedBy="room", cascade=CascadeType.ALL, fetch=FetchType.EAGER)
private List<Equipment> equipments;
```



### 10.1.6 JPQL(Java Persistence Query Language)

지금까지 **EntityManager**가 제공하는 **API**를 사용해 **기본키**를 지정한 후 데이터베이스를 조회하거나 갱신하는 방법을 살펴봤다. 하지만 일반적인 애플리케이션을 개발할 때는 **기본키**를 지정하지 않고도 데이터베이스의 데이터를 처리해야 하는 일이 많은데, 예를 들어 전체 목록을 조회하거나 특정 검색에 대한 결과를 표시해야 할 때, 혹은 일괄적으로 데이터를 갱신해야 하는 경우가 이에 해당한다. 그래서 **JPA**에서는 **기본키**를 지정해 DB를 조작하는 방법 외에도 **기본키를 사용하지 않는** 처리 방법도 함께 제공한다. **JPA**는 이러한 처리를 위해 **SQL**과 같은 유연한 쿼리를 가능하게 하는 기법을 몇 가지 제공하고 있으며, 다음과 같은 특징이 있다.



**JPQL 쿼리를 기술하는 방법**

| 방법                                      | 설명                                                         |
| ----------------------------------------- | ------------------------------------------------------------ |
| **JPQL(Java Persistence Query Language)** | SQL처럼 JPA 독자적인 쿼리 언어를 써서 Entity를 가져오거나 값을 변경할 때 사용한다. SQL이 데이터를 입출력할 때 테이블이나 테이블의 행과 열로 표현하는 반면, JPQL은 Entity나 Entity의 컬렉션, 그리고 Entity의 프로퍼티명으로 표현한다는 점이 다르다. |
| **Criteria Query**                        | JPQL과 개념은 비슷하나 좀 더 객체지향적으로 기술하는 방법으로, JPA 2.0에서 추가됐다. JPQL은 문자열로 기술되기 때문에 타입 검사를 컴파일 시점으로 할 수 없어 타입 불일치 같은 오류가 잠재적으로 발생할 수 있다. 한편 Criteria Query는 문자열이 아니라 Builder 패턴의 CriteriaQuery 객체를 이용해 자바 코드처럼 쿼리를 기술하기 때문에 컴파일 시점에 타입 검사를 할 수 있어 쿼리 작성 과정에서 발생하는 실수를 미연에 방지할 수 있다. |
| **Native Query**                          | SQL을 직접 작성해서 Entity를 취득하거나 갱신하는 방법으로, 성능 등의 다양한 이유로 데이터베이스 제품에 의존적인 최적화된 기능이 필요할 때 이 방법을 사용하낟. |



이 책에서는 **JPQL**을 활용하는 방법에 한해서 소개하겠다. **JPQL**에는 **SQL**에 비해 많은 기능이 있지만 여기서는 기본적인 **JPQL**의 사용 예를 보면서 **JPQL**의 사용법을 살펴보겠다.



- **JPQL 이용 예**

```java
@PersistenceContext
EntityManager entityManager;

@Transactional(readOnly=true)
public List<Room> getRoomByName(String roomName) {
    String jpql = "SELECT r FROM Room r WHERE r.roomName = :roomName";
    // 이 예에서는 room 테이블의 모든 레코드를 Entity로 읽어 들이기 위한 JPQL을 기술하고 있다.
    // SQL로 대체하면 SELECT * FROM room r WHERE r.room_name = ? 처럼 되고
    // 테이블 이름 'room'과 칼럼명 'room_name', *를 사용하게 되는데 JPQL에서는 이것들을
    // Entity명이나 그것의 프로퍼티명으로 대체한다. SQL의 * 대신 JPQL에서는 Entity의 별명을 사용한다.
    TypedQuery<Room> query = entityManager.createQuery(jpql, Room.class);
    // EntityManager에서 제공되는 API를 사용해 문자열의 JPQL을 TypedQuery로 컴파일한다. 
    // 엄밀히 말하면 다르지만 JDBC의 PreparedStatement에 해당한다.
    query.setParameter("roomName", roomName);
    // JPQL에 설정한 바인드 변수(':변수명' 형식)에 바인드 값을 설정한다.
    return query.getResultList();
    // 쿼리를 실행한다. 여기서는 데이터베이스의 데이터를 조회하는 SELECT 쿼리를 실행한다.
    // 조회 결과를 List 타입으로 취득하고 싶은 경우에는 TypedQuery.getResultList 메서드를 사용한다.
}
```



> 이 예에서는 문자열 객체로부터 **TypedQuery**를 생성하지만 **JPQL**의 수가 늘어나면 문자열 객체 관리가 번거로워진다. **Named Query**라는 **JPA** 기능을 사용하면 **JPQL**에 이름을 붙여 관리할 수 있고 **@NamedQuery** 애너테이션에 **JPQL**을 기술할 수 있게 된다. 관계가 강한 **Entity**에 애너테이션을 부여하고 정리하면 **JPQL**을 관리하기가 쉬워진다.



**JPA**에서는 **Entity**를 통해 **SQL** 존재를 감추면서 한편으로는 자유로운 고급 쿼리 기능을 제공함으로써 기능성을 해치지 않는다. 한편 쿼리를 생성하고 매개변수를 바인드해서 쿼리를 실행하는 구조는 **JDBC**로 되돌아가는 것처럼 보여 **JPA**의 장점이 훼손됐다고 느끼는 사람도 있을 것이다. 지금부터 소개하는 **스프링 데이터 JPA**를 이용하면 **JPA 쿼리**를 사용할 때의 번거로움이 조금 해소된다.



## 10.2 JPA를 이용한 데이터베이스 접근 기초

이번 절에서는 **JPA**를 사용하는 데 필수적인 기본적인 DB 조작 방법을 구체적으로 살펴보겠다.



### 10.2.1 JPA에 의한 CRUD 작업

먼저 **EntityManager**에서 제공하는 **Entity** 조작 **API**를 이용해 **Entity**에 대해 **CRUD** 작업을 해보겠다.



- **JPA를 이용한 CRUD 조작**

```java
@Service
public class RoomServiceImpl implements RoomService {
    
    @PersistenceContext
    EntityManager entityManager;
    // JPA 구현이 제공하는 EntityManager를 주입한다. 이 인스턴스가 EJB 컨테이너에서 관리되지 않는
    // 경우에도 스프링의 DI 기능에 따라 @PersistenceContext를 통해 EntityManager를 주입할 수 있다. // 다만 10.5절 '스프링 데이터 JPA 설정'에서 설명한 방법으로 스프링이 JPA 구현을 인지할 수 있는
    // 타입으로 설정할 필요가 있다.
    
    @Transactional(readOnly=true)
    // 스프링이 제공하는 트랜잭션 기능인 @Transactional 애너테이션을 통해 JPA 트랜잭션 관리를 선언적으로 한다. 다만 10.5절 '스프링 데이터 JPA 설정'에서 설명한 방법으로 트랜잭션을 설정할 필요가 있다. 스프링 기능을 사용하지 않고 JPA 트랜잭션을 관리하는 경우 EntityManager.getTransaction 메서드로 EntityTransaction을 취득하고 명령적으로 트랜잭션을 시작 및 종료한다.
    public Room getRoom(Integer id) {
        Room room = entityManager.find(Room.class, id);
        // find 메서드로 Entity를 1건 취득한다. 취득한 Entity는 관리 상태가 된다. 대상 Entity가
        // 존재하지 않는 경우에는 null이 반환된다.
        if(room == null) {
            // 검색 대상이 없을 때 처리할 내용 (생략)
        }
        return room;
    }
    
    
    @Transactional
    public Room createRoom(String roomName, Integer capacity) {
        Room room = new Room();
        room.setRoomName(roomName);
        room.setCapacity(capacity);
        entityManager.persist(room);
        // persist 메서드로 새롭게 생성된 new 상태의 Entity를 관리 상태로 전환한다. persist 메서드가 호출된 시점에는 데이터베이스에 반영되지 않는다. 메서드가 호출된 후 Entity에 대한 변경 사항을 포함해서 트랜잭션이 종료될 때 데이터베이스에 반영된다.
        return room;
    }
    
    
    public Room updateRoomName(Integer id, String roomName) {
        Room room = getRoom(id);
        // Entity의 내용을 변경하기 위해 기본키를 지정해 Entity를 영속성 컨텍스트에서 취득한다.
        // 취득한 Entity는 이미 관리 상태이므로 트랜잭션이 종료될 때 변경 사항이 데이터베이스에 반영된다.
        room.setRoomName(roomName);
        return room;
    }
    
    
    @Transactional
    public void deleteRoom(Integer id) {
        Room room = getRoom(id);
        // Entity 삭제를 위해 기본키를 지정해 Entity를 영속성 컨텍스트에서 취득한다. 
        // 삭제하려면 대상 Entity가 관리 상태로 돼 있어야 하기 때문이다.
        entityManager.remove(room);
        // 관리 상태의 Entity를 remove 메서드로 삭제한다. Entity는 삭제된 상태가 되고 트랜잭션이 
        // 종료될 때 데이터베이스의 레코드가 삭제된다.
    }
}
```





### 10.2.2 JPA의 JPQL을 활용한 데이터 접근

**JPQL**을 사용해 **SQL**을 실행하는 방법을 다시 살펴보자. 여기서는 조금 더 응용한 예로 **JOIN FETCH 절**을 사용해 **관계 Entity**를 읽어오는 쿼리를 작성해 보겠다.

| room_id | room_name |
| ------- | --------- |
| **1**   | Room A    |
| **2**   | Room B    |
| **3**   | Room C    |



| equipment_id | equipment_name | room_id |
| ------------ | -------------- | ------- |
| **1**        | White Board    | 1       |
| **2**        | Desk           | 1       |
| **3**        | Chair          | 1       |
| **4**        | Computer       | 2       |
| **5**        | Monitor        | 2       |
| **6**        | Telephone      | 3       |



![](https://docs.google.com/drawings/d/svFMOQug0CCgKFQrKNtYbLQ/image?parent=e/2PACX-1vSgWs9-SkKDVDplyC96i2vj5TuZvbmLHvACG_NMQ3eEa1hFtJwCGbFZVRaHriS3HOeS8O0XIQ6gqqII&rev=184&h=356&w=601&ac=1)



앞에서 관계를 맺는 **Entity**에 대해 **SELECT** 문의 **JPQL**을 실행한 경우 **관계 Entity** 프로퍼티의 데이터가 읽어 들이는 시점은 **페치 종류**에 따라 달라진다고 소개한 바 있다. 하지만 **관계 Entity**의 프로퍼티를 읽을 때면 **페치 종류**와는 상관없이 **SQL**이 실행되고 설상가상으로 **Lazy 페치**를 하게 될 때는 관계 **Entity**의 개수만큼 **SQL**이 실행될 수도 있다. 특히 **Entity**의 건수가 많다면 대량의 **SQL**이 실행되어 성능 저하가 발생할 우려가 있다.



**JPQL**에서는 데이터를 조회할 때 **JPQL**에 **JOIN FETCH** 절을 사용하면 **본체 Entity**와 **관계 Entity**를 결합한 후, **본체 Entity**와 **관계 Entity** 모두를 한 번의 **SQL** 실행으로 읽어 들일 수 있다. **JOIN FETCH**는 **관계 Entity**에 지정한 **페치**의 종류와 관계없이 적용할 수 있다. 원래 **JPQL**에서는 **SQL**처럼 **SELECT **절에 **JOIN**이나 **LEFT JOIN** 절을 추가해서 **Entity**를 결합할 수 있지만 **FETCH**를 지정하지 않는 한 관계 **Entity** 프로퍼티에 읽어 들이지 않는다. **JOIN FECTH**를 사용할 때 주의할 점은 일대다나 다대다 관계의 **Entity**인 경우에는 그 수만큼 **본체 Entity**가 중복으로 반환된다는 것이다. 이러한 중복을 피하려면 **DISTINCT** 절을 추가한다.



- **JOIN FECTH를 이용해 관계 Entity 읽어오기**

```java
@Service
public class RoomServiceImpl implements RoomService {
	// 생략
    
    @Transactional(readOnly=true)
    public List<Room> getRoomsByFetch(String roomName) {
        String jpql = "SELECT DISTINCT r FROM Room r" +
            	"LEFT JOIN FETCH r.equipments " +
            	"WHERE r.roomName = :roomName";
        TypedQuery<Room> query = entityManager.createQuery(jpql, Room.class);
        query.setParameter("roomName", roomName);
        return query.getResultList();
    }

}
```





#### 데이터베이스 갱신

데이터베이스를 **참조**하는 **SELECT 문**의 JPQL 사용법을 소개했지만 데이터베이스를 **갱신**하는 **INSERT, UPDATE, DELETE**를 사용한 **JPQL**을 실행하는 것도 가능하다. 데이터를 변경하는 쿼리를 실행하는 것은 대량의 데이터를 갱신할 때 성능 면에서 효과적인 수단이다. 대량의 데이터를 갱신하고 싶은 경우 **Entity**를 1건씩 관리 상태로 갱신해버리면 성능 저하의 원인이 되기 때문에 일반적인 쿼리를 사용한다.



- **JPQL로 데이터를 변경하는 예**

```java
@Service
public class RoomServiceImpl implements RoomService {
    // 생략
    
    @Transactional
    public Integer updateCapacityAll(Integer capacity) {
        String jpql = "UPDATE Room r SET r.capacity = :capacity";
        // UPDATE 문의 JPQL을 정의한다. 데이터를 조회하는 쿼리와 마찬가지로 SQL에 대한 데이터베이스의 테이블명이나 칼럼명 대신 Entity 이름이나 프로퍼티명을 지정한다.
        Query query = entityManager.createQuery(jpql);
        // Query를 생성한다. 데이터를 변경하는 경우에는 TypedQuery 대신 Query를 사용한다.
        query.setParameter("capacity", capacity);
        // 쿼리에 매개변수를 바인드한다. 데이터를 조회하는 쿼리와 차이는 없다.
        
        return query.executeUpdate();
        // 데이터를 변경하는 쿼리를 실행하는 경우 TypedQuery.executeUpdate 메서드를 사용한다.
        // UPDATE 문의 SQL과 마찬가지로 갱신된 레코드 건수가 int 타입의 반환값으로 반환된다.
        // Entity 조작과 달리 SQL은 바로 실행된다.
    }
}
```



다만 데이터를 변경하는 **JPQL**을 실행한 경우에는 이미 관리 상태로 된 **Entity**에 변경 사항이 반영되지 않는다. 이미 관리 상태로 돼 있는 Entity에 변경 사항을 반영하고 싶은 경우에는 **JPQL**을 실행한 후 refresh 메서드를 호출해야 한다.





### 10.2.3 배타 제어

웹 애플리케이션에서는 **동시**에 여러 트랜잭션이 실행되는 것이 일반적이다. 그래서 데이터베이스에 대한 갱신 처리에서는 **배타 제어**를 고려해야 한다. 당연히 JPA에는 배타 제어하는 기능이 있다. 데이터베이스의 배타 제어 방법에는 **낙관적 잠금(optimistic lock)**과 **비관적 잠금(pessimistic lock)**이 있으며 시스템 요구사항이나 처리 특성을 고려해서 방식을 선택하는 것이 일반적이다. **JPA 2.0**에서 비관적 잠금이 추가되어 현재 JPA에서는 낙관적 잠금과 비관적 잠금이 모두 지원된다.



#### 낙관적 잠금

- **낙관적 잠금의 사용 예**

```java
@Entity
@Table(name="room")
public class Room implements Serializable {
	// 생략
	
	@Version
    // 낙관적 잠금을 사용하는 경우 대상 Entity가 서로 구분되도록 반드시 Versioning해야 한다. 
    // Versioning하려면 Versioning 전용 프로퍼티를 준비하고 @javax.persistence.Version
    // 애너테이션을 부여한다.
	@Column(name="version")
	private Integer version;
    // Versioning을 하기 위한 프로퍼티를 준비한다. 타입은 Integer 같은 정수형 외에도 Timestamp를
    // 사용할 수 있다. Versioning을 위해 JPA 내부에서 이 프로퍼티가 갱신되기 때문에 애플리케이션에서
    // 직접 이 프로퍼티를 갱신하는 것은 금지된다.
    
	
	// 생략
}
```



```java
@Service
public class RoomServiceImpl implements RoomService {
	@PersistenceContext
    private EntityManager entityManager;
    
    // 생략
    
    @Transactional
    public void updateRoomWithOptimisticLock(Integer id, String roomName, Integer capacity) {
        Room room = entityManager.find(Room.class, id);
        entityManager.lock(room, LockModeType.OPTIMISTIC);
        // 낙관적 잠금을 활성화한다. EntityManager.lock 메서드 외에도 EntityManager.find 메서드의 인수에 LockModeType을 지정해 락을 활성화할 수도 있다. 쿼리에 대해 락을 활성화하는 경우에는 TypedQuery.setLockMode 메서드를 사용한다. 다만 활성화할 수 있는 쿼리는 데이터를 조회하는 쿼리로 제한된다.  
        
        
        // 갱신 처리 (생략)
        // 낙관적 잠금이 실패할 경우 트랜잭션이 종료되는 시점에 OptimisticLockException이 발생.
    }
}
```



**낙관적 잠금**은 잠금 대상 Entity의 **Versioning**에 의해 구현된다. 따라서 **Entity**의 **@Version** 애너테이션을 부여한 프로퍼티가 반드시 있어야 한다. 갱신 작업이 정상적으로 처리된 경우에는 트랜잭션마다 버전이 증가한다. 만약 다른 트랜잭션에 의해 같은 행에 갱신이 완료된 경우에는 데이터베이스에 갱신 정보를 반영하려고 한 시점(트랜잭션 종료 시점 등)에서 예상한 버전과 다른 버전이 감지되고 **JPA** 구현이 **OptimisticLockException**을 발생시킨다. 또한 트랜잭션이 종료될 때 낙관적 잠금에 실패한 경우는 **OptimisticLickException**을 wrapping한 **RollbackException**이 발생한다.



> **JPA** 구현이 **Hibernate**인 경우 트랜잭션이 종료될 때 낙관적 잠금이 실행된 경우 **JPA**에서 규정돼 있는 **OptimisticLockException**이 아니라 **Hibernate** 고유 예외인 **org.hibernate.StaleObjectStateException**이 발생한다. **flush()**를 사용할 때 낙관적 잠금에 실패한 경우에는 **OptimisticLockException**이 발생한다.







