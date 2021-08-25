# [JPA - Best Practice] Step-04 Embedded를 이용해 응집력 높이기

이번 포스팅에서는 Embedded를 이용해 Password 클래스를 통해서 Password 관련 응집력을 높이는 방법과

JPA에서 LocalDateTime을 활용하는 방법에 대해 중점적으로 포스팅을 진행해 보겠습니다.



### 중요 포인트

- Embeddable 타입의 Password 클래스 정의





## 1. Embeddable 타입의 Password 클래스 정의

### (1) 비밀번호 요구 사항

- 비밀번호 만료 기본 14일 기간이 있다.
- 비밀번호 만료 기간이 지나는 것을 알 수 있어야 한다.
- 비밀번호 5회 이상 실패했을 경우 더 이상 시도를 못하게 해야 한다.
- 비밀번호가 일치하는 경우 실패 카운트를 초기화해야 한다.
- 비밀번호 변경시 만료일이 현재시간 기준 14일로 연장되어야 한다.



```java
@Embeddable
public class Password {
    @Column(name = "password", nullable = false)
    private String value;

    @Column(name = "password_expiration_date")
    private LocalDateTime expirationDate;

    @Column(name = "password_failed_count", nullable = false)
    private int failedCount;

    @Column(name = "password_ttl")
    private long ttl;

    @Builder
    public Password(final String value) {
        this.ttl = 1209_604; // 1209_604 is 14 days
        this.value = encodePassword(value);
        this.expirationDate = extendExpirationDate();
    }

    public boolean isMatched(final String rawPassword) {
        if (failedCount >= 5)
            throw new PasswordFailedExceededException();

        final boolean matches = isMatches(rawPassword);
        updateFailedCount(matches);
        return matches;
    }

    public void changePassword(final String newPassword, final String oldPassword) {
        if (isMatched(oldPassword)) {
            value = encodePassword(newPassword);
            extendExpirationDate();
        }
    }
}
```

- 객체의 변경이나 질의는 반드시 해당 객체에 의해서 이루어져야 하는데 위의 요구 사항을 만족하는 로직들은 Password 객체 안에 있고 Password 객체를 통해서 모든 작업이 이루어진다.

  - 결과적으로 Password 관련 테스트 코드도 작성하기 쉬워지고 작은 단위로 테스트 코드를 작성하면 실패했을 때 원인도 찾기 쉬워진다.

  - 또한, Password의 책임이 명확해진다. 만약 Embeddable 타입으로 분리하지 않았을 경우

    해당 로직들은 모두 Account 클래스에 들어가 Account의 책임이 증가하는 것을 방지할 수 있다.