# [JPA] 즉시 로딩&middot;지연 로딩 예시 정리







## 1. 다대일 



### 도메인

Member:Team  N:1 

양방향 연관관계



**Member**

```java
@Entity
@Getter @Setter
@ToString(of={"id", "username"})
public class Member {

    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "member_id")
    private Long id;

    private String username;

    @ManyToOne
    @JoinColumn(name = "team_id")
    private Team team;
}
```



**Team**

```java
@Entity
@Getter @Setter
public class Team {

    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "team_id")
    private Long id;

    private String name;

    @OneToMany(mappedBy = "team")
    private List<Member> members;
}
```



**MemberRepository**

```java
public interface MemberRepository extends JpaRepository<Member, Long> {
}
```

**TeamRepository**

```java
public interface TeamRepository extends JpaRepository<Team, Long> {
}
```



![테이블](https://lh4.googleusercontent.com/p82CHicO6hR0-abSjhtaXtykWSFH0PAu6S5PIGAP3O6A9sZazYOfTx0joBF93FCY5MUKuttfG-gerYkLoNH5RNbs46zipQOq3HGKi2HnbivenXJC8dvKNvFk6-GJ3OoBQxxw6uiI)





### 1번 즉시로딩으로 멤버 하나 가져오기 (다대일에서 다 하나 가져오기)

```java
public class Member {
    ...
         
    @ManyToOne   // 즉시 로딩으로 설정되어 있음.
    @JoinColumn(name = "team_id")
    private Team team;
}
```



**멤버 하나 가져오기**

```java
@Test
public void test() {
    Member member = memberRepository.findById(1L).get();
}
```



**쿼리**

```sql
    select
        member0_.member_id as member_i1_0_0_,
        member0_.team_id as team_id3_0_0_,
        member0_.username as username2_0_0_,
        team1_.team_id as team_id1_4_1_,
        team1_.name as name2_4_1_ 
    from
        member member0_ 
    left outer join
        team team1_ 
            on member0_.team_id=team1_.team_id 
    where
        member0_.member_id=1
```

![](https://lh5.googleusercontent.com/pLjBiOevYBz1goEfihlDBj8xH3Q-AYf_170lmHIyPEiBXiutjKgjiHKGq7jLldSTAlH3tFKIWc43aFZypiT0Ipk3PHB4ijkTR7P4iXZYZFoLq1HbOyJ_l7R-RQ8tRGMS3dmkbsoo)



- 대부분의 구현체가 즉시 로딩을 최적화하기 위해 가능하면 **조인 쿼리**를 사용한다.

- Member 입장에서 Team이 null인 경우도 있을 수 있기 때문에 기본적으로 LEFT OUTER JOIN을 사용한다.

  - ```java
    @ManyToOne(optional = false)
    @JoinColumn(name = "team_id")
    private Team team;
    ```

    위처럼 `optional = false`로 설정하면 해당 team 객체에 null이 들어갈 수 없다.

    ```sql
        select
            member0_.member_id as member_i1_0_0_,
            member0_.team_id as team_id3_0_0_,
            member0_.username as username2_0_0_,
            team1_.team_id as team_id1_4_1_,
            team1_.name as name2_4_1_ 
        from
            member member0_ 
        inner join
            team team1_ 
                on member0_.team_id=team1_.team_id 
        where
            member0_.member_id=1
    ```

    따라서 inner join 쿼리가 나간다.

    

  - ```java
    @ManyToOne
    @JoinColumn(name = "team_id", nullable = false)
    private Team team;
    ```

    같은 원리로 Member 테이블의 team_id 컬럼이 null이 될 수 없으므로 얘도 inner join 쿼리가 나간다.





### 2번 즉시로딩으로 멤버 리스트 가져오기 (다대일에서 다 여러개 가져오기)

**member 전부 다 가져오기**

```java
@Test
public void test() {
    List<Member> all = memberRepository.findAll();
}
```



**쿼리**

```sql
    -- member 일단 다 가져오고
    select
        member0_.member_id as member_i1_0_,
        member0_.team_id as team_id3_0_,
        member0_.username as username2_0_ 
    from
        member member0_
        
    -- 팀 가져오기
    select
        team0_.team_id as team_id1_4_0_,
        team0_.name as name2_4_0_ 
    from
        team team0_ 
    where
        team0_.team_id=1
        
        
    select
        team0_.team_id as team_id1_4_0_,
        team0_.name as name2_4_0_ 
    from
        team team0_ 
    where
        team0_.team_id=2
        
        
    select
        team0_.team_id as team_id1_4_0_,
        team0_.name as name2_4_0_ 
    from
        team team0_ 
    where
        team0_.team_id=3
```



![](https://docs.google.com/drawings/d/szwsHRpbHPBxSFci-NRhSJw/image?parent=e/2PACX-1vRVvbIKomKLDTKGV_Zpd2jnSflB7IGvZt3_68Ps6WLyfUO3_b16JGyGje2BnfNAFQnyCXgohLQOH9QO&rev=132&drawingRevisionAccessToken=MOSKHFYSmFoTEQ&h=474&w=563&ac=1)





### 3번 지연로딩으로 멤버 가져오기

**지연로딩 설정**

```java
@Entity
@Getter @Setter
@ToString(of={"id", "username"})
public class Member {
	// ...
        
    @ManyToOne(fetch = FetchType.LAZY)  // 지연로딩으로 설정
    @JoinColumn(name = "team_id")
    private Team team;
}
```



**멤버 가져오기**

```java
@Test
public void test() {

    Member member = memberRepository.findById(1L).get();
    List<Member> all = memberRepository.findAll();
}
```



**쿼리**

```sql
    select
        member0_.member_id as member_i1_0_0_,
        member0_.team_id as team_id3_0_0_,
        member0_.username as username2_0_0_ 
    from
        member member0_ 
    where
        member0_.member_id=1
        
    select
        member0_.member_id as member_i1_0_,
        member0_.team_id as team_id3_0_,
        member0_.username as username2_0_ 
    from
        member member0_
```

- 딱 member만 가져온다.





### 4번 지연 로딩 + FETCH JOIN

**FETCH JOIN를 이용한 쿼리 정의**

```java
public interface MemberRepository extends JpaRepository<Member, Long> {

    @Query("select m from Member m join fetch m.team t")
    List<Member> findMembersWithTeam();


    @Query("select m from Member m join fetch m.team t where m.id = :id")
    Optional<Member> findMemberWithTeamByMemberId(@Param("id") Long id);
}
```





**멤버 하나를 팀이랑 FETCH JOIN 해서 가져오기**

```java
@Test
public void test() {

    Member member = memberRepository.findMemberWithTeamByMemberId(1L).get();
}
```



**쿼리**

```sql
    select
        member0_.member_id as member_i1_0_0_,
        team1_.team_id as team_id1_4_1_,
        member0_.team_id as team_id3_0_0_,
        member0_.username as username2_0_0_,
        team1_.name as name2_4_1_ 
    from
        member member0_ 
    inner join
        team team1_ 
            on member0_.team_id=team1_.team_id 
    where
        member0_.member_id=1
```



![](https://docs.google.com/drawings/d/sBqUw0HN4Uat5X_eBdZPaMg/image?parent=e/2PACX-1vRVvbIKomKLDTKGV_Zpd2jnSflB7IGvZt3_68Ps6WLyfUO3_b16JGyGje2BnfNAFQnyCXgohLQOH9QO&rev=189&drawingRevisionAccessToken=fg_NnsX6QdmNrQ&h=487&w=563&ac=1)





**모든 멤버 팀이랑 FETCH JOIN해서 가져오기**

```java
@Test
public void test() {
    List<Member> membersWithTeam = memberRepository.findMembersWithTeam();
}
```



**쿼리**

```sql
    select
        member0_.member_id as member_i1_0_0_,
        team1_.team_id as team_id1_4_1_,
        member0_.team_id as team_id3_0_0_,
        member0_.username as username2_0_0_,
        team1_.name as name2_4_1_ 
    from
        member member0_ 
    inner join
        team team1_ 
            on member0_.team_id=team1_.team_id
```



![](https://docs.google.com/drawings/d/sGXCQr6GYs5gXj0yfWE4g2A/image?parent=e/2PACX-1vRVvbIKomKLDTKGV_Zpd2jnSflB7IGvZt3_68Ps6WLyfUO3_b16JGyGje2BnfNAFQnyCXgohLQOH9QO&rev=5&drawingRevisionAccessToken=PxgzJQ9m3W0bBA&h=985&w=563&ac=1)







### 5번 객체 그래프 탐색으로 가져오기

```java
@Test
@Rollback(value = false)
public void test() {

    Member member1 = memberRepository.findById(1L).get();
    member1.getTeam();
}
```

- member1으로 `getTeam()`을 호출해도 Team의 필드를 직접 접근하기 전까지는 조회 안한다.

- **쿼리**

  - ```sql
        select
            member0_.member_id as member_i1_0_0_,
            member0_.team_id as team_id3_0_0_,
            member0_.username as username2_0_0_ 
        from
            member member0_ 
        where
            member0_.member_id=1
    ```

    



```java
@Test
@Rollback(value = false)
public void test() {

    Member member1 = memberRepository.findById(1L).get();
    Team team = member1.getTeam();
    team.getName();
}
```

- team의 `name` 필드를 접근하면 그때 조회 쿼리가 나간다.

- **쿼리**

  - ```sql
        select
            member0_.member_id as member_i1_0_0_,
            member0_.team_id as team_id3_0_0_,
            member0_.username as username2_0_0_ 
        from
            member member0_ 
        where
            member0_.member_id=1
            
        select
            team0_.team_id as team_id1_4_0_,
            team0_.name as name2_4_0_ 
        from
            team team0_ 
        where
            team0_.team_id=1
    ```

    





```java
@Test
public void test() {

    List<Member> all = memberRepository.findAll();

    for (Member member : all) {
        member.getTeam().getName();
    }
}
```



**쿼리**

```sql
    select
        member0_.member_id as member_i1_0_,
        member0_.team_id as team_id3_0_,
        member0_.username as username2_0_ 
    from
        member member0_
        
        
    select
        team0_.team_id as team_id1_4_0_,
        team0_.name as name2_4_0_ 
    from
        team team0_ 
    where
        team0_.team_id=1
        
        
    select
        team0_.team_id as team_id1_4_0_,
        team0_.name as name2_4_0_ 
    from
        team team0_ 
    where
        team0_.team_id=2
        
        
    select
        team0_.team_id as team_id1_4_0_,
        team0_.name as name2_4_0_ 
    from
        team team0_ 
    where
        team0_.team_id=3
```







### 6번 다대일에서 일(Team) 가져오기 (Default가 지연로딩임) + FETCH JOIN



**팀 하나 가져오기**

```java
@Test
public void test() {
    Team team = teamRepository.findById(1L).get();
}
```

- 지연 로딩으로 되어 있음



**쿼리**

```sql
    select
        team0_.team_id as team_id1_4_0_,
        team0_.name as name2_4_0_ 
    from
        team team0_ 
    where
        team0_.team_id=1
```

- 팀만 딱 가져옴





**FETCH JOIN 이용한 쿼리 정의**

```java
public interface TeamRepository extends JpaRepository<Team, Long> {

    @Query("select t from Team t join fetch t.members")
    List<Team> findAllTeamsWithMembers();


    @Query("select t from Team t join fetch t.members where t.id = :id")
    Optional<Team> findTeamWithMembersById(@Param("id") Long id);
}
```





**팀 전부다 가져오기**

```java
@Test
public void test() {
    List<Team> teams = teamRepository.findAllTeamsWithMembers();
}
```



**쿼리**

```sql
    select
        team0_.team_id as team_id1_4_0_,
        members1_.member_id as member_i1_0_1_,
        team0_.name as name2_4_0_,
        members1_.team_id as team_id3_0_1_,
        members1_.username as username2_0_1_,
        members1_.team_id as team_id3_0_0__,
        members1_.member_id as member_i1_0_0__ 
    from
        team team0_ 
    inner join
        member members1_ 
            on team0_.team_id=members1_.team_id
```



![](https://docs.google.com/drawings/d/sGXCQr6GYs5gXj0yfWE4g2A/image?parent=e/2PACX-1vRVvbIKomKLDTKGV_Zpd2jnSflB7IGvZt3_68Ps6WLyfUO3_b16JGyGje2BnfNAFQnyCXgohLQOH9QO&rev=5&drawingRevisionAccessToken=PxgzJQ9m3W0bBA&h=985&w=563&ac=1)







**팀 ID로 하나 가져오기**

```java
@Test
public void test() {
    Team team1 = teamRepository.findTeamWithMembersById(1L).get();
}
```



**쿼리**

```sql
    select
        team0_.team_id as team_id1_4_0_,
        members1_.member_id as member_i1_0_1_,
        team0_.name as name2_4_0_,
        members1_.team_id as team_id3_0_1_,
        members1_.username as username2_0_1_,
        members1_.team_id as team_id3_0_0__,
        members1_.member_id as member_i1_0_0__ 
    from
        team team0_ 
    inner join
        member members1_ 
            on team0_.team_id=members1_.team_id 
    where
        team0_.team_id=1
```





### 7번 다대일에서 일 가져오기 By 객체 그래프 탐색

```java
@Test
@Rollback(value = false)
public void test() {
    Team team1 = teamRepository.findById(1L).get();

    for (Member member : team1.getMembers()) {
        member.getUsername();
    }
}
```



**쿼리**

```sql
    select
        team0_.team_id as team_id1_4_0_,
        team0_.name as name2_4_0_ 
    from
        team team0_ 
    where
        team0_.team_id=1
        
        
    select
        members0_.team_id as team_id3_0_0_,
        members0_.member_id as member_i1_0_0_,
        members0_.member_id as member_i1_0_1_,
        members0_.team_id as team_id3_0_1_,
        members0_.username as username2_0_1_ 
    from
        member members0_ 
    where
        members0_.team_id=1
```







```java
@Test
@Rollback(value = false)
public void test() {
    List<Team> teams = teamRepository.findAll();

    for (Team team : teams) {
        for (Member member : team.getMembers()) {
            member.getUsername();
        }
    }
}
```



**쿼리**

```sql
    select
        team0_.team_id as team_id1_4_,
        team0_.name as name2_4_ 
    from
        team team0_
        
        
    select
        members0_.team_id as team_id3_0_0_,
        members0_.member_id as member_i1_0_0_,
        members0_.member_id as member_i1_0_1_,
        members0_.team_id as team_id3_0_1_,
        members0_.username as username2_0_1_ 
    from
        member members0_ 
    where
        members0_.team_id=1
        
        
    select
        members0_.team_id as team_id3_0_0_,
        members0_.member_id as member_i1_0_0_,
        members0_.member_id as member_i1_0_1_,
        members0_.team_id as team_id3_0_1_,
        members0_.username as username2_0_1_ 
    from
        member members0_ 
    where
        members0_.team_id=2
        
        
    select
        members0_.team_id as team_id3_0_0_,
        members0_.member_id as member_i1_0_0_,
        members0_.member_id as member_i1_0_1_,
        members0_.team_id as team_id3_0_1_,
        members0_.username as username2_0_1_ 
    from
        member members0_ 
    where
        members0_.team_id=3
```

