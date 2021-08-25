# [Chapter 08] DB 연동



## 트랜잭션 처리

이메일이 유효한지 여부를 판단하기 위해 실제로 검증 목적의 메일을 발송하는 서비스를 경험해봤을 것이다.

이들 서비스는 이메일에 함께 보낸 링크를 클릭하면 최종적으로 이메일이 유효하다고 판단하고 해당 이메일을 사용할 수 있도록 한다.



이렇게 이메일 인증 시점에 테이블의 데이터를 변경하는 기능은 

다음 코드처럼 회원 정보에서 이메일을 수정하고 인증 상태를 변경하는 두 쿼리를 실행할 것이다.

```java
jdbcTemplate.update("update MEMBER set EMAIL = ?", email);
jdbcTemplate.update("insert into EMAIL_AUTH values (?, 'T')", email);
```



그런데 만약 첫 번째 쿼리를 실행한 후 두 번째 쿼리를 실행하는 시점에 문제가 발생하면 어떻게 될까?

예를 들어, 코드를 잘못 수정/배포해서 두 번째 쿼리에서 사용할 테이블 이름이 잘못되었을 수도 있고,

중복된 값이 존재해서 INSERT 쿼리를 실행하는데 실패할 수도 있다.



두 번째 쿼리가 실패했음에도 불구하고 첫 번째 쿼리 실행 결과가 DB에 반영되면 

이후 해당 사용자의 이메일 주소는 인증되지 않은 채로 계속 남아 있게 될 것이다.

따라서 두 번째 쿼리 실행에 실패하면 첫 번째 쿼리 실행 결과도 취소해야 올바른 상태를 유지한다.

이렇게 두 개 이상의 쿼리를 한 작업으로 실행해야 할 때 사용하는 것이 **트랜잭션(transaction)**이다.







**트랜잭션**은 여러 쿼리를 논리적으로 하나의 작업으로 묶어준다.

한 트랜잭션으로 묶인 쿼리 중 하나라도 실패하면 전체 쿼리를 실패로 간주하고 실패 이전에 실행한 쿼리를 취소한다.

쿼리 실행 결과를 취소하고 DB를 기존 상태로 되돌리는 것을 **롤백(rollback)**이라고 부른다.

반면에 트랜잭션으로 묶인 모든 쿼리가 성공해서 쿼리 결과를 DB에 실제로 반영하는 것을 **커밋(commit)**이라고 부른다.



트랜잭션을 시작하면 트랜잭션을 커밋하거나 롤백할 때까지 실행한 쿼리들이 하나의 작업 단위가 된다.

JDBC는 `Connection`의 `setAutoCommit(false)`를 이용해서 트랜잭션을 시작하고

`commit()`과 `rollback()`을 이용해서 트랜잭션을 반영(commit)하거나 취소(rollback)한다.

```java
Connection conn = null;
try {
    conn = DriverManager.getConnection(jdbcUrl, user, pw);
    
    conn.setAutoCommit(false); // 트랜잭션 범위 시작
    ... 쿼리 실행
    conn.commit();             // 트랜잭션 범위 종료: 커밋
} catch (SQLException ex) {
    if(conn != null) {
        // 트랜잭션 범위 종료: 롤백
        try { conn.rollback(); } catch (SQLException ex) { ... }
    }
} finally {
    if(conn != null) {
        try { conn.close(); } catch (SQLException ex) { ... }
    }
}
```

위와 같은 방식은 코드로 직접 트랜잭션 범위를 관리하기 때문에 

개발자가 트랜잭션을 커밋하는 코드나 롤백하는 코드를 누락하기가 쉽다.

게다가 구조적인 중복이 반복되는 문제도 있다.

스프링이 제공하는 트랜잭션 기능을 사용하면 중복이 없는 매우 간단한 코드로 트랜잭션 범위를 지정할 수 있따.





### `@Transactional`을 이용한 트랜잭션 처리

스프링이 제공하는 `@Transactional` 애너테이션을 이용하면 트랜잭션 범위를 매우 쉽게 지정할 수 있다.

다음과 같이 트랜잭션 범위에서 실행하고 싶은 메서드에 `@Transactional` 애너테이션만 붙이면 된다.

```java
@Transactional
public void changePassword(String email, String oldPwd, String newPwd) {
    Member member = memberDao.selectByEmail(email);
    if(member == null) throw new MemberNotFoundException();
    
    member.changePassword(oldPwd, newPwd);
    memberDao.update(member);
}
```



스프링은 `@Transactional` 애너테이션이 붙은 `changePassword()` 메서드를 동일한 트랜잭션 범위에서 실행한다.

따라서 `memberDao.selectByEmail()`에서 실행하는 쿼리와 `member.changePassword()`에서 실행하는 쿼리는 한 트랜잭션에 묶인다.



`@Transactional` 애너테이션이 제대로 동작하려면 다음의 두 가지 내용을 스프링 설정에 추가해야 한다.

- `PlatformTransactionManager` 빈 설정
- `@Transactional` 애너테이션 활성화 설정



다음은 설정 예를 보여준다.

```java
@Configuration
@EnableTransactionManagement
public class AppCtx {
    
    @Bean(destroyMethod = "close")
    public DataSource dataSource() { ... }
    
    @Bean
    public PlatformTransactionManager transactionManager() {
        DataSourceTransactionManager tm = new DataSourceTransactionManager();
        tm.setDataSource(dataSource());
        return tm;
    }
    ...
}
```



`PlatformTransactionManager`는 스프링이 제공하는 트랜잭션 매니저 인터페이스다.

스프링은 구현기술에 상관없이 동일한 방식으로 트랜잭션을 처리하기 위해 이 인터페이스를 사용한다.

JDBC는 `DataSourceTransactionManager` 클래스를 `PlatformTransactionManager`로 사용한다.

