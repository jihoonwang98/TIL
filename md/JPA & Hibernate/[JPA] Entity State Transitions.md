# [JPA] Entity State Transitions



> https://vladmihalcea.com/a-beginners-guide-to-jpa-hibernate-entity-state-transitions/



## 1. Introduction

Hibernate는 개발자의 사고를 SQL식 사고에서 Entity State Transitions식 사고로 전환할 것을 요구한다.

한번 엔티티가 Hibernate에 의해 관리되기 시작하면, 엔티티에 대한 모든 변화들은 자동으로 DB로 전파될 것이다.

도메인 모델 엔티티들을 조작하는 것은 SQL 문을 작성하는 것보다 쉬워질 것이다.



하지만 Hibernate도 만능은 아니다. 그러므로 실제 실행되는 SQL 문을 항상 염두에 두고 생각하며 코드를 짜야 한다.





## 2. Entity States

Entity State의 종류는 아래와 같다.



- **New (Transient(일시적인&middot;순간의))**
  - 객체가 새롭게 생성된 후 Hibernate Session(Persistence Context)와 아무 관련이 없는 상태.
  - DB 테이블의 어떤 row와도 매핑되지 않은 상태.
  - 이 상태의 엔티티를 저장하려면 `EntityManager`의 `persist` 메서드를 호출하거나 transitive persistence 메커니즘을 이용해야 한다.



- **Persistent (Managed)**
  - persistent 엔티티는 DB 테이블의 row와 매핑되어 있는 상태.
  - persistent 상태의 엔티티는 Persistence Context에 의해 관리된다.
  - 이 상태에 있는 엔티티에 가해지는 모든 변화가 감지되며 이러한 변화는 Session flush-time에 모두 DB에 전파되어 반영된다.



- **Detached**

  - 현재 동작하고 있는 Persistence Context가 닫히면 managed(persistent) 상태에 있던 모든 엔티티들이 detached 엔티티가 된다.

  - 이 상태가 되면 모든 변화들은 더 이상 감지되지 않고 DB 자동 synchronization도 일어나지 않는다.

  - detached 엔티티를 active Hibernate Session가 다시 관리하게 하고 싶으면, 아래와 같은 선택지 중 하나를 선택하자.

    - **Reattaching**:

      Hibernate(하지만 JPA 2.1버전은 안댐)는 `Session#update` 메서드를 통해 reattaching을 지원한다.

      Hibernate Session은 주어진 DB 행 한 개에 대해 오직 한 개의 엔티티만을 연관시킬 수 있다.

      그래서 현재 Hibernate Session과 연관된(같은 DB 행에 매핑된) 다른 JVM 객체가 없는 경우에만 reattaching이 가능하다. 

      

    - **Merging**:

       [merge 연산](https://vladmihalcea.com/jpa-persist-and-merge/)은 detached된 엔티티를 managed 엔티티로 <u>복사</u>하는 것이다.

      만약 merge되는 엔티티와 동일한 엔티티가 현재 Session에 없다면, DB에서 하나 가져온다.

      merge는 복사를 하는 것이기 때문에 detached된 엔티티는 merge 후에도 계속 detached된 상태를 유지한다.



- **Removed**

  - removed 상태의 엔티티는 Session flush-time에 실제 DB로 DELETE 쿼리가 나간다.

  - 참고로, JPA는 managed 상태에 있는 엔티티들만 삭제되는 것을 허용하지만, 

    Hibernate는 detached 엔티티들을 삭제하는 것도 가능하다(`Session#delete` 메서드).

    

![](https://vladmihalcea.com/wp-content/uploads/2014/07/jpaentitystates.png)

