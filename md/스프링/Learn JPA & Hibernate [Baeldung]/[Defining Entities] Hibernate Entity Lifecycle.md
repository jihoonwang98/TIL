# Hibernate Entity Lifecycle :cyclone:



> **[참고]**: Hibernate Entity Lifecycle
>
> https://www.baeldung.com/hibernate-entity-lifecycle
>
> 의역과 오역에 주의...





## 1. Overview

모든 Hibernate Entity는 **lifecycle**을 가지고 있다.

**Lifecycle**의 종류는 다음과 같다.

- **transient (new, 비영속)** 
- **managed (영속)**
- **detached (준영속)**
- **deleted (삭제)**





## 2. Helper Methods

우리는 이 JPA tutorial을 진행하면서, 몇몇 **helper method**들을 꾸준히 사용할 것이다.



- *HibernateLifecycleUtil.getManagedEntities(session)*
  - 이 메서드는 Session의 내부 저장소에서 모든 managed entitiy들을 꺼내오는데 사용한다.

- *DirtyDataInspector.getDirtyEntities()*
  - 이 메서드는 'dirty'한 모든 entity들의 list를 가져오는데 사용한다.

- *HibernateLifecycleUtil.queryCount(query)*
  - count(*) query를 날리는 편리한 메서드다.





## 3. It's All About Persistence Context

이번 포스팅의 주제인 **Entity Lifecycle**에 대한 얘기를 하기 앞서,

우리는 먼저 **Persistence Context(영속성 컨텍스트)**에 대한 이해가 필요하다.



간단히 말해, **Persistence Context**는 **client code**와 **실제 data 저장소(DB)** 사이에 위치하는 놈이다.

이 공간은 persistent data가 entity로 변환되고, 값이 읽혀질 준비를 하고, client code에 의해 값이 바뀌어지는 임시 공간이다.

