# 7장 WHERE 절

WHERE 절은 FROM 절 다음에 기술하며, FROM 절이 수행된 후 수행된다.

```sql
SELECT 절 -- (3)
  FROM 절 -- (1)
 WHERE 절 -- (2)
```



WHERE 절의 구문은 아래와 같다. 조건(condition)은 행마다 평가되며, TRUE, FALSE, UNKNOWN 중 하나의 값을 반환한다. SELECT 문은 결과가 TRUE인 행만 반환한다.

```sql
WHERE condition
```



비교 조건, 논리 조건, IN 조건, BETWEEN 조건, LIKE 조건, 널 조건을 차례대로 살펴보자.



## 7.1 비교 조건

비교 조건(comparison condition)으로 표현식을 비교할 수 있다. 아래와 같은 비교 조건을 사용할 수 있다.

| 비교 조건 | 설명             | 비교 조건  | 설명             |
| --------- | ---------------- | ---------- | ---------------- |
| =         | 같음             | <>, !=, ^= | 다름             |
| >         | 큼               | <          | 작음             |
| >=        | 크거나 같음      | <=         | 작거나 같음      |
| ANY, SOME | 목록 일부를 비교 | ALL        | 목록 전체를 비교 |



아래 쿼리는 job이 ANALYST인 행을 조회한다.

```sql
SELECT ename, job FROM emp WHERE job = 'ANALYST';

// output

ENAME                JOB
-------------------- ------------------
SCOTT                ANALYST
FORD                 ANALYST
```



아래 쿼리는 sal가 2500보다 큰 행을 조회한다.

```sql
SELECT ename, sal FROM emp WHERE sal > 2500;

// output

ENAME                       SAL
-------------------- ----------
JONES                      2975
BLAKE                      2850
SCOTT                      3000
KING                       5000
FORD                       3000
```



아래 쿼리는 hiredate가 1981년 이전인 행을 조회한다.

```sql
SELECT ename, hiredate FROM emp WHERE hiredate < DATE '1981-01-01';

// output 

ENAME                HIREDATE
-------------------- --------
SMITH                80/12/17
```



널은 비교 조건에서 UNKNOWN으로 평가된다. 널 처리는 조금 후에 살펴보자.

```sql
SELECT ename, comm FROM emp WHERE comm = NULL;

선택된 레코드가 없습니다.
```



아래 쿼리는 comm이 0이 아닌 행을 조회한다. NULL <> 0 조건은 UNKNOWN으로 평가되기 때문에 comm이 널인 행은 반환되지 않는다.

```sql
SELECT ename, comm FROM emp WHERE comm <> 0;

// output

ENAME                      COMM
-------------------- ----------
ALLEN                       300
WARD                        500
MARTIN                     1400
```



ANY 조건은 목록의 일부를 비교하여 조건을 만족하는 행을 반환한다. 목록을 일부 만족하면 행이 반환된다. 아래 쿼리는 sal > 1000 조건과 동일한 결과를 반환한다.

```sql
SELECT ename, sal FROM emp WHERE sal > ANY (1000, 2500);

// output

ENAME                       SAL
-------------------- ----------
ALLEN                      1600
WARD                       1250
JONES                      2975
MARTIN                     1250
BLAKE                      2850
CLARK                      2450
SCOTT                      3000
KING                       5000
TURNER                     1500
ADAMS                      1100
FORD                       3000
MILLER                     1300
```



위 쿼리는 아래 쿼리와 논리적으로 동일하다. LEAST 함수를 사용하는 편이 명시적이다.

```sql
SELECT ename, sal FROM emp WHERE sal > LEAST (1000, 2500);
```



ALL 조건은 목록의 전체를 비교하여 조건을 만족하는 행을 반환한다. 목록을 모두 만족해야 행이 반환된다.

```sql
SELECT ename, sal FROM emp WHERE sal > ALL (1000, 2500);

// output

ENAME                       SAL
-------------------- ----------
JONES                      2975
BLAKE                      2850
SCOTT                      3000
KING                       5000
FORD                       3000
```



위 쿼리는 아래 쿼리와 논리적으로 동일하다.

```sql
SELECT ename, sal FROM emp WHERE sal > GREATEST (1000, 2500);
```



