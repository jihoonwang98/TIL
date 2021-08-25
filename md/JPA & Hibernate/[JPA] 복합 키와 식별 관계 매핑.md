# [JPA] 복합 키와 식별 관계 매핑



## 1. 식별 관계 vs 비식별 관계

분류 기준: 외래 키가 기본 키에 포함되는지 여부

- **식별 관계**: 부모 테이블의 기본 키를 내려 받아서 자식 테이블의 **기본 키 + 외래 키**로 사용
- **비식별 관계**: 부모 테이블의 기본 키를 내려 받아서 자식 테이블의 **외래키로만** 사용
  - **필수적 비식별 관계**(Mandatory): 외래 키에 `NULL`을 허용하지 않음
  - **선택적 비식별 관계**(Optional): 외래 키에 `NULL`을 허용





## 2. 복합키

기본 키를 구성하는 컬럼이 하나면 아래처럼 매핑한다.

```java
@Entity
public class Hello {
	@Id 
	private String id;
}
```



그럼 기본 키를 구성하는 컬럼이 두개면 어떻게 할까?

```java
@Entity
public class Hello {

	@Id 
	private String id1;
	
	@Id
	private String id2;
}
```

이렇게 하면 되지 않을까??

-> 안된다... 매핑 오류가 발생한다..



JPA에서는 식별자를 둘 이상 사용하는 **복합키**를 사용하려면 무조건 별도의 **<u>식별자 클래스</u>**를 만들어야 한다.

JPA는 복합키를 지원하기 위해 `@IdClass`와 `@EmbeddedId` 2가지 방법을 제공한다.





### 식별자 클래스의 조건

- `Serializable` 인터페이스를 구현해야 한다.
- `equals`, `hashCode`를 구현해야 한다.
- 기본 생성자가 있어야 한다.
- 식별자 클래스는 public이어야 한다.





### (1) `@IdClass` - RDB에 가까운 방법

**식별자 클래스**

```java
@AllArgsConstructor
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@EqualsAndHashcode
public class HelloId implements Serializable {

	private String id1;
    private String id2;
}
```



**엔티티**

```java
@Entity
@IdClass(HelloId.class)
public class Hello {
	
    @Id
    @Column(name="hello_id1")
    private Long id1;
    
    @Id
    @Column(name="hello_id2")
    private Long id2;
}
```





**<u>주의점</u>**

- **식별자 클래스의 속성명과 엔티티에서 사용하는 식별자의 속성명이 같아야 한다.**
  - Hello.**id1** == HelloId.**id1**
  - Hello.**id2** == HelloId.**id2**





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



   

### (2) `@EmbeddedId` - 객체지향적인 방법

**식별자 클래스**

```java
@Embeddable
@AllArgsConstructor
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@EqualsAndHashcode
public class HelloId implements Serializable {

    @Column(name = "hello_id1")
	private String id1;
    
    @Column(name = "hello_id2")
    private String id2;
}
```



**엔티티**

```java
@Entity
public class Hello {

    @EmbeddedId
    private HelloId id;   
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







## 3. 비식별 관계 매핑

아래와 같은 비식별 관계를 매핑해보자.



![](https://docs.google.com/drawings/d/sRkMZLWe-Hwqd34_d9mTu2A/image?parent=e/2PACX-1vRVvbIKomKLDTKGV_Zpd2jnSflB7IGvZt3_68Ps6WLyfUO3_b16JGyGje2BnfNAFQnyCXgohLQOH9QO&rev=50&h=125&w=601&ac=1)





**부모 엔티티 식별자 클래스**

```java
@AllArgsConstructor
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@EqualsAndHashcode
public class HelloId implements Serializable {

	private String id1;
    private String id2;
}
```



**부모 엔티티**

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



**자식 엔티티**

```java
@Entity
public class Child {

    @Id
    private String id;

    @ManyToOne
    @JoinColumns({
            @JoinColumn(name = "parent_id1",
                        referencedColumnName = "parent_id1"),
            @JoinColumn(name = "parent_id2",
                        referencedColumnName = "parent_id2")
    })
    private Parent parent;
}
```



- 외래 키를 매핑할 때 여러 컬럼을 매핑해야 하므로 `@JoinColumns` 애너테이션을 사용하고,

  각각의 외래 키 컬럼을 `@JoinColumn`으로 매핑한다.







## 4. 식별 관계 매핑

아래 그림과 같은 식별 관계를 매핑해보자.



![](https://docs.google.com/drawings/d/sBLAMESb_AHppeYmXVxAVKw/image?parent=e/2PACX-1vRVvbIKomKLDTKGV_Zpd2jnSflB7IGvZt3_68Ps6WLyfUO3_b16JGyGje2BnfNAFQnyCXgohLQOH9QO&rev=143&h=156&w=601&ac=1)



위의 그림을 보면 부모, 자식, 손자까지 계속 기본 키를 전달하는 식별 관계다.

식별 관계에서 자식 테이블은 부모 테이블의 기본 키를 포함해서 복합 키를 구성해야 하므로 

`@IdClass`나 `@EmbeddedId`를 사용해서 식별자를 매핑해야 한다.





### (1) `@IdClass`와 식별 관계

**Parent**

```java
@Entity
public class Parent {

    @Id
    @Column(name = "parent_id")
    private String id;
    private String name;
}
```



**ChildId**

```java
@AllArgsConstructor
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@EqualsAndHashCode
public class ChildId implements Serializable {

    private String parent;
    private String childId;
}
```



**Child**

```java
@Entity
@IdClass(ChildId.class)
public class Child {

    @Id
    @ManyToOne
    @JoinColumn(name = "parent_id")
    private Parent parent;

    @Id
    @Column(name = "child_id")
    private String childId;

    private String name;
}
```



**GrandChildId**

```java
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@AllArgsConstructor
@EqualsAndHashCode
public class GrandChildId implements Serializable {

    private ChildId child;
    private String id;
}
```



**GrandChild**

```java
@Entity
@IdClass(GrandChildId.class)
public class GrandChild {

    @Id
    @ManyToOne
    @JoinColumns({
            @JoinColumn(name = "parent_id"),
            @JoinColumn(name = "child_id")
    })
    private Child child;

    @Id
    @Column(name = "grandchild_id")
    private String id;

    private String name;
}
```





### (2) `@EmbeddedId`와 식별 관계

`@EmbeddedId`로 식별 관계를 구성할 때는 `@MapsId`를 사용해야 한다.



**Parent**

```java
@Entity
public class Parent {

    @Id
    @Column(name = "parent_id")
    private String id;
    
    private String name;
}
```



**ChildId**

```java
@Embeddable
@AllArgsConstructor
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@EqualsAndHashCode
public class ChildId implements Serializable {

    private String parentId;
    
    @Column(name = "child_id")
    private String childId;
}
```













