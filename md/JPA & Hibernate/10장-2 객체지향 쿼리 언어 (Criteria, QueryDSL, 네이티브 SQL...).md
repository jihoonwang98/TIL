# 10장 객체지향 쿼리 언어 - 2편



## 10.3 Criteria

Criteria 쿼리는 JPQL을 자바 코드로 작성하도록 도와주는 빌더 클래스 API다.

Criteria를 사용하면 문자가 아닌 코드로 JPQL을 작성하므로 문법 오류를 컴파일 단계에서 잡을 수 있고

문자 기반의 JPQL보다 동적 쿼리를 안전하게 생성할 수 있는 장점이 있다.

하지만 실제 Criteria를 사용해서 개발해보면 코드가 복잡하고 장황해서 직관적으로 이해가 힘들다는 단점도 있다.



Criteria는 결국 JPQL의 생성을 돕는 클래스 모음이다.



(중략)





## 10.4 QueryDSL

JPA Criteria는 문자가 아닌 코드로 JPQL을 작성하므로

문법 오류를 컴파일 단계에서 잡을 수 있고 

IDE 자동완성 기능의 도움을 받을 수 있는 등 여러가지 장점이 있다.

하지만 Criteria의 가장 큰 단점은 너무 복잡하고 어렵다는 것이다.

작성된 코드를 보면 그 복잡성으로 인해 어떤 JPQL이 생성될지 파악하기가 쉽지 않다.



쿼리를 문자가 아닌 코드로 작성해도, 쉽고 간결하며

그 모양도 쿼리와 비슷하게 개발할 수 있는 프로젝트가 바로 **QueryDSL**이다.

QueryDSL도 Criteria처럼 JPQL 빌더 역할을 하는데 JPA Criteria를 대체할 수 있다.



QueryDSL은 오픈소스 프로젝트다.

처음에는 HQL(하이버네이트 쿼리언어)을 코드로 작성할 수 있도록 해주는 프로젝트로 시작해서

지금은 JPA, JDO, JDBC, Lucene, Hibernate Search, 몽고DB, 자바 컬렉션 등을 다양하게 지원한다.

참고로 QueryDSL은 이름 그대로 쿼리 즉 데이터를 조회하는 데 기능이 특화되어 있다.





### 10.4.1 QueryDSL 설정



#### 필요 라이브러리

참고로 예제에 사용한 버전은 3.6.3이다.

아래와 같이 QueryDSL 라이브러리를 추가하자.



**pom.xml 추가**

```xml
<dependency>
	<groupId>com.mysema.querydsl</groupId>
    <artifactId>querydsl-jpa</artifactId>
    <version>3.6.3</version>
</dependency>

<dependency>
	<groupId>com.mysema.querydsl</groupId>
    <artifactId>querydsl-apt</artifactId>
    <version>3.6.3</version>
    <scope>provided</scope>
</dependency>
```

- **querydsl-jpa**: QueryDSL JPA 라이브러리
- **querydsl-apt**: 쿼리 타입(Q)을 생성할 때 필요한 라이브러리





#### 환경설정

QueryDSL을 사용하려면 Criteria의 메타 모델처럼 엔티티를 기반으로 

**쿼리 타입**이라는 클래스를 생성해야 한다.

아래의 예제와 같이 **쿼리 타입** 생성용 플러그인을 pom.xml에 추가하자.



**쿼리 타입 생성용 pom.xml 추가**

```xml
<build>
	<plugins>
    	<plugin>
        	<groupId>com.mysema.maven</groupId>
            <artifactId>apt-maven-plugin</artifactId>
            <version>1.1.3</version>
            <executions>
            	<execution>
                	<goals>
                    	<goal>process</goal>
                    </goals>
                    <configuration>
                    	<outputDirectory>target/generated-sources/java</outputDirectory>
                        <processor>com.mysem.query.apt.jpa.JPAAnnotationProcessor</processor>
                    </configuration>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```



콘솔에서 mvn compile을 입력하면

outputDirectory에 지정한 target/generated-sources 위치에 QMember.java처럼 Q로 시작하는 쿼리 타입들이 생성된다.

이제 target/generated-sources를 소스 경로에 추가하면 된다.





### 10.4.2 시작

QueryDSL을 어떻게 사용하는지 아래의 예제로 알아보자.



**QueryDSL 시작**

```java
public void queryDSL() {
    
    EntityManager em = emf.createEntityManager();
    JPAQuery query = new JPAQuery(em);
    QMember qMember = new QMember("m");  // 생성되는 JPQL의 별칭이 m
    List<Member> members = 
        query.from(qMember)
        .where(qMember.name.eq("회원1"))
        .orderBy(qMember.name.desc())
        .list(qMember);
}
```



**QueryDSL**을 사용하려면 우선 **com.mysema.query.jpa.impl.JPAQuery** 객체를 생성해야 하는데

이때 엔티티 매니저를 생성자에 넘겨준다.

다음으로 사용할 쿼리 타입(Q)을 생성하는데 생성자에는 별칭을 주면 된다.

이 별칭을 JPQL에서 별칭으로 사용한다.



그 다음에 나오는 from, where, orderBy, list는 코드만 보아도 쉽게 이해가 될 것이다.

다음 실행된 JPQL을 보면 둘이 얼마나 비슷한지 알 수 있다.

