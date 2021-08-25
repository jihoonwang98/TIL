# [JPA] Query 결과를 Map으로 리턴받기



> https://vladmihalcea.com/jpa-query-map-result/



## 1. 도메인 모델

아래와 같은 **`Post`** 엔티티를 사용한다.



![](https://vladmihalcea.com/wp-content/uploads/2020/01/PostMapQuery.png)



**Post 엔티티**

```java
@Entity
@Getter @Setter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class Post {
    
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "post_id")
    private Long id;
    
    private String title;
    
    private LocalDate createdOn;
    
    public Post(String title, LocalDate createdOn) {
        this.title = title;
        this.createdOn = createdOn;
    }
}
```



`Post` 엔티티를 저장하는 코드를 작성하자.

```java
@SpringBootTest
@Transactional
class PostTest {

    @Autowired
    EntityManager em;

    @Test
    public void test1() {
        Post post1 = new Post("post1", LocalDate.of(2016, 10, 12));
        Post post2 = new Post("post2", LocalDate.of(2018, 1, 30));
        Post post3 = new Post("post3", LocalDate.of(2018, 5, 8));
        Post post4 = new Post("post4", LocalDate.of(2019, 3, 19));

        em.persist(post1);
        em.persist(post2);
        em.persist(post3);
        em.persist(post4);
    }
}
```





## 2. 요구사항

우리는 각 출판년도마다 나온 post의 개수를 알고 싶다.

ex) 위의 예시를 예로 들면, 아래와 같다.

| 출판년도 (year) | Post 개수 (postCount) |
| --------------- | --------------------- |
| 2016            | 1                     |
| 2018            | 2                     |
| 2019            | 1                     |



위와 같은 테이블 정보를 JPQL 쿼리로 가져오려면 아래와 같은 쿼리를 작성하면 된다.

```sql
select
	YEAR(p.createdOn) as year,
	count(p) as postCount
from 
	Post p
group by
	YEAR(p.createdOn);
```



위와 같은 쿼리를 `em.createQuery(...).getResultList()` 메서드로 가져오면 `List` 형태로 결과를 가져올 수는 있다.

하지만, 여기서는 출판년도라는 컬럼의 값이 unique하므로 `Map`을 사용하면 데이터를 더 쉽게 다룰 수 있을 것 같다.





## 3. JPA Query `getResultStream`으로 결과 Map으로 리턴받기

JPA 2.2 버전부터 `getResultStream` 메서드를 이용해

`List<Tuple>` 결과를 `Map`으로 변환시킬 수 있다.

```java
@SpringBootTest
@Transactional
class PostTest {

    @Autowired
    EntityManager em;

    @Test
    public void test1() {
        Post post1 = new Post("post1", LocalDate.of(2016, 10, 12));
        Post post2 = new Post("post2", LocalDate.of(2018, 1, 30));
        Post post3 = new Post("post3", LocalDate.of(2018, 5, 8));
        Post post4 = new Post("post4", LocalDate.of(2019, 3, 19));

        em.persist(post1);
        em.persist(post2);
        em.persist(post3);
        em.persist(post4);

        em.flush();
        em.clear();

        Map<Integer, Integer> results = em.createQuery(
                "select " +
                        "   YEAR(p.createdOn) as year, " +
                        "   count(p) as postCount " +
                        "from " +
                        "   Post p " +
                        "group by " +
                        "   YEAR(p.createdOn)", Tuple.class)
                .getResultStream()
                .collect(
                        Collectors.toMap(
                                tuple -> ((Number) tuple.get("year")).intValue(),
                                tuple -> ((Number) tuple.get("postCount")).intValue()
                        )
                );

        System.out.println(results);

    }
}
```





## 4. JPA Query `getResultList`로 Map 리턴받기

```java
Map<Integer, Integer> postCountByYearMap = entityManager
.createQuery(
    "select " +
    "   YEAR(p.createdOn) as year, " +
    "   count(p) as postCount " +
    "from " +
    "   Post p " +
    "group by " +
    "   YEAR(p.createdOn)", Tuple.class)
.getResultList()
.stream()
.collect(
    Collectors.toMap(
        tuple -> ((Number) tuple.get("year")).intValue(),
        tuple -> ((Number) tuple.get("postCount")).intValue()
    )
);
```



