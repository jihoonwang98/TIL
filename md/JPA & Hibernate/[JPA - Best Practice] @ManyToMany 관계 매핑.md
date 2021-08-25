# [JPA - Best Practice] @ManyToMany 관계 매핑



> https://vladmihalcea.com/the-best-way-to-use-the-manytomany-annotation-with-jpa-and-hibernate/





## 1. 도메인 모델

![](https://vladmihalcea.com/wp-content/uploads/2017/05/many-to-many-post-tag.png)



다대다 관계는 각각의 부모 테이블을 참조하는 FK를 가지고 있는 새로운 테이블(`post_tag`)을 통해 연결된다.





## 2. List를 사용해서 다대다 관계 구현하기

다대다 관계를 매핑하는 첫번째 방법은 `List` 컬렉션을 사용해서 구현하는 방법이다.

```java
@Entity
@Getter @Setter
@NoArgsConstructor
public class Post {
 
    @Id @GeneratedValue
    private Long id;
 
    private String title;
 
    public Post(String title) {
        this.title = title;
    }
 
    @ManyToMany(cascade = {
        CascadeType.PERSIST,
        CascadeType.MERGE
    })
    @JoinTable(name = "post_tag",
        joinColumns = @JoinColumn(name = "post_id"),
        inverseJoinColumns = @JoinColumn(name = "tag_id")
    )
    private List<Tag> tags = new ArrayList<>();
 
    
    
    //== 연관관계 편의 메서드 ==//
    public void addTag(Tag tag) {
        tags.add(tag);
        tag.getPosts().add(this);
    }
 
    public void removeTag(Tag tag) {
        tags.remove(tag);
        tag.getPosts().remove(this);
    }
    
    
 
    //== Equals And HashCode ==//
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Post)) return false;
        return id != null && id.equals(((Post) o).getId());
    }
 
    @Override
    public int hashCode() {
        return getClass().hashCode();
    }
}
 
@Entity
@Getter @Setter
@NoArgsConstructor
public class Tag {
 
    @Id @GeneratedValue
    private Long id;
 
    @NaturalId
    private String name;
 
    @ManyToMany(mappedBy = "tags")
    private List<Post> posts = new ArrayList<>();
    
    
 	//== Equals And HashCode ==//
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Tag tag = (Tag) o;
        return Objects.equals(name, tag.name);
    }
 
    @Override
    public int hashCode() {
        return Objects.hash(name);
    }
}
```



- `Post` 엔티티에 `CascadeType`으로 `PERSIST`와 `MERGE`만을 정의했다.
  - As explained in [this article](https://vladmihalcea.com/a-beginners-guide-to-jpa-and-hibernate-cascade-types/), the `REMOVE` [entity state transition](https://vladmihalcea.com/a-beginners-guide-to-jpa-hibernate-entity-state-transitions/) doesn’t make any sense for a `@ManyToMany` JPA association since it could trigger a chain deletion that would ultimately wipe both sides of the association.
  - `Post` 엔티티와 연관된 `Tag`를 지워버리면 해당 `Tag`가 다른 `Post`와도 연관되어 있을 수도 있기 때문에 지우면 큰일 날 수 있어서 그런것 같다.



- `addTag()`와 `removeTag()` 메서드를 `Post` 엔티티에 두어서 연관관계를 관리하게 했다.

  - 보통 로직이 `Post` 엔티티에서 시작하지, `Tag`에서 시작하는 경우는 없으므로 `Post`에서 한번에 연관관계를 관리하게 한 것 같다.

  













