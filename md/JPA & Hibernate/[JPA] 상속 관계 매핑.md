# [JPA] 상속 관계 매핑



**매핑**

- OOP: 상속
- RDB: 슈퍼타입-서브타입 관계



**방법 3가지**

슈퍼타입-서브타입 논리 모델을 실제 물리 모델인 테이블로 구현할 때는 3가지 방법을 선택할 수 있다.

- **각각의 테이블로 변환(Join 전략)**

![](https://docs.google.com/drawings/d/sg7Thdh6olUFBbq4HkFFlvw/image?parent=e/2PACX-1vRVvbIKomKLDTKGV_Zpd2jnSflB7IGvZt3_68Ps6WLyfUO3_b16JGyGje2BnfNAFQnyCXgohLQOH9QO&rev=181&h=340&w=601&ac=1)



- **통합 테이블로 변환**

![](https://docs.google.com/drawings/d/shfwfBJS7gwegx-ScpuAVdA/image?parent=e/2PACX-1vRVvbIKomKLDTKGV_Zpd2jnSflB7IGvZt3_68Ps6WLyfUO3_b16JGyGje2BnfNAFQnyCXgohLQOH9QO&rev=12&h=652&w=601&ac=1)



- **서브타입 테이블로 변환**
  - 서브 타입마다 하나의 테이블을 만든다.
  - 쓰지말자...







## 1. 조인 전략

![](https://docs.google.com/drawings/d/sg7Thdh6olUFBbq4HkFFlvw/image?parent=e/2PACX-1vRVvbIKomKLDTKGV_Zpd2jnSflB7IGvZt3_68Ps6WLyfUO3_b16JGyGje2BnfNAFQnyCXgohLQOH9QO&rev=181&h=340&w=601&ac=1)



- 엔티티 각각을 모두 테이블로 만들고 (`ITEM`, `ALBUM`, `MOVIE`, `BOOK` 전부다)

  자식 테이블이 부모 테이블의 기본 키를 받아서 (기본 키 + 외래 키)로 사용하는 전략이다.

- 조회할 때 Join을 자주 사용하게 된다.

- 객체는 타입으로 구분할 수 있지만 테이블은 타입의 개념이 없다. 따라서 타입을 구분하는 컬럼을 추가해야 한다.



**Item**

```java
@Entity
@Inheritance(strategy = InheritanceType.JOINED)
@DiscriminatorColumn(name = "DTYPE")
public abstract class Item {

    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "item_id")
    private Long id;

    private String name;
    private String price;
}
```



**Album**

```java
@Entity
@DiscriminatorValue("A")
@PrimaryKeyJoinColumn(name = "album_id")
public class Album extends Item{

    private String artist;
}
```



**Book**

```java
@Entity
@DiscriminatorValue("B")
public class Book extends Item {

    private String author;
}
```



**Movie**

```java
@Entity
@DiscriminatorValue("M")
public class Movie extends Item {

    private String director;
    private String actor;
}
```





**매핑 방법**

- 부모 엔티티 (`Item`)
  - 상속할거니까 `@Inheritance` 붙여주고 속성으로 `(strategy = InheritanceType.JOINED)` 주면 조인 전략됨.
  - 구분 컬럼도 만들어야 되니까 `@DiscriminatorColumn` 붙여주고 `(name = DTYPE)` 속성으로 컬럼 이름 지정.
  - 부모 엔티티는 직접 생성해서 쓰는 경우가 없으므로 추상 클래스로 만들어 놓자.
- 자식 엔티티 (`Album`, `Book`, `Movie`)
  - 자신만의 필드 정의
  - 부모 테이블의 구분 컬럼에 들어갈 값을 `@DiscriminatorValue()`로 지정해주자.
  - 자식 테이블의 기본 키는 부모 테이블의 기본 키를 상속받아서 그대로 사용하기 때문에 컬럼명도 그대로 상속받는다.
    - 자식 테이블에서 기본 키 컬럼명을 바꾸고 싶으면, `@PrimaryKeyJoinColumn`을 사용하면 된다.



**조인 전략 장&middot;단점**

- 장점
  - 테이블이 정규화됨
  - 외래 키 참조 무결성 제약조건을 활용 가능
  - 저장공간 효율적으로 사용
- 단점
  - 조인이 많이 사용 -> 성능 저하될 수 있음
  - 조회 쿼리가 복잡함
  - 데이터를 등록할 때 INSERT 쿼리 두 번 나감







## 2. 단일 테이블 전략

![](https://docs.google.com/drawings/d/shfwfBJS7gwegx-ScpuAVdA/image?parent=e/2PACX-1vRVvbIKomKLDTKGV_Zpd2jnSflB7IGvZt3_68Ps6WLyfUO3_b16JGyGje2BnfNAFQnyCXgohLQOH9QO&rev=12&h=652&w=601&ac=1)



- 테이블 하나에 다 때려박는 전략
- 여기서도 역시 구분 컬럼(`DTYPE`)을 통해 어떤 자식 데이터가 저장되었는지 구분한다.
- 조회할 때 조인 사용 X -> 빠름
- **주의**: 자식 엔티티가 매핑한 컬럼은 모두 `null`을 허용해야 함





코드는 조인 전략에서의 코드랑 완전 똑같은데 

`Item` 엔티티의 속성을 `@Inheritance(strategy = InheritanceType.SINGLE_TABLE)` 얘로 바꿔주기만 하면 된다.





**단일 테이블 전략 장&middot;단점 및 특징**

- 장점
  - 조인 사용 X -> 조회 성능 빠름
  - 조회 쿼리가 단순함
- 단점
  - 자식 엔티티가 매핑한 컬럼은 모두 `null`을 허용해야 함.
  - 단일 테이블에 모든 것을 저장하므로 테이블이 커질 수 있다. 그러므로 상황에 따라서는 조회 성능이 오히려 느려진다.