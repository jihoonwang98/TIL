# Defining JPA Entites :duck:



> **[참고]**: Defining JPA Entites
>
> https://www.baeldung.com/jpa-entities
>
> 의역과 오역에 주의...



# 1. Introduction :earth_africa:

이번 포스팅에서는, JPA에서 엔터티를 정의하고 커스터마이징하는 다양한 어노테이션들과 함께 엔터티의 기초를 다진다.





## 2. Entity :eagle:

JPA의 **Entity**들은 DB에 persist 될 수 있는 data를 표현하는 **POJO**들일 뿐이다.

**Entity**는 DB에 저장된 **테이블**을 나타낸다.

모든 **Entity**의 **인스턴스**들은 테이블의 한 **행**을 나타낸다.





### 2.1 The *Entity* Annotation 

student 데이터를 표현하는 *Student*라는 POJO가 있고, 

우리가 이 객체를 DB에 저장하고 싶다고 하자.

```java
public class Student {
    
    // fields, getters and setters
    
}
```





이를 위해서, 우리는 JPA가 이 객체를 인지할 수 있게 하기 위해 이 객체를 **Entity**로 지정해야 한다.

그럼, 이 Student 클래스 위에 **@Entity** 어노테이션을 붙여보자. (반드시 이 어노테이션은 class level에서 지정되어야 한다.)

**또한, <u>Entity는 no-arg 생성자와 primary key를 가져야만 한다</u>.**

```java
@Entity
public class Student {
    
    // fields, getters and setters
    
}
```





**Entity의 이름**은 기본적으로 **class의 이름**으로 설정된다.

바꾸고 싶을 때는 **@Entity** 어노테이션의 **name** 속성에 지정해주면 된다.

```java
@Entity(name="student")
public class Student {
    
    // fields, getters and setters
    
}
```





다양한 JPA 구현체에서 우리가 정의한 Entity를 subclassing 해야 하기 때문에,

**<u>절대로 Entity class를 *final* class로 선언하면 안된다</u>**.







### 2.2 The *Id* Annotation

각각의 JPA Entity는 반드시 그 객체를 유일하게 식별하는 **Primary key(주식별자)**를 가져야만 한다.

이떄, **@Id** 어노테이션을 사용하면 **Primary Key**를 지정할 수 있다.

우리는 **@Id** 어노테이션과 함께 **@GeneratedValue** 어노테이션을 지정함으로써 식별자를 다양한 방법으로 만들어낼 수 있다.



**@GeneratedValue** 어노테이션의 **strategy** 속성으로 가능한 값은

- **AUTO**
- **TABLE**
- **SEQUENCE**
- **IDENTITY** 

가 있다.



```java
@Entity
public class Student {
    @Id
    @GeneratedValue(strategy=GenerationType.AUTO)
    private Long id;
    
    private String name;
    
    // getters and setters
}
```

**strategy**로 *GenerationType.AUTO*를 지정하면,

JPA provide가 식별자를 만들 때 아무 strategy나 사용한다.



If we annotate the entity's fields, the JPA provider will use these fields to get and set the entity's state. In addition to Field Access, we can also do Property Access or Mixed Access, which enables us to use both Field and Property access in the same entity. (??)





### 2.3 The *Table* Annotation

대부분의 경우, 

**DB의 Table 이름**과

**Entity의 이름**이

일치하지 않을 것이다.



이러한 경우에는, Entity 객체에 **@Table** 어노테이션을 이용해 **테이블 이름**을 지정할 수 있다.

즉, **@Table(name="DB\_테이블\_이름")**을 붙여주면 그 Entity가 지정한 테이블과 매핑된다.

```java
@Entity
@Table(name="STUDENT")
public class Student {
    
    // fields, getters and setters
    
}
```



***schema*** 속성을 통해 schema를 지정할 수도 있따.

```java
@Entity
@Table(name="STUDENT", schema="SCHOOL")
public class Student {
    
    // fields, getters and setters
    
}
```





만약, **@Table** 어노테이션을 사용하지 않는다면, **Entity 이름**이 **Table 이름**으로 간주될 것이다.





### 2.4 The *Column* Annotation