```jpql
select m from Member m
where m.name = ?1
order by m.name desc
```





#### 기본 Q 생성

쿼리 타입(Q)은 사용하기 편리하도록 아래의 예제와 같이 기본 인스턴스를 보관하고 있다.

하지만 같은 엔티티를 조인하거나 같은 엔티티를 서브쿼리에 사용하면 같은 별칭이 사용되므로 

이때는 별칭을 직접 지정해서 사용해야 한다.



**Member 쿼리 타입**

```java
public class QMember extends EntityPathBase<Member> {

	public static final QMember member = new QMember("member1");
    ...
}
```



쿼리 타입은 아래의 예제와 같이 사용한다.



**쿼리 타입 사용**

```java
QMember qMember = new QMember("m");   // 직접 지정
QMember qMember = QMember.member;     // 기본 인스턴스 사용
```





쿼리 타입의 기본 인스턴스를 사용하면 아래의 예제와 같이

**import static**을 활용해서 코드를 더 간결하게 작성할 수 있다.



**import static 활용**

```java
import static jpabook.jpashop.domain.QMember.member;   // 기본 인스턴스

public void basic() {
    
    EntityManager em = emf.createEntityManager();
    
    JPAQuery query = new JPAQuery(em);
    List<Member> members =
        query.from(member)
        	.where(member.name.eq("회원1"))
        	.orderBy(member.name.desc())
        	.list(member);
}
```





### 10.4.3 검색 조건 쿼리

QueryDSL의 기본 쿼리 기능을 아래의 예제를 통해 알아보자.

```java
JPAQuery query = new JPAQuery(em);
QItem item = QItem.item;
List<Item> list = query.from(item)
    .where(item.name.eq("좋은상품").and(item.price.gt(20000)))
    .list(item);  // 조회할 프로젝션 지정
```



위의 예제를 실행하면 아래의 JPQL이 생성되고, 실행된다.

```jpql
select item
from Item item
where item.name = ?1 and item.price > ?2
```



QueryDSL의 **where 절**에는 and나 or을 사용할 수 있다.

또한 다음처럼 여러 검색 조건을 사용해도 된다.

이때는 and 연산이 된다.

`.where(item.name.eq("좋은 상품"), item.price.gt(20000))`



쿼리 타입의 필드는 필요한 대부분의 메서드를 명시적으로 제공한다.

몇 가지만 예를 들어 보자.

다음은 **where()**에서 사용되는 메서드다.

```java
item.price.between(10000, 20000);  // 가격이 10000원 ~ 20000원
item.name.contains("상품1");       // 상품1이라는 이름을 포함한 상품,
                                  // SQL에서 like '%상품1%' 검색
item.name.startsWith("고급");      // 이름이 고급으로 시작하는 상품,
								  // SQL에서 liek '고급%' 검색
```



코드로 작성되어 있으므로 IDE가 제공하는 코드 자동 완성 기능의 도움을 받으면 필요한 메서드를 손쉽게 찾을 수 있다.





### 10.4.4 결과 조회

쿼리 작성이 끝나고 결과 조회 메서드를 호출하면 실제 DB를 조회한다.

보통 **uniqueResult()**나 **list()**를 사용하고 파라미터로 프로젝션 대상을 넘겨준다.

결과 조회 API는 **com.mysema.query.Projectable**에 정의되어 있다.



대표적인 결과 조회 메서드는 다음과 같다.

- **uniqueResult()**: 조회 결과가 한 건일 때 사용한다. 

  조회 결과가 없으면 null을 반환하고 

  결과가 하나 이상이면 com.mysema.query.NonUniqueResultException 예외가 발생한다.

- **singleResult()**: uniqueResult()와 같지만 결과가 하나 이상이면 처음 데이터를 반환한다.

- **list()**: 결과가 하나 이상일 때 사용한다. 결과가 없으면 빈 컬렉션을 반환한다.





### 10.4.5 페이징과 정렬

아래의 예제를 통해 페이징과 정렬을 알아보자.

```java
QItem item = QItem.item;

query.from(item)
	.where(item.price.gt(20000))
    .orderBy(item.price.desc(), item.stockQuantity.asc())
    .offset(10).limit(20)
    .list(item);
```



정렬은 **orderBy**를 사용하는데 쿼리 타입(Q)이 제공하는 **asc(), desc()**를 사용한다.

페이징은 **offset**과 **limit**을 적절히 조합해서 사용하면 된다.



페이징은 아래의 예제와 같이 **restrict()** 메서드에 **com.mysema.query.QueryModifiers**를 파라미터로 사용해도 된다.

```java
QueryModifiers queryModifiers = new QueryModifiers(20L, 10L); // limit, offset
List<Item> list =
    query.from(item)
    .restrict(queryModifiers)
    .list(item)
```



실제 페이징 처리를 하려면 검색된 전체 데이터 수를 알아야 한다.

이때는 **list()** 대신에 아래의 예제와 같이 **listResults()**를 사용한다.

```java
SearchResults<Item> result = 
	query.from(item)
		.where(item.price.gt(10000))
		.offset(10).limit(20)
		.listResults(item);

long total = result.getTotal();   // 검색된 전체 데이터 수
long limit = result.getLimit();   
long offset = result.getOffset();
List<Item> results = result.getResults();  // 조회된 데이터
```



