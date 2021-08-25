# JPA 지연 로딩과 조회 성능 최적화



> 참고:  인프런 강의 by 김영한
>
> [실전! 스프링 부트와 JPA 활용2 - API 개발과 성능 최적화](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8-JPA-API%EA%B0%9C%EB%B0%9C-%EC%84%B1%EB%8A%A5%EC%B5%9C%EC%A0%81%ED%99%94/dashboard)



대부분의 성능 이슈는 **조회**에서 일어난다.



주문 + 배송정보 + 회원을 조회하는 API를 최적화해보자.

지연 로딩 때문에 발생하는 성능 문제(N+1)를 단계적으로 해결해보자.





**도메인 모델**

![](https://lh5.googleusercontent.com/XuDW7se07mK2b_UhTicRlhRWghizWbYgelpFAzeycZmHdF62FTYTNIaOgxf5tyTn9zqwUOjdfwX14mK8nj_eZ5RT5jgCiuGk3mRz3GjO5RgN8IL_L0OHnUnUL9h-af00nSasmVGS)





**Order 엔티티**

```java
public class Order {

    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "order_id")
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "member_id")
    private Member member;


    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL)
    private List<OrderItem> orderItems = new ArrayList<>();


    @OneToOne(cascade = CascadeType.ALL, fetch = FetchType.LAZY)
    @JoinColumn(name = "delivery_id")
    private Delivery delivery;

    private LocalDateTime orderDate;

    @Enumerated(EnumType.STRING)
    private OrderStatus status;   // 주문 상태 [ORDER, CANCEL]
    
    ...
}
```



Order : Member는 다대일

Order : Delivery는 일대일

두개 모두 지연 로딩으로 설정.





## 엔티티를 직접 노출



```java
/**
 *  주문 조회 V1. 엔티티 직접 노출
 *
 */
@GetMapping("/api/v1/simple-orders")
public List<Order> ordersV1() {
    List<Order> all = orderRepository.findAll();    
   
    // Lazy 강제 초기화
    for (Order order : all) {
        order.getMember().getName();
        order.getDelivery().getAddress();
    }
    
    return all;    
}
```



Order의 member 필드나 Member의 orders 필드 

둘 중 하나에 **@JsonIgnore** 어노테이션을 붙여주지 않으면 

양방향 관계여서 무한 참조가 발생한다.



그리고, Order의 member 필드와 delivery 필드를

Lazy 로딩으로 설정해 놓았으므로 

위에서는 for 문을 통해 강제 초기화를 시켜줬다.



Lazy 로딩은 초기화를 시켜주지 않으면 프록시 객체가 들어있게 된다.

**jackson 라이브러리**는 기본적으로 이 프록시 객체를 json으로 어떻게 생성할지 모르기 때문에

예외가 발생한다.



이때는 **Hibernate5Module**을 스프링 빈으로 등록하면 해결할 수 있다.



> **Hibernate5Module 등록**
>
> ```java
> @Bean
> Hibernate5Module hibernate5Module() {
>     return new Hibernate5Module();
> }
> ```
>
> - 기본적으로 초기화된 프록시 객체만 노출, 
>
>   초기화 되지 않은 프록시 객체는 노출 안함.





<u>**근데 그냥 이렇게 하지말고,**</u> 

<u>**Entity 대신 DTO로 변환해서 반환하는 것이 더 좋다.**</u>







## 엔티티를 DTO로 변환 (fetch join 사용 X)

이번에는 DTO로 응답해보자.



API 컨트롤러에 가서 아래와 같이 작성한다.

```java
/**
 *  주문 조회 V2. 엔티티를 DTO로 반환 (fetch join 사용 X)
 *  - 단점: 지연로딩으로 쿼리 N번 호출
 */
@GetMapping("/api/v2/simple-orders")
public List<OrderResponseV2DTO> ordersV2() {
    List<Order> orders = orderRepository.findAll();
     
    return orders.stream()    
        .map(o -> new OrderResponseV2DTO(o))
        .collect(Collectors.toList());
}
```



위에서 findAll 메서드는 그냥 모든 Order를 가져오는 메서드다.

findAll 메서드에서 **fetch join**을 사용하지 않았기 때문에



불러온 모든 Order 엔티티의 

member 필드와 delivery 필드는 

초기화되지 않은 **프록시 객체**가 들어가 있다.





여기서 DTO 코드를 살펴보면,

```java
public class OrderResponseV2DTO {

    private Long orderId;
    private String name;
    private LocalDateTime orderDate;
    private OrderStatus orderStatus;
    private Address address;

    public OrderResponseV2DTO(Order order) {
        orderId = order.getId();
        orderDate = order.getOrderDate();
        orderStatus = order.getStatus();
        
        name = order.getMember().getName();          // 지연로딩
        address = order.getDelivery().getAddress();  // 지연로딩
    }
}
```



Order 엔티티를 OrderResponseV2DTO로 변환하는 과정에서

지연로딩이 일어나게 된다.



이때, Order 객체의 member, delivery에 들어있는 프록시 객체를 초기화하기 위해

**추가적인 조회 쿼리가 발생한다.**





<u>총 몇 번의 쿼리</u>가 나가게 될까?

- Order 조회 1번 (이때 order 조회 결과 수를 N이라고 하자)

- Order의 member 필드 초기화 -> 지연 로딩 조회 N번
- Order의 delivery 필드 초기화 -> 지연 로딩 조회 N번

즉, 1 + N + N번의 쿼리가 실행된다. (최악의 경우)





![](https://docs.google.com/drawings/d/e/2PACX-1vR00SJBigWQrzIUmjXzbXuk_z1pcHDR-vAA9sVGQSOrBxDBvAm1ehgoTG7q5PIzi7JQSnjtz9XkCY9L/pub?w=2247&h=1135)

위의 그림과 같은 경우

Order의 조회 결과가 3건(N == 3)이므로,

1 + 3 + 3 = 7번의 쿼리가 나가게 된다.





어떤 쿼리가 나가는지 직접 살펴보자.





### 실행된 쿼리

```sql
(1번 쿼리)
    select
        order0_.order_id as order_id1_6_,
        order0_.delivery_id as delivery4_6_,
        order0_.member_id as member_i5_6_,
        order0_.order_date as order_da2_6_,
        order0_.status as status3_6_ 
    from
        orders order0_
        
    
(2번 쿼리)
    select
        member0_.member_id as member_i1_4_0_,
        member0_.city as city2_4_0_,
        member0_.street as street3_4_0_,
        member0_.zipcode as zipcode4_4_0_,
        member0_.name as name5_4_0_ 
    from
        member member0_ 
    where
        member0_.member_id=6
        
        
(3번 쿼리)
    select
        delivery0_.delivery_id as delivery1_2_0_,
        delivery0_.city as city2_2_0_,
        delivery0_.street as street3_2_0_,
        delivery0_.zipcode as zipcode4_2_0_,
        delivery0_.status as status5_2_0_ 
    from
        delivery delivery0_ 
    where
        delivery0_.delivery_id=1


(4번 쿼리)
    select
        member0_.member_id as member_i1_4_0_,
        member0_.city as city2_4_0_,
        member0_.street as street3_4_0_,
        member0_.zipcode as zipcode4_4_0_,
        member0_.name as name5_4_0_ 
    from
        member member0_ 
    where
        member0_.member_id=7


(5번 쿼리)
    select
        delivery0_.delivery_id as delivery1_2_0_,
        delivery0_.city as city2_2_0_,
        delivery0_.street as street3_2_0_,
        delivery0_.zipcode as zipcode4_2_0_,
        delivery0_.status as status5_2_0_ 
    from
        delivery delivery0_ 
    where
        delivery0_.delivery_id=2


(6번 쿼리)
    select
        member0_.member_id as member_i1_4_0_,
        member0_.city as city2_4_0_,
        member0_.street as street3_4_0_,
        member0_.zipcode as zipcode4_4_0_,
        member0_.name as name5_4_0_ 
    from
        member member0_ 
    where
        member0_.member_id=8


(7번 쿼리)
    select
        delivery0_.delivery_id as delivery1_2_0_,
        delivery0_.city as city2_2_0_,
        delivery0_.street as street3_2_0_,
        delivery0_.zipcode as zipcode4_2_0_,
        delivery0_.status as status5_2_0_ 
    from
        delivery delivery0_ 
    where
        delivery0_.delivery_id=3
```









## 엔티티를 DTO로 변환 (fetch join 사용 버전)

그럼 이번에는 **fetch join**까지 사용해서 쿼리를 최적화 해보자.



```java
/**
 *  주문 조회 V3. 엔티티를 조회해서 DTO로 변환 (fetch join 사용 O)
 *  - fetch join으로 쿼리 1번 호출
*/
@GetMapping("/api/v3/simple-orders")
public List<OrderResponseDTO> ordersV3() {
    
    List<Order> orders = orderRepository.findAllWithMemberDelivery();
    
    
    return orders.stream()
        .map(o -> new OrderResponseDTO(o))
        .collect(Collectors.toList());
}
```





바뀐 것은 findAll() 메서드 대신 **findAllWithMemberDelivery()** 메서드를 호출한다는 것 뿐이다.

findAllWithMemberDelivery() 메서드는 다음과 같다.

```java
public List<Order> findAllWithMemberDelivery() {
        
    return em.createQuery(    
        "select o from Order o " +        
                 "join fetch o.member m " +        
                 "join fetch o.delivery d", Order.class)       
        .getResultList();    
}
```



위의 fetch join 쿼리는

Order를 가져올 때 inner join으로 member와 delivery까지 함께 들고온다.





### 실행된 쿼리

```sql
    select
        order0_.order_id as order_id1_6_0_,
        member1_.member_id as member_i1_4_1_,
        delivery2_.delivery_id as delivery1_2_2_,
        order0_.delivery_id as delivery4_6_0_,
        order0_.member_id as member_i5_6_0_,
        order0_.order_date as order_da2_6_0_,
        order0_.status as status3_6_0_,
        member1_.city as city2_4_1_,
        member1_.street as street3_4_1_,
        member1_.zipcode as zipcode4_4_1_,
        member1_.name as name5_4_1_,
        delivery2_.city as city2_2_2_,
        delivery2_.street as street3_2_2_,
        delivery2_.zipcode as zipcode4_2_2_,
        delivery2_.status as status5_2_2_ 
    from
        orders order0_ 
    inner join
        member member1_ 
            on order0_.member_id=member1_.member_id 
    inner join
        delivery delivery2_ 
            on order0_.delivery_id=delivery2_.delivery_id
```

한 방 쿼리로 다 들고온다.



**<u>7번 나가던게 1번으로 줄었다!!</u>**

만약 Order의 조회 건수가 1000000건이었다면 쿼리가 (1 + 1000000 + 1000000) = 2000001번이 나갔을 것이다.

하지만 fetch join을 사용하면 이런 경우에도 쿼리가 1번 나간다.









## JPA에서 DTO로 바로 조회

```java
/**
 *  V4. JPA에서 DTO로 바로 조회
 *  - 쿼리 1번 호출
 *  - select 절에서 원하는 데이터만 선택해서 조회
 */
@GetMapping("/api/v4/simple-orders")
public List<OrderSimpleQueryDTO> ordersV4() {
    return orderSimpleQueryRepository.findOrderDtos();    
}
```





엔티티를 조회하는 repository와 분리하여

OrderSimpleQueryRepository를 만든다.

```java
@Repository
@RequiredArgsConstructor
public class OrderSimpleQueryRepository {

    private final EntityManager em;

    public List<OrderSimpleQueryDTO> findOrderDtos() {
        return em.createQuery(
                "select new com.example.apioptimization.dto.order." +
                        "OrderSimpleQueryDTO(o.id, m.name, o.orderDate, o.status, d.address) " +
                        "from Order o " +
                        "join o.member m " +
                        "join o.delivery d", OrderSimpleQueryDTO.class)
                .getResultList();
    }
}
```



바로 위 버전에서는 fetch join을 이용해서

member 테이블의 모든 컬럼들을 다 들고 왔는데 (delivery도 마찬가지)

여기서는 응답에 필요한 데이터(member 이름)만 쏙 뽑아서 

DTO로 변환해 반환한다.





### 실행된 쿼리

```sql
    select
        order0_.order_id as col_0_0_,
        member1_.name as col_1_0_,
        order0_.order_date as col_2_0_,
        order0_.status as col_3_0_,
        delivery2_.city as col_4_0_,
        delivery2_.street as col_4_1_,
        delivery2_.zipcode as col_4_2_ 
    from
        orders order0_ 
    inner join
        member member1_ 
            on order0_.member_id=member1_.member_id 
    inner join
        delivery delivery2_ 
            on order0_.delivery_id=delivery2_.delivery_id
```







이 방법의 **장점**은 다음과 같다.

- SELECT 절에서 원하는 데이터를 직접 선택하므로 

  `DB -> 애플리케이션` 네트워크 용량을 최적화할 수 있다.

  (하지만 생각보다 미비하다고 한다.)



**단점**

- Repository 재사용성이 떨어진다.

  API 스펙에 맞춘 코드가 Repository에 들어가게 된다.









## 정리

그렇다면 **엔티티를 조회해서 DTO로 변환하는 방법**과, **DTO로 바로 조회하는 방법** 

두 가지 중에 어떤 방법을 선택해야 할까?

두 가지 방법은 각각 장단점이 있다.

엔티티로 조회하면 Repository 재사용성도 좋고, 개발도 단순해진다.



따라서 권장하는 방법은 다음과 같다.



**쿼리 방식 선택 권장 순서 (by 김영한)**

1. 우선 엔티티를 DTO로 변환하는 방법을 선택한다.

2. 필요하면 fetch join으로 성능 최적화 한다.   (-> 대부분의 성능 이슈가 여기서 해결된다.)

3. 그래도 안되면 DTO로 직접 조회하는 방법을 사용한다.

4. 최후의 방법은 JPA가 제공하는 네이티브 SQL이나 스프링 JDBC Template을 사용해서

   SQL을 직접 사용한다.