**@Table** 어노테이션과 비슷하게, **@Column** 어노테이션을 사용해서 **테이블의 칼럼**의 세부 사항을 지정할 수 있다.

**@Column** 어노테이션은 *name, length, nullable, unique* 같은 다양한 속성들을 가지고 있다.

```java
@Entity
@Table(name="STUDENT")
public class Student {
    @Id
    @GeneratedValue(strategy=GenerationType.AUTO)
    private Long id;
    
    @Column(name="STUDENT_NAME", length=50, nullable=false, unique=false)
    private String name;
    
    // other fields, getters and setters
}
```



**@Column**의 *name* 속성은 **테이블의 column 이름**을 지정한다.

**@Column**의 *length* 속성은 해당 칼럼의 **length**를 지정한다.

**@Column**의 *nullable* 속성은 해당 칼럼이 **nullable** 한지 아닌지를 지정한다.

**@Column**의 *unique* 속성은 해당 칼럼이 **unique** 한지 아닌지를 지정한다.



만약, **@Column**을 붙이지 않는다면, **필드명**이 테이블의 **칼럼명**으로 간주된다.







### 2.5 The *Transient* Annotation

때때로, 우리는 필드를 non-persistent 하게 만들고 싶을 때도 있을 수 있다.

이럴 때 **@Trasient** 어노테이션을 사용할 수 있다. 이 어노테이션을 붙이면 해당 필드는 persist 되지 않는다.



```java
@Entity
@Table(name="STUDENT")
public class Student {
    @Id
    @GeneratedValue(strategy=GenerationType.AUTO)
    private Long id;
    
    @Column(name="STUDENT_NAME", length=50, nullable=false)
    private String name;
    
    @Transient
    private Integer age;
    
    // other fields, getters and setters
}
```

위의 예에서, *age* 필드는 테이블에 persist 되지 않는다.







### 2.6 The *Temporal* Annotation

때때로, 우리는 테이블에 Date 값들을 저장하고 싶을 수도 있다.

이럴 때 **@Temporal** 어노테이션을 사용할 수 있다. 



```java
@Entity
@Table(name="STUDENT")
public class Student {
    @Id
    @GeneratedValue(strategy=GenerationType.AUTO)
    private Long id;
    
    @Column(name="STUDENT_NAME", length=50, nullable=false, unique=false)
    private String name;
    
    @Transient
    private Integer age;
    
    @Temporal(TemporalType.DATE)
    private Date birthDate;
    
    @Temporal(TemporalType.DATE)
    Date regDate;
    @Temporal(TemporalType.TIME)
    Date regTime;
    @Temporal(TemporalType.TIMESTAMP)
    Date regTimestamp;

    
    // other fields, getters and setters
}
```



JPA 2.2 버전에서부터는, 

*java.time.LocalDate, java.time.LocalTime, java.time.LocalDateTime, java.time.OffsetTime, java.time.OffsetDateTime*을 지원한다.





### 2.7 The *Enumerated* Annotation

때때로, Java **Enum** 타입을 persist 해야 할 상황이 있을 수 있다.

이럴 때 **@Enumerated** 어노테이션을 지정해서 **Enum**이 **<u>이름</u>**으로 persist 될지 **<u>순서(숫자)</u>**로 persist 될지를 결정할 수 있다.



```java
public enum Gender {
    MALE, 
    FEMALE
}
```



```java
@Entity
@Table(name="STUDENT")
public class Student {
    @Id
    @GeneratedValue(strategy=GenerationType.AUTO)
    private Long id;
    
    @Column(name="STUDENT_NAME", length=50, nullable=false, unique=false)
    private String name;
    
    @Transient
    private Integer age;
    
    @Temporal(TemporalType.DATE)
    private Date birthDate;
    
    @Enumerated(EnumType.STRING)
    private Gender gender;
    
    // other fields, getters and setters
}
```

**@Enumerated** 어노테이션을 지정하지 않으면 **Enum** 타입은 **순서 값(숫자)**으로 persist 된다.

**Enum** 타입을 **Enum 이름**으로 persist 하고 싶을 때, 어노테이션 값으로 *EnumType.STRING*을 지정해주면 된다.