**listResults()**를 사용하면 전체 데이터 조회를 위한 **count** 쿼리를 한 번 더 실행한다.

그리고 **SearchResults**를 반환하는데 이 객체에서 전체 데이터 수를 조회할 수 있다.





### 10.4.6 그룹

그룹은 아래의 예제와 같이 **groupBy**를 사용하고 

그룹화된 결과를 제한하려면 **having**을 사용하면 된다.

```java
query.from(item)
    .groupBy(item.price)
    .having(item.price.gt(1000))
    .list(item)
```





### 10.4.7 조인

조인은 **innerJoin(join), leftJoin, rightJoin, fullJoin**을 사용할 수 있고

추가로 JPQL의 **on**과 성능 최적화를 위한 **fetch** 조인도 사용할 수 있다.



조인의 기본 문법은 첫 번째 파라미터에 조인 대상을 지정하고,

두 번째 파라미터에 별칭(alias)으로 사용할 쿼리 타입을 지정하면 된다.

`join(조인 대상, 별칭으로 사용할 쿼리 타입)`



아래의 예제는 가장 기본적인 조인 방법이다.

```java
QOrder order = QOrder.order;
QMember member = QMember.member;
QOrderItem orderItem = QOrderItem.orderItem;

query.from(order)
    .join(order.member, member)
    .leftJoin(order.orderItems, orderItem)
    .list(order);
```



아래의 예제는 조인에 **on**을 사용햇다.

```java
query.from(order)
    .leftJoin(order.orderItems, orderItem)
    .on(orderItem.count.gt(2))
    .list(order);
```





아래의 예제는 페치 조인을 사용하는 방법이다.

```java
query.from(order)
    .innerJoin(order.member, member).fetch()
    .leftJoin(order.orderItems, orderItem).fetch()
    .list(order);
```





아래의 예제는 **from** 절에 여러 조인을 사용하는 세타 조인 방법이다.

```java
QOrder order = QOrder.order;
QMember member = QMember.member;

query.from(order, member)
    .where(order.member.eq(member))
    .list(order);
```





### 10.4.8 서브 쿼리

서브 쿼리는아래의 예제와 같이 **com.mysema.query.jpa.JPASubQuery**를 생성해서 사용한다.

서브 쿼리의 결과가 하나면 **unique()**, 여러 건이면 **list()**를 사용할 수 있다.

```java
QItem item = QItem.item;
QItem itemSub = new QItem("itemSub");

query.from(item)
	.where(item.price.eq(
		new JPASubQuery().from(itemSub).unique(itemSub.price.max())
	))
	.list(item);
```



아래의 예제는 여러 건의 서브 쿼리를 사용하는 방법이다.

```java
QItem item = QItem.item;
QItem itemSub = new QItem("itemSub");

query.from(item)
    .where(item.in(
    	new JPASubQuery().from(itemSub)
        	.where(item.name.eq(itemSub.name))
        	.list(itemSub)
    ))
    .list(item);
```





### 10.4.9 프로젝션과 결과 반환

**select** 절에 조회 대상을 지정하는 것을 프로젝션이라 한다.



#### 프로젝션 대상이 하나

프로젝션 대상이 하나면 아래의 예제와 같이 해당 타입으로 반환한다.

```java
QItem item = QItem.item;
List<String> result = query.from(item).list(item.name);

for(String name: result) {
    System.out.println("name = " + name);
}
```





#### 여러 컬럼 반환과 튜플

프로젝션 대상으로 여러 필드를 선택하면

QueryDSL은 기본으로 **com.mysema.query.Tuple**이라는 **Map**과 비슷한 내부 타입을 사용한다.

조회 결과는 **tuple.get()** 메서드에 조회한 쿼리 타입을 지정하면 된다.



아래의 예제를 통해 튜플 사용 방법을 알아보자.

```java
QItem item = QItem.item;

List<Tuple> result = query.from(item).list(item.name, item.price);
// List<Tuple> result = query.from(item).list(new QTuple(item.name, item.price));
// 같다.

for(Tuple tuple: result) {
    System.out.println("name = " + tuple.get(item.name));
    System.out.println("price = " + tuple.get(item.price));
}
```





#### 빈 생성

쿼리 결과를 엔티티가 아닌 특정 객체로 받고 싶으면

**빈 생성 (Bean Population)** 기능을 사용한다.

QueryDSL은 객체를 생성하는 다양한 방법을 제공한다.

- 프로퍼티 접근
- 필드 직접 접근
- 생성자 사용



원하는 방법을 지정하기 위해 **com.mysema.query.type.Projections**를 사용하면 된다.

다양한 방법으로 아래 예제의 **ItemDTO**에 값을 채워보자.

```java
public class ItemDTO {
    
    private String username;
    private int price;
    
    public ItemDTO() { }
    
    public ItemDTO(String username, int price) {
        this.username = username;
        this.price = price;
    }
    
    // Getter, Setter
}
```



**프로퍼티 접근(Setter)**

```java
QItem item = QItem.item;
List<ItemDTO> result = query.from(item).list(
	Projections.bean(ItemDTO.class, item.name.as("username"), item.price));
```



위의 예제의 **Projections.bean()** 메서드는 수정자(Setter)를 사용해서 값을 채운다.

예제를 보면 쿼리 결과는 **name**인데 **ItemDTO**는 **username** 프로퍼티를 가지고 있다.

