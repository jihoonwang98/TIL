# [JPA - Best Practice] Step-05 OneToMany 관계 설정 팁



### 도메인 

배송:배송상태 == 1:N



## 1. 배송 - 배송상태

- 배송이 있고 배송상태를 나타내는 배송 로그가 있다.
- 각각의 관계는 1:N 관계다.
- 아래와 같은 JSON을 갖는다.

```json
{
  "address": {
    "address1": "서울 특별시...",
    "address2": "신림 ....",
    "zip": "020...."
  },
  "logs": [
    {
      "status": "PENDING"
    },
    {
      "status": "DELIVERING"
    },
    {
      "status": "COMPLETED"
    }
  ]
}
```





## 2. 관계 설정

```java
public class Delivery {

    @Embedded
    private Address address;

    @OneToMany(mappedBy = "delivery", cascade = CascadeType.PERSIST, orphanRemoval = true, fetch = FetchType.EAGER)
    private List<DeliveryLog> logs = new ArrayList<>();

    @Embedded
    private DateTime dateTime;
    ....
}

public class DeliveryLog {

    @Enumerated(EnumType.STRING)
    @Column(name = "status", nullable = false, updatable = false)
    private DeliveryStatus status;

    @ManyToOne
    @JoinColumn(name = "delivery_id", nullable = false, updatable = false)
    private Delivery delivery;

    @Embedded
    private DateTime dateTime;

    ....
}


@Embeddable
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class Address {

    @NotEmpty
    @Column(name = "address1", nullable = false)
    private String address1;

    @NotEmpty
    @Column(name = "address2", nullable = false)
    private String address2;

    @NotEmpty
    @Column(name = "zip", nullable = false)
    private String zip;

    @Builder
    public Address(String address1, String address2, String zip) {
        this.address1 = address1;
        this.address2 = address2;
        this.zip = zip;
    }
}


@Embeddable
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class DateTime {

    @CreatedDate
    @Column(name = "created_at")
    private LocalDateTime createdAt;

    @LastModifiedDate
    @Column(name = "update_At")
    private LocalDateTime updatedAt;
}
```

- @Embedded 타입으로 빼놓은 Address를 그대로 사용했다.
  - 이처럼 핵심 도메인에 대해서 데이터의 연관성이 있는 것들을 Embedded로 분리해놓으면 좋다.
- DateTime 클래스도 Embedded 타입으로 지정해서 반복적인 생성일, 수정일 칼럼들을 일관성 있고 편리하게 생성할 수 있다.



**지금부터는 1:N 관계 팁에 관한 이야기입니다.**

- Delivery를 통해서 DeliveryLog를 관리하므로 `CascadeType.PERSIST` 설정을 주었습니다.

- 1:N 관계를 맺을 경우 `List`를 주로 사용하는데 객체 생성을 `null`로 설정하는 것보다 

  `new ArrayList<>();`로 초기화하는 것이 바람직합니다.

  - 이유:

    ```java
    private void verifyStatus(DeliveryStatus status, Delivery delivery) {
        if (!delivery.getLogs().isEmpty()) {
        ...
        }
    }
    ```

    초기화하지 않았을 경우에는 `null`로 초기화되며 `ArrayList`에서 지원해주는 함수를 사용할 수 없게됩니다.

    1:N 관계에서 N이 없는 경우 `null`인 상태인 것보다 Empty 상태가 훨씬 직관적입니다.

    `null`의 경우 값을 못 가져온 것인지 값이 없는 것인지 의미가 분명하지 않습니다.



## 3. 객체의 상태는 언제나 자기 자신이 관리합니다.

```java
public class DeliveryLog {
  @Id
  @GeneratedValue
  private long id;
  ....

  private void cancel() {
      verifyNotYetDelivering();
      this.status = DeliveryStatus.CANCELED;
  }

  private void verifyNotYetDelivering() {
      if (isNotYetDelivering()) throw new DeliveryAlreadyDeliveringException();
  }

  private void verifyAlreadyCompleted() {
    if (isCompleted())
        throw new IllegalArgumentException("It has already been completed and can not be changed.");
  }
}
```

- 객체의 상태는 언제나 자기 자신이 관리합니다. 즉, 자신이 생성되지 못할 이유도 자기 자신이 관리해야 한다고 생각합니다. 위의 로직은 다음과 같습니다.
  - `cancel()`: 배송을 취소하기 위해서는 아직 배달이 시작하기 이전의 상태여야 가능합니다.
  - `verifyAlreadyCompleted()`: 마지막 로그가 COMPLETED인 경우 더는 로그를 기록할 수 없습니다.



