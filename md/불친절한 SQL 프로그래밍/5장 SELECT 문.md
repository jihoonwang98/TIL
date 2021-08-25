# 5 SELECT 문

**SELECT 문**은 데이터를 <u>조회</u>하는 구문이다.

기본 SELECT 문은 SELECT 절과 FROM 절로 구성된다. SELECT 절 다음에 FROM 절을 기술하며, <u>FROM 절이 수행된 후 SELECT 절이 수행된다</u>.



```sql
SELECT 절 -- (2)
  FROM 절 -- (1)
```



| 절        | 설명                        |
| --------- | --------------------------- |
| SELECT 절 | 조회할 열이나 표현식을 기술 |
| FROM 절   | 조회할 테이블을 기술        |



## 5.1 SELECT 절

SELECT 절에 조회할 열이나 표현식을 기술할 수 있다. *(asterisk), 열, 열 별칭, DISTINCT 키워드를 차례대로 살펴보자.



```sql
SELECT [{{DISTINCT | UNIQUE} | ALL}]
		{* | {t_alias.* | expr [[AS] c_alias]} [, {t_alias.*} | expr [[AS] c_alias]}] ...}
```



### 5.1.1 Asterisk (*)

SELECT 절에 asterisk(*)를 기술하면 테이블의 전체 열이 조회된다. 결과의 열 순서는 테이블의 열 순서와 동일하다.



아래는 asterisk를 사용한 쿼리다.

```sql
SELECT * FROM dept;

// output

```



> DESC[RIBE] 명령어
>
> DESC[RIBE] 명령어로 테이블의 열 정보를 확인할 수 있다. 
>
> ```sql
> DESC dept
> 
> // output
> ```



### 5.1.2 열

SELECT 절에 조회할 열(column)을 기술할 수 있다. 열은 쉼표(,)로 구분한다.



아래 쿼리는 dname 열과 deptno 열을 조회한다. 테이블의 열 순서와 무관하게 결과의 열 순서를 지정할 수 있다.

```sql
SELECT dname, deptno FROM dept;

// output
```



위 쿼리는 아래 쿼리로 해석된다. 테이블에 열이 종속된 구조({테이블}.{열})다.

```sql
SELECT dept.dname, dept.deptno FROM dept;
```



아래 쿼리는 구문상의 제약으로 에러가 발생한다.

```sql
SELECT *, deptno FROM dept;

ORA-00923: FROM 키워드가 필요한 위치에 없습니다.
```



asterisk와 열을 함께 사용하려면 asterisk를 테이블로 한정해야 한다.

```sql
SELECT dept.*, deptno FROM dept;

// output
```



### 5.1.3 열 별칭

열은 별칭(column alias)을 지정할 수 있다. 별칭을 지정하면 열이나 표현식을 간결하게 나타낼 수 있다. 



**별칭 선언 제약**