이처럼 쿼리 결과와 매핑할 프로퍼티 이름이 다르면 **as**를 사용해서 별칭을 주면 된다.





**필드 직접 접근**

```java
QItem item = QItem.item;
List<ItemDTO> result = query.from(item).list(
	Projections.fields(ItemDTO.class, item.name.as("username"),
                      item.price));
```



위의 예제의 **Projections.fields()** 메서드를 사용하면 필드에 직접 접근해서 값을 채워준다.

필드를 **private**으로 설정해도 동작한다.





**생성자 사용**

```java
QItem item = QItem.item;
List<ItemDTO> result = query.from(item).list(
	Projections.constructor(ItemDTO.class, item.name, item.price));
```



위 예제의 **Projections.constructor()** 메서드는 생성자를 사용한다.

물론 지정한 프로젝션과 파라미터 순서가 같은 생성자가 필요하다.





#### DISTINCT

**distinct**는 다음과 같이 사용한다.

`query.distinct().from(item)...`





### 10.4.10 수정, 삭제 배치 쿼리

QueryDSL도 수정, 삭제 같은 배치 쿼리를 지원한다.

JPQL 배치 쿼리와 같이 영속성 컨텍스트를 무시하고 DB를 직접 쿼리한다는 점에 유의하자.

JPQL 배치 쿼리는 10.6.1절에서 다룬다.



**수정 배치 쿼리**

```java
QItem item = QItem.item;
JPAUpdateClause updateClause = new JPAUpdateClause(em, item);
long count = updateClause.where(item.name.eq("시골개발자의 JPA 책"))
	.set(item.price, item.price.add(100))
	.execute();
```



위의 예제와 같이 **수정 배치 쿼리**는 **com.mysema.query.jpa.impl.JPAUpdateClause**를 사용한다.

위의 예제는 상품의 가격을 100원 증가시킨다.





**삭제 배치 쿼리**

```java
QItem item = QItem.item;
JPADeleteClause deleteClause = new JPADeleteClause(em, item);
long count = deleteClause.where(item.name.eq("시골개발자의 JPA 책"))
    .execute();
```



위의 예제와 같이 **삭제 배치 쿼리**는 **com.mysema.query.jpa.impl.JPADeleteClause**를 사용한다.

예제는 이름이 같은 상품을 삭제한다.







### 10.4.11 동적 쿼리

**com.mysema.query.BooleanBuilder**를 사용하면 

특정 조건에 따른 동적 쿼리를 편리하게 생성할 수 있다.



**동적 쿼리 예제**

```java
SearchParam param = new SearchParam();
param.setName("시골개발자");
param.setPrice(10000);

QItem item = QItem.item;

BooleanBuilder builder = new BooleanBuilder();
if(StringUtils.hasText(param.getName())) {
    builder.and(item.name.contains(param.getName()));
}

if(param.getPrice() != null) {
    builder.and(item.price.gt(param.getPrice()));
}

List<Item> result = query.from(item)
    .where(builder)
    .list(item);
```



위의 예제는 상품 이름과 가격 유무에 따라 동적으로 쿼리를 생성한다.





### 10.4.12 메서드 위임

**메서드 위임 (Delegate Methods)** 기능을 사용하면

쿼리 타입에 검색 조건을 직접 정의할 수 있다.



**검색 조건 정의**

```java
public class ItemExpression {
    
    @QueryDelegate(Item.class)
    public static BooleanExpression isExpensive(QItem item, Integer price) {
        return item.price.gt(price);
    }
}
```



메서드 위임 기능을 사용하려면 우선 위의 예제와 같이 

**정적 static** 메서드를 만들고 **@com.mysema.query.annotations.QueryDelegate** 어노테이션에 

속성으로 이 기능을 적용할 엔티티를 지정한다.

정적 메서드의 첫 번째 파라미터에는 대상 엔티티의 쿼리 타입(Q)을 지정하고

나머지는 필요한 파라미터를 정의한다.





**쿼리 타입에 생성된 결과**

```java
public class QItem extends EntityPathBase<Item> {
	...
    public com.mysema.query.types.expr.BooleanExpression
        isExpensive(Integer price) {
        return ItemExpression.isExpensive(this, price);
    }
}
```



위의 예제에 생성된 쿼리 타입을 보면 기능이 추가된 것을 확인할 수 있다.

이제 메서드 위임 기능을 사용해보자.

다음 코드를 보자.

`query.from(item).where(item.isExpensive(30000)).list(item);`



필요하다면 **String, Date** 같은 자바 기본 내장 타입에도 메서드 위임 기능을 사용할 수 있다.

```java
@QueryDelegate(String.class)
public static BooleanExpression isHelloStart(StringPath stringPath) {
	return stringPath.startsWith("Hello");
}
```







## 10.5 네이티브 SQL

JPQL은 표준 SQL이 지원하는 대부분의 문법과 SQL 함수들을 지원하지만

특정 DB에 종속적인 기능은 지원하지 않는다.

예를 들어 다음과 같은 것들이다.

- 특정 DB만 지원하는 함수, 문법, SQL 쿼리 힌트
- 인라인 뷰(From 절에서 사용하는 서브쿼리), UNION, INTERSECT
- 스토어드 프로시저