비교 조건은 데이터 타입이 동일한 값을 비교해야 한다. 데이터 타입이 다르면 암시적 데이터 변환이 발생한다. 아래에서 좌측 쿼리는 문자 값이 SMITH가 숫자 값으로, 우측 쿼리는 VARCHAR2 타입인 ename이 숫자 값으로 변환되기 때문에 에러가 발생한다.

```sql
SELECT * FROM emp WHERE empno = 'SMITH'           SELECT * FROM emp WHERE ename = 7369;
```



## 7.2 논리 조건

논리 조건(logical condition)으로 조건을 결합하거나 부정할 수 있다. AND 조건, OR 조건, NOT 조건을 사용할 수 있다. 



아래 쿼리는 deptno가 30이고 job이 CLERK인 행을 조회한다.

```sql
SELECT ename, deptno, job FROM emp WHERE deptno = 30 AND job = 'CLERK';

// output

ENAME                    DEPTNO JOB
-------------------- ---------- ------------------
JAMES                        30 CLERK
```



아래 쿼리는 deptno가 30이거나 job이 CLERK인 행을 조회한다.

```sql
SELECT ename, deptno, job FROM emp WHERE deptno = 30 OR job = 'CLERK';

// output

ENAME                    DEPTNO JOB
-------------------- ---------- ------------------
SMITH                        20 CLERK
ALLEN                        30 SALESMAN
WARD                         30 SALESMAN
MARTIN                       30 SALESMAN
BLAKE                        30 MANAGER
TURNER                       30 SALESMAN
ADAMS                        20 CLERK
JAMES                        30 CLERK
MILLER                       10 CLERK
```



아래 쿼리는 deptno가 30이거나 job이 CLERK인 행이 아닌 행을 조회한다.

```sql
SELECT ename, deptno, job FROM emp WHERE NOT (deptno = 30 OR job = 'CLERK');

// output

ENAME                    DEPTNO JOB
-------------------- ---------- ------------------
JONES                        20 MANAGER
CLARK                        10 MANAGER
SCOTT                        20 ANALYST
KING                         10 PRESIDENT
FORD                         20 ANALYST
```



## 7.3 BETWEEN 조건

BETWEEN 조건은 값의 범위에 해당하는 행을 반환한다. BETWEEN 조건을 범위 조건으로 부르기도 한다.

```sql
expr1 [NOT] BETWEEN expr2 AND expr3
```



아래 쿼리는 sal가 2500 이상, 3000 이하인 행을 조회한다.

```sql
SELECT ename, sal FROM emp WHERE sal BETWEEN 2500 AND 3000;

// output

ENAME                       SAL
-------------------- ----------
JONES                      2975
BLAKE                      2850
SCOTT                      3000
FORD                       3000
```



범위 속성을 조회할 경우 BETWEEN 조건을 아래의 방식으로도 사용할 수 있다.

```sql
SELECT * FROM salgrade WHERE 1500 BETWEEN losal AND hisal;

// output

     GRADE      LOSAL      HISAL
---------- ---------- ----------
         3       1401       2000
```



위 쿼리는 아래 쿼리로 해석된다.

```sql
SELECT * FROM salgrade WHERE losal <= 1500 AND 1500 <= hisal;
```



## 7.4 IN 조건

IN 조건은 expr의 목록과 일치하거나 일치하지 않는 행을 반환한다.

```sql
expr [NOT] IN (expr [, expr])
```



아래 쿼리는 job이 ANALYST거나 MANAGER인 행을 조회한다.

```sql
SELECT ename, job FROM emp WHERE job IN ('ANALYST', 'MANAGER');

// output

ENAME                JOB
-------------------- ------------------
JONES                MANAGER
BLAKE                MANAGER
CLARK                MANAGER
SCOTT                ANALYST
FORD                 ANALYST
```



위 쿼리는 아래 쿼리와 논리적으로 동일하다. IN 조건은 OR 조건으로 평가된다.

```sql
SELECT ename, job FROM emp WHERE (job = 'ANALYST' OR job = 'MANAGER');
```



IN 조건은 다중 열(multiple column)을 사용할 수 있다. 아래 쿼리는 deptno가 10이고 job이 MANAGER인 행이나, deptno가 20이고 job이 ANALYST인 행을 조회한다.

