## 1. DCL 개요

**SQL 문장 분류**

- DDL: CREATE, ALTER, DROP, TRUNCATE
- DML: INSERT, DELETE, UPDATE, SELECT
- TCL: ROLLBACK, COMMIT, SAVEPOINT
- **DCL**: GRANT, REVOKE



**DCL**(Data Control Language)는 유저를 생성하고 권한을 제어하는 명령어다.



GRANT -> 권한 부여

REVOKE -> 권한 취소



## 2. 유저와 권한



**가. 유저 생성과 시스템 권한 부여**

사용자가 실행하는 모든 DDL 문장(CREATE, ALTER, DROP, RENAME 등)은 그에 해당하는 적절한 권한이 있어야만 문장을 실행할 수 있다. 이러한 권한을 **시스템 권한**이라고 하며 약 100개 이상의 종류가 있다. 일반적으로 시스템 권한은 일일이 유저에게 부여되지 않는다. 100개 이상의 시스템 권한을 일일이 사용자에게 설정하는 것은 너무 복잡하고, 특히 유저로부터 권한을 관리하기가 어렵기 때문이다. 그래서 **ROLE**을 이용하여 간편하고 쉽게 권한을 부여하게 된다.



**[예제] SCOTT 유저로 접속한 다음 PJS 유저(패스워드: KOREA7)를 생성하기**

```sql
CONN SCOTT/TIGER

CREATE USER PJS IDENTIFIED BY KOREA7; -- 유저를 생성하려면 유저 생성 권한(CREATE USER)이 있어야 한다.
```



SCOTT 유저가 유저를 생성할 권한이 없다면 위의 코드는 오류가 발생한다. Oracle의 DBA 권한을 가지고 있는 SYSTEM 유저로 접속하면 유저 생성 권한(CREATE USER)을 다른 유저에게 부여할 수 있다.



**[예제] SCOTT 유저에게 유저생성 권한(CREATE USER)을 부여한 후 다시 PJS 유저를 생성하기**

```sql
GRANT CREATE USER TO SCOTT;

CONN SCOTT/TIGER

CREATE USER PJS IDENTIFIED BY KOREA7;
```





**[예제] 생성된 PJS 유저로 로그인하기**

```sql
CONN PJS/KOREA7;

-- ERROR: 사용자 PJS는 CREATE SESSION 권한을 가지고 있지 않음; 로그온이 거절되었다.
```



PJS 유저가 생성됐지만 아무런 권한도 부여받지 못했기 때문에 로그인을 하면 CREATE SESSION 권한이 없다는 오류가 발생한다. 유저가 **로그인을 하려면 CREATE SESSION 권한을 부여받아야 한다**.





**[예제] PJS 유저가 로그인할 수 있도록 CREATE SESSION 권한을 부여한다**

```sql
CONN SCOTT/TIGER

GRANT CREATE SESSION TO PJS;

CONN PJS/KOREA7
연결되었다.
```





**[예제] PJS 유저로 테이블을 생성한다**

```sql
SELECT * FROM TAB;
선택된 레코드가 없다.

CREATE TABLE MENU (
  MENU_SEQ NUMBER NOT NULL,
  TITLE VARCHAR2(10)
);
-- ERROR: 권한이 불충분하다.
```



PJS 유저는 로그인 권한만 부여되었기 때문에 테이블을 생성하려면 테이블 생성 권한(CREATE TABLE)이 불충분하다는 오류가 발생한다.



**[예제] SYSTEM 유저를 통하여 PJS 유저에게 CREATE TABLE 권한을 부여한 후 다시 테이블을 생성한다**

```sql
CONN SYSTEM/MANAGER

GRANT CREATE TABLE TO PJS;

CONN PJS/KOREA7

CREATE TABLE MENU (
	MENU_SEQ NUMBER NOT NULL,
	TITLE VARCHAR2(10)
);
```





**나. OBJECT에 대한 권한 부여**





## 3. Role을 이용한 권한 부여

유저를 생성하면 기본적으로

- CREATE SESSION,

- CREATE TABLE,

- CREATE PROCEDURE 등

많은 권한을 부여해야 한다. DB 관리자는 유저가 생성될 때마다 각각의 권한들을 유저에게 부여하는 작업을 수행해야 하며 간혹 권한을 빠뜨릴 수도 있으므로 

DB 관리자는 **ROLE**을 생성하고, ROL에 각종 권한들을 부여한 후 ROLE을 다른 ROLE이나 유저에게 부여할 수 있다. 

ROLE에는 시스템 권한과 오브젝트 권한을 모두 부여할 수 있으며, ROLE은 유저에게 직접 부여될 수도 있고, 다른 ROLE에 포함하여 유저에게 부여될 수도 있다.



**[예제] JISUNG 유저에게 CREATE SESSION과 CREATE TABLE 권한을 가진 ROLE을 생성한 후 ROLE을 이용하여 다시 권한을 할당한다. 권한을 취소할 때는 REVOKE를 사용한다.**

```sql
CONN SYSTEM/MANAGER

REVOKE CREATE SESSION, CREATE TABLE FROM JISUNG;
```



```sql
CONN SYSTEM/MANAGER

CREATE ROLE LOGIN_TABLE;

GRANT CREATE SESSION, CREATE TABLE TO LOGIN_TABLE;

GRANT LOGIN_TABLE TO JISUNG;

CONN JISUNG/KOREA7

CREATE TABLE MENU2(
  MENU_SEQ NUMBER NOT NULL,
  TITLE VARCHAR2(10)
);
```







유저를 삭제하는 명령어는 DROP USER이고, CASCADE 옵션을 주면 해당 유저가 생성한 오브젝트를 먼저 삭제한 후 유저를 삭제한다.



**[예제] 앞에서 MENU라는 테이블을 생성했기 때문에 CASCADE 옵션을 사용하여 JISUNG 유저를 삭제한 후, 유저 재생성 및 기본적인 ROLE을 부여한다.**

```sql
CONN SYSTEM/MANAGER

DROP USER JISUNG CASCADE;
-- JISUNG 유저가 만든 MENU 테이블도 같이 삭제된다.

CREATE USER JISUNG IDENTIFIED BY KOREA7;

GRAN CONNECT, RESOURCE TO JISUNG;

CONN JISUNG/KOREA7

CREATE TABLE MENU (
  MENU_SEQ NUMBER NOT NULL,
  TITLE VARCHAR2(10)
);
```