때로는 특정 DB에 종속적인 기능이 필요하다.

JPA는 특정 DB에 종속적인 기능을 사용할 수 있는 다양한 방법을 열어두었다.

그리고 JPA 구현체들은 JPA 표준보다 더 다양한 방법을 지원한다.



특정 DB에 종속적인 기능을 지원하는 방법은 다음과 같다.

- **특정 DB만 사용하는 함수**

  - JPQL에서 네이티브 SQL 함수를 호출할 수 있다(JPA 2.1).
  - Hibernate는 DB 방언에 각 DB에 종속적인 함수들을 정의해두었다. 또한 직접 호출할 함수를 정의할 수도 있다.

- **특정 DB만 지원하는 SQL 쿼리 힌트**

  - Hibernate를 포함한 몇몇 JPA 구현체들이 지원한다.

- **인라인 뷰(From 절에서 사용하는 서브쿼리), UNION, INTERSECT**

  - Hibernate는 지원하지 않지만 일부 JPA 구현체들이 지원한다.

- **스토어 프로시저**

  - JPQL에서 스토어드 프로시저를 호출할 수 있다(JPA 2.1).

- **특정 DB만 지원하는 문법**

  - 오라클의 CONNECT BY처럼 특정 DB에 너무 종속적인 SQL 문법은 지원하지는 않는다.

    이때는 네이티브 SQL을 사용해야 한다.



다양한 이유로 JPQL을 사용할 수 없을 때

JPA는 SQL을 직접 사용할 수 있는 기능을 제공하는데 이것을 **네이티브 SQL**이라 한다.

JPQL을 사용하면 JPA가 SQL을 생성한다.

**네이티브 SQL**은 이 SQL을 개발자가 직접 정의하는 것이다.



그럼 JPA가 지원하는 **네이티브 SQL**과 **JDBC API**를 직접 사용하는 것에는 어떤 차이가 있을까?

**네이티브 SQL을 사용하면 엔티티를 조회할 수 있고, JPA가 지원하는 영속성 컨텍스트의 기능을 그대로 사용할 수 있다.**

반면에 JDBC API를 직접 사용하면 단순히 데이터의 나열을 조회할 뿐이다.





### 10.5.1 네이티브 SQL 사용

**네이티브 쿼리 API**는 다음 3가지가 있다.

```java
// 결과 타입 정의
public Query createNativeQuery(String sqlString, Class resultClass);

// 결과 타입을 정의할 수 없을 때
public Query createNativeQuery(String sqlString);

public Query createNativeQuery(String sqlString, String resultSetMapping); // 결과 매핑 사용
```



우선 엔티티를 조회하는 것부터 보자.



#### 엔티티 조회

네이티브 SQL은 아래의 예제와 같이 **em.createNativeQuery(SQL, 결과 클래스)**를 사용한다.

첫 번째 파라미터는 네이티브 SQL을 입력하고

두 번째 파라미터는 조회할 엔티티 클래스의 타입을 입력한다.

JPQL를 사용할 때와 거의 비슷하지만 실제 DB SQL을 사용한다는 것과 

위치기반 파라미터만 지원하다는 차이가 있다.



**엔티티 조회 코드**

```java
// SQL 정의
String sql = "SELECT ID, AGE, NAME, TEAM_ID FROM MEMBER WHERE AGE > ?";
Query nativeQuery = em.createNativeQuery(sql, Member.class)
    .setParameter(1, 20);

List<Member> resultList = nativeQuery.getResultList();
```



**여기서 가장 중요한 점은 네이티브 SQL로 SQL만 직접 사용할 뿐이지 나머지는 JPQL을 사용할 때와 같다.**

**조회한 엔티티도 영속성 컨텍스트에서 관리된다.**



> **[참고]**
>
> JPA는 공식적으로 네이티브 SQL에서 이름 기반 파라미터를 지원하지 않고 위치 기반 파라미터만 지원한다.
>
> 하지만 하이버네이트는 네이티브 SQL에 이름 기반 파라미터를 사용할 수 있다.
>
> 따라서 하이버네이트 구현체를 사용한다면 예제를 이름 기반 파라미터로 변경해도 동작한다.



> **[참고]**
>
> em.createNativeQuery()를 호출하면서 타입 정보를 주었는데도 TypeQuery가 아닌 Query가 리턴되는데
>
> 이것은 JPA 1.0에서 API 규약이 정의되어 버려서 그렇다. 너무 신경 쓰지 않아도 된다.





#### 값 조회

이번에는 단순히 값으로만 조회하는 방법을 알아보자.

```java
// SQL 정의
String sql = "SELECT ID, AGE, NAME, TEAM_ID FROM MEMBER WHERE AGE > ?";

Query nativeQuery = em.createNativeQuery(sql)
    .setParameter(1, 10);

List<Object[]> resultList = nativeQuery.getResultList();
for(Object[] row: resultList) {
    System.out.println("id = " + row[0]);
    System.out.println("age = " + row[1]);
    System.out.println("name = " + row[2]);
    System.out.println("team_id = " + row[3]);
}
```



위의 예제는 엔티티로 조회하지 않고 단순히 값으로 조회햇다.

이렇게 여러 값으로 조회하려면 **em.createNativeQuery(SQL)**의 두 번째 파라미터를 사용하지 않으면 된다.

