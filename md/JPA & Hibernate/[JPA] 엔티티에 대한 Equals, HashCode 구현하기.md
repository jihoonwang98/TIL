# [JPA] 엔티티에 대한 Equals, HashCode 구현하기



> https://vladmihalcea.com/hibernate-facts-equals-and-hashcode/
>
> https://vladmihalcea.com/how-to-implement-equals-and-hashcode-using-the-jpa-entity-identifier/



## 1. Introduction

모든 자바 객체들은 `equals`, `hashCode` 메서드들을 상속받지만, 이것들은 Value Object들에 한해서만 유용하고, 

stateless behavior-oriented object들에 대해서는 쓸모가 없다.



참조 값(주소)을 `==` 연산자를 통해 비교하는 것은 간단하지만,

객체 동등성(객체 값)을 비교하는 것은 약간 더 복잡하다.





## 2. Requirements

Hibernate는 dirty checking 할 때 대부분의 경우에는  `equals`와 `hashCode` 메서드를 필요로 하지는 않는다.



Hibernate 공식 문서가 위의 두 가지 메서드를 구현해야 한다고 명시해놓은 상황은 아래와 같다.

- `Set` 컬렉션에 엔티티를 추가할 때
- 새로운 Persistence Context에 엔티티를 reattaching 할 때



These requirements arise from the `Object.equals` “*consistent*” constraint, leading us to the following principle:



**An entity must be equal to itself across all [JPA object states](https://vladmihalcea.com/a-beginners-guide-to-jpa-hibernate-entity-state-transitions/)**:

- transient
- attached
- detached
- removed (as long as the object is marked to be removed and it still living on the Heap)



Therefore, we can conclude that:

- We can’t use an auto-incrementing database id in the `hashCode` method since the transient and the attached object versions will no longer be located in the same hashed bucket.
- We can’t rely on the default `Object` `equals` and `hashCode` implementations since two entities loaded in two different persistence contexts will end up as two different Java objects, therefore breaking the all-states equality rule.
- So, if Hibernate uses the equality to uniquely identify an `Object`, for its whole lifetime, we need to find the right combination of properties satisfying this requirement.





## 3. Business key equality

Business key는 모든 엔티티 객체들이 각각 고유한 값을 가져야 하는 엔티티 필드를 말한다.

Business key는 또한 어떠한 persistence 기술에 대해서도 독립적이다. (DB auto incremented id와는 반대로..)



그래서, Business key는 엔티티를 생성하는 바로 그 순간 세팅되고 그 뒤로는 쭉 변화되어서는 안된다.

그러면 적절한 Business key를 찾는 과정을 담은 예시를 살펴보자.



### Root Entity Use Case (어떠한 Parent 종속성도 가지고 있지 않은 엔티티)

```java
@Entity
public class Company {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;
 
    @Column(unique = true, updatable = false)
    private String name;
 
    @Override
    public int hashCode() {
        HashCodeBuilder hcb = new HashCodeBuilder();
        hcb.append(name);
        return hcb.toHashCode();
    }
 
    @Override
    public boolean equals(Object obj) {
        if (this == obj) {
            return true;
        }
        if (!(obj instanceof Company)) {
            return false;
        }
        Company that = (Company) obj;
        EqualsBuilder eb = new EqualsBuilder();
        eb.append(name, that.name);
        return eb.isEquals();
    }
}
```

위의 `Company` 엔티티의 `name` 필드는 unique한 Business key의 역할을 할 수 있다.

그러므로, `unique=true` 옵션과 `updatable=false` 옵션을 줘서 변하지 않게 했다.

그래서 만약 두 `Company`가 같은 `name`을 가지고 있으면 다른 필드들이 어떻든 상관없이 같은 `Company`임을 알 수 있게 된다.

그래서 `hashCode`와 `equals` 메서드 모두 `name` 필드를 이용해 구현했다.





### Children entities with an EAGER fetched parent

```java
@Entity
public class Product {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;
 
    @Column(updatable = false)
    private String code;
 
    @ManyToOne(fetch = FetchType.EAGER)
    @JoinColumn(name = "company_id",
                nullable = false, updatable = false)
    private Company company;
 
    @OneToMany(fetch = FetchType.LAZY,
               cascade = CascadeType.ALL,
               mappedBy = "product",
               orphanRemoval = true)
    @OrderBy("index")
    private Set images = new LinkedHashSet();
 
    @Override
    public int hashCode() {
        HashCodeBuilder hcb = new HashCodeBuilder();
        hcb.append(code);
        hcb.append(company);
        return hcb.toHashCode();
    }
 
    @Override
    public boolean equals(Object obj) {
        if (this == obj) {
            return true;
        }
        if (!(obj instanceof Product)) {
            return false;
        }
        Product that = (Product) obj;
        EqualsBuilder eb = new EqualsBuilder();
        eb.append(code, that.code);
        eb.append(company, that.company);
        return eb.isEquals();
    }
}
```

이번 예제에서는 위의 `Product` 엔티티를 조회할 때 `Company`를 즉시 로딩하게 설정했다.

그리고 `Product`의 `code` 필드는 unique 하지가 않아서 Business key의 역할을 하지 못하는 상황이다.

이 경우에는 `Company` 엔티티의 Business key를 활용할 수 있다.

`Company` 엔티티에 대한 참조를 non-updatable하게 설정해서 `equals/hashCode` 제약을 깨는 것을 방지하자.



그런데 이 방법은 문제가 있다.

만약에 `Company` 엔티티가 `Product` 엔티티를 Set 형태로 가지고 있고, 아래와 같은 메서드를 호출하면,

```java
public void removeChild(Child child) {
    children.remove(child);
    child.setParent(null);
}
```

`equals/hashCode` 제약이 깨지게 된다.

그래서 양방향 연관관계에서는 이런 방식의 `equals/hashCode`를 사용하는 것은 조심해야 한다.





### Children entities with a LAZY fetched parent

```java
@Entity
public class Image {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;
 
    @Column(updatable = false)
    private String name;
 
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "product_id", nullable = false,
                updatable = false)
    private Product product;
 
    @Override
    public int hashCode() {
        HashCodeBuilder hcb = new HashCodeBuilder();
        hcb.append(name);
        hcb.append(product);
        return hcb.toHashCode();
    }
 
    @Override
    public boolean equals(Object obj) {
        if (this == obj) {
            return true;
        }
        if (!(obj instanceof Image)) {
            return false;
        }
        Image that = (Image) obj;
        EqualsBuilder eb = new EqualsBuilder();
        eb.append(name, that.name);
        eb.append(product, that.product);
        return eb.isEquals();
    }
}
```

위와 같이 LAZY 로딩으로 설정해놓고 `equals/hashCode`에 Parent 엔티티의 필드를 사용하는 경우,

LazyInitializationException을 만나게 될 수 있다.

그러므로, 이렇게 LAZY 로딩으로 설정한 경우 이런 방식은 위험하다.





### Children entities ignoring the parent

따라서 결론부터 말하자면, Business key를 설정할 때는 parent reference는 고려하지 않는 게 좋다.





---



# [JPA] JPA Entity Identifier로 어떻게 equals와 HashCode를 구현할 것인가?



# 1. Introduction

바로 위의 포스팅에서 살펴 본 것처럼, `equals`와 `hashCode`를 구현할 때 JPA Entity Business key를 사용하는 것이 가장 좋은 선택이다.

하지만, 모든 엔티티들이 unique한 business key를 가지고 있는 것은 아니다.

그래서 우리는 PK를 이용해야만 할 수도 있다.

하지만 Entity Id로 equality를 구현하는 것은 매우 힘들다. 

이번 포스팅은 어떻게 Entity Id로 equality를 어떠한 문제도 없이 구현하는 지를 다룬다.



## 2. Test harness

`equals`와 `hashCode`를 구현할 때, 딱 한 가지 머리 속에 새겨둬야 할 규칙이 있다.

> **`Equals`와 `hashCode`는 반드시 모든 Entity State Transitions에 걸쳐 일관성 있게 동작해야 한다.**



위 규칙을 테스트하기 위해 아래와 같은 테스트 케이스를 작성할 수 있다.

```java
protected void assertEqualityConsistency(
        Class<T> clazz,
        T entity) {
 
    Set<T> tuples = new HashSet<>();
 
    assertFalse(tuples.contains(entity));
    tuples.add(entity);
    assertTrue(tuples.contains(entity));
 
    doInJPA(entityManager -> {
        entityManager.persist(entity);
        entityManager.flush();
        assertTrue(
            "The entity is not found in the Set after it's persisted.",
            tuples.contains(entity)
        );
    });
 
    assertTrue(tuples.contains(entity));
 
    doInJPA(entityManager -> {
        T entityProxy = entityManager.getReference(
            clazz,
            entity.getId()
        );
        assertTrue(
            "The entity proxy is not equal with the entity.",
            entityProxy.equals(entity)
        );
    });
 
    doInJPA(entityManager -> {
        T entityProxy = entityManager.getReference(
            clazz,
            entity.getId()
        );
        assertTrue(
            "The entity is not equal with the entity proxy.",
            entity.equals(entityProxy));
    });
 
    doInJPA(entityManager -> {
        T _entity = entityManager.merge(entity);
        assertTrue(
            "The entity is not found in the Set after it's merged.",
            tuples.contains(_entity)
        );
    });
 
    doInJPA(entityManager -> {
        entityManager.unwrap(Session.class).update(entity);
        assertTrue(
            "The entity is not found in the Set after it's reattached.",
            tuples.contains(entity)
        );
    });
 
    doInJPA(entityManager -> {
        T _entity = entityManager.find(clazz, entity.getId());
        assertTrue(
            "The entity is not found in the Set after it's loaded in a different Persistence Context.",
            tuples.contains(_entity)
        );
    });
 
    doInJPA(entityManager -> {
        T _entity = entityManager.getReference(clazz, entity.getId());
        assertTrue(
            "The entity is not found in the Set after it's loaded as a proxy in a different Persistence Context.",
            tuples.contains(_entity)
        );
    });
 
    T deletedEntity = doInJPA(entityManager -> {
        T _entity = entityManager.getReference(
            clazz,
            entity.getId()
        );
        entityManager.remove(_entity);
        return _entity;
    });
 
    assertTrue(
        "The entity is not found in the Set even after it's deleted.",
        tuples.contains(deletedEntity)
    );
}
```





## 3. Natural Id

첫 번째로 테스트할 Use case는 natural id 매핑이다.

```java
@Entity
public class Book implements Identifiable<Long> {
 
    @Id
    @GeneratedValue
    private Long id;
 
    private String title;
 
    @NaturalId
    private String isbn;
 
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Book)) return false;
        Book book = (Book) o;
        return Objects.equals(getIsbn(), book.getIsbn());
    }
 
    @Override
    public int hashCode() {
        return Objects.hash(getIsbn());
    }
 
    //Getters and setters omitted for brevity
}
```

isbn 프로퍼티는 `@NaturalId`이다.

그러므로 unique하고 not nullable해야 한다.

`equals`와 `hashCode` 모두 isbn 프로퍼티를 이용해 구현하고 있다.



아래와 같은 테스트를 잘 통과한다.

```java
Book book = new Book();
book.setTitle("High-PerformanceJava Persistence");
book.setIsbn("123-456-7890");
 
assertEqualityConstraints(Book.class, book);
```





## 4. Default Object `equals` and `hashCode`

만약 엔티티가 어떠한 칼럼도 `@NaturalId`로 사용할 수 없다면 어떻게 할까?

처음으로 드는 생각은 그냥 `equals`랑 `hashCode` 구현안하고 내버려두는 것이다.

```java
@Entity(name = "Book")
public class Book implements Identifiable<Long> {
 
    @Id
    @GeneratedValue
    private Long id;
 
    private String title;
 
    //Getters and setters omitted for brevity
}
```



그러나 아래 테스트를 돌리면,

```java
Book book = new Book();
book.setTitle("High-PerformanceJava Persistence");
 
assertEqualityConstraints(Book.class, book);
```



Hibernate가 예외를 던진다.

```java
java.lang.AssertionError: The entity is not found after it's merged
```





## 5. Using the Entity Identifier for `equals` and `hashCode`

default `equals`와 `hashCode`가 잘 작동 안하면, 이번에는 Entity Identifier를 사용해보자.

IDE를 이용해서 id 필드를 이용해 `equals`와 `hashCode`를 구현해 보자.

```java
@Entity
public class Book implements Identifiable<Long> {
 
    @Id
    @GeneratedValue
    private Long id;
 
    private String title;
 
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Book)) return false;
        Book book = (Book) o;
        return Objects.equals(getId(), book.getId());
    }
 
    @Override
    public int hashCode() {
        return Objects.hash(getId());
    }
 
    //Getters and setters omitted for brevity
}
```



테스트를 돌려보면, Hibernate가 오류를 던진다.

```java
java.lang.AssertionError: The entity is not found after it's persisted
```



엔티티가 처음 `Set`에 저장될 때, identifier가 null이다.

엔티티가 저장된 후에는, identifier가 자동으로 생성된 값이 할당되므로 hashCode 값이 바뀐다.

이러한 이유로 처음 `Set`에 저장한 엔티티를 저장한 후에는 찾을 수 없게 되는 것이다.





## 6. Fixing the entity identifier `equals` and `hashCode`

이전의 문제를 고치기 위해서는 오직 한가지 해결책이 존재한다.

`hashCode`가 언제나 같은 값을 반환해야 하게 만들면 된다.

```java
@Entity
public class Book implements Identifiable<Long> {
 
    @Id
    @GeneratedValue
    private Long id;
 
    private String title;
 
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
 
        if (!(o instanceof Book))
            return false;
 
        Book other = (Book) o;
 
        return id != null &&
               id.equals(other.getId());
    }
 
    @Override
    public int hashCode() {
        return getClass().hashCode();
    }
 
    //Getters and setters omitted for brevity
}
```

Also, when the entity identifier is `null`, we can guarantee equality only for the same object references. Otherwise, no transient object is equal to any other transient or persisted object. That’s why the identifier equality check is done only if the current `Object` identifier is not null.

With this implementation, the `equals` and `hashCode` test runs fine for all entity state transitions. The reason why it works is that the hashCode value does not change, hence, we can rely on the `java.lang.Object` reference equality as long as the identifier is `null`.

