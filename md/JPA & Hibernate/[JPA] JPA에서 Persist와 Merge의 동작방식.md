# [JPA] JPA에서 Persist와 Merge의 동작방식

> https://vladmihalcea.com/jpa-persist-and-merge/



## 1. Introduction

JPA를 사용하면, Entity State Transition들이 자동으로 SQL 문으로 번역된다.

이번 포스팅에서는 언제 `persist()`를 사용하고 언제 `merge()`를 사용할지에 대해 설명한다.





## 2. Persist

**<u>`persist` 연산은 오직 new(transient) 엔티티에 대해서만 수행되어야 한다.</u>**

JPA 관점에서 보면, 엔티티가 새로 만들어지고 어떠한 DB row와도 연관된 적이 없으면 new(transient) 상태라고 볼 수 있다.

다시 말해, 엔티티와 매핑되는 DB record가 존재하지 않는 상황을 말한다.



예를 들어, 아래와 같은 테스트 케이스를 수행했다고 하자

```java
@Test
public void test() {
    Post post = new Post();
    post.setTitle("title!");
    em.persist(post);
    System.out.println("post entity identifier: " + post.getId());

    System.out.println("flush Persistence Context");
    em.flush();
}
```

Hibernate는 `Post` 엔티티를 현재 작동중인 Persistence Context에 attach 시킬 것이다.

그런데 INSERT 문은 즉시 실행될 수도 있고 flush-time 까지 연기될 수도 있다.

이건 Id가 생성되는 방식에 따라 다르다.



### IDENTITY 방식

만약에 엔티티가 Id를 IDENTITY generator를 사용하면,

```java
@Entity
@Getter @Setter
public class Post {

    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String title;
}
```

```java
@Test
public void test() {
    Post post = new Post();
    post.setTitle("title!");
    em.persist(post);
    System.out.println("post entity identifier: " + post.getId());

    System.out.println("flush Persistence Context");
    em.flush();
}

// 결과
2021-06-25 17:40:26.959  INFO 7928 --- [           main] p6spy
    insert 
    into
        post
        (id, title) 
    values
        (null, 'title!')
post entity identifier: 1
flush Persistence Context
```

엔티티가 persist 될 때마다, Hibernate는 해당 엔티티를 반드시 현재 동작중인 Persistence Context에 attach 해야 한다.

이 Persistence Context는 엔티티를 담는 `Map`처럼 작동하는데, 이 `Map`의 Key는 엔티티 타입과 엔티티 identifier로 구성된다.



IDENTITY 전략의 경우 엔티티 identifier 값을 아는 방법은 실제로 DB에 값을 넣고 증가된 ID를 가져와서 확인하는 방법밖에 없다.

그래서 `post.getId()`를 호출하는 순간 identifier 값을 찾기 위해 DB에 INSERT 쿼리를 날리고 값을 가져와서 출력한다.

참고로 flush-time에 이 INSERT 쿼리를 되돌릴 수 없다.





### SEQUENCE

ID 생성 전략으로 SEQUENCE 전략을 적용하고 위의 예시를 다시 돌려보면, 아래와 같은 결과가 나온다.

```java
@Entity
@Getter @Setter
public class Post {

    @Id @GeneratedValue(strategy = GenerationType.SEQUENCE)
    private Long id;

    private String title;
}
```

```java
@Test
public void test() {
    Post post = new Post();
    post.setTitle("title!");
    em.persist(post);
    System.out.println("post entity identifier: " + post.getId());

    System.out.println("flush Persistence Context");
    em.flush();
}

// 결과
2021-06-25 17:47:39.451  INFO 20356 --- [           main] p6spy
call next value for hibernate_sequence
    
post entity identifier: 1
flush Persistence Context

2021-06-25 17:47:39.487  INFO 20356 --- [           main] p6spy
    insert 
    into
        post
        (title, id) 
    values
        ('title!', 1)
```

이번에는 INSERT 문이 flush-time 까지 미뤄진다.

그리고 만약 batch size 설정 값을 주면 Hibernate는 batch insert 최적화를 적용해 줄 수도 있다.