위 설정에서 보듯이 `dataSource` 프로퍼티를 이용해서 트랜잭션 연동에 사용할 `DataSource`를 지정한다.



`@EnableTransactionManagement` 애너테이션은 `@Transactional` 애너테이션이 붙은 메서드를 트랜잭션 범위에서 실행하는 기능을 활성화한다. 

등록된 `PlatformTransactionManager` 빈을 사용해서 트랜잭션을 적용한다.





트랜잭션 처리를 위한 설정이 끝나면 트랜잭션 범위에서 실행하고 싶은 스프링 빈 객체의 메서드에 `@Transactional` 애너테이션을 붙이면 된다.







### `@Transactional`과 프록시

트랜잭션도 공통 기능이므로 AOP를 사용해서 처리한다.

스프링에서 AOP는 프록시를 통해서 구현된다는 것을 기억한다면 트랜잭션 처리도 프록시를 통해서 이루어진다고 유추할 수 있을 것이다.





### `@Transactional` 적용 메서드의 롤백 처리









### `@Transactional`의 주요 속성

`@Transactional` 애너테이션의 주요 속성은 아래와 같다.

| 속성        | 타입        | 설명                                                         |
| ----------- | ----------- | ------------------------------------------------------------ |
| value       | String      | 트랜잭션을 관리할 때 사용할 PlatformTransactionManager 빈의 이름을 지정한다. 기본값은 " "이다. |
| propagation | Propagation | 트랜잭션 전파 타입을 지정한다. 기본값은 Propagation.REQUIRED이다. |
| isolation   | Isolation   | 트랜잭션 격리 레벨을 지정한다. 기본값은 Isolation.DEFAULT이다. |
| timeout     | int         | 트랜잭션 제한 시간을 지정한다. 기본값은 -1로 이 경우 데이터베이스의 타임아웃 시간을 사용한다. 초 단위로 지정한다. |

- `Propagation`과 `Isolation` 열거 타입은 `org.springframework.transaction.annotation` 패키지에 정의되어 있다.
- `@Transactional` 애너테이션의 `value` 속성 값이 없으면 등록된 빈 중에서 타입이 `PlatformTransactionManager`인 빈을 사용한다.





`Propagation` 열거 타입에 정의되어 있는 값 목록은 아래와 같다.

| 값            | 설명                                                         |
| ------------- | ------------------------------------------------------------ |
| REQUIRED      | 메서드를 수행하는 데 트랜잭션이 필요하다는 것을 의미한다. 현재 진행중인 트랜잭션이 존재하면 해당 트랜잭션을 사용한다. 존재하지 않으면 새로운 트랜잭션을 생성한다. |
| MANDATORY     | 메서드를 수행하는 데 트랜잭션이 필요하다는 것을 의미한다. 하지만 REQUIRED와 달리 진행 중인 트랜잭션이 존재하지 않을 경우 익셉션이 발생한다. |
| REQUIRES_NEW  | 항상 트랜잭션을 시작한다. 진행 중인 트랜잭션이 존재하면 기존 트랜잭션을 일시 중지하고 새로운 트랜잭션을 시작한다. 새로 시작된 트랜잭션이 종료된 뒤에 기존 트랜잭션이 계속된다. |
| SUPPORTS      | 메서드가 트랜잭션을 필요로 하지는 않지만, 진행 중인 트랜잭션이 존재하면 트랜잭션을 사용한다는 것을 의미한다. 진행 중인 트랜잭션이 존재하지 않더라도 메서드는 정상적으로 동작한다. |
| NOT_SUPPORTED | 메서드가 트랜잭션을 필요로 하지 않음을 의미한다. SUPPORTS와 달리 진행 중인 트랜잭션이 존재할 경우 메서드가 실행되는 동안 트랜잭션은 일시 중지되고 메서드 실행이 종료된 후에 트랜잭션을 계속 진행한다. |
| NEVER         | 메서드가 트랜잭션을 필요로 하지 않는다. 만약 진행 중인 트랜잭션이 존재하면 익셉션이 발생한다. |
| NESTED        | 진행 중인 트랜잭션이 존재하면 기존 트랜잭션에 중첩된 트랜잭션에서 메서드를 실행한다. 진행 중인 트랜잭션이 존재하지 않으면 REQUIRED와 동일하게 동작한다. 이 기능은 JDBC 3.0 드라이버를 사용할 때에만 적용된다(JTA Provider가 이 기능을 지원할 경우에도 사용 가능하다). |





`Isolation` 열거 타입에 정의된 값은 아래와 같다.

| 값               | 설명                                                         |
| ---------------- | ------------------------------------------------------------ |
| DEFAULT          | 기본 설정을 사용한다.                                        |
| READ_UNCOMMITTED | 다른 트랜잭션이 커밋하지 않은 데이터를 읽을 수 있다.         |
| READ_COMMITTED   | 다른 트랜잭션이 커밋한 데이터를 읽을 수 있다.                |
| REPEATABLE_READ  | 처음에 읽어 온 데이터와 두 번째 읽어 온 데이터가 동일한 값을 갖는다. |
| SERIALIZABLE     | 동일한 데이터에 대해서 동시에 두 개 이상의 트랜잭션을 수행할 수 없다. |





### `@EnableTransactionManagement` 애너테이션의 주요 속성

| 속성             | 설명                                                         |
| ---------------- | ------------------------------------------------------------ |
| proxyTargetClass | 클래스를 이용해서 프록시를 생성할지 여부를 지정한다. 기본값은 false로서 인터페이스를 이용해서 프록시를 생성한다. |
| order            | AOP 적용 순서를 지정한다. 기본값은 가장 낮은 우선순위에 해당하는 int의 최댓값이다. |