JPA는 조회한 값들을 **Object[]**에 담아서 반환한다.

여기서는 스칼라 값들을 조회했을 뿐이므로 결과를 영속성 컨텍스트가 관리하지 않는다.

마치 JDBC로 데이터를 조회한 것과 비슷하다.





#### 결과 매핑 사용

지금까지 특정 엔티티를 조회하거나 스칼라 값들을 나열해서 조회하는 단순한 조회 방법을 설명했다.

엔티티와 스칼라 값을 함께 조회하는 것처럼 매핑이 복잡해지면

**@SqlResultSetMapping**을 정의해서 결과 매핑을 사용해야 한다.



이번에는 아래의 예제를 통해 회원 엔티티와 회원이 주문한 상품 수를 조회해 보자.

```java
// SQL 정의
String sql = 
    "SELECT M.ID, AGE, NAME, TEAM_ID, I.ORDER_COUNT " +
    "FROM MEMBER M " +
    "LEFT JOIN " +
    "	(SELECT IM.ID, COUNT(*) AS ORDER_COUNT " +
    "	FROM ORDERS O, MEMBER IM " + 
    "	WHERE O.MEMBER_ID = IM.ID) I " +
    "ON M.ID = I.ID";

Query nativeQuery = em.createNativeQuery(sql, "memberWithOrderCount");

List<Object[]> resultList = nativeQuery.getResultList();
for(Object[] row: resultList) {
    Member member = (Member) row[0];
    BigInteger orderCount = (BigInteger)row[1];
    System.out.println("member = " + member);
    System.out.println("orderCount = " + orderCount);
}

```



**em.createNativeQuery(sql, "memberWithOrderCount")**의 두 번째 파라미터에 결과 매핑 정보의 이름이 사용되었다.

아래의 예제를 통해 결과 매핑을 정의하는 코드를 보자.

```java
@Entity
@SqlResultSetMapping(name = "memberWithOrderCount",
	entities = {@EntityResult(entityClass = Member.class)},
	columns = {@ColumnResult(name = "ORDER_COUNT")}
)
public class Member {...}
```



위의 예제에 있는 **memberWithOrderCount**의 결과 매핑을 잘 보면

**회원 엔티티**와 `ORDER_COUNT` 컬럼을 매핑했다.



쿼리 결과에서 **ID, AGE, NAME, TEAM_ID**는 **Member** 엔티티와 매핑하고

`ORDER_COUNT`는 단순히 값으로 매핑한다.

그리고 **entities, columns**라는 이름에서 알 수 있듯이 여러 엔티티와 여러 컬럼을 매핑할 수 있다.





이번에는 JPA 표준 명세에 있는 예제 코드로 결과 매핑을 어떻게 하는지 좀 더 자세히 알아보자.

아래의 예제는 네이티브 SQL을 사용한다.



**표준 명세 예제 - SQL**

```java
Query q = em.createNativeQuery(
	"SELECT o.id       AS order_id, " +
           "o.quantity AS order_quantity, "+
           "o.item     AS order_item, " +
           "i.name     AS item_name " +
    "FROM Order o, Item i " +
    "WHERE (order_quantity > 25) AND (order_item = i.id)", "OrderResults");
)
```



**표준 명세 예제 - 매핑 정보**

```java
@SqlResultSetMapping(name="OrderResults", 
	entities={
        @EntityResult(entityClass=com.acme.Order.class, fields={
            @FieldResult(name="id", column="order_id"),
            @FieldResult(name="quantity", column="order_quantity"),
            @FieldResult(name="item", column="order_item")
        })
    },
    columns={
        @ColumnResult(name="item_name")
    }
)
```



위의 코드를 보면 **@FieldResult**를 사용해서 컬럼명과 필드명을 직접 매핑한다.

이 설정은 엔티티의 필드에 정의한 **@Column**보다 앞선다.

조금 불편한 것은 **@FieldResult**를 한 번이라도 사용하면 전체 필드를 **@FieldResult**로 매핑해야 한다.



다음처럼 두 엔티티를 조회하는데 컬럼명이 중복될 때도 **@FieldResult**를 사용해야 한다.

`SELECT A.ID, B.ID FROM A, B`



A, B 둘다 ID라는 필드를 가지고 있으므로 컬럼명이 충돌한다.

따라서 다음과 같이 별칭을 적절히 사용하고 **@FieldResult**로 매핑하면 된다.

```SQL
SELECT
	A.ID AS A_ID,
	B.ID AS B_ID
FROM A, B
```





#### 결과 매핑 어노테이션

결과 매핑에 사용하는 어노테이션을 다음 표에 정리했다

- @SqlResultSetMapping 속성
- @EntityResult 속성
- @FieldResult 속성
- @ColumnResult 속성





**@SqlResultSetMapping 속성**

| 속성     | 기능                                               |
| -------- | -------------------------------------------------- |
| name     | 결과 매핑 이름                                     |
| entities | @EntityResult를 사용해서 엔티티를 결과로 매핑한다. |
| columns  | @ColumnResult를 사용해서 컬럼을 결과로 매핑한다.   |



**@EntityResult 속성**

| 속성                | 기능                                                    |
| ------------------- | ------------------------------------------------------- |
| entityClass         | 결과로 사용할 엔티티 클래스를 지정한다.                 |
| fields              | @FieldResult를 사용해서 결과 컬럼을 필드와 매핑한다.    |
| discriminatorColumn | 엔티티의 인스턴스 타입을 구분하는 필드(상속에서 사용됨) |



