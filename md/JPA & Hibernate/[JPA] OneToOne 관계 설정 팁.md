# [JPA] OneToOne 관계 설정 팁

> https://www.popit.kr/spring-boot-jpa-step-08-onetoone-%EA%B4%80%EA%B3%84-%EC%84%A4%EC%A0%95-%ED%8C%81/



**알아볼 것**

- 외래 키는 어느 테이블에 두는 것이 좋은가?
- 양방향 연관 관계 편의 메서드
- 제약 조건으로 인한 안정성 및 성능 향상



## 1. 도메인

**Coupon**

```java
public class Coupon {
    @Id @GeneratedValue
    private long id;
    
    private double discountAmount;
    
    private boolean use;
    
    @OneToOne()
    private Order order;
}
```



**Order**

```java
public class Order {
    @Id @GeneratedValue
    private long id;
    
    private double price;
    
    @OneToOne
    @JoinColumn()
    private Coupon coupon;
}
```







## 2. 외래 키는 어느 테이블에 두는 것이 좋은가?

일대다 관계에서는 다 쪽에 외래 키를 관리하게 되지만,

상대적으로 일대일 관계 설정에서는 외래 키를 어디 두어야 할지 고민해야 한다.



JPA에서는 외래 키를 갖는 쪽이 연관 관계의 주인이 되고 

연관 관계의 주인만이 DB 테이블과 매핑되고 외래 키를 관리(등록&middot;수정&middot;삭제)할 수 있기 때문이다.



**(1) Order가 연관관계의 주인인 경우**

```java
public class Order {
    @Id @GeneratedValue
    private long id;
    
    private double price;
    
    @OneToOne
    @JoinColumn(name = "coupon_id")
    private Coupon coupon;
}

public class Coupon {
    @Id @GeneratedValue
    private long id;
    
    private double discountAmount;
    
    private boolean use;
    
    @OneToOne(mappedBy = "coupon")
    private Order order;
}
```





**(2) Coupon이 연관관계의 주인인 경우**

```java
public class Order {
    @Id @GeneratedValue
    private long id;
    
    private double price;
    
    @OneToOne(mappedBy = "order")
    private Coupon coupon;
}

public class Coupon {
    @Id @GeneratedValue
    private long id;
    
    private double discountAmount;
    
    private boolean use;
    
    @OneToOne
    @JoinColumn(name = "order_id")
    private Order order;
}
```





### Sample Code

```java
// 주문시 1,000 할인 쿠폰을 적용해본 간단한 코드입니다. 
public Order order() {
    final Order order = Order.builder().price(1_0000).build(); // 10,000 상품주문
    Coupon coupon = couponService.findById(1); // 1,000 할인 쿠폰
    order.applyCoupon(coupon);
    return orderRepository.save(order);
}
@Test
public void order_쿠폰할인적용() {
    final Order order = orderService.order();
    assertThat(order.getPrice(), is(9_000D)); // 1,000 할인 적용 확인
    final Order findOrder = orderService.findOrder(order.getId());
    System.out.println("couponId : "+ findOrder.getCoupon().getId()); // couponId : 1 (coupon_id 외래 키를 저장 완료)
}
```





**Order가 주인일 경우 장점: INSERT SQL이 한 번만 실행**

```sql
-- order가 연관 관계의 주인일 경우 SQL
insert into orders (id, coupon_id, price) values (null, ?, ?) 

-- coupon이 연관 관계의 주인일 경우 SQL
insert into orders (id, price) values (null, ?)
update coupon set discount_amount=?, order_id=?, use=? where id=?
```

어쨋든 Order 테이블에 데이터를 저장해야 하는데,

Order 엔티티가 연관관계의 주인이면 쿼리 한 방이면 충분하지만 

Coupon이 연관관계의 주인이면 Order 테이블에 저장하는 쿼리 한 방, 해당 Order가 사용한 쿠폰 정보를 업데이트 하기 위한 쿼리 한 방이 나가게된다.





**Order가 주인일 경우 단점: 연관 관계 변경 시 취약**

위의 상황은 주문 한 개에 쿠폰을 한 개 사용할 수 있는 상황이었지만,

요구사항이 바뀌어서 주문 한 개에 쿠폰을 여러 개 사용할 수 있게 되면  쿠폰에 외래 키가 있어야 된다.

따라서 DB 컬럼들을 이전해야 하기 때문에 상당히 골치 아프다.







## 2. 양방향 연관관계 편의 메서드



```java
// Order가 연관관계의 주인일 경우 예제
class Coupon {
    ...
    // 연관관계 편의 메소드
    public void use(final Order order) {
        this.order = order;
        this.use = true;
    }
}

class Order {
    private Coupon coupon; //(1)
    ...
    // 연관관계 편의 메소드
    public void applyCoupon(final Coupon coupon) {
        this.coupon = coupon;
        coupon.use(this);
        price -= coupon.getDiscountAmount();
    }
}


// 주문 생성시 1,000 할인 쿠폰 적용
public Order order() {
    final Order order = Order.builder().price(1_0000).build(); // 10,000 상품주문
    Coupon coupon = couponService.findById(1); // 1,000 할인 쿠폰
    order.applyCoupon(coupon);
    return orderRepository.save(order);
}
```





양방향 연관 관계 편의 메서드를 작성해야 하는 이유??

```java
public void use(final Order order) {
//  this.order = order; 해당코드를 주석했을 때 테스트 코드
    this.use = true;
} 

@Test
public void use_메서드에_order_set_필요이유() {
    final Order order = orderService.order();
    assertThat(order.getPrice(), is(9_000D)); // 1,000 할인 적용 확인
    final Coupon coupon = order.getCoupon();
    assertThat(coupon.getOrder(), is(notNullValue())); // 해당 검사는 실패한다.
}
```

order를 바인딩하는 코드를 주석하고 해당 코드를 돌려보면 실패하게 됩니다. 

일반적으로 생각했을 때 order 생성 시 1,000할인 쿠폰을 적용했기 때문에 해당 쿠폰에도 주문 객체가 들어갔을 거로 생각할 수 있습니다. 

하지만 위의 주석시킨 코드가 그 기능을 담당했기 때문에 쿠폰 객체의 주문 값은 null인 상태입니다. 

**즉 순수한 객체까지 고려한 양방향 관계를 고려하는 것이 바람직하고 그것이 안전합니다.**







## 3. 제약 조건으로 인한 안정성 및 성능 향상

모든 주문에 할인 쿠폰이 반드시 적용된다면 `@JoinColumn`의 `nullable` 옵션을 false로 주는 것이 좋다.

`NOT NULL` 제약 조건을 준수해서 안정성이 보장된다.

```java
public class Order {
    ...
    @OneToOne
    @JoinColumn(name = "coupon_id", referencedColumnName = "id", nullable = false)
    private Coupon coupon;
}
```





또한, 외래 키에 `NOT NULL` 제약 조건을 설정하면 값이 있는 것을 보장한다.

따라서 JPA는 이때 내부 조인 SQL을 만들어주고, 이것은 외부 조인보다 성능과 최적화에 더 좋다.







- **nullable = false 해준 경우 / inner join 나감**

![](https://i.imgur.com/94To549.png)



- **nullable = false 안 해준 경우 / outer join 나감**

![](https://i.imgur.com/bHfKh8m.png)

