# [JPA] 테이블 없는 상속 @MappedSuperclass



부모 클래스는 테이블과 매핑하지 않고 

부모 클래스를 상속받는 자식 클래스에게 매핑 정보만 제공하고 싶을 때 `@MappedSuperclass`를 사용한다.





아래와 같은 테이블이 있을 때

ID, NAME 컬럼이 두 테이블 모두에 존재한다.

그런데 이렇게 중복되는 컬럼들을 쏙 뽑아서 한 번에 관리하고 싶다면 어떡할까??

이럴때 `@MappedSuperclass`를 사용한다.



![](https://docs.google.com/drawings/d/s-jm9BNr0Uio2ceALVKF7gg/image?parent=e/2PACX-1vRVvbIKomKLDTKGV_Zpd2jnSflB7IGvZt3_68Ps6WLyfUO3_b16JGyGje2BnfNAFQnyCXgohLQOH9QO&rev=58&h=156&w=601&ac=1)







아래 그림처럼 중복된 컬럼들만 쏙 뽑아서 필드로 갖고 있는 `BaseEntity`라는 엔티티를 정의한 뒤,

여기에다가 `@MappedSuperclass`를 붙여주면 `BaseEntity` 자체는 테이블로 매핑되지는 않지만

자신이 갖고 있는 필드들을 자식 엔티티들에게 상속할 수 있다.



![](https://docs.google.com/drawings/d/smT2VA-sgb3hjXj-VexEAaQ/image?parent=e/2PACX-1vRVvbIKomKLDTKGV_Zpd2jnSflB7IGvZt3_68Ps6WLyfUO3_b16JGyGje2BnfNAFQnyCXgohLQOH9QO&rev=172&h=245&w=601&ac=1)



```java
@MappedSuperclass
@Getter @Setter
public abstract class BaseEntity {

    @Id @GeneratedValue
    private Long id;
    private String name;
}

@Entity
@Getter @Setter
public class Member extends BaseEntity {

    private String email;
}

@Entity
@Getter @Setter
public class Seller extends BaseEntity{
    private String shopName;
}
```





**매핑 방법**

- Base 엔티티 
  - 자식 엔티티들이 공통으로 갖고 있는 컬럼들을 필드로 정의하고, `@MappedSuperclass`를 붙여준다.
  - 이 엔티티를 직접 사용할 일은 거의 없으므로 추상 클래스로 만들어 준다.
- 자식 엔티티
  - Base 엔티티를 상속받기만 하면 됨







### 매핑 정보 재정의 (`@AttributeOverrides`, `@AttributeOverride`)



- **`@AttributeOverride`**
  - 부모 엔티티의 필드(컬럼)를 한 개만 재정의할 때 사용
  - `name` 속성: 재정의하고 싶은 부모 엔티티의 필드명을 입력
  - `column` 속성: 재정의하고 싶은 컬럼 정보를 `@Column` 애너테이션으로 넣어준다.



- **`@AttributeOverrides`**
  - 부모 엔티티의 필드(컬럼)를 둘 이상 재정의할 때 사용
  - `@AttributeOverrides({@AttributeOverride(...), @AttributeOverride(...), ... })` 꼴로 사용



```java
@Entity
@AttributeOverride(name="id", column=@Column(name = "MEMBER_ID"))
public class Member extends BaseEntity {
    ...
}
```



```java
@Entity
@AttributeOverrides({
	@AttributeOverride(name="id", column=@Column(name="MEMBER_ID")),
    @AttributeOverride(name="name", column=@Column(name="MEMBER_NAME"))
})
public class Member extends BaseEntity {
    ...
}
```