**@FieldResult 속성**

| 속성   | 기능               |
| ------ | ------------------ |
| name   | 결과를 받을 필드명 |
| column | 결과 컬럼명        |



**@ColumnResult 속성**

| 속성 | 기능        |
| ---- | ----------- |
| name | 결과 컬럼명 |









### 10.5.2 Named 네이티브 SQL

JPQL처럼 네이티브 SQL도 Named 네이티브 SQL을 사용해서 정적 SQL을 작성할 수 있다.

아래의 예제와 같이 엔티티를 조회해보자.



**엔티티 조회**

```java
@Entity
@NamedNativeQuery(
    name = "Member.memberSQL",
    query = "SELECT ID, AGE, NAME, TEAM_ID FROM MEMBER WHERE AGE > ?",
    resultClass = Member.class
)
public class Member {...}
```



**@NamedNativeQuery**로 Named 네이티브 SQL을 등록했다.

다음으로 사용하는 예제를 보자.

```java
TypedQuery<Member> nativeQuery =
	em.createNamedQuery("Member.memberSQL", Member.class)
		.setParameter(1, 20);
```



흥미로운 점은 JPQL Named 쿼리와 같은 **createNamedQuery** 메서드를 사용한다는 것이다.

따라서 **TypeQuery**를 사용할 수 있다.



다음으로 아래의 예제와 같이 Named 네이티브 SQL에서 결과 매핑을 사용해 보자.

```java
@Entity
@SqlResultSetMapping( name = "memberWithOrderCount",
	entities = {@EntityResult(entityClass = Member.class)},
    columns = {@ColumnResult(name = "ORDER_COUNT")}
)
@NamedNativeQuery(
	name = "Member.memberWithOrderCount",
    query = "SELECT M.ID, AGE, NAME, TEAM_ID, I.ORDER_COUNT " +
            "FROM MEMBER M " +
            "LEFT JOIN " +
            "     (SELECT IM.ID, COUNT(*) AS ORDER_COUNT " +
            "      FROM ORDERS O, MEMBER IM " +
            "      WHERE O.MEMBER_ID = IM.ID) I " +
            "ON M.ID = I.ID",
    resultSetMapping = "memberWithOrderCount"
)
public class Member { ... }
```



Named 네이티브 쿼리에서 **resultSetMapping = "memberWithOrderCount"**로 조회 결과를 매핑할 대상까지 지정했다.

다음은 위에서 정의한 Named 네이티브 쿼리를 사용하는 코드다.

```java
List<Object[]> resultList = 
    em.createNamedQuery("Member.memberWithOrderCount")
    	.getResultList();
```





#### @NamedNativeQuery

**@NamedNativeQuery**의 속성을 알아보자.



**@NamedNativeQuery 속성**

| 속성             | 기능                   |
| ---------------- | ---------------------- |
| name             | 네임드 쿼리 이름(필수) |
| query            | SQL 쿼리(필수)         |
| hints            | 벤더 종속적인 힌트     |
| resultClass      | 결과 클래스            |
| resultSetMapping | 결과 매핑 사용         |



각 기능은 이미 설명했으므로 표로 이해가 될 것이다.

여기서 **hints** 속성이 있는데 이것은 SQL 힌트가 아니라 Hiberante 같은 JPA 구현체에 제공하는 힌트다.



여러 Named 네이티브 쿼리를 선언하려면 다음처럼 사용하면 된다.

```java
@NameNativeQueries({
    @NamedNativeQuery(...),
    @NamedNativeQuery(...)
})
```







### 10.5.3 네이티브 SQL XML에 정의

Named 네이티브 쿼리를 아래의 예제와 같이 XML에 정의해보자.

```xml
<entity-mappings ...>

	<named-native-query name="Member.memberWithOrderCountXml"
                        result-set-mapping="memberWithOrderCountResultMap" >
    	<query><CDATA[
            SELECT M.ID, AGE, NAME, TEAM_ID, I.ORDER_COUNT
            FROM MEMBER M
            LEFT JOIN 
                (SELECT IM.ID, COUNT(*) AS ORDER_COUNT
                 FROM ORDERS O, MEMBER IM
                 WHERE O.MEMBER_ID = IM.ID) I
            ON M.ID = I.ID
        ]></query>
    </named-native-query>
    
    <sql-result-set-mapping name="memberWithOrderCountResultMap">
        <entity-result entity-class="jpabook.domain.Member" />
        <column-result name="ORDER_COUNT" />
    </sql-result-set-mapping>
        
</entity-mappings>
```



XML에 정의할 때는 순서를 지켜야 하는데

**\<named-native-query>**를 먼저 정의하고 **\<sql-result-set-mapping>**를 정의해야 한다.



XML과 어노테이션 둘 다 사용하는 코드는 다음과 같다.

```java
List<Object[]> resultList =
	em.createNamedQuery("Member.memberWithOrderCount")
		.getResultList();
```





### 10.5.4 네이티브 SQL 정리

네이티브 SQL도 JPQL을 사용할 때와 마찬가지로 Query, TypeQuery(Named 네이티브 쿼리의 경우에만)를 반환한다.