- 대소문자 구분 X
- 숫자로 시작 X
- 공백이나 특수 문자(#, $, _는 제외) 포함 X
- <u>단, 별칭을 큰 따옴표(")로 감싸면 제약을 회피할 수 있다</u>



```sql
expr [[AS] c_alias]
```



아래 쿼리는 열에 별칭을 지정했다. AS 키워드를 사용하면 가독성을 높일 수 있다. 가급적 AS 키워드를 사용하자. 큰 따옴표는 사용하지 않는 편이 바람직하다.

```sql
SELECT deptno dept_no, dname AS dept_nm, loc AS "Location" FROM dept;
```



### 5.1.4 DISTINCT 키워드

SELECT 절에 DISTINCT 키워드나 UNIQUE 키워드를 기술하면 중복 행이 제거된 결과가 반환된다. ALL 키워드를 기술하면 중복 행을 기술하지 않는다. 기본값은 ALL이다.



```sql
[{{DISTINCT | UNIQUE} | ALL}]
```



```sql
-- 아래 쿼리는 전체 행을 반환
SELECT deptno FROM emp;

-- 아래 쿼리는 중복 행이 제거된 결과를 반환
SELECT DISTINCT deptno FROM emp;
```





## 5.2 FROM 절

FROM 절에 조회할 테이블을 기술할 수 있다. 테이블은 쉼표(,)로 구분한다. 2개 이상의 테이블을 기술하면 조인이 수행된다.



```sql
FROM [schema.]table [sample_clause] [t_alias]
```



### 5.2.1 스키마

테이블에 스키마(schema)를 지정하면 해당 스키마의 테이블을 조회할 수 있다. 스키마를 지정하지 않으면 현재 사용자의 테이블, 전용 synonym, 공용 synonym 순서로 구문이 해석된다. 



현재 사용자가 SCOTT인 경우 아래의 쿼리들은

```sql
SELECT deptno FROM dept;
SELECT deptno FROM scott.dept;
SELECT dept.deptno FROM dept;
SELECT dept.deptno FROM scott.dept;
```

모두 아래의 쿼리로 해석된다.

```sql
SELECT scott.dept.deptno FROM scott.dept;
```



열은 테이블, 테이블은 스키마에 종속된 구조({스키마}.{테이블}.{열})다.



### 5.2.2 테이블 별칭

테이블도 별칭(table alias)을 지정할 수 있다. [schema.]table이 t_alias로 대체된다.

```sql
[schema.]table [t_alias]
```



아래 쿼리는 dept 테이블에 별칭을 부여한다. scott.dept가 a로 대체된다. 테이블 별칭으로 열을 한정하면 쿼리의 가독성을 높일 수 있다.

```sql
SELECT a.deptno FROM dept a;
```



아래 쿼리는 에러가 발생한다. scott.dept가 a로 대체되어 dept로 열을 한정할 수 없기 때문이다.

```sql
SELECT dept.deptno FROM a;

ORA-00904: "DEPT"."DEPTNO": 부적합한 식별자
```



아래 쿼리도 에러가 발생한다. FROM 절에 2개의 테이블을 기술하여 WHERE 절에 기술한 deptno 열이 어느 테이블의 열인지 알 수 없기 때문이다. 

```sql
SELECT * FROM dept, emp WHERE deptno = deptno;

ORA-00918: 열의 정의가 애매합니다
```



아래와 같이 테이블로 열을 한정하면 에러가 발생하지 않는다.

```sql
SELECT * FROM dept, emp WHERE emp.deptno = dept.deptno
```



테이블 대신 테이블 별칭으로 열을 한정하면 쿼리를 간결하게 작성할 수 있다.

```sql
SELECT * FROM dept a, emp b WHERE b.deptno = a.deptno;
```





### 5.2.3 SAMPLE 절

(일단 생략)





## 5.3 기본 요소

SQL에서 공통적으로 사용하는 기본 요소를 살펴보자.



### 5.3.1 리터럴

리터럴(literal)은 변하지 않는 값이다. 다른 프로그래밍 언어의 상수와 유사하다. 문자, 숫자, 날짜, 인터벌 리터럴을 사용할 수 있다.



#### 5.3.1.1 문자 리터럴

문자 리터럴은 문자 값을 지정한다. 문자열은 작은 따옴표(')로 감싸서 기술한다.



리터럴은 전체 행에서 동일한 값을 반환한다.

```sql
SELECT 'Department' AS c1, deptno FROM dept;

// output
```



> **DUAL 테이블**
>
> DUAL 테이블은 dummy 열로 구성되며, 1개의 행을 가지고 있다. DUAL 테이블은 리터럴 조회, 행 복제 등의 다양한 용도로 활용할 수 있다. SELECT 절에 *나 dummy 열을 기술하면 쿼리의 성능이 저하될 수 있으므로 리터럴만 기술하도록 하자.
>
> ```sql
> SELECT * FROM DUAL;
> 
> // output
> ```



#### 5.3.1.2 숫자 리터럴

숫자 리터럴은 숫자 값을 지정한다.



#### 5.3.1.3 날짜 리터럴

날짜 리터럴은 날짜 값을 지정한다. DATE, TIMESTAMP, TIMESTAMP WITH TIME ZONE 리터럴을 사용할 수 있다. 



날짜 값은 NLS 파라미터 설정에 따라 출력 포맷이 결정된다. 날짜 값의 출력 포맷과 관련된 NLS 파라미터를 아래와 같이 설정하자.

```sql
ALTER SESSION SET NLS_DATE_FORMAT = "YYYY-MM-DD HH24:MI:SS";
ALTER SESSION SET NLS_TIMESTAMP_FORMAT = "YYYY-MM-DD HH24:MI:SS.FF";
ALTER SESSION SET NLS_TIMESTAMP_TZ_FORMAT = "YYYY-MM-DD HH24:MI:SS.FF TZH:TZM";
```



DATE 리터럴은 연월일을 지정한다. 아래 쿼리의 c1 열은 DATE 리터럴로 연월일을 지정했다. 시분초까지 지정하려면 c2 열처럼 TO_DATE 함수를 사용해야 한다.

```sql
SELECT DATE '2050-01-01' AS c1
	 , TO_DATE ('2050-01-01 23:59:59', 'YYYY-MM-DD HH24:MI:SS') AS c2
  FROM DUAL;
  
// output

```



TIMESTAMP 리터럴은 소수점 이하 초를 지정할 수 있다.

```sql
SELECT TIMESTAMP '2050-01-01 23:59:59.999999999' AS c1 FROM DUAL;

// output
```



TIMESTAMP WITH TIME ZONE 리터럴은 TIMESTAMP에 시간대 변위 값을 포함시킬 수 있다. 시간대 변위 값에 대한 내용은 나중에 살펴보자.

```sql
SELECT TIMESTAMP '2050-01-01 23:59:59.999999999 +09:00' AS c1 FROM DUAL;
```



#### 5.3.1.4 인터벌 리터럴

인터벌 리터럴은 시간의 간격을 지정한다. YEAR TO MONTH, DAY TO SECOND 리터럴을 사용할 수 있다.



YEAR TO MONTH 리터럴은 년에서 월까지의 간격을 지정할 수 있다. precision의 범위는 0~9, 기본값은 2다.

```sql
INTERVAL 'integer [- integer]' {YEAR | MORTH} [(precision)] [TO {YEAR | MONTH}]
```



아래는 YEAR TO MONTH 리터럴을 사용한 쿼리다. c3 열처럼 연월을 함께 기술하면 월은 11까지 지정할 수 있다.

```sql
SELECT INTERVAL '99' 	YEAR 		  AS c1
	 , INTERVAL '99' 	MONTH 		  AS c2
	 , INTERVAL '99-11' YEAR TO MONTH AS c3
  FROM DUAL;
```



DAY TO SECOND 리터럴은 일에서 초까지의 간격을 지정할 수 있다.

```sql
INTERVAL '{integer | integer time_expr | time_expr}'
	     {{DAY | HOUR | MINUTE} [(leading_precision)]
	     | SECOND [(leading_precision [, fractional_seconds_precision])]}
	 [TO {DAY | HOUR | MINUTE | SECOND [(fractional_seconds_precision)]}]
```



| 항목                         | 설명                      |
| ---------------------------- | ------------------------- |
| leading_precision            | 범위는 0 ~ 9 (기본값은 2) |
| fractional_seconds_precision | 범위는 1 ~ 9 (기본값은 6) |



아래는 DAY TO SECOND 리터럴을 사용한 쿼리다. c2 열처럼 SECOND만 기술한 경우 쉼표로 정밀도를 지정할 수 있다. 쉼표의 앞자리는 일 정밀도, 뒷자리는 초 정밀도를 나타낸다. c3 열처럼 일과 시분초를 함께 기술하면 시는 23, 분은 59, 초는 59.999999999까지 지정할 수 있다.

```sql
SELECT INTERVAL '99'			       DAY				   AS c1
     , INTERVAL '863999.999999999'	   SECOND(1, 9) 	   AS c2
     , INTERVAL '9 23:59:59.999999999' DAY(1) TO SECOND(9) AS c3
     , INTERVAL '239:59'               HOUR(1) TO MINUTE   AS c4
  FROM DUAL;
```





### 5.3.2 널

널(null)은 값이 없거나 정해지지 않은 것을 의미한다. 오라클 DB는 <u>널과 빈 문자('')을 동일하게 처리</u>한다.



아래 쿼리에서 c1 열은 NULL 키워드, c2 열은 빈 문자로 널을 지정했다. NULL 키워드를 사용하는 편이 가독성 측면에서 바람직하다.

```sql
SELECT NULL AS c1, '' AS c2 FROM DUAL;
```





### 5.3.3 연산자

연산자(operator)는 피연산자(operand)에 대한 연산을 수행한다. 오라클 DB는 다양한 연산자를 제공한다. 이번 장에서 산술 연산자와 연결 연산자를 살펴보고, 나머지 연산자는 각각의 장에서 살펴보자.



| 연산자           | 설명                                      | 장   |
| ---------------- | ----------------------------------------- | ---- |
| 산술 연산자      | 숫자 값과 날짜 값에 대한 산술 연산을 수행 | 5장  |
| 연결 연산자      | 피연산자를 연결한 문자 값을 반환          | 5장  |
| 집합 연산자      | 데이터 집합을 수평으로 연결               | 13장 |
| 계층 쿼리 연산자 | 계층 쿼리의 노드 값을 반환                | 16장 |
| MULTISET 연산자  | 중첩 테이블을 검색                        | 28장 |



#### 5.3.3.1 산술 연산자

산술 연산자(arithmetic operator)는 숫자 값과 날짜 값에 대한 산술 연산을 수행한다.



| 구분 | 연산자     | 설명                     | 예시              |
| ---- | ---------- | ------------------------ | ----------------- |
| 단항 | +, -       | 양수, 음수               | -1                |
| 이항 | +, -, *, / | 덧셈, 뺄셈, 곱셈, 나눗셈 | 1 + 2 - 3 * 4 / 5 |



아래 쿼리는 숫자 값에 대한 산술 연산을 수행한다. c1, c2 열은 결과가 동일하다. c2 열처럼 괄호를 사용하는 편이 가독성 측면에서 바람직하다. 

```sql
SELECT 1 + 2 - 3 * 4 / 5 AS c1, 1 + 2 - ((3 * 4) / 5) AS c2 FROM DUAL;
```



나눗셈의 제수가 0이면 에러가 발생한다. 제수 처리에 대한 내용은 6장에서 살펴보자.

```sql
SELECT 1 / 0 FROM DUAL;
```



아래 쿼리는 날짜 값에 산술 연산을 수행한다. 날짜 값의 산술 연산에 사용되는 숫자 값은 일수(number of days)로 계산된다. c1 열은 31일, c2, c3 열은 1초를 더한다. c3 열처럼 인터벌 값을 사용하는 편이 가독성 측면에서 바람직하다.

```sql
SELECT DATE '2050-01-31' + 31                  AS c1
     , DATE '2050-01-31' + (1 / 24 / 60 / 60)  AS c2
     , DATE '2050-01-31' + INTERVAL '1' SECOND AS c3
  FROM DUAL;
```



아래 쿼리는 TIMESTAMP 값에 산술 연산을 수행한다. c1 열은 TIMESTAMP 값에 숫자 값을 연산하여 소수점 이하 초가 유실되었다. c2 열처럼 인터벌 값을 사용하면 소수점 이하 초를 유지할 수 있다. 

```sql
SELECT TIMESTAMP '2050-01-31 23:59:59.999999999' + 31                AS c1
     , TIMESTAMP '2050-01-31 23:59:59.999999999' + INTERVAL '31' DAY AS c2
```



날짜 값에 YEAR TO MONTH 리터럴을 연산하면 에러가 발생할 수 있다. 아래 쿼리는 2050-02-31이 존재하지 않기 때문에 에러가 발생한다.

```sql
SELECT DATE '2050-01-31' + INTERVAL '1' MONTH AS c1 FROM DUAL;

ORA-01839: 지정된 월에 대한 날짜가 부적합합니다
```



날짜 값의 월 연산은 ADD_MONTHS 함수를 사용해야 한다. ADD_MONTHS 함수는 6장에서 살펴보자.

```sql
SELECT ADD_MONTHS (DATE '2050-01-31', 1) AS c1 FROM DUAL;

// output

C1
------------------------------
2050-02-28 00:00:00
```



DATE 값의 연산 결과는 일수를 나타내는 숫자 값(c1), TIMESTAMP 값의 연산 결과는 INTERVAL 값(c2), INTERVAL 값의 연산 결과는 INTERVAL 값(c3)이 반환된다.

```sql
SELECT DATE      '2050-01-02'          - DATE '2050-01-01'               AS c1
     , TIMESTAMP '2050-01-02 00:00:00' - TIMESTAMP '2050-01-01 00:00:00' AS c2
     , INTERVAL  '2' DAY 			   - INTERVAL '1' DAY                AS c3
  FROM DUAL;
```



산술 연산에 NULL을 사용하면 NULL이 반환된다. NULL 처리에 대한 내용은 6장에서 살펴보자.

```sql
SELECT 1 + NULL AS c1
     , DATE '2050-01-01' + NULL AS c2
     , TIMESTAMP '2050-01-01 00:00:00' + NULL AS c3
  FROM DUAL;
```



#### 5.3.3.2 연결 연산자

연결 연산자(concatenation operator)는 피연산자를 연결한 문자 값을 반환한다. 문자 값이 아닌 피연산자는 문자 값으로 변환된다. 널은 무시된다.



아래는 연결 연산자를 사용한 쿼리다.

```sql
SELECT 1 || NULL || 'A' AS c1 FROM DUAL;

// output
C1
------------------------------
1A
```



#### 5.3.3.3 연산자 우선순위

표현식은 연산자 우선순위에 따라 계산된다. 우선순위가 동일하면 좌측부터 계산된다.



| 우선순위 | 연산자                                    |
| -------- | ----------------------------------------- |
| 1        | 단항 산술 연산자(+, -)                    |
| 2        | 다항 산술 연산자(*, /)                    |
| 3        | 다항 산술 연산자(+, -), 연결 연산자(\|\|) |



+, - 연산자와 연결 연산자는 우선순위가 동일하다. 아래의 쿼리는 연결 연산자가 먼저 수행되어 에러가 발생한다. 

```sql
SELECT '$' || 1 + 2 AS c1 FROM DUAL;

ORA-01722: 수치가 부적합합니다
```



아래의 쿼리는 산술 연산자가 먼저 수행되어 에러가 발생하지 않는다.

```sql
SELECT 1 + 2 || '$' AS c1 FROM DUAL;

// output

C1
------------------------------
3$
```





### 5.3.4 표현식

표현식(expression)은 값으로 평가될 수 있는 리터럴, 연산자, SQL 함수 등의 조합이다. 오라클 DB는 다양한 표현식을 제공한다. 이번 장에서 CASE 표현식을 살펴보고, 나머지 표현식은 각각의 장에서 살펴보자.



#### 5.3.4.1 CASE 표현식

CASE 표현식을 사용하면 IF THEN ELSE 논리를 평가할 수 있다. 단순 CASE 표현식과 검색 CASE 표현식을 사용할 수 있다.



- **단순 CASE 표현식**

  단순(simple) CASE 표현식은 expr과 comparison_expr이 일치하는 첫 번째 return_expr, 일치하는 comparison_expr이 없으면 else_expr을 반환한다.

  ```sql
  CASE expr
     {WHEN comparison_expr THEN return_expr}...
     [ELSE else_expr]
  END		
  ```

  

  아래는 단순 CASE 표현식을 사용한 쿼리다. 단순 CASE 표현식은 등가 비교만 가능하다.

  ```sql
  SELECT deptno
       , CASE deptno
              WHEN 10 THEN 1
              WHEN 20 THEN 2
              ELSE 9
         END AS c1
    FROM dept;
  ```

  

  아래 쿼리는 ELSE 절을 기술하지 않았다. ELSE 절을 기술하지 않으면 일치하는 WHEN 절이 없는 경우 널이 반환된다.

  ```sql
  SELECT deptno
       , CASE deptno
              WHEN 10 THEN 1
              WHEN 20 THEN 2
         END AS c1
    FROM dept;
  ```

  

  단순 CASE 표현식은 expr과 comparison_expr의 데이터 타입이 동일하지 않으면 에러가 발생한다.

  ```sql
  SELECT CASE deptno
              WHEN '10' THEN 1
              WHEN 20   THEN 2
         END AS c1 
    FROM dept;
    
  ORA-00932: 일관성 없는 데이터 유형: NUMBER이(가) 필요하지만 CHAR임
  ```

  

  CASE 표현식의 return_expr와 else_expr도 데이터 타입이 동일해야 한다.

  ```sql
  SELECT CASE deptno 
              WHEN 10 THEN '1'
              WHEN 20 THEN 2
         END AS c1
    FROM dept;
  
  ORA-00932: 일관성 없는 데이터 유형: CHAR이(가) 필요하지만 NUMBER임
  ```



- **검색 CASE 표현식**

  검색 CASE 표현식은 condition이 TRUE인 첫 번째 return_expr를 반환한다. TRUE인 condition이 없으면 else_expr이 반환된다.

  ```sql
  CASE 
     {WHEN condition THEN return_expr} ...
     [ELSE else_expr]
  END
  ```

  

  아래는 검색 CASE 표현식을 사용한 쿼리다. 등가 조건 외에도 다양한 조건을 사용할 수 있다. BETWEEN 조건은 범위를 평가하는 조건이다. 7장에서 관련 내용을 살펴보자.

  ```sql
  SELECT deptno
       , CASE
             WHEN deptno BETWEEN 10 AND 20 THEN 1
             WHEN deptno BETWEEN 20 AND 30 THEN 2
             ELSE 9
         END AS c1
    FROM dept;
  ```

  

  검색 CASE 문은 WHEN 절의 기술 순서에 주의해야 한다. 아래 쿼리의 deptno >= 20 조건은 deptno >= 10 조건에 포함되기 때문에 평가되지 않는다.

  ```sql
  SELECT deptno
       , CASE
             WHEN deptno >= 10 THEN 1
             WHEN deptno >= 20 THEN 2
         END AS c1
    FROM dept;
  ```



### 5.3.5 슈도 칼럼

슈도 칼럼(pseudocolumn)은 테이블에 저장되지 않은 의사 칼럼으로 쿼리 수행 시점에 값이 결정된다. 아래 표처럼 구문에 따라 사용할 수 있는 슈도 칼럼이 다르다. 이번 장에서 ROWID 슈도 칼럼을 살펴보고, 나머지 슈도 칼럼은 각각의 장에서 살펴보자.



| 구문      | 슈도 칼럼                                    | 장              |
| --------- | -------------------------------------------- | --------------- |
| 일반      | ROWID, ROWNUM, ORA_ROWSCN                    | 5장, 15장, 19장 |
| 계층 쿼리 | LEVEL, CONNECT_BY_ISLEAF, CONNECT_BY_ISCYCLE | 16장            |
| 시퀀스    | CURRVAL, NEXTVAL                             | 20장            |
| 버전 쿼리 | VERSIONS_STARTSCN, VERSIONS_STARTTIME, ...   | 31장            |



아래 쿼리의 c1 열은 행이 저장된 주소(ROWID)를 반환한다.

```sql
SELECT deptno, ROWID AS c1 FROM dept;

// output 

    DEPTNO C1
---------- ------------------------------
        10 AAAR6zAAHAAAACVAAA
        20 AAAR6zAAHAAAACVAAB
        30 AAAR6zAAHAAAACVAAC
        40 AAAR6zAAHAAAACVAAD
```



> **ROWID**
>
> ROWID는 DB에서 행을 식별할 수 있는 고유 값이다. 오브젝트, 파일, 블록, 행 번호의 조합으로 계산된다. 



### 5.3.6 주석

SQL도 다른 프로그래밍 언어처럼 주석(comment)을 기술할 수 있다. 단일 행 주석과 다중 행 주석을 사용할 수 있다.



| 유형         | 문법                    |
| ------------ | ----------------------- |
| 단일 행 주석 | --로 시작함             |
| 다중 행 주석 | /*로 시작해서 */로 끝남 |





## 5.4 바인드 변수

SQL도 바인드 변수(bind variable)를 사용할 수 있다. 바인드 변수를 사용하면 쿼리의 재사용성을 높일 수 있다.



아래는 바인드 변수를 사용한 쿼리다. 바인드 변수를 사용하지 않았기 때문에 에러가 발생한다.

```sql
SELECT :v1 AS c1 FROM DUAL;

SP2-0552: 바인드 변수 "V1" 가 정의되지 않았습니다
```



SQL*PLUS는 VAR[IABLE] 명령어로 바인드 변수를 선언하고, EXEC[UTE] 명령어로 바인드 변수에 값을 할당한다. 쿼리를 다시 수행하면 v1 바인드 변수에 할당한 1이 반환된다.

```sql
VAR v1 VARCHAR2(10);
EXEC :v1 := 1;

SELECT :v1 AS c1 FROM DUAL;

// output 

C1
------------------------------
1
```



v1 바인드 변수에 '2'를 할당해보자. 쿼리를 다시 수행하면 '2'가 반환된다.

```sql
EXEC :v1 := '2';

SELECT :v1 AS c1 FROM DUAL;

// output 

C1
------------------------------
2
```