> ID 생성 전략으로 TABLE 전략은 사용하지 말아야 한다.
>
> The `TABLE` strategy behaves like `SEQUENCE`, but you should avoid it at any cost because it uses a separate transaction to generate the entity identifier, therefore putting pressure on the underlying connection pool and the database transaction log.
>
> Even worse, row-level locks are used to coordinate multiple concurrent requests, and, just like [Amdhal’s Law](https://en.wikipedia.org/wiki/Amdahl's_law) tells us, introducing a serializability execution can affect scalability.
>
> For more details about why you should avoid the `TABLE` strategy, check out [this article](https://vladmihalcea.com/why-you-should-never-use-the-table-identifier-generator-with-jpa-and-hibernate/).





## 3. Merge

**<u>Merging 연산은 오직 detached 엔티티들에 대해서만 필요한 연산</u>**이다.



아래와 같은 엔티티를 살펴보자.

```java
Post post = doInJPA(entityManager -> {
    Post _post = new Post();
    _post.setTitle("High-Performance Java Persistence");
 
    entityManager.persist(_post);
    return _post;
});
```



`Post` 엔티티를 불러온 `EntityManager`가 닫혔기 때문에, `Post` 엔티티는 detached 된다.

그리고 이제 Hibernate는 더 이상 변화를 추적하지 않는다.

detached 엔티티에 생긴 변화를 반영하기 위해서는 새로운 Persistence Context에 reattach 해야 한다.

```java
post.setTitle("High-Performance Java Persistence Rocks!");
 
doInJPA(entityManager -> {
    LOGGER.info("Merging the Post entity");
    Post post_ = entityManager.merge(post);
});
```



위의 테스트 케이스를 돌려보면, Hibernate는 다음과 같은 statement들을 수행한다.

```sql
-- Merging the Post entity
 
SELECT p.id AS id1_0_0_ ,
       p.title AS title2_0_0_
FROM   post p
WHERE  p.id = 1
 
UPDATE post
SET title='High-Performance Java Persistence Rocks!'
WHERE id=1
```



Hibernate는 merge된 엔티티의 최신 상태의 DB record를 가져오기 위해 SELECT 문을 일단 날린다.

그 다음에, detached 상태의 엔티티를 **복사**해서 새로운 managed 상태의 엔티티를 만들어 낸다.

이렇게 되면 다시 dirty checking 메커니즘으로 변화를 감지하고 DB에 반영할 수 있게 된다.



IDENTITY나 SEQUENCE 제네레이터 전략의 경우, 엔티티를 저장하기 위해 `persist` 대신 `merge`를 사용할 수도 있다. 

근데 이렇게 하면 SELECT 쿼리 한 번 더나가서 비효율적이다.





## 4. The redundant save anti-pattern

이제, new(transient) 엔티티들은 `persist`로 저장하고, detached 엔티티들은 `merge`로 reattach 해야 한다는 것을 깨달았을 것이다.



그러나, 필자(vladmihalcea)는 많은 프로젝트를 리뷰하면서, 아래와 같은 안티 패턴이 널리 퍼져있음을 발견했다.

```java
@Transactional
public void savePostTitle(Long postId, String title) {
    Post post = postRepository.findOne(postId);
    post.setTitle(title);
    postRepository.save(post);
}
```

위의 코드에서 `save()` 메서드는 하는 일이 없다.

`save()` 메서드를 지워도, Hibernate는 UPDATE 문을 날릴 것이다.

왜냐면 엔티티가 managed 상태이기 때문에 어떠한 변화도 DB로 반영될 것이기 때문이다.



이런 방법은 안티패턴이다. 왜냐면 `save()` 메서드가 `MergeEvent`를 일으키고,

이 이벤트는 `DefaultMergeEventListener`에 의해 처리된다.

이 Listener는 아래와 같은 연산을 수행한다.

```java
protected void entityIsPersistent(MergeEvent event, Map copyCache) {
    LOG.trace( "Ignoring persistent instance" );
 
    final Object entity = event.getEntity();
    final EventSource source = event.getSession();
    final EntityPersister persister = source
        .getEntityPersister( event.getEntityName(), entity );
 
    ( (MergeContext) copyCache ).put( entity, entity, true );
 
    cascadeOnMerge( source, persister, entity, copyCache );
    copyValues( persister, entity, entity, source, copyCache );
 
    event.setResult( entity );
}
```

`copyValues` 메서드가 호출되면, [hydrated state](https://vladmihalcea.com/how-does-hibernate-store-second-level-cache-entries/)가 다시 복사되고, 그러면 새로운 배열이 불필요하게 생성된다.

그러므로 CPU Cycle을 낭비하게 된다.