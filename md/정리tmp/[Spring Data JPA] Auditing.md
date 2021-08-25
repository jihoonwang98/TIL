# [Spring Data JPA] Auditing



```java
@Getter
@MappedSuperclass 
@EntityListeners(AuditingEntityListener.class) 
public abstract class BaseTimeEntity{

    // Entity가 생성되어 저장될 때 시간이 자동 저장됩니다.
    @CreatedDate
    private LocalDateTime createdDate;

    // 조회한 Entity 값을 변경할 때 시간이 자동 저장됩니다.
    @LastModifiedDate
    private LocalDateTime modifiedDate;
}
```

 



| 어노테이션                                     | 설명                                            |
| ---------------------------------------------- | ----------------------------------------------- |
| @EntityListeners(AuditingEntityListener.class) | 해당 클래스에 Auditing 기능을 포함              |
| @CreatedDate                                   | Entity가 생성되어 저장될 때 시간이 자동 저장    |
| @LastModifiedDate                              | 조회한 Entity의 값을 변경할 때 시간이 자동 저장 |



























