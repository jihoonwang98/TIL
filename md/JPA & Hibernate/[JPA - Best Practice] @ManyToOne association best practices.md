# [JPA / Best Practice] @ManyToOne association best practices



## 1. Table Relationships

다대일 관계 테이블은 아래와 같다.

![](https://vladmihalcea.com/wp-content/uploads/2019/04/one-to-many-table-relationship-1024x267.png)







## 2. The `@ManyToOne` association

```java
@Entity
@Getter @Setter
public class PostComment {
 
    @Id
    @GeneratedValue
    private Long id;
 
    private String review;
 
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "post_id")
    private Post post;
}
```





## 3. Persisting a ManyToOne association with JPA

부모 `Post` 엔티티를 아래와 같은 코드로 미리 저장했다고 하자.

```java
entityManager.persist(
    new Post()
        .setId(1L)
        .setTitle("High-Performance Java Persistence")
);
```



개발자들이 자식 엔티티를 저장할 때 흔히 하는 실수는 

**<u>부모 엔티티를 `find()`로 찾는 것</u>**이다.

```java
Post post = entityManager.find(Post.class, 1L);
 
entityManager.persist(
    new PostComment()
        .setId(1L)
        .setReview("Amazing book!")
        .setPost(post)
);
```



또는 Spring Data JPA를 사용하고 있는 경우, `JpaRepository`의 `findById` 메서드를 사용하면서 같은 문제가 발생한다.

```java
Post post = postRepository.findById(1L);
 
commentRepository.save(
    new PostComment()
        .setId(1L)
        .setReview("Amazing book!")
        .setPost(post)
);
```



위와 같이 `find()` 메서드를 사용해서 자식 엔티티인 `PostComment`를 저장하게 되면, 

하이버네이트는 아래와 같은 SQL 문을 만들어 낼 것이다.

```sql
-- 먼저 POST 찾고,
SELECT
    p.id AS id1_0_0_,
    p.title AS title2_0_0_
FROM post p
WHERE p.id=1
 
-- 찾은 POST의 id를 이용해서 자식 엔티티인 POST_COMMENT를 INSERT한다.
INSERT INTO post_comment (
    post_id,
    review, 
    id
) VALUES (
    1,
    'Amazing book!',
    1
)
```



하지만, 위의 쿼리에서 자식 엔티티인 `POST_COMMENT`를 INSERT 하기 위해서 해야 하는 것은 외래 키 컬럼인 `post_id` 값을 세팅하는 것 뿐이다.

`POST` 엔티티의 모든 정보를 가져올 필요가 없다.





**<u>따라서, `find()` 메서드 대신에, `getReference()` 메서드를 사용해야 한다.</u>**

```java
Post post = entityManager.getReference(Post.class, 1L);
 
entityManager.persist(
    new PostComment()
        .setId(1L)
        .setReview("Amazing book!")
        .setPost(post)
);
```



또는 Spring Data JPA를 사용하고 있다면, `getOne()` 메서드를 사용해야 한다.

```java
Post post = postRepository.getOne(1L);
 
commentRepository.save(
    new PostComment()
        .setId(1L)
        .setReview("Amazing book!")
        .setPost(post)
);
```





위와 같이 `getReference()`나 `getOne()` 메서드를 사용해 프록시 객체를 가져오면,

하이버네이트는 SELECT 문을 실행하지 않고 자식 엔티티를 저장할 수 있게 된다.

```java
INSERT INTO post_comment (
    post_id,
    review, id
)
VALUES (
    1,
    'Amazing book!',
    1
)
```





## 4. Fetching a ManyToOne association with JPA and Hibernate

다대일 관계의 fetch 전략으로 `FetchType.LAZY`를 사용하고 있다고 가정하고,

아래와 같이 `PostComment` 엔티티를 갖고와보자.

```java
PostComment comment = entityManager.find(PostComment.class, 1L);
 
LOGGER.info(
    "The post '{}' got the following comment '{}'",
    comment.getPost().getTitle(),
    comment.getReview()
);
```



하이버네이트는 `comment.getPost().getTitle()`을 실행하려면 `Post` 엔티티를 가져와야 하므로,

`PostComment` 엔티티에 대한 SELECT문 외에도 `Post` 엔티티에 대한 SELECT문을 추가로 날린다.

```sql
SELECT
    pc.id AS id1_1_0_,
    pc.post_id AS post_id3_1_0_,
    pc.review AS review2_1_0_
FROM post_comment pc
WHERE pc.id = 1
 
SELECT
    p.id AS id1_0_0_,
    p.title AS title2_0_0_
FROM post p
WHERE p.id = 1
 
The post 'High-Performance Java Persistence' got the following comment 'Amazing book!'
```





SQL문이 두 방 날라가는 걸 막기 위해서는, `find()` 메서드로 찾지 말고 JPQL의 `join fetch`문으로 한방에 갖고 와야 한다.

```java
PostComment comment = entityManager.createQuery("""
    select pc
    from PostComment pc
    join fetch pc.post
    where pc.id = :id
    """, PostComment.class)
.setParameter("id", 1L)
.getSingleResult();
 
LOGGER.info(
    "The post '{}' got the following comment '{}'",
    comment.getPost().getTitle(),
    comment.getReview()
);
```



이제 하이버네이트는 부모 엔티티와 자식 엔티티를 조인해서 한방에 갖고 온다.

```sql
SELECT
    pc.id AS id1_1_0_,
    p.id AS id1_0_1_,
    pc.post_id AS post_id3_1_0_,
    pc.review AS review2_1_0_,
    p.title AS title2_0_1_
FROM post_comment pc
INNER JOIN post p ON pc.post_id = p.id 
WHERE pc.id = 1
 
The post 'High-Performance Java Persistence' got the following comment 'Amazing book!'
```







## 5. 정리



### `@ManyToOne` 전략

- 기본 fetch 전략: `@ManyToOne(fetch = FetchType.LAZY)`
- 자식 엔티티 저장할 때
  - 자식 엔티티에 부모 엔티티 set 할 때 `find()` 메서드 대신 `getReference()` 사용
  - JPA는 `findById()` 대신 `getOne()` 사용
- 자식 엔티티 조회할 때
  - 부모 엔티티의 PK 값을 제외한 값도 같이 조회해야 하는 경우에는 `FETCH JOIN`을 사용하자.