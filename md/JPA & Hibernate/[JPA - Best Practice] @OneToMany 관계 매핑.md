# [JPA - Best Practice] @OneToMany 관계 매핑



> https://vladmihalcea.com/the-best-way-to-map-a-onetomany-association-with-jpa-and-hibernate/



![](https://vladmihalcea.com/wp-content/uploads/2017/03/many-to-one.png)



## 1. 단방향 @OneToMany

```java
@Entity
@Getter @Setter
public class Post {
 
    @Id @GeneratedValue
    private Long id;
 
    private String title;
 
    @OneToMany(
        cascade = CascadeType.ALL,
        orphanRemoval = true
    )
    private List<PostComment> comments = new ArrayList<>();
}
 

@Entity
@Getter @Setter
public class PostComment {
 
    @Id @GeneratedValue
    private Long id;
 
    private String review;
}
```



테이블 매핑이 아래처럼 됨.



![](https://vladmihalcea.com/wp-content/uploads/2017/03/one-to-many.png)





이 방법은 쓰면 안됨





## 2. 단방향 `@OneToMany` with `@JoinColumn`

```java
@Entity
@Getter @Setter
public class Post {
 
    @Id @GeneratedValue
    private Long id;
 
    private String title;
 
    
    @JoinColumn(name="post_id")
    @OneToMany(
        cascade = CascadeType.ALL,
        orphanRemoval = true
    )
    private List<PostComment> comments = new ArrayList<>();
}
 

@Entity
@Getter @Setter
public class PostComment {
 
    @Id @GeneratedValue
    private Long id;
 
    private String review;
}
```

앞에서 언급한 join table이 하나 더 생기는 문제를 해결하기 위해, `@JoinColumn` 애너테이션을 붙여보자.

`@JoinColumn`을 붙이면 `Post_Comment` 테이블에 `post_id` 라는 FK 컬럼이 있다고 하이버네이트에게 알려준다.





이 상태에서 `PostComment` 엔티티 세 개를 저장해보면, 아래와 같은 SQL 문이 나간다.

```sql
insert into post (title, id)
values ('First post', 1)
 
insert into post_comment (review, id)
values ('My first review', 2)
 
insert into post_comment (review, id)
values ('My second review', 3)
 
insert into post_comment (review, id)
values ('My third review', 4)
 
update post_comment set post_id = 1 where id = 2
 
update post_comment set post_id = 1 where id =  3
 
update post_comment set post_id = 1 where id =  4
```

만약 너가 [Hibernate flush order](https://vladmihalcea.com/hibernate-facts-knowing-flush-operations-order-matters/)를 알고 있다면, collection 요소들이 처리되기 전에 persist가 처리되는 것을 알고 있을 것이다.

하이버네이트는 FK 없이 child record들을 먼저 집어 넣는다. 왜냐면 자식 엔티티들은 FK 정보가 없기 때문이다.

그 다음에 collection을 처리하는 단계가 되어서야, FK 컬럼들이 업데이트 된다.





이건 자식 엔티티를 삭제할 때도 마찬가지다.

아래와 같이 child collection에서 첫 번째 요소를 삭제해보자.

```java
post.getComments().remove(0);
```



그러면 하이버네이트는 아래와 같이 한 개가 아니라 두 개의 쿼리를 날린다.

```sql
update post_comment set post_id = null where post_id = 1 and id = 2
 
delete from post_comment where id=2
```





## 3. 양방향 `@OneToMany`

 `@OneToMany` 관계를 매핑하는 가장 좋은 방법은 `@ManyToOne` 측에 의존해 모든 엔티티 상태 변경을 전파하는 것이다.



```java
@Entity
@Getter @Setter
public class Post {
 
    @Id
    @GeneratedValue
    private Long id;
 
    private String title;
 
    @OneToMany(
        mappedBy = "post",
        cascade = CascadeType.ALL,
        orphanRemoval = true
    )
    private List<PostComment> comments = new ArrayList<>();
 
    public void addComment(PostComment comment) {
        comments.add(comment);
        comment.setPost(this);
    }
 
    public void removeComment(PostComment comment) {
        comments.remove(comment);
        comment.setPost(null);
    }
}

 
@Entity
@Getter @Setter
public class PostComment {
 
    @Id
    @GeneratedValue
    private Long id;
 
    private String review;
 
    @ManyToOne(fetch = FetchType.LAZY)
    private Post post;
 
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof PostComment )) return false;
        return id != null && id.equals(((PostComment) o).getId());
    }
 
    @Override
    public int hashCode() {
        return getClass().hashCode();
    }
}
```

- 부모 엔티티인 `Post`는 두 가지 연관관계 편의 메서드를 가진다. (`addComment()`, `removeComment()`)
  - 이 메서드들은 양쪽 엔티티의 연관관계를 synchronize 하기 위해 사용된다.
  - 양방향 연관관계를 사용하면 항상 이런 편의 메서드들을 제공해야 한다.

- 자식 엔티티인 `PostComment`는 `equals()`와 `hashCode()` 메서드를 구현했다.
  - Since we cannot rely on a [natural identifier for equality checks](https://vladmihalcea.com/hibernate-facts-equals-and-hashcode/), we need to use the entity identifier instead for the `equals` method. However, you need to do it properly so that [equality is consistent across all entity state transitions](https://vladmihalcea.com/how-to-implement-equals-and-hashcode-using-the-jpa-entity-identifier/), which is also the reason why the `hashCode` has to be a constant value. Because we rely on equality for the `removeComment`, it’s good practice to override `equals` and `hashCode` for the child entity in a bidirectional association.
  - 자식 엔티티가 부모 엔티티의 `List`나 `Set`에 들어가는 경우 `equals()`와 `hashCode()`를 재정의해주어야 한다.





이제 `PostComment` 엔티티 세 개를 저장해보자.

```java
Post post = new Post("First post");
 
post.addComment(
    new PostComment("My first review")
);
post.addComment(
    new PostComment("My second review")
);
post.addComment(
    new PostComment("My third review")
);
 
entityManager.persist(post);
```



하이버네이트는 각각의 statement마다 한 개의 SQL 문을 만들어낸다.

```sql
insert into post (title, id)
values ('First post', 1)
 
insert into post_comment (post_id, review, id)
values (1, 'My first review', 2)
 
insert into post_comment (post_id, review, id)
values (1, 'My second review', 3)
 
insert into post_comment (post_id, review, id)
values (1, 'My third review', 4)
```





만약에 `PostComment`를 삭제하면,

```java
Post post = entityManager.find( Post.class, 1L );
PostComment comment1 = post.getComments().get(0);
post.removeComment(comment1);
```



SQL 문이 하나만 나간다.

```sql
delete from post_comment where id = 2
```