```sql
SELECT ename, deptno, job 
  FROM emp
 WHERE (deptno, job) IN ((10, 'MANAGER'), (20, 'ANALYST'));
 
 // output
 
 ENAME                    DEPTNO JOB
-------------------- ---------- ------------------
CLARK                        10 MANAGER
SCOTT                        20 ANALYST
FORD                         20 ANALYST
```



아래 쿼리는 job이 ANALYST와 MANAGER가 아닌 행을 조회한다.

```sql
SELECT ename, job FROM emp WHERE job NOT IN ('ANALYST', 'MANAGER');

// output

ENAME                JOB
-------------------- ------------------
SMITH                CLERK
ALLEN                SALESMAN
WARD                 SALESMAN
MARTIN               SALESMAN
KING                 PRESIDENT
TURNER               SALESMAN
ADAMS                CLERK
JAMES                CLERK
MILLER               CLERK
```



위 쿼리는 아래 쿼리로 해석된다. NOT IN 조건은 AND 조건으로 평가된다.

| 순서 | 쿼리                                                         |
| ---- | ------------------------------------------------------------ |
| 1    | SELECT ename, job FROM emp WHERE NOT (job IN ('ANALYST', 'MANAGER')); |
| 2    | SELECT ename, job FROM emp WHERE NOT (job = 'ANALYST' OR job = 'MANAGER'); |
| 3    | SELECT ename, job FROM emp WHERE job <> 'ANALYST' AND job <> 'MANAGER'; |



NOT IN 조건은 expr에 널이 입력되면 결과가 반환되지 않는다.

```sql
SELECT ename, mgr FROM emp WHERE mgr NOT IN (7839, NULL);

선택된 레코드가 없습니다.
```



위 쿼리는 아래 쿼리로 해석된다. mgr <> NULL 조건은 UNKNOWN으로 평가되고, AND 조건은 조건이 하나라도 UNKNOWN이면 UNKNOWN이므로 결과가 반환되지 않는다.

| 순서 | 쿼리                                                         |
| ---- | ------------------------------------------------------------ |
| 1    | SELECT ename, mgr FROM emp WHERE NOT (mgr IN (7839, NULL));  |
| 2    | SELECT ename, mgr FROM emp WHERE NOT (mgr = 7839 OR mgr = NULL); |
| 3    | SELECT ename, mgr FROM emp WHERE mgr <> 7839 AND mgr <> NULL; |



IN 조건은 널이 입력되어도 결과가 반환된다.

```sql
SELECT ename, mgr FROM emp WHERE mgr IN (7839, NULL);

// output

ENAME                       MGR
-------------------- ----------
JONES                      7839
BLAKE                      7839
CLARK                      7839
```



위 쿼리는 아래 쿼리로 해석된다. mgr = NULL 조건은 UNKNOWN으로 평가되고, OR 조건은 조건이 하나라도 TRUE면 TRUE이므로 mgr = NULL 조건이 무시된다.

```sql
SELECT ename, mgr FROM emp WHERE mgr = 7839 OR mgr = NULL;
```



IN 조건을 아래의 방식으로도 사용할 수 있다. empno나 mgr가 7839인 행을 조회한다.

```sql
SELECT ename, empno, mgr FROM emp WHERE 7839 IN (empno, mgr);

// output

ENAME                     EMPNO        MGR
-------------------- ---------- ----------
JONES                      7566       7839
BLAKE                      7698       7839
CLARK                      7782       7839
KING                       7839
```



위 쿼리는 아래 쿼리로 해석된다.

```sql
SELECT ename, empno, mgr FROM emp WHERE (empno = 7839 OR mgr = 7839);
```



IN 조건은 목록을 1000개까지만 지정할 수 있는 제약이 있다.



## 7.5 LIKE 조건

LIKE 조건은 char1이 char2 패턴과 일치하는 행을 반환한다. char1과 char2는 문자 값을 사용해야 한다. LIKE 조건을 패턴 일치 조건(patter-matching condition)으로 부르기도 한다.

```sql
char1 [NOT] LIKE char2 [ESCAPE esc_char]
```



char2에 아래의 특수문자를 사용할 수 있다.

| 특수문자 | 설명                   |
| -------- | ---------------------- |
| %        | 0개 이상의 문자와 일치 |
| _        | 하나의 문자와 일치     |