좀더 이론적으로 설명하자면, **Persistence Context**는 [Unit of Work](https://martinfowler.com/eaaCatalog/unitOfWork.html) 패턴의 구현체다.

이 컨텍스트는 모든 load된 data들을 추적하며, 이 data들의 변화를 추적하고,

business 트랜잭션의 마지막에는 모든 변화들을 DB와 동기화시키는 책임을 가지고 있다.



**JPA EntityManager**와 **Hibernate의 Session**은 모두 **Persistence Context** 컨셉의 구현체들이다.

이번 포스팅에서는, 편의상 **Persistence Context**를 **Hibernate Session**로 표현한다.



**Hibernate Entity Lifecycle 상태**는 어떻게 **Entity**가 **Persistence Context**와 연관되는지 잘 설명해준다.





## 4. Managed Entity (영속 엔터티)

**Managed Entity**는 **DB 테이블 행**을 나타낸다. (비록 그 행이 DB에는 아직 존재하지 않지만..)

이 Entity는 현재 작동중인 *Session*에 의해 관리된다. 

그리고 이 Entity에 생긴 모든 변화들은 자동적으로 추적되고 DB로 전파된다.



*Session*은 또한 DB로부터 Entity를 불러오거나 detached Entity를 다시 re-attach 하기도 한다.





코드를 통해 직접 확인해보도록 하자.

우리의 sample 애플리케이션은 *FootballPlayer*라는 클래스를 하나의 **Entity**로 정의하고 있다.

애플리케이션이 시작할 때, 우리는 DB에 sample data로 초기화를 시킬 것이다.

```
+-------------------+-------+
| Name              |  ID   |
+-------------------+-------+
| Cristiano Ronaldo | 1     |
| Lionel Messi      | 2     |
| Gigi Buffon       | 3     |
+-------------------+-------+
```



우리가 위 DB의 3번 ID의 data를 **`Gifi Buffon`** 대신 full name인 **`Gianluigi Buffon`**으로 바꾸고 싶다고 하자.



먼저, 우리는 *Session* 객체를 얻음으로써 우리의 unit of work를 시작해야 한다.

```java
Session session = sessionFactory.openSession();
```



In a server environment, we may inject a *Session* to our code via a context-aware proxy. The principle remains the same: we need a *Session* to encapsulate the business transaction of our unit of work. (??)



다음으로, 우리는 *Session*에게 persistent store에서 data를 불러오라고 지시해야 한다.

```java
assertThat(getManagedEntities(session)).isEmpty();

List<FootballPlayer> players = session.createQuery("from FootballPlayer").getResultList();

assertThat(getManagedEntities(session)).size().isEqualTo(3);
```



우리가 처음으로 *Session*을 얻어올 때, *Session*의 **persistence context store**는 비어있다. (첫 줄의 assert문이 증명하는 것처럼)

다음으로, 우리는 DB로부터 data를 받아오는 query를 수행한다. 

내부적으로 *Session*은 자신이 **persistence context store**에서 불러온 모든 entity들을 추적한다.

위의 사례에서는 query 수행 이후, *Session*의 내부 **store**에 3개의 entity가 포함되어 있을 것이다.





이제 부폰의 이름을 바꿔보자. :do

```java
Transaction transaction = session.getTransaction();
transaction.begin();

FootballPlayer gigiBuffon = players.stream()
  .filter(p -> p.getId() == 3)
  .findFirst()
  .get();

gigiBuffon.setName("Gianluigi Buffon");
transaction.commit();

assertThat(getDirtyEntities()).size().isEqualTo(1);
assertThat(getDirtyEntities().get(0).getName()).isEqualTo("Gianluigi Buffon");
```



트랜잭션의 *commit()*이나 *flush()* 메서드가 호출될 때, 

**Session**은 자신의 tracking list에 있는 모든 **dirty entity**들을 찾아서 DB에 동기화시킨다.



<u>우리는 **Session**에게 Entity를 변경했다고 알리기위해 어떠한 메서드도 호출할 필요가 없음을 주목하라</u>.

왜냐하면 이 Entity는 **managed entity**이기 때문에, 모든 변화들은 DB에 자동적으로 전파된다.



**Managed entity**는 언제나 persistent(영속화되는, 저장되는) entity이다. 

**managed entity**는 DB의 행이 아직 만들어지지 않았더라도 DB 식별자를 가지고 있어야 한다.

(INSERT 문은 unit of work의 마지막까지 연기된다.)







## 5. Detached Entity

**Detached entity**는 그저 identity value가 DB 행과 매칭되는 평범한 POJO entity일 뿐이다.

**Managed entity**와의 차이점이라면 어떠한 **persistence context**에 의해서도 추적되지 않는다는 것이다.



Entity는 해당 Entity를 부른 Session이 이미 close 되었거나, 우리가 *Session.evict(entity)* 또는 *Session.clear()*를 호출할때 **detached** 될 수 있다. 



코드로 살펴보자..

```java
FootballPlayer cr7 = session.get(FootballPlayer.class, 1L);

assertThat(getManagedEntities(session)).size().isEqualTo(1);
assertThat(getManagedEntities(session).get(0).getId()).isEqualTo(cr7.getId());

session.evict(cr7);

assertThat(getManagedEntities(session)).size().isEqualTo(0);
```



**persistence context**는 detached entity들의 변화를 추적하지 않는다.

```java
cr7.setName("CR7");
transaction.commit();

assertThat(getDirtyEntities()).isEmpty();
```



*Session.merge(entity) / Session.update(entity)*는 Session에 Entity를 다시 attach 시킬 수 있다.

```java
FootballPlayer messi = session.get(FootballPlayer.class, 2L);

session.evict(messi);
messi.setName("Leo Messi");
transaction.commit();

assertThat(getDirtyEntities()).isEmpty();

transaction = startTransaction(session);
session.update(messi);
transaction.commit();

assertThat(getDirtyEntities()).size().isEqualTo(1);
assertThat(getDirtyEntities().get(0).getName()).isEqualTo("Leo Messi");
```





### 5.1 The Identity Field Is All That Matters

아래의 코드의 로직을 살펴보자.

```java
FootballPlayer gigi = new FootballPlayer();
gigi.setId(3);
gigi.setName("Gigi the Legend");
session.update(gigi);
```



위의 예에서는, 해당 객체의 생성자에 따라 평범한 방법으로 entity의 인스턴스를 생성했다.

우리는 필드들을 값으로 채우고 identity 값을 3으로 세팅했다.

여기서 3은 Gigi Buffon persistent data의 identity와 일치한다.

*update()*를 호출하는 것은 마치 우리가 다른 **persistence context**로부터 entity를 불러온 것과 같은 효과를 낸다.

사실상, *Session*은 re-attached entity가 어디서 굴러들어왔는지 구별하지 않는다.

이것은 웹 애플리케이션에서 HTML form value들로부터 detached entity들을 만들때 자주 발생하는 scenario이다.







As far as *Session* is concerned, a detached entity is just a plain entity whose identity value corresponds to persistent data.



Be aware that the example above just serves a demo purpose. and we need to know exactly what we're doing. Otherwise, we could end up with null values across our entity if we just set the value on the field we want to update, leaving the rest untouched (so, effectively null).







## 6. Transient Entity

**Transient Entity**는 persistent store에 표현되지 않는 간단한 entity 객체고, 어떠한 *Session*으로부터도 관리되지 않는다.



Transient Entity의 전형적인 사례는 해당 Entity의 생성자로 새로운 인스턴스를 만들어 내는 경우다.

Transient Entity를 *persistent* 하게 만드려면, *Session.save(entity)* 또는 *Session.saveOrUpdate(entity)*를 호출해야 한다.

```java
FootballPlayer neymar = new FootballPlayer();
neymar.setName("Neymar");
session.save(neymar);

assertThat(getManagedEntities(session)).size().isEqualTo(1);
assertThat(neymar.getId()).isNotNull();

int count = queryCount("select count(*) from Football_Player where name='Neymar'");

assertThat(count).isEqualTo(0);

transaction.commit();
count = queryCount("select count(*) from Football_Player where name='Neymar'");

assertThat(count).isEqualTo(1);
```



우리가 *Session.save(entity)*를 수행하면, 그 Entity는 identity value를 부여받고 *Session*에 의해 관리된다.

그러나 INSERT 연산이 unit of work의 마지막까지 연기될 수 있기 때문에 DB에서는 아직 이용가능하지 않을 수 있다.





## 7. Deleted Entity

**Entity**는 *Session.delete(entity)*가 호출되었을 때 **deleted(reomved)** 상태가 된다.

그리고 *Session*은 그 entity를 삭제되었다고 기록해놓는다.

DELETE 명령 그 자체는 unit of work의 마지막에 수행된다.



아래 코드를 보자.

```java
session.delete(neymar);

assertThat(getManagedEntities(session).get(0).getStatus()).isEqualTo(Status.DELETED);
```

하지만, unit of work의 마지막까지 해당 객체가 **persistence context store**에 stay 한다는 것을 주의해라.