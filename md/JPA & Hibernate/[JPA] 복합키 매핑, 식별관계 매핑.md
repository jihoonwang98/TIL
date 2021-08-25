# [JPA] 복합키 매핑, 식별관계 매핑



> 참고: https://steady-hello.tistory.com/106



## 복합키 매핑

복합키(PK가 두 개 이상으로 구성)를 표현할 때 JPA에서는 무조건 **식별자 클래스**를 만들어야 한다.

그리고 식별자 클래스는 `@IdClass`, `@EmbeddedId` 둘 애너테이션 중 하나를 선택해서 사용한다.

- `@EmbeddedId`: 객체지향스러운 방법
- `@IdClass`: DB스러운 방법



식별자 클래스는 `equals()`, `hashCode()`를 재정의해야 한다.

그래야 영속성 컨텍스트가 식별자를 비교할 때 사용할 수 있기 때문이다.





### 복합키 매핑 방법 두 가지

#### (1) `@IdClass` 사용

**Parent**

```java
@Entity
@IdClass(ParentId.class)
public class Parent {

    @Id
    @Column(name = "parent_id1")
    private String id1;

    @Id
    @Column(name = "parent_id2")
    private String id2;

    private String name;
}
```





**ParentId**

```java
@AllArgsConstructor
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@EqualsAndHashCode
public class ParentId implements Serializable {

    private String id1;
    private String id2;
}
```





**`@IdClass` 복합키 만드는 방법?**

- 식별자 클래스
  - `Serializable` 인터페이스 구현해야 함.
  - `equals()`, `hashCode()` 구현해야 함.
  - 기본 생성자 선언해야 함.
  - 클래스의 접근자 public이어야 함.
  - **<u>식별자 클래스의 필드명과 엔티티에서 사용하는 식별자의 필드명이 동일해야 함.</u>**

- 엔티티
  - 클래스에다가 `@IdClass(식별자_클래스)` 애너테이션 붙이기
  - 식별자 클래스에 정의된 필드명 그대로 필드 정의하고 `@Id` 붙이기
    - 칼럼명 바꾸고 싶으면 `@Column(name = "")` 애너테이션으로 바꾸자







#### (2) `@EmbeddedId` 사용

**Parent**

```java
@Entity
public class Parent {

    @EmbeddedId
    private ParentId id;    
    
    private String name;
}
```



**ParentId**

```java
@Embeddable
@AllArgsConstructor
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@EqualsAndHashCode
public class ParentId implements Serializable {

    @Column(name = "parent_id1")
    private String id1;

    @Column(name = "parent_id2")
    private String id2;
}
```



**`@EmbeddedId` 복합키 만드는 방법?**

- 식별자 클래스
  - `Serializable` 인터페이스 구현해야 함.
  - `equals()`, `hashCode()` 구현해야 함.
  - 기본 생성자 선언해야 함.
  - 클래스의 접근자 public이어야 함.
  - <u>**식별자 클래스에 `@Embeddable`을 붙여줘야 함.**</u>

- 엔티티
  - 식별자 클래스를 필드로 정의하고 `@EmbeddedId` 애너테이션을 붙여준다.









## 식별, 비식별 관계

기준: FK(외래 키)가 PK(기본 키)에 포함되는 지 여부에 따라 분류

- **식별 관계**: 부모 테이블의 기본 키를 받아서 자식 테이블의 **기본 키 + 외래 키**로 사용
- **비식별 관계**: 부모 테이블의 기본 키를 받아서 자식 테이블의 **외래 키**로 사용
  - 필수적 비식별 관계: 외래 키에 NULL을 허용하지 않음
  - 선택적 비식별 관계: 외래 키에 NULL을 허용