아래 쿼리는 ename이 A로 시작하는 행을 조회한다.

```sql
SELECT ename FROM emp WHERE ename LIKE 'A%';

// output

ENAME
--------------------
ALLEN
ADAMS
```



아래 쿼리는 ename이 A로 시작하고 S로 끝나는 행을 조회한다.

```sql
SELECT ename FROM emp WHERE ename LIKE 'A%S';

// output

ENAME
--------------------
ADAMS
```



아래 쿼리는 ename에 ON이 포함된 행을 조회한다.

```sql
SELECT ename FROM emp WHERE ename LIKE '%ON%';

// output

ENAME
--------------------
JONES
```



아래 쿼리는 ename의 세 번째 문자가 M이고 길이가 5자리인 행을 조회한다.

```sql
SELECT ename FROM emp WHERE ename LIKE '__M__';

// output

ENAME
--------------------
JAMES
```



아래 쿼리는 ename에 A가 포함되지 않은 행을 조회한다.

```sql
SELECT ename FROM emp WHERE ename NOT LIKE '%A%';

// output

ENAME
--------------------
SMITH
JONES
SCOTT
KING
TURNER
FORD
MILLER
```



특수문자(%, _)를 검색할 경우 ESCAPE 문자를 사용할 수 있다. 아래 쿼리는 \를 ESCAPE 문자로 사용하여 세 번째 글자가 %인 패턴을 조회한다.

```sql
WITH w1 AS (SELECT '10%'  AS c1 FROM DUAL UNION ALL
            SELECT '100%' AS c1 FROM DUAL)
SELECT c1 FROM w1 WHERE c1 LIKE '__\%' ESCAPE '\';
```



LIKE 조건을 아래의 방식으로도 사용할 수 있다. 아래 쿼리는 ename에 AGENT SMITH의 일부가 포함된 행을 조회한다.

```sql
SELECT ename FROM emp WHERE 'AGENT SMITH' LIKE '%' || ename || '%';
```



아래 쿼리는 위 쿼리와 결과가 동일하다. INSTR 함수를 사용하는 편이 성능 측면에서 유리할 수 있다.

```sql
SELECT ename FROM emp WHERE INSTR ('AGENT SMITH', ename) > 0;
```



아래 쿼리는 1980년에 입사한 사원을 조회한다. 문자 타입이 아닌 열에 LIKE 연산자를 사용하면 암시적 데이터 변환이 발생한다.

```sql
SELECT ename, hiredate FROM emp WHERE hiredate LIKE '1980%';

// output

ENAME                HIREDATE
-------------------- -------------------
SMITH                1980-12-17 00:00:00
```



위 쿼리는 아래 쿼리로 변환된다. 내부 함수인 INTERNAL_FUNCTION 함수를 통해 hiredate가 문자 값으로 변경된다. 

```sql
SELECT ename, hiredate FROM emp WHERE INTERNAL_FUNCTION (hiredate) LIKE '1980%';
```



아래와 같이 NLS_DATE_FORMAT 파라미터를 변경해보자.

```sql
ALTER SESSION SET NLS_DATE_FORMAT = 'MM-DD-YYYY';
```



쿼리를 다시 수행하면 결과가 반환되지 않는다. NLS_DATE_FORMAT에 따라 INTERNAL_FUNCTION 함수의 결과가 달라질 수 있기 때문이다. 날짜 기간 조회에 대한 내용은 조금 후에 살펴보자.

```sql
SELECT ename, hiredate FROM emp WHERE hiredate LIKE '1980%';

선택된 레코드가 없습니다.
```



다음 예제를 위해 NLS_DATE_FORMAT 파라미터를 YYYY-MM-DD HH24:MI:SS로 되돌리자.

```sql
ALTER SESSION SET NLS_DATE_FORMAT = "YYYY-MM-DD HH24:MI:SS";
```



## 7.6 널 조건

널 조건은 널을 포함하거나 포함하지 않는 행을 반환한다.

```sql
expr IS [NOT] NULL
```



아래 쿼리는 comm이 널인 행을 조회한다.

```sql
SELECT ename, comm FROM emp WHERE comm IS NULL;

// output

ENAME                      COMM
-------------------- ----------
SMITH
JONES
BLAKE
CLARK
SCOTT
KING
ADAMS
JAMES
FORD
MILLER
```



