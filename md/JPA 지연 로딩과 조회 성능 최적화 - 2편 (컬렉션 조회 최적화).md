# JPA 지연 로딩과 조회 성능 최적화 - 2편 (컬렉션 조회 최적화)



> 참고:  인프런 강의 by 김영한
>
> [실전! 스프링 부트와 JPA 활용2 - API 개발과 성능 최적화](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8-JPA-API%EA%B0%9C%EB%B0%9C-%EC%84%B1%EB%8A%A5%EC%B5%9C%EC%A0%81%ED%99%94/dashboard)



[1편](https://kgvovc.tistory.com/64)에 이어지는 내용이다.



1편에서는 xToOne(ManyToOne, OneToOne) 관계를 최적화 했다면,

이번편에서는 **OneToMany** 관계를 최적화 해보자.

즉, 엔티티가 필드로 **리스트**를 갖고 있는 경우를 최적화 해보자는 것이다.





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

**Order : OrderItem은 <u>일대다</u>**



> @OneToMany는 기본적으로  fetch 전략이 LAZY로 설정되어 있다.









## V1: 엔티티 직접 노출

권장하지 않는 방법이므로 간단히 보고 넘어가자.



```java
@RestController
@RequiredArgsConstructor
public class OrderApiController {
    
    private final OrderRepository orderRepository;
 
 /**
   * V1. 엔티티 직접 노출
   * - Hibernate5Module 모듈 등록, LAZY=null 처리
   * - 양방향 관계 문제 발생 -> @JsonIgnore
   */
    
    @GetMapping("/api/v1/orders")
    public List<Order> ordersV1() {
        List<Order> all = orderRepository.findAll();
        for (Order order : all) {
            order.getMember().getName(); //Lazy 강제 초기화
            order.getDelivery().getAddress(); //Lazy 강제 초기환
            List<OrderItem> orderItems = order.getOrderItems();
            orderItems.stream().forEach(o -> o.getItem().getName()); //Lazy 강제초기화
        }
        return all;
    }
}
```



**엔티티를 직접 노출하는 방법의 단점**

- 엔티티가 바뀌면 API 스펙이 바뀐다.

- 양방향 연관관계면 무한 루프에 걸리지 않게 엔티티 중 한 곳에 `@JsonIgnore`를 추가해야 한다. 

  (엔티티에 화면과 관련된 로직이 들어간다.)



그냥 이 방법으로 하지 말자.







## V2: 엔티티로 조회해서 DTO로 변환 후 반환 (fetch join 안 쓴 버전)



```java
/**
  *  V2. 엔티티를 조회해서 DTO로 변환 (fetch join 사용 X)
  *  - 트랜잭션 안에서 지연 로딩 필요
  */
    
@GetMapping("/api/v2/orders")
public List<OrderDTO> ordersV2() {
    
    List<Order> orders = orderRepository.findAll();
    
    return orders.stream()
        .map(o -> new OrderDTO(o))
        .collect(Collectors.toList());
}
```





**DTO 클래스 만들기**

```java
@Data
public class OrderDTO {

    private Long orderId;
    private String name;
    private LocalDateTime orderDate;
    private OrderStatus orderStatus;
    private Address address;
    private List<OrderItemDTO> orderItems;


    public OrderDTO(Order order) {
        orderId = order.getId();
        orderDate = order.getOrderDate();
        orderStatus = order.getStatus();
        name = order.getMember().getName();        // 지연 로딩
        address = order.getMember().getAddress();  // 지연 로딩(위에서 가져와서 쿼리는 안나감)
        orderItems = order.getOrderItems().stream()
                .map(orderItem -> new OrderItemDTO(orderItem))
                .collect(Collectors.toList());
    }
}


@Data
public class OrderItemDTO {

    private String itemName;
    private int orderPrice;
    private int count;

    public OrderItemDTO(OrderItem orderItem) {
        itemName = orderItem.getItem().getName();
        orderPrice = orderItem.getOrderPrice();
        count = orderItem.getCount();
    }
}
```





쿼리는 총 몇방이 나갈까?

`List<Order> orders = orderRepository.findAll()`에서

**모든 Order들을 가져오는데 1방**이 나가고,





아래 코드에서 알 수 있듯이,

가져온 order들을 DTO로 반환하는 과정에서 지연로딩이 일어나 추가적인 쿼리가 나가게 된다.

`return orders.stream().map(o -> new OrderDTO(o)) .collect(Collectors.toList())`

이 과정을 좀 더 자세히 살펴보자.





먼저, order 한 개를 OrderDTO로 변환하는 과정에서

아래와 같은 변환 과정을 거친다.

```java
public OrderDTO(Order order) {
        orderId = order.getId();
        orderDate = order.getOrderDate();
        orderStatus = order.getStatus();
        name = order.getMember().getName();        // 지연 로딩
        address = order.getMember().getAddress();  // 지연 로딩(위에서 가져와서 쿼리는 안나감)
        orderItems = order.getOrderItems().stream()  // 지연 로딩
                .map(orderItem -> new OrderItemDTO(orderItem))
                .collect(Collectors.toList());
}
```



orderId, orderDate, orderStatus는 Order 테이블을 통째로 가져와서 이미 가지고 있는 상태이므로, 추가 쿼리가 나가지 않는다.



문제는 `order.getMember().getName()`과 `order.getMember().getAddress()`이다.

우리는 order의 member_id만 알고, 해당 member_id의 member 이름과 주소는 모른다.

(member 필드는 프록시 객체가 들어가 있는 상황이다.)

따라서 member의 name과 address에 접근할 때 

프록시 객체를 초기화(지연 로딩)하기 위해 추가적인 쿼리가 나간다.



여기서는 `order.getMember().getName()`이 실행될 때 쿼리가 나가서 해당 member를 가져오고,

`order.getMember().getAddress()`는 가져온 member에서 꺼내 쓰기 때문에 추가적인 쿼리는 나가지 않는다.





마지막으로 `order.getOrderItems().stream().map(...)` 코드에서도

orderItem의 필드에 접근하게 되면 위에서와 같은 원리로 지연 로딩이 일어나게 된다.

따라서, order_id에 해당하는 OrderItem들을 조회하는 쿼리가 추가로 나가게 된다.





그런데, 여기서 끝이 아니다.

OrderItemDTO는 아래와 같이 변환 과정을 거칠 때, 또 다시 지연 로딩이 일어난다.

```java
public OrderItemDTO(OrderItem orderItem) {
        itemName = orderItem.getItem().getName();    // 지연 로딩
        orderPrice = orderItem.getOrderPrice();
        count = orderItem.getCount();
}
```



orderItem의 item도 프록시 객체이기 때문에

`orderItem.getItem().getName()`을 하는 순간 프록시 객체를 초기화하기 위해

item 조회 쿼리가 나간다.





이 과정을 그림으로 나타내면 아래와 같다.



![](https://docs.google.com/drawings/d/e/2PACX-1vRv8H29Sk9Q1y6n1iQsx-oHCmHHgtoYdO4f-W8kwsryzndPHyMjvioOG9IrFqk1R-tXWbUcn5vfZ9rb/pub?w=3527&h=1161)





### 실행된 쿼리

```sql
(1번 쿼리 / orderRepository.findAll();)
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
        orderitems0_.order_id as order_id5_5_0_,
        orderitems0_.order_item_id as order_it1_5_0_,
        orderitems0_.order_item_id as order_it1_5_1_,
        orderitems0_.count as count2_5_1_,
        orderitems0_.item_id as item_id4_5_1_,
        orderitems0_.order_id as order_id5_5_1_,
        orderitems0_.order_price as order_pr3_5_1_ 
    from
        order_item orderitems0_ 
    where
        orderitems0_.order_id=1
        

(4번 쿼리)  
    select
        item0_.item_id as item_id2_3_0_,
        item0_.name as name3_3_0_,
        item0_.price as price4_3_0_,
        item0_.stock_quantity as stock_qu5_3_0_,
        item0_.author as author6_3_0_,
        item0_.isbn as isbn7_3_0_,
        item0_.dtype as dtype1_3_0_ 
    from
        item item0_ 
    where
        item0_.item_id=1
        

(5번 쿼리)  
    select
        item0_.item_id as item_id2_3_0_,
        item0_.name as name3_3_0_,
        item0_.price as price4_3_0_,
        item0_.stock_quantity as stock_qu5_3_0_,
        item0_.author as author6_3_0_,
        item0_.isbn as isbn7_3_0_,
        item0_.dtype as dtype1_3_0_ 
    from
        item item0_ 
    where
        item0_.item_id=2
        
        
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
        member0_.member_id=7
        

(7번 쿼리)
    select
        orderitems0_.order_id as order_id5_5_0_,
        orderitems0_.order_item_id as order_it1_5_0_,
        orderitems0_.order_item_id as order_it1_5_1_,
        orderitems0_.count as count2_5_1_,
        orderitems0_.item_id as item_id4_5_1_,
        orderitems0_.order_id as order_id5_5_1_,
        orderitems0_.order_price as order_pr3_5_1_ 
    from
        order_item orderitems0_ 
    where
        orderitems0_.order_id=2
        
     
(8번 쿼리)
    select
        item0_.item_id as item_id2_3_0_,
        item0_.name as name3_3_0_,
        item0_.price as price4_3_0_,
        item0_.stock_quantity as stock_qu5_3_0_,
        item0_.author as author6_3_0_,
        item0_.isbn as isbn7_3_0_,
        item0_.dtype as dtype1_3_0_ 
    from
        item item0_ 
    where
        item0_.item_id=3
        
 
(9번 쿼리)
    select
        item0_.item_id as item_id2_3_0_,
        item0_.name as name3_3_0_,
        item0_.price as price4_3_0_,
        item0_.stock_quantity as stock_qu5_3_0_,
        item0_.author as author6_3_0_,
        item0_.isbn as isbn7_3_0_,
        item0_.dtype as dtype1_3_0_ 
    from
        item item0_ 
    where
        item0_.item_id=4
        

(10번 쿼리)
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
        

(11번 쿼리)
    select
        orderitems0_.order_id as order_id5_5_0_,
        orderitems0_.order_item_id as order_it1_5_0_,
        orderitems0_.order_item_id as order_it1_5_1_,
        orderitems0_.count as count2_5_1_,
        orderitems0_.item_id as item_id4_5_1_,
        orderitems0_.order_id as order_id5_5_1_,
        orderitems0_.order_price as order_pr3_5_1_ 
    from
        order_item orderitems0_ 
    where
        orderitems0_.order_id=3
        

(12번 쿼리)
    select
        item0_.item_id as item_id2_3_0_,
        item0_.name as name3_3_0_,
        item0_.price as price4_3_0_,
        item0_.stock_quantity as stock_qu5_3_0_,
        item0_.author as author6_3_0_,
        item0_.isbn as isbn7_3_0_,
        item0_.dtype as dtype1_3_0_ 
    from
        item item0_ 
    where
        item0_.item_id=5
        

(13번 쿼리)
    select
        item0_.item_id as item_id2_3_0_,
        item0_.name as name3_3_0_,
        item0_.price as price4_3_0_,
        item0_.stock_quantity as stock_qu5_3_0_,
        item0_.author as author6_3_0_,
        item0_.isbn as isbn7_3_0_,
        item0_.dtype as dtype1_3_0_ 
    from
        item item0_ 
    where
        item0_.item_id=6
        

(14번 쿼리)
    select
        item0_.item_id as item_id2_3_0_,
        item0_.name as name3_3_0_,
        item0_.price as price4_3_0_,
        item0_.stock_quantity as stock_qu5_3_0_,
        item0_.author as author6_3_0_,
        item0_.isbn as isbn7_3_0_,
        item0_.dtype as dtype1_3_0_ 
    from
        item item0_ 
    where
        item0_.item_id=7
```





정리하면 

**SQL 실행 수**는

- order - 1번
- member, address - N번(order 조회 수만큼)
- orderItem - N번(order 조회 수만큼)
- item - M번(orderItem 조회 수만큼)

총 1 + N + N + M번의 쿼리가 실행된다.



위의 예에서는 

order 조회 건수가 3건이니까 N == 3,

orderItem 조회 건수가 7건이니까 M == 7,

즉, 1 + 3 + 3 + 7 == 14번의 쿼리가 실행된다.





## V3: 엔티티로 조회해서 DTO로 변환 후 반환 (fetch join 최적화 버전)



```java
/**
  *  V3. 엔티티를 조회해서 DTO로 변환 (fetch join 사용 O)
  *  - 페이징 시에는 N 부분을 포기해야 함.
  *    (대신에 batch fetch size 옵션 주면 N -> 1 쿼리로 변경 가능)
  */

@GetMapping("/api/v3/orders")
public List<OrderDTO> ordersV3() {

    List<Order> orders = orderRepository.findAllWithItem();
    List<OrderDTO> result = orders.stream()
            .map(o -> new OrderDTO(o))
            .collect(Collectors.toList());
        return result;    
}
```



이전 버전과 차이는 **findAllWithItem()** 메서드 밖에 없다.





**OrderRepository에 findAllWithItem 메서드 추가**

```java
public List<Order> findAllWithItem() {
        
    return em.createQuery(
        "select o from Order o " +
            "join fetch o.member m " +        
            "join fetch o.delivery d " +
            "join fetch o.orderItems oi " +
            "join fetch oi.item i", Order.class)
        .getResultList();
}
```



위와 같이 fetch join을 이용하면 쿼리가 딱 1번만 실행된다.





### 실행된 쿼리

```sql
    select
        order0_.order_id as order_id1_6_0_,
        member1_.member_id as member_i1_4_1_,
        delivery2_.delivery_id as delivery1_2_2_,
        orderitems3_.order_item_id as order_it1_5_3_,
        item4_.item_id as item_id2_3_4_,
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
        delivery2_.status as status5_2_2_,
        orderitems3_.count as count2_5_3_,
        orderitems3_.item_id as item_id4_5_3_,
        orderitems3_.order_id as order_id5_5_3_,
        orderitems3_.order_price as order_pr3_5_3_,
        orderitems3_.order_id as order_id5_5_0__,
        orderitems3_.order_item_id as order_it1_5_0__,
        item4_.name as name3_3_4_,
        item4_.price as price4_3_4_,
        item4_.stock_quantity as stock_qu5_3_4_,
        item4_.author as author6_3_4_,
        item4_.isbn as isbn7_3_4_,
        item4_.dtype as dtype1_3_4_ 
    from
        orders order0_ 
    inner join
        member member1_ 
            on order0_.member_id=member1_.member_id 
    inner join
        delivery delivery2_ 
            on order0_.delivery_id=delivery2_.delivery_id 
    inner join
        order_item orderitems3_ 
            on order0_.order_id=orderitems3_.order_id 
    inner join
        item item4_ 
            on orderitems3_.item_id=item4_.item_id
```





**가져오는 테이블**



![](https://docs.google.com/drawings/d/e/2PACX-1vQHuzJJ2MVGCoVoBBX-5-hdUrOXcGQsIr-XETy0lZ3A7Lv9W0JMKkg1cjhW-IIb1rIzxsFGfgE5w7hW/pub?w=1431&h=232)



그런데, Order: OrderItems가 일대다 조인이라 

데이터가 뻥튀기된 것을 볼 수 있다.





이것 때문에 

```java
@GetMapping("/api/v3/orders")
public List<OrderDTO> ordersV3() {

    List<Order> orders = orderRepository.findAllWithItem();
    
    System.out.println("orders.size() = " + orders.size());     // size 찍어보기
    orders.stream().forEach(order -> System.out.println(order));  // 모든 order 출력
    
    List<OrderDTO> result = orders.stream()
            .map(o -> new OrderDTO(o))
            .collect(Collectors.toList());
        return result;    
}

// output
orders.size() = 7
Order(id=1, orderDate=2021-05-28T11:06:34.439958, status=ORDER)
Order(id=1, orderDate=2021-05-28T11:06:34.439958, status=ORDER)
Order(id=2, orderDate=2021-05-28T11:06:34.468880, status=ORDER)
Order(id=2, orderDate=2021-05-28T11:06:34.468880, status=ORDER)
Order(id=3, orderDate=2021-05-28T11:40:48.771289, status=ORDER)
Order(id=3, orderDate=2021-05-28T11:40:48.771289, status=ORDER)
Order(id=3, orderDate=2021-05-28T11:40:48.771289, status=ORDER)
```

orders의 size를 찍어보면 3이 아니라 7이 나온다.

orders에 들어있는 Order들도 중복 데이터가 들어있는 것을 볼 수 있다.





이것 때문에 우리가 API 응답으로 

아래와 같이 중복 데이터가 포함된 결과를 받게 된다.

```json

[
  {
    "orderId": 1,
    "name": "userA",
    "orderDate": "2021-05-28T11:06:34.439958",
    "orderStatus": "ORDER",
    "address": {
      "city": "서울",
      "street": "1",
      "zipcode": "1111"
    },
    "orderItems": [
      {
        "itemName": "JPA1 BOOK",
        "orderPrice": 10000,
        "count": 1
      },
      {
        "itemName": "JPA2 BOOK",
        "orderPrice": 20000,
        "count": 2
      }
    ]
  },
  {
    "orderId": 1,
    "name": "userA",
    "orderDate": "2021-05-28T11:06:34.439958",
    "orderStatus": "ORDER",
    "address": {
      "city": "서울",
      "street": "1",
      "zipcode": "1111"
    },
    "orderItems": [
      {
        "itemName": "JPA1 BOOK",
        "orderPrice": 10000,
        "count": 1
      },
      {
        "itemName": "JPA2 BOOK",
        "orderPrice": 20000,
        "count": 2
      }
    ]
  },
  {
    "orderId": 2,
    "name": "userB",
    "orderDate": "2021-05-28T11:06:34.46888",
    "orderStatus": "ORDER",
    "address": {
      "city": "진주",
      "street": "2",
      "zipcode": "2222"
    },
    "orderItems": [
      {
        "itemName": "SPRING1 BOOK",
        "orderPrice": 20000,
        "count": 3
      },
      {
        "itemName": "SPRING2 BOOK",
        "orderPrice": 40000,
        "count": 4
      }
    ]
  },
  {
    "orderId": 2,
    "name": "userB",
    "orderDate": "2021-05-28T11:06:34.46888",
    "orderStatus": "ORDER",
    "address": {
      "city": "진주",
      "street": "2",
      "zipcode": "2222"
    },
    "orderItems": [
      {
        "itemName": "SPRING1 BOOK",
        "orderPrice": 20000,
        "count": 3
      },
      {
        "itemName": "SPRING2 BOOK",
        "orderPrice": 40000,
        "count": 4
      }
    ]
  },
  {
    "orderId": 3,
    "name": "userC",
    "orderDate": "2021-05-28T11:40:48.771289",
    "orderStatus": "ORDER",
    "address": {
      "city": "부산",
      "street": "3",
      "zipcode": "3333"
    },
    "orderItems": [
      {
        "itemName": "Hibernate Master Book1",
        "orderPrice": 39500,
        "count": 16
      },
      {
        "itemName": "Hibernate Master Book2",
        "orderPrice": 23800,
        "count": 30
      },
      {
        "itemName": "Hibernate Master Book3",
        "orderPrice": 13000,
        "count": 5
      }
    ]
  },
  {
    "orderId": 3,
    "name": "userC",
    "orderDate": "2021-05-28T11:40:48.771289",
    "orderStatus": "ORDER",
    "address": {
      "city": "부산",
      "street": "3",
      "zipcode": "3333"
    },
    "orderItems": [
      {
        "itemName": "Hibernate Master Book1",
        "orderPrice": 39500,
        "count": 16
      },
      {
        "itemName": "Hibernate Master Book2",
        "orderPrice": 23800,
        "count": 30
      },
      {
        "itemName": "Hibernate Master Book3",
        "orderPrice": 13000,
        "count": 5
      }
    ]
  },
  {
    "orderId": 3,
    "name": "userC",
    "orderDate": "2021-05-28T11:40:48.771289",
    "orderStatus": "ORDER",
    "address": {
      "city": "부산",
      "street": "3",
      "zipcode": "3333"
    },
    "orderItems": [
      {
        "itemName": "Hibernate Master Book1",
        "orderPrice": 39500,
        "count": 16
      },
      {
        "itemName": "Hibernate Master Book2",
        "orderPrice": 23800,
        "count": 30
      },
      {
        "itemName": "Hibernate Master Book3",
        "orderPrice": 13000,
        "count": 5
      }
    ]
  }
]
```







이게 다 Order : OrderItem이 **일대다 관계**라서 데이터가 뻥튀기돼서 그렇다.

그럼 이걸 어떡하냐?

fetch join 쿼리에 가서 **<u>distinct</u>** 키워드를 붙여주자.

```java
public List<Order> findAllWithItem() {
        
    return em.createQuery(
        "select distinct o from Order o " +
            "join fetch o.member m " +        
            "join fetch o.delivery d " +
            "join fetch o.orderItems oi " +
            "join fetch oi.item i", Order.class)
        .getResultList();
}
```





### DISTINCT 붙인 후 실행된 쿼리

```sql
    select
        distinct order0_.order_id as order_id1_6_0_,
        member1_.member_id as member_i1_4_1_,
        delivery2_.delivery_id as delivery1_2_2_,
        orderitems3_.order_item_id as order_it1_5_3_,
        item4_.item_id as item_id2_3_4_,
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
        delivery2_.status as status5_2_2_,
        orderitems3_.count as count2_5_3_,
        orderitems3_.item_id as item_id4_5_3_,
        orderitems3_.order_id as order_id5_5_3_,
        orderitems3_.order_price as order_pr3_5_3_,
        orderitems3_.order_id as order_id5_5_0__,
        orderitems3_.order_item_id as order_it1_5_0__,
        item4_.name as name3_3_4_,
        item4_.price as price4_3_4_,
        item4_.stock_quantity as stock_qu5_3_4_,
        item4_.author as author6_3_4_,
        item4_.isbn as isbn7_3_4_,
        item4_.dtype as dtype1_3_4_ 
    from
        orders order0_ 
    inner join
        member member1_ 
            on order0_.member_id=member1_.member_id 
    inner join
        delivery delivery2_ 
            on order0_.delivery_id=delivery2_.delivery_id 
    inner join
        order_item orderitems3_ 
            on order0_.order_id=orderitems3_.order_id 
    inner join
        item item4_ 
            on orderitems3_.item_id=item4_.item_id
```

자세히 보면 이전의 쿼리랑 distinct 키워드만 하나 추가된 것을 알 수 있다.





그런데 테이블에 중복된 행이 존재하지 않기 때문에 

가져오는 테이블은 이전 쿼리랑 똑같이 아래와 같다.

![](https://docs.google.com/drawings/d/e/2PACX-1vQHuzJJ2MVGCoVoBBX-5-hdUrOXcGQsIr-XETy0lZ3A7Lv9W0JMKkg1cjhW-IIb1rIzxsFGfgE5w7hW/pub?w=1431&h=232)





아니 그러면 뭣하러 **distinct**를 붙인거냐?? 라고 물어보면,

아래 결과가 달라진다.

```java
@GetMapping("/api/v3/orders")
public List<OrderDTO> ordersV3() {

    List<Order> orders = orderRepository.findAllWithItem();
    
    System.out.println("orders.size() = " + orders.size());     // size 찍어보기
    orders.stream().forEach(order -> System.out.println(order));  // 모든 order 출력
    
    List<OrderDTO> result = orders.stream()
            .map(o -> new OrderDTO(o))
            .collect(Collectors.toList());
        return result;    
}

// output
orders.size() = 3
Order(id=1, orderDate=2021-05-28T11:06:34.439958, status=ORDER)
Order(id=2, orderDate=2021-05-28T11:06:34.468880, status=ORDER)
Order(id=3, orderDate=2021-05-28T11:40:48.771289, status=ORDER)
```



JPA의 **distinct**는 SQL에 distinct를 추가할 뿐만 아니라,

그 외에도 같은 엔티티가 조회되면, **<u>애플리케이션에서 중복을 걸러준다</u>**.



따라서, API 응답 결과도 아래와 같이

우리가 원했던 결과가 나오게 된다.

```json
[
  {
    "orderId": 1,
    "name": "userA",
    "orderDate": "2021-05-28T11:06:34.439958",
    "orderStatus": "ORDER",
    "address": {
      "city": "서울",
      "street": "1",
      "zipcode": "1111"
    },
    "orderItems": [
      {
        "itemName": "JPA1 BOOK",
        "orderPrice": 10000,
        "count": 1
      },
      {
        "itemName": "JPA2 BOOK",
        "orderPrice": 20000,
        "count": 2
      }
    ]
  },
  {
    "orderId": 2,
    "name": "userB",
    "orderDate": "2021-05-28T11:06:34.46888",
    "orderStatus": "ORDER",
    "address": {
      "city": "진주",
      "street": "2",
      "zipcode": "2222"
    },
    "orderItems": [
      {
        "itemName": "SPRING1 BOOK",
        "orderPrice": 20000,
        "count": 3
      },
      {
        "itemName": "SPRING2 BOOK",
        "orderPrice": 40000,
        "count": 4
      }
    ]
  },
  {
    "orderId": 3,
    "name": "userC",
    "orderDate": "2021-05-28T11:40:48.771289",
    "orderStatus": "ORDER",
    "address": {
      "city": "부산",
      "street": "3",
      "zipcode": "3333"
    },
    "orderItems": [
      {
        "itemName": "Hibernate Master Book1",
        "orderPrice": 39500,
        "count": 16
      },
      {
        "itemName": "Hibernate Master Book2",
        "orderPrice": 23800,
        "count": 30
      },
      {
        "itemName": "Hibernate Master Book3",
        "orderPrice": 13000,
        "count": 5
      }
    ]
  }
]
```





그런데 이 방법도 **단점**이 있다.

바로 **페이징이 불가능**하다는 점이다.



> **[참고]**
>
> **컬렉션 fetch join**을 사용하면 **<u>페이징이 불가능</u>**하다.
>
> Hibernate는 경고 로그를 남기면서 모든 데이터를 DB에서 읽어오고,
>
> <u>메모리에서 페이징 해버린다. **(매우 위험!!!!)**</u>





> **[참고]**
>
> 컬렉션 fetch join은 1개만 사용할 수 있다.
>
> 컬렉션 둘 이상에 fetch join을 사용하면 안된다.
>
> 데이터가 부정합하게 조회될 수 있다.





그래서 얘도 결국에는 쓰면 안된다. (데이터가 많으면 위험함, 페이징도 안됨)







## V3.1: 엔티티를 DTO로 변환 후 반환 (fecth join + @BatchSize 이용)



### 페이징

- 컬렉션을 fetch join하면 페이징이 불가능하다.

  - 일대다에서 일(1)을 기준으로 페이징하는 것이 목적이다.

    그런데 데이터는 다(N)를 기준으로 row가 생성된다.

  - Order를 기준으로 페이징하고 싶은데, 다(N)인 OrderItem을 조인하면 

    OrderItem이 기준이 돼버린다.





### 한계 돌파

그러면 **페이징 + 컬렉션 엔티티**를 함께 조회하려면 어떻게 해야할까?



- 먼저 **xToOne**(OneToOne, ManyToOne) 관계를 모두 fetch join한다.

  ToOne 관계는 row 수를 증가시키지 않으므로 페이징 쿼리에 영향을 주지 않는다.

- 컬렉션은 지연 로딩으로 조회한다.

- 지연 로딩 성능 최적화를 위해 

  `hibernate.default_batch_fetch_size`, `@BatchSize`를 적용한다.

  - hibernate.default_batch_fetch_size: 글로벌 설정

  - @BatchSize: 개별 최적화

  - 이 옵션을 사용하면 컬렉션이나, 프록시 객체를 한꺼번에 설정한 size 만큼

    **IN 쿼리로 조회**한다.







**OrderRepository에 findAllWithMemberDelivery 메서드 추가**

```java
public List<Order> findAllWithMemberDelivery(int offset, int limit) {
    return em.createQuery(
            "select o from Order o " +
                    "join fetch o.member m " +
                    "join fetch o.delivery d", Order.class)
            .setFirstResult(offset)
            .setMaxResults(limit)
            .getResultList();
}
```





```java
/**
 *  V3.1 엔티티를 조회해서 DTO로 변환, 페이징 고려
 *  - ToOne 관계만 우선 fetch join으로 모두 최적화
 *  - 컬렉션 관계는 hibernate.default_batch_fetch_size, @BatchSize로 최적화
 */
@GetMapping("/api/v3.1/orders")
public List<OrderDTO> ordersV3_page(
        @RequestParam(value = "offset", defaultValue = "0") int offset,
        @RequestParam(value = "limit", defaultValue = "100") int limit) {
    
    List<Order> orders = orderRepository.findAllWithMemberDelivery(offset, limit);
   
    List<OrderDTO> result = orders.stream()
            .map(o -> new OrderDTO(o))
            .collect(Collectors.toList());
    return result;
}
```





**최적화 옵션**

```yaml
spring:
  jpa:
    properties:
      hibernate:
        default_batch_fetch_size: 1000
```

- 개별로 설정하려면 `@BatchSize`를 적용하면 된다.







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
            on order0_.delivery_id=delivery2_.delivery_id limit 100
            
            
           
    select
        orderitems0_.order_id as order_id5_5_1_,
        orderitems0_.order_item_id as order_it1_5_1_,
        orderitems0_.order_item_id as order_it1_5_0_,
        orderitems0_.count as count2_5_0_,
        orderitems0_.item_id as item_id4_5_0_,
        orderitems0_.order_id as order_id5_5_0_,
        orderitems0_.order_price as order_pr3_5_0_ 
    from
        order_item orderitems0_ 
    where
        orderitems0_.order_id in (
            1, 2, 3
        )
        
        
        
    select
        item0_.item_id as item_id2_3_0_,
        item0_.name as name3_3_0_,
        item0_.price as price4_3_0_,
        item0_.stock_quantity as stock_qu5_3_0_,
        item0_.author as author6_3_0_,
        item0_.isbn as isbn7_3_0_,
        item0_.dtype as dtype1_3_0_ 
    from
        item item0_ 
    where
        item0_.item_id in (
            1, 2, 3, 4, 5, 6, 7
        )
```







이 방법의 **장점**

- **장점**

  - 쿼리 호출 수가 `1 + N` -> `1 + 1`으로 최적화 된다.

  - 조인보다 DB 데이터 전송량이 최적화 된다.

    (Order와 OrderItem을 조인하면 Order가 OrderItem만큼 중복해서 조회된다.

    이 방법은 각각 조회하므로 전송해야할 중복 데이터가 없다.)

  - fetch join 방식과 비교해서 쿼리 호출 수가 약간 증가하지만, DB 데이터 전송량이 감소한다.

  - 컬렉션 fetch join은 페이징이 불가능 하지만, 이 방법은 페이징이 가능하다.



- **결론**

  - ToOne 관계는 fetch join해도 페이징에 영향을 주지 않는다.

    따라서 ToOne 관계는 fetch join으로 쿼리 수를 줄이고, 

    나머지는 `hibernate.default_batch_fetch_size`로 최적화 하자.





> **[참고] `default_bacth_fetch_size` 크기 결정**
>
> 100 ~ 1000 사이를 선택하는 것을 권장.
>
> 이 전략은 **SQL IN 절**을 사용하는데,
>
> DB에 따라 IN 절 파라미터를 1000으로 제한하기도 한다.
>
> 
>
> 1000으로 잡으면 한 번에 1000개를 DB에서 애플리케이션에 불러오므로
>
> DB에 순간 부하가 증가할 수 있다.
>
> 하지만 애플리케이션은 100이든 1000이든 결국 전체 데이터를 로딩해야 하므로
>
> 메모리 사용량이 같다.
>
> 
>
> 1000으로 설정하는 것이 성능상 가장 좋지만, 결국 DB든 애플리케이션이든 
>
> 순간 부하를 어디까지 견딜 수 있는지로 결정하면 된다.







## V4: JPA에서 DTO로 직접 조회



```java
/**
 * V4. JPA에서 DTO로 바로 조회, 컬렉션 N 조회 (1 + N Query)
 * - 페이징 가능
 */
@GetMapping("/api/v4/orders")
public List<OrderQueryDTO> ordersV4() {
    return orderQueryRepository.findOrderQueryDtos();
}
```





```java
public class OrderQueryRepository {

    private final EntityManager em;


    /**
     *  컬렉션은 별도로 조회
     *  Query: 루트 1번, 컬렉션 N번
     *  단건 조회에서 많이 사용하는 방식
     */
    public List<OrderQueryDTO> findOrderQueryDtos() {

        // 루트 조회 (toOne 코드를 모두 한 번에 조회)
        List<OrderQueryDTO> result = findOrders();


        // 루프를 돌면서 컬렉션 추가 (추가 쿼리 실행)
        result.forEach(o -> {
            List<OrderItemQueryDTO> orderItems = findOrderItems(o.getOrderId());
            o.setOrderItems(orderItems);
        });

        return result;
    }


    /**
     *  1:N 관계(컬렉션)를 제외한 나머지를 한 번에 조회
     */
    private List<OrderQueryDTO> findOrders() {
        return em.createQuery(
                "select new com.example.apioptimization.dto.order." +
                        "OrderQueryDTO(o.id, m.name, o.orderDate, o.status, d.address) " +
                        "from Order o " +
                        "join o.member m " +
                        "join o.delivery d", OrderQueryDTO.class)
                .getResultList();
    }


    /**
     *  1:N 관계인 orderItems 조회
     */
    private List<OrderItemQueryDTO> findOrderItems(Long orderId) {
        return em.createQuery(
                "select new com.example.apioptimization.dto.order." +
                        "OrderItemQueryDTO(oi.order.id, i.name, oi.orderPrice, oi.count) " +
                        "from OrderItem oi " +
                        "join oi.item i " +
                        "where oi.order.id = :orderId", OrderItemQueryDTO.class)
                .setParameter("orderId", orderId)
                .getResultList();
    }

}
```





**DTO 클래스**

```java
@Data
@EqualsAndHashCode(of = "orderId")
public class OrderQueryDTO {

    private Long orderId;
    private String name;
    private LocalDateTime orderDate;
    private OrderStatus orderStatus;
    private Address address;
    private List<OrderItemQueryDTO> orderItems;

    public OrderQueryDTO(Long orderId, String name, LocalDateTime orderDate, OrderStatus orderStatus, Address address) {
        this.orderId = orderId;
        this.name = name;
        this.orderDate = orderDate;
        this.orderStatus = orderStatus;
        this.address = address;
    }
}

@Data
@AllArgsConstructor
public class OrderItemQueryDTO {

    @JsonIgnore
    private Long orderId;
    private String itemName;
    private int orderPrice;
    private int count;
}
```







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
            
            
    select
        orderitem0_.order_id as col_0_0_,
        item1_.name as col_1_0_,
        orderitem0_.order_price as col_2_0_,
        orderitem0_.count as col_3_0_ 
    from
        order_item orderitem0_ 
    inner join
        item item1_ 
            on orderitem0_.item_id=item1_.item_id 
    where
        orderitem0_.order_id=1
        
        
    select
        orderitem0_.order_id as col_0_0_,
        item1_.name as col_1_0_,
        orderitem0_.order_price as col_2_0_,
        orderitem0_.count as col_3_0_ 
    from
        order_item orderitem0_ 
    inner join
        item item1_ 
            on orderitem0_.item_id=item1_.item_id 
    where
        orderitem0_.order_id=2
        
        
    select
        orderitem0_.order_id as col_0_0_,
        item1_.name as col_1_0_,
        orderitem0_.order_price as col_2_0_,
        orderitem0_.count as col_3_0_ 
    from
        order_item orderitem0_ 
    inner join
        item item1_ 
            on orderitem0_.item_id=item1_.item_id 
    where
        orderitem0_.order_id=3
```





- Query: 루트 1번, 컬렉션 N번 실행

- ToOne(N:1, 1:1) 관계들을 먼저 조회하고, ToMany(1:N) 관계는 각각 별도로 처리한다.

  - 이런 방식을 택한 이유는 다음과 같다.
  - ToOne 관계는 조인해도 데이터 row 수가 증가하지 않는다.
  - ToMany(1:N) 관계는 조인하면 row 수가 증가한다.

- row 수가 증가하지 않는 ToOne 관계는 조인으로 최적화하기 쉬우므로 한번에 조회하고,

  ToMany 관계는 최적화하기 어려우므로 `findOrderItems()`같은 별도의 메서드로 조회한다.