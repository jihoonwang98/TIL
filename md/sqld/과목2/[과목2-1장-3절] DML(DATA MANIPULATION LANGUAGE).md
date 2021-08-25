## 1. INSERT

INSERT 문은 테이블에 데이터를 입력하는 명령이다. 테이블에 데이터를 입력하는 방법은 두 가지 유형이 있으며 한 번에 한 건만 입력된다.



```sql
INSERT INTO 테이블명 (COLUMN_LIST) VALUES (COLUMN_LIST에_넣을_VALUE_LIST);

또는

INSERT INTO 테이블명 VALUES (전체_COLUMN에_넣을_VALUE_LIST);
```



- 해당 칼럼명과 입력되어야 하는 값을 1:1로 매핑해서 입력한다.
- 해당 칼럼의 데이터 유형이 CHAR나 VARCHAR2 등 문자 유형일 경우 ' (SINGLE QUOTATION)로 입력할 값을 입력한다.
- 첫 번째 방법을 사용 시 정의하지 않은 칼럼은 Default로 NULL 값이 입력된다 (Default 값이 제약 조건으로 설정되어 있다면 설정되어 있는 Default 값으로 입력된다). 단, PK나 Not NULL로 지정된 칼럼은 NULL이 허용되지 않는다.
- 두 번째 방법을 사용 시 모든 칼럼에 데이터를 입력하는 경우로 칼럼의 순서대로 빠짐없이 데이터가 입력되어야 한다.







## 2. UPDATE

UPDATE는 테이블의 정보를 수정하는 명령이다. 



```sql
UPDATE 테이블명 SET 수정되어야_할_칼럼명 = 수정되기를_원하는_새로운_값;
```



위의 구문을 실행하면 해당 칼럼의 모든 값이 일괄적으로 수정된다.



> 보통 UPDATE 문을 사용할 때는 WHERE 절과 함께 변경을 원하는 행을 선택해 해당 데이터만 수정한다.







## 3. DELETE

DELETE는 데이터를 삭제하는 명령이다. 



```sql
DELETE [FROM] 테이블명;
```



UPDATE 문과 마찬가지로 WHERE 절을 사용하지 않는다면 테이블 내의 전체 데이터가 일괄적으로 삭제된다.



**[참고 DDL 명령어 vs DML 명령어]**

DDL(CREATE, ALTER, RENAME, DROP) 명령어의 경우에는 직접 DB의 테이블에 영향을 미치기 때문에 DDL 명령어를 입력하는 순간 명령어에 해당하는 작업이 즉시(AUTO COMMIT) 완료된다.

하지만 DML(INSERT, UPDATE, DELETE, SELECT) 명령어의 경우, 조작하려는 테이블을 메모리 버퍼에 올려놓고 작업을 하기 때문에 실시간으로 테이블에 영향을 미치는 것은 아니다. 따라서 버퍼에서 처리한 DML 명령어가 실제 테이블에 반영되기 위해서는 COMMIT 명령어를 입력하여 TRANSACTION을 종료해야 한다.







## 4. SELECT

SELECT 문은 테이블에 저장된 데이터들을 조회하는 명령이다.



```sql
SELECT [ALL / DISTINCT] 칼럼명1, 칼럼명2, ... FROM 테이블명;

- ALL: Default 옵션이므로 별도로 표시하지 않아도 됨. 중복된 데이터가 있어도 모두 출력한다.
- DISTINCT: 중복된 데이터가 있는 경우 1건으로 처리해서 출력한다.
```



- WILDCARD 사용하기

  해당 테이블의 모든 칼럼 정보를 보고 싶을 때는 와일드카드로 애스터리스크(*)를 사용해서 조회할 수 있다.

  

- ALIAS 부여하기

  조회된 결과에 일종의 별명(ALIAS, ALIASES)을 부여해서 칼럼 레이블을 변경할 수 있다.







## 5. 산술 연산자와 합성 연산자

- 산술 연산자 (+, -, *, /)

  산술 연산자는 NUMBER와 DATE 자료형에 대해 적용된다. 



- 합성(CONCATENATION) 연산자

  합성 연산자는 문자열과 문자열을 연결하는 연산자다. 

  Oracle은 **2개의 수직 바**(||)를 사용하고,

  SQL Server는 **더하기**(+)를 사용한다.

  두 벤더 모두 공통적으로 **CONCAT (string1, string2)** 함수를 사용할 수 있다.