아래 쿼리는 comm이 널이 아닌 행을 조회한다.

```sql
SELECT ename, comm FROM emp WHERE comm IS NOT NULL;

// output

ENAME                      COMM
-------------------- ----------
ALLEN                       300
WARD                        500
MARTIN                     1400
TURNER                        0
```



아래 쿼리는 comm이 널이거나 0인 행을 조회한다.

```sql
SELECT ename, comm FROM emp WHERE (comm IS NULL OR comm = 0);

// output

ENAME                      COMM
-------------------- ----------
SMITH
JONES
BLAKE
CLARK
SCOTT
KING
TURNER                        0
ADAMS
JAMES
FORD
MILLER
```



NVL 함수를 사용해도 동일한 결과를 얻을 수 있다.

```sql
SELECT ename, comm FROM emp WHERE NVL (comm, 0) = 0;
```



아래 쿼리는 불필요한 NVL 함수를 사용했다. comm이 널이면 조건이 0 > 0으로 해석되고, 0 > 0 조건은 항상 FALSE이기 때문이다.

```sql
SELECT ename, comm FROM emp WHERE NVL (comm, 0) > 0;
```



위 쿼리는 아래 쿼리와 결과가 동일하다.

```sql
SELECT ename, comm FROM emp WHERE comm > 0;
```



### LNNVL 함수

LNNVL 함수는 condition이 FALSE나 UNKNOWN이면 TRUE, TRUE면 FALSE를 반환한다. WHERE 절과 CASE 표현식에 사용할 수 있다.

```sql
LNNVL (condition)
```



아래는 LNNVL 함수를 사용한 쿼리다. comm이 널이거나 0인 행을 조회한다.

```sql
SELECT ename, comm FROM emp WHERE LNNVL (comm <> 0);

// output

ENAME                      COMM
-------------------- ----------
SMITH
JONES
BLAKE
CLARK
SCOTT
KING
TURNER                        0
ADAMS
JAMES
FORD
MILLER
```



위 쿼리는 아래 쿼리와 논리적으로 동일하다.

```sql
SELECT ename, comm FROM emp WHERE (comm IS NULL OR comm = 0);
```



## 7.7 조건 우선순위

조건은 아래의 우선순위에 따라 평가된다.

| 우선순위 | 조건                                      |
| -------- | ----------------------------------------- |
| 1        | 연산자                                    |
| 2        | 비교 조건(=, <>, >, <, >=, <=)            |
| 3        | IN 조건, LIKE 조건, BETWEEN 조건, 널 조건 |
| 4        | 논리 조건 (NOT)                           |
| 5        | 논리 조건 (AND)                           |
| 6        | 논리 조건 (OR)                            |



deptno가 10이나 20이고 job이 CLERK인 행을 조회해보자. 아래의 쿼리는 의도하지 않은 결과가 반환된다.

```sql
SELECT ename, deptno, job
  FROM emp
 WHERE deptno = 10
    OR deptno = 20
   AND job = 'CLERK';
```



아래와 같이 괄호를 사용하면 의도한 결과를 얻을 수 있다. OR 조건은 반드시 괄호를 사용해야 한다. 

```sql
SELECT ename, deptno, job
  FROM emp
 WHERE job = 'CLERK'
   AND (   deptno = 10
        OR deptno = 20);
```



또는 아래의 쿼리처럼 IN 절을 사용하면 쿼리를 간결하게 작성할 수 있다.

```sql
SELECT ename, deptno, job
  FROM emp
 WHERE job = 'CLERK'
   AND deptno IN (10, 20);
```



## 7.8 활용 예제

WHERE 절의 활용 예제를 살펴보자.



### 열 가공

WHERE 절의 열을 가공하면 쿼리의 성능이 저하될 수 있다. 가급적 열을 가공하지 않는 편이 바람직하다.



아래 쿼리는 연봉이 36000 이상인 행을 조회한다. 아래 쿼리는 산술 연산이 행의 개수만큼 수행된다.

```sql
SELECT *
  FROM emp
 WHERE sal * 12 >= 36000;
```



반면 아래 쿼리는 산술 연산을 수행한 결과로 조건을 평가할 수 있다.

```sql
SELECT * 
  FROM emp
 WHERE sal >= 36000 / 12;
```



(생략)

