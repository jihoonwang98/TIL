# [JPA - Best Practice] @OneToOne 관계 Best Practice



> 참고: https://vladmihalcea.com/the-best-way-to-map-a-onetoone-relationship-with-jpa-and-hibernate/





## 1. 도메인 모델

![](https://vladmihalcea.com/wp-content/uploads/2016/07/onetoone.png)



`Post` 엔티티가 부모 엔티티, `PostDetails`가 자식 엔티티.





## 2. 전형적인 흔한 매핑 방법 (비효율적)



**자식 엔티티 `PostDetails`**

```java
@Entity
@Getter @Setter
@NoArgsConstructor
public class PostDetails {
 
    @Id
    @GeneratedValue
    private Long id;
    
    private Date createdOn;
 
    private String createdBy;
 
    @OneToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "post_id")
    private Post post;
 
    
    public PostDetails(String createdBy) {
        createdOn = new Date();
        this.createdBy = createdBy;
    }

}
```





**부모 엔티티 `Post`**

```java
@Entity
@Getter @Setter
public class Post {
 
    @Id
    @GeneratedValue
    private Long id;
 
    private String title;
 
    @OneToOne(mappedBy = "post", cascade = CascadeType.ALL,
              fetch = FetchType.LAZY, optional = false)
    private PostDetails details;
 
    // 자식 엔티티의 연관관계도 부모 엔티티가 같이 설정해준다
    public void setDetails(PostDetails details) {
        if (details == null) {
            if (this.details != null) {
                this.details.setPost(null);
            }
        }
        else {
            details.setPost(this);
        }
        this.details = details;
    }
}
```



그러나, 이런 매핑 방법은 가장 효율적인 방법은 아니다.



`post_details` 테이블은 PK 컬럼(id)과 FK 컬럼(post_id)를 포함하고 있다.

![](https://vladmihalcea.com/wp-content/uploads/2016/07/one-to-one.png)





그러나, 한 개의 `post` row와 연관된 `post_details` row는 오직 한 개 존재할 수 있으므로,

`post_details` PK는 `post`의 PK를 그대로 내려 받는게 더 말이 된다.

즉, 식별관계로 만드는 것이다.

![](https://vladmihalcea.com/wp-content/uploads/2016/07/one-to-one-shared-pk.png)



이렇게 하면, `post_details`의 PK는 FK의 역할도 하게 된다.

그리고 두 테이블이 PK를 공유하게 된다.



> PK와 FK 컬럼들은 대부분의 경우 인덱싱된다.
>
> 그래서 PK를 공유하는 것은 index footprint를 절반으로 줄일 수 있다.





단방향 일대일 관계는 LAZY 전략이 먹히지만, 





(뭔 소린지 모르겠다)

