따라서 JPQL API를 그대로 사용할 수 있다.

예를 들어 네이티브 SQL을 사용해도 아래의 예제와 같이 페이징 처리 API를 적용할 수 있다.

```java
String sql = "SELECT ID, AGE, NAME, TEAM_ID FROM MEMBER";
Query nativeQuery = em.createNativeQuery(sql, Member.class)
	.setFirstResult(10)
    .setMaxResults(20)
```



DB 방언에 따라 결과는 다르겠지만,

다음처럼 페이징 정보를 추가한 SQL을 실행한다.

```sql
SELECT ID, AGE, NAME, TEAM_ID
FROM MEMBER
limit ? offset ?   // 페이징 정보 추가
```



네이티브 SQL은 JPQL이 자동 생성하는 SQL을 수동으로 직접 정의하는 것이다.

따라서 JPA가 제공하는 기능 대부분을 그대로 사용할 수 있다.



네이티브 SQL은 관리하기 쉽지 않고 자주 사용하면 특정 DB에 종속적인 쿼리가 증가해서 이식성이 떨어진다.

그렇다고 현실적으로 네이티브 SQL을 사용하지 않을 수는 없다.

될 수 있으면 표준 JPQL을 사용하고 기능이 부족하면 차선책으로 Hibernate 같은 JPA 구현체가 제공하는 기능을 사용하자.

그래도 안되면 마지막 방법으로 네이티브 SQL을 사용하자.

그리고 네이티브 SQL로도 부족함을 느낀다면 MyBatis나 스프링 프레임워크가 제공하는 JdbcTemplate 같은 SQL 매퍼와

JPA를 함께 사용하는 것도 고려할만하다.





### 10.5.5 스토어드 프로시저 (JPA 2.1)

JPA는 2.1부터 스토어드 프로시저를 지원한다.





#### 스토어드 프로시저 사용

아래의 예제와 같이 단순히 입력 값을 두 배로 증가시켜 주는 **proc_multiply**라는 스토어드 프로시저가 있다.

이 프로시저는 첫 번째 파라미터로 값을 입력받고 두 번째 파라미터로 결과를 반환한다.



**proc_multiply MySQL 프로시저**

```SQL
DELIMITER //

CREATE PROCEDURE proc_multiply (INOUT inParam INT, INOUT outParam INT)
BEGIN
	SET outParam = inParam * 2;
END //
```



JPA로 이 스토어드 프로시저를 호출해보자.

먼저 아래의 예제의 순서 기반 파라미터 호출 코드를 보자.



**순서 기반 파라미터 호출**

```java
StoredProcedureQuery spq =
    em.createStoredProcedureQuery("proc_multiply");

spq.registerStoredProcedureParamter(1, Integer.class, ParameterMode.IN);
spq.registerStoredProcedureParamter(2, Integer.class, ParameterMode.OUT);

spq.setParamter(1, 100);
spq.execute();

Integer out = (Integer) spq.getOutputParameterValue(2);
System.out.println("out = " + out);  // 결과 200
```



스토어드 프로시저를 사용하려면

**em.createStoredProcedureQuery()** 메서드에 사용할 스토어드 프로시저 이름을 입력하면 된다.



그리고 **registerStoredProcedureParameter()** 메서드를 사용해서 

프로시저에서 사용할 파라미터를 순서, 타입, 파라미터 모드 순으로 정의하면 된다.

사용할 수 있는 **ParameterMode**는 아래와 같다.

```JAVA
public enum ParameterMode {
	IN,             // INPUT 파라미터
	INOUT,          // INPUT, OUTPUT 파라미터
	OUT,            // OUTPUT 파라미터
	REF_CURSOR      // CURSOR 파라미터
}
```





아래의 예제와 같이 파라미터에 순서 대신에 이름을 사용할 수 있다.

```java
StoredProcedureQuery spq =
	em.createStoredProcedureQuery("proc_multiply");

spq.registerStoredProcedureParameter("inParam", Integer.class, ParameterMode.IN);
spq.registerStoredProcedureParameter("outParam", Integer.class, ParameterMode.OUT);

spq.setParameter("inParam", 100);
spq.execute();

Integer out = (Integer)spq.getOutputParameterValue("outParam");
System.out.println("out = " + out);  // 결과 = 200
```





#### Named 스토어드 프로시저 사용

스토어드 프로시저 쿼리에 아래의 예제와 같이 이름을 부여해서 사용하는 것을 

**Named 스토어드 프로시저**라 한다.



**Named 스토어드 프로시저 어노테이션에 정의하기**

```java
@NamedStoredProcedureQuery(
	name = "multiply",
    procedureName = "proc_multiply",
    parameters = {
        @StoredProcedureParameter(name = "inParam", mode = ParameterMode.IN, type = Integer.class),
        @StoredProcedureParameter(name = "outParam", mode = ParameterMode.OUT, type = Integer.class)
    }
)
@Entity
public class Member { ... }
```



**@NamedStoredProcedureQuery**로 정의하고

**name** 속성으로 이름을 부여하면 된다.

**procedureName** 속성에 실제 호출할 프로시저 이름을 적어주고 

**@StoredProcedureParameter**를 사용해서 파라미터 정보를 정의하면 된다.

참고로 둘 이상을 정의하려면 **@NamedStoredProcedureQueries**를 사용하면 된다.





























