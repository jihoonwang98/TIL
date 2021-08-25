# [JPA - Best Practice] Cascade 영속성 전이 Best Practice



> https://vladmihalcea.com/a-beginners-guide-to-jpa-and-hibernate-cascade-types/



## 1. Introduction

JPA는 Entity State Transitions 들을 DB DML 문으로 번역해준다.

이런 엔티티들은 Entity Graph 형태로 존재하는 것이 흔하기 때문에 JPA는 이러한 Entity State 변화들을 

Parent 엔티티로부터 Child 엔티티로 전파하는 기능을 제공해준다.



## 2. JPA vs Hibernate Cascade Types

Hibernate는 모든 JPA Cascade Type들을 지원하고 몇가지 추가적인 legacy cascading style들도 제공한다.



| *JPA ENTITYMANAGER* ACTION                                   | *JPA CASCADETYPE*                                            | *HIBERNATE* NATIVE *SESSION* ACTION                          | *HIBERNATE* NATIVE *CASCADETYPE*                             | *EVENT LISTENER*                                             |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| [detach(entity)](http://docs.oracle.com/javaee/7/api/javax/persistence/EntityManager.html#detach(java.lang.Object)) | [DETACH](http://docs.oracle.com/javaee/7/api/javax/persistence/CascadeType.html#DETACH) | [evict(entity)](https://docs.jboss.org/hibernate/orm/4.3/javadocs/org/hibernate/Session.html#evict(java.lang.Object)) | [DETACH](https://docs.jboss.org/hibernate/orm/3.5/api/org/hibernate/annotations/CascadeType.html#DETACH) or [~~EVICT~~](https://docs.jboss.org/hibernate/orm/3.5/api/org/hibernate/annotations/CascadeType.html#EVICT) | [Default Evict Event Listener](https://docs.jboss.org/hibernate/orm/4.3/javadocs/org/hibernate/event/internal/DefaultEvictEventListener.html) |
| [merge(entity)](http://docs.oracle.com/javaee/7/api/javax/persistence/EntityManager.html#merge(T)) | [MERGE](http://docs.oracle.com/javaee/7/api/javax/persistence/CascadeType.html#MERGE) | [merge(entity)](https://docs.jboss.org/hibernate/orm/4.3/javadocs/org/hibernate/Session.html#merge(java.lang.Object)) | [MERGE](https://docs.jboss.org/hibernate/orm/3.5/api/org/hibernate/annotations/CascadeType.html#MERGE) | [Default Merge Event Listener](https://docs.jboss.org/hibernate/orm/4.3/javadocs/org/hibernate/event/internal/DefaultMergeEventListener.html) |
| [persist(entity)](http://docs.oracle.com/javaee/7/api/javax/persistence/EntityManager.html#persist(java.lang.Object)) | [PERSIST](http://docs.oracle.com/javaee/7/api/javax/persistence/CascadeType.html#PERSIST) | [persist(entity)](https://docs.jboss.org/hibernate/orm/4.3/javadocs/org/hibernate/Session.html#persist(java.lang.Object)) | [PERSIST](https://docs.jboss.org/hibernate/orm/3.5/api/org/hibernate/annotations/CascadeType.html#PERSIST) | [Default Persist Event Listener](https://docs.jboss.org/hibernate/orm/4.3/javadocs/org/hibernate/event/internal/DefaultPersistEventListener.html) |
| [refresh(entity)](http://docs.oracle.com/javaee/7/api/javax/persistence/EntityManager.html#refresh(java.lang.Object)) | [REFRESH](http://docs.oracle.com/javaee/7/api/javax/persistence/CascadeType.html#REFRESH) | [refresh(entity)](https://docs.jboss.org/hibernate/orm/4.3/javadocs/org/hibernate/Session.html#refresh(java.lang.Object)) | [REFRESH](https://docs.jboss.org/hibernate/orm/3.5/api/org/hibernate/annotations/CascadeType.html#REFRESH) | [Default Refresh Event Listener](https://docs.jboss.org/hibernate/orm/4.3/javadocs/org/hibernate/event/internal/DefaultRefreshEventListener.html) |
| [remove(entity)](http://docs.oracle.com/javaee/7/api/javax/persistence/EntityManager.html#remove(java.lang.Object)) | [REMOVE](http://docs.oracle.com/javaee/7/api/javax/persistence/CascadeType.html#REMOVE) | [delete(entity)](https://docs.jboss.org/hibernate/orm/4.3/javadocs/org/hibernate/Session.html#delete(java.lang.Object)) | [REMOVE](https://docs.jboss.org/hibernate/orm/3.5/api/org/hibernate/annotations/CascadeType.html#REMOVE) or [DELETE](https://docs.jboss.org/hibernate/orm/3.5/api/org/hibernate/annotations/CascadeType.html#DELETE) | [Default Delete Event Listener](https://docs.jboss.org/hibernate/orm/4.3/javadocs/org/hibernate/event/internal/DefaultDeleteEventListener.html) |
|                                                              |                                                              | [saveOrUpdate(entity)](https://docs.jboss.org/hibernate/orm/4.3/javadocs/org/hibernate/Session.html#saveOrUpdate(java.lang.Object)) | [SAVE_UPDATE](https://docs.jboss.org/hibernate/orm/3.5/api/org/hibernate/annotations/CascadeType.html#SAVE_UPDATE) | [Default Save Or Update Event Listener](https://docs.jboss.org/hibernate/orm/4.3/javadocs/org/hibernate/event/internal/DefaultSaveOrUpdateEventListener.html) |
|                                                              |                                                              | [replicate(entity, replicationMode)](https://docs.jboss.org/hibernate/orm/4.3/javadocs/org/hibernate/Session.html#replicate(java.lang.Object, org.hibernate.ReplicationMode)) | [REPLICATE](https://docs.jboss.org/hibernate/orm/3.5/api/org/hibernate/annotations/CascadeType.html#REPLICATE) | [Default Replicate Event Listener](https://docs.jboss.org/hibernate/orm/4.3/javadocs/org/hibernate/event/internal/DefaultReplicateEventListener.html) |
| [lock(entity, lockModeType)](http://docs.oracle.com/javaee/7/api/javax/persistence/EntityManager.html#lock(java.lang.Object, javax.persistence.LockModeType)) |                                                              | [buildLockRequest(entity, lockOptions)](https://docs.jboss.org/hibernate/orm/4.3/javadocs/org/hibernate/Session.html#buildLockRequest(org.hibernate.LockOptions)) | [LOCK](https://docs.jboss.org/hibernate/orm/3.5/api/org/hibernate/annotations/CascadeType.html#LOCK) | [Default Lock Event Listener](https://docs.jboss.org/hibernate/orm/4.3/javadocs/org/hibernate/event/internal/DefaultLockEventListener.html) |
| All the above EntityManager methods                          | [ALL](http://docs.oracle.com/javaee/7/api/javax/persistence/CascadeType.html#ALL) | All the above Hibernate Session methods                      | [ALL](https://docs.jboss.org/hibernate/orm/3.5/api/org/hibernate/annotations/CascadeType.html#ALL) |                                                              |





## 2. Cascading Best Practices

Cascading은 Parent - Child 관계에서만 말이 된다.

Parent 엔티티의 Entity State 변화가 Child 엔티티로 전파되는 것이 Cascading이니 말이다.



Child -> Parent로 Cascading 하는 것은 유용하지 않고, 보통 mapping code smell이다(?) 뭔소리고..



### (1) One-To-One

가장 흔한 경우는 일대일 양방향 연관관계다.

```java
@Entity
@Getter @Setter
public class Post {
 
    @Id @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;
 
    private String name;
 
    @OneToOne(mappedBy = "post",
        cascade = CascadeType.ALL, orphanRemoval = true)
    private PostDetails details;
 
    
    //== 연관관계 편의 메서드 ==//
    public void addDetails(PostDetails details) {
        this.details = details;
        details.setPost(this);
    }
 
    public void removeDetails(PostDetails details) {
        if (details != null) {
            details.setPost(null);
        }
        this.details = null;
    }
}
 
@Entity
@Getter @Setter
public class PostDetails {
 
    @Id
    private Long id;
 
    @Temporal(TemporalType.TIMESTAMP)
    private Date createdOn = new Date();
 
    private boolean visible;
 
    @OneToOne
    @MapsId
    private Post post;
}
```

여기서는 `Post` 엔티티가 부모 엔티티가 되고, `PostDetails` 엔티티가 자식 엔티티가 된다.



위의 경우에는, `CascadeType.ALL`과 `orphanRemoval = true` 속성이 말이 된다.

왜냐하면 `PostDetails` 엔티티의 life-cycle이 `Post` 부모 엔티티에 종속적이기 때문이다.



### - Cascading the one-to-one persist operation

`CascadeType.PERSIST` 옵션은 `CascadeType.ALL`로 설정하면 자동으로 설정되므로,

우리는 `Post` 엔티티 하나만 저장하면 연관된 `PostDetails` 엔티티를 자동으로 저장할 수 있다.

```java
Post post = new Post();
post.setName("Hibernate Master Class");
 
PostDetails details = new PostDetails();
 
post.addDetails(details);
 
session.persist(post);
```



우리는 위에서 `session.persist(details)` 같은 코드를 작성하지 않았지만 

cascade에 의해 아래와 같이 자동으로 INSERT 문이 함께 나간다.

```sql
INSERT INTO post(id, NAME)
VALUES (DEFAULT, Hibernate Master Class'')
 
insert into PostDetails (id, created_on, visible)
values (1, '2015-03-03 10:17:19.14', false)
```





### - Cascading the one-to-one merge operation

`CascadeType.ALL`로 설정하면 자동으로 `CascadeType.MERGE`도 설정된다.

그래서 `Post` 엔티티 하나만 merge 하면, 연관된 `PostDetails` 엔티티도 같이 merge 된다.

```java
Post post = newPost();
post.setName("Hibernate Master Class Training Material");
post.getDetails().setVisible(true);
 
doInTransaction(session -> {
    session.merge(post);
});
```



merge 연산은 아래와 같이 만들어진다.

```sql
SELECT onetooneca0_.id     AS id1_3_1_,
   onetooneca0_.NAME       AS name2_3_1_,
   onetooneca1_.id         AS id1_4_0_,
   onetooneca1_.created_on AS created_2_4_0_,
   onetooneca1_.visible    AS visible3_4_0_
FROM   post onetooneca0_
LEFT OUTER JOIN postdetails onetooneca1_
    ON onetooneca0_.id = onetooneca1_.id
WHERE  onetooneca0_.id = 1
 
UPDATE postdetails SET
    created_on = '2015-03-03 10:20:53.874', visible = true
WHERE  id = 1
 
UPDATE post SET
    NAME = 'Hibernate Master Class Training Material'
WHERE  id = 1
```





### - Cascading the one-to-one delete operation

`CascadeType.ALL`로 설정하면 자동으로 `CascadeType.DELETE`도 설정된다.

그래서 `Post` 엔티티 하나만 delete 하면, 연관된 `PostDetails` 엔티티도 같이 delete 된다.



```java
Post post = newPost();
 
doInTransaction(session -> {
    session.delete(post);
});
```



delete 연산은 아래와 같이 만들어진다.

```sql
delete from PostDetails where id = 1
delete from Post where id = 1
```



### - 일대일 delete orphan cascading 연산

만약에 자식 엔티티가 부모 엔티티와의 연관관계가 끊기면, 자식의 FK가 NULL이 된다.

이런 경우에 자식 row를 지워버리고 싶다면, orphan removal 기능을 사용하면 된다.



```java
doInTransaction(session -> {
    Post post = (Post) session.get(Post.class, 1L);
    post.removeDetails();
});
```



orphan removal은 아래와 같이 생성된다.

```sql
SELECT onetooneca0_.id         AS id1_3_0_,
       onetooneca0_.NAME       AS name2_3_0_,
       onetooneca1_.id         AS id1_4_1_,
       onetooneca1_.created_on AS created_2_4_1_,
       onetooneca1_.visible    AS visible3_4_1_
FROM   post onetooneca0_
LEFT OUTER JOIN postdetails onetooneca1_
    ON onetooneca0_.id = onetooneca1_.id
WHERE  onetooneca0_.id = 1
 
delete from PostDetails where id = 1
```





### - 단방향 one-to-one 관계

대부분의 경우, 자식 엔티티가 연관관계의 주인이 되고 부모 엔티티가 반대 편(mappedBy)이 돼서 양방향 매핑이 된다.

그러나 cascade는 양방향 관계에만 쓸 수 있는 것은 아니다.

단방향 관계에도 쓸 수 있다.



```java
@Entity
@Getter
@NoArgsConstructor
public class Commit {
 
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;
 
    private String comment;
 
    @OneToOne(cascade = CascadeType.ALL)
    private BranchMerge branchMerge;
 
    public Commit(String comment) {
        this.comment = comment;
    }
 
    
    //== 연관관계 편의 메서드 ==//
    public void addBranchMerge(
        String fromBranch, String toBranch) {
        this.branchMerge = new BranchMerge(
             fromBranch, toBranch
        );
    }
 
    public void removeBranchMerge() {
        this.branchMerge = null;
    }
}
 
@Entity
@NoArgsConstructor
@Getter
public class BranchMerge {
 
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;
 
    private String fromBranch;
 
    private String toBranch;
 
    public BranchMerge(
        String fromBranch, String toBranch) {
        this.fromBranch = fromBranch;
        this.toBranch = toBranch;
    }
}
```







### (2) One-To-Many



```java
@Entity
@Getter @Setter
public class Post {
 
    @Id @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;
 
    private String name;
 
    @OneToMany(cascade = CascadeType.ALL,
        mappedBy = "post", orphanRemoval = true)
    private List<Comment> comments = new ArrayList<>();
 
    
 	//== 연관관계 편의 메서드 ==//
    public void addComment(Comment comment) {
        comments.add(comment);
        comment.setPost(this);
    }
 
    public void removeComment(Comment comment) {
        comment.setPost(null);
        this.comments.remove(comment);
    }
}
 
@Entity
@Getter @Setter
public class Comment {
 
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;
 
    @ManyToOne
    private Post post;
 
    private String review; 
}
```

일대일 관계에서처럼, `CascadeType.ALL`과 orphan removal 조합은 적합하다.

왜냐하면 `Comment`의 Life-cycle이 `Post` 부모 엔티티에 종속적이기 때문이다.





### - Cascading the one-to-many persist operation

### - Cascading the one-to-many merge operation

### - Cascading the one-to-many delete operation

### - The one-to-many delete orphan cascading operation

자세한 코드는 직접 홈페이지를 들어가보자.







### (3) Many-To-Many













