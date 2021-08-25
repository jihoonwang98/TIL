## 1. STANDARD SQL 개요

**가. 일반 집합 연산자**

- UNION 연산(합집합) -> UNION 기능
- INTERSECTION 연산(교집합) -> INTERSECT 기능
- DIFFERENCE 연산(차집합) -> EXCEPT(Oracle은 MINUS) 기능
- PRODUCT 연산(카티션 곱) -> CROSS JOIN 기능

 

**나. 순수 관계 연산자**

순수 관계 연산자는 관계형 데이터베이스를 구현하기 위해 새롭게 만들어진 연산자이다.

- SELECT 연산 -> WHERE 절
- PROJECT 연산 -> SELECT 절
- (NATURAL) JOIN 연산 -> JOIN 기능
- DIVIDE 연산은 현재 사용되지 않는다.







## 2. FROM 절 JOIN 형태

ANSI/ISO SQL에서 표시하는 FROM 절의 JOIN 형태는 다음과 같다.

- INNER JOIN: WHERE 절에서부터 사용하던 JOIN의 DEFAULT 옵션으로 JOIN 조건에서 동일한 값이 있는 행만 반환된다.

  

- NATURAL JOIN: INNER JOIN의 하위 개념으로 NATURAL JOIN은 두 테이블 간의 동일한 이름을 갖는 모든 칼럼들에 대해 EQUI(=) JOIN을 수행한다.



- USING 조건절
- ON 조건절
- CROSS JOIN
- OUTER JOIN



## 3. INNER JOIN

INNER JOIN은 JOIN 조건에서 동일한 값이 있는 행만 반환한다. INNER JOIN 표시는 그 동안 WHERE 절에서 사용하던 JOIN 조건을 FROM 절에서 정의하겠다는 표시이므로 USING 조건절이나 ON 조건절을 필수적으로 사용해야 한다.



```sql
[WHERE 절 JOIN 조건]
SELECT emp.deptno, empno, ename, dname
FROM emp, dept
WHERE emp.deptno = dept.deptno;


[FROM 절 JOIN 조건]
SELECT emp.deptno, empno, ename, dname
FROM emp INNER JOIN dept
ON emp.deptno = dept.deptno;
```







## 4. NATURAL JOIN

NATURAL JOIN은 두 테이블 간의 동일한 이름을 갖는 모든 칼럼들에 대해 EQUI(=) JOIN을 수행한다. NATURAL JOIN이 명시되면, 추가로 USING 조건절, ON 조건절, WHERE 절에서 JOIN 조건을 정의할 수 없다. SQL Server에서는 지원하지 않는 기능이다.



```sql
SELECT deptno, empno, ename, dname 
FROM emp NATURAL JOIN dept;
```



위 SQL은 별도의 JOIN 칼럼을 지정하지 않았지만, 두 개의 테이블에서 DEPTNO라는 공통된 칼럼을 자동으로 인식하여 JOIN을 처리한 것이다. JOIN에 사용된 칼럼들은 같은 데이터 유형이어야 하며, ALIAS나 테이블 명과 같은 접두사를 붙일 수 없다.

```sql
SELECT emp.deptno, empno, ename, dname
FROM emp NATURAL JOIN dept;

1행에 오류:
ORA-25155: NATURAL 조인에 사용된 열은 식별자를 가질 수 없음
```



```sql
SELECT * FROM dept;

DEPTNO DNAME           LOC
------ --------------- ---------------
    10 ACCOUNTING      NEW YORK
    20 RESEARCH        DALLAS
    30 SALES           CHICAGO
    40 OPERATIONS      BOSTON


SELECT * FROM dept_temp;

DEPTNO DNAME           LOC
------ --------------- ---------------
    10 ACCOUNTING      NEW YORK
    20 R_D             DALLAS
    30 marketing       CHICAGO
    40 OPERATIONS      BOSTON
        
        

SELECT * FROM dept NATURAL [INNER] JOIN dept_temp;
        
DEPTNO DNAME           LOC
------ --------------- ---------------
    10 ACCOUNTING      NEW YORK
    40 OPERATIONS      BOSTON
```







## 5. USING 조건절

NATURAL JOIN에서는 모든 일치되는 칼럼들에 대해 JOIN이 이루어지지만, FROM 절의 USING 조건절을 이용하면 같은 이름을 가진 칼럼들 중에서 원하는 칼럼에 대해서만 선택적으로 EQUI JOIN을 할 수가 있다. 다만, 이 기능은 SQL Server에서는 지원하지 않는다.



```sql
SELECT * 
FROM dept JOIN dept_temp
USING (deptno);

DEPTNO DNAME           LOC             DNAME           LOC
------ --------------- --------------- --------------- ---------------
    10 ACCOUNTING      NEW YORK        ACCOUNTING      NEW YORK
    20 RESEARCH        DALLAS          R_D             DALLAS
    30 SALES           CHICAGO         marketing       CHICAGO
    40 OPERATIONS      BOSTON          OPERATIONS      BOSTON
```



- USING 조건절을 이용한 EQUI JOIN에서도 NATURAL JOIN과 마찬가지로 JOIN 칼럼에 대해서는 ALIAS나 테이블 이름과 같은 접두사를 붙일 수 없다.



```sql
SELECT * 
FROM dept JOIN dept_temp
USING (dname);

DNAME           DEPTNO LOC             DEPTNO LOC
--------------- ------ --------------- ------ ---------------
ACCOUNTING          10 NEW YORK            10 NEW YORK
OPERATIONS          40 BOSTON              40 BOSTON
```



```sql
SELECT *
FROM dept JOIN dept_temp
USING (loc, deptno);

LOC             DEPTNO DNAME           DNAME
--------------- ------ --------------- ---------------
NEW YORK            10 ACCOUNTING      ACCOUNTING
DALLAS              20 RESEARCH        R_D
CHICAGO             30 SALES           marketing
BOSTON              40 OPERATIONS      OPERATIONS
```



```sql
SELECT * 
FROM dept JOIN dept_temp
USING (deptno, dname);

DEPTNO DNAME           LOC             LOC
------ --------------- --------------- ---------------
    10 ACCOUNTING      NEW YORK        NEW YORK
    40 OPERATIONS      BOSTON          BOSTON
```



## 6. ON 조건절

JOIN 서술부(ON 조건절)와 비 JOIN 서술부(WHERE 조건절)를 분리하여 이해가 쉬우며, 칼럼명이 다르더라도 JOIN 조건을 사용할 수 있는 장점이 있다.



```sql
SELECT e.empno, e.ename, e.deptno, d.dname
FROM emp e JOIN dept d
ON (e.deptno = d.deptno);
```



NATURAL JOIN의 JOIN 조건은 기본적으로 같은 이름을 가진 모든 칼럼들에 대한 동등 조건이지만, 임의의 JOIN 조건을 지정하거나, 이름이 다른 칼럼명을 JOIN 조건으로 사용하거나, JOIN 칼럼을 명시하기 위해서는 ON 조건절을 사용한다. ON 조건절에 사용된 괄호는 옵션 사항이다.



USING 조건절을 이용한 JOIN에서는 JOIN 칼럼에 대해서 ALIAS나 테이블 명과 같은 접두사를 사용하면 SYNTAX 에러가 발생하지만, 반대로 ON 조건절을 사용한 JOIN의 경우는 ALIAS나 테이블 명과 같은 접두사를 사용하여 SELECT에 사용되는 칼럼을 논리적으로 명확하게 지정해주어야 한다.





**가. WHERE 절과의 혼용**

```sql
SELECT e.ename, e.deptno, d.deptno, d.dname
FROM emp e JOIN dept d ON (e.deptno = d.deptno)
WHERE e.deptno = 30;

ENAME                DEPTNO DEPTNO DNAME
-------------------- ------ ------ ---------------
ALLEN                    30     30 SALES
WARD                     30     30 SALES
MARTIN                   30     30 SALES
BLAKE                    30     30 SALES
TURNER                   30     30 SALES
JAMES                    30     30 SALES
```



**나. ON 조건절 + 데이터 검증 조건 추가**

ON 조건절에 JOIN 조건 외에도 데이터 검색 조건을 추가할 수는 있으나, 검색 조건 목적인 경우는 WHERE 절을 사용할 것을 권고한다. (다만, 아우터 조인에서 조인의 대상을 제한하기 위한 목적으로 사용되는 추가 조건의 경우는 ON 절에 표기되어야 한다.)



```sql
SELECT e.ename, e.mgr, d.deptno, d.dname
FROM emp e JOIN dept d
ON (e.deptno = d.deptno AND e.mgr = 7698);

ENAME                       MGR DEPTNO DNAME
-------------------- ---------- ------ ---------------
ALLEN                      7698     30 SALES
WARD                       7698     30 SALES
JAMES                      7698     30 SALES
TURNER                     7698     30 SALES
MARTIN                     7698     30 SALES
```





**다. 다중 테이블 JOIN **

```sql
SELECT e.empno, d.deptno, d.dname, t.dname New_DNAME
FROM emp e JOIN dept d
ON (e.deptno = d.deptno)
JOIN dept_temp t
ON (e.deptno = t.deptno);
```









## 7. CROSS JOIN

CROSS JOIN은 일반 집합 연산자의 PRODUCT의 개념으로 테이블 간 JOIN 조건이 없는 경우 생길 수 있는 모든 데이터의 조합을 말한다.



```
SELECT ename, dname
FROM emp CROSS JOIN dept
ORDER BY ename;

SELECT ename, dname
FROM emp, dept
ORDER BY ename;
```





## 8. OUTER JOIN

OUTER JOIN 역시 JOIN 조건을 FROM 절에서 정의하겠다는 표시이므로 USING 조건절이나 ON 조건절을 필수적으로 사용해야 한다. 



**가. LEFT OUTER JOIN**

조인 수행시 먼저 표기된 좌측 테이블에 해당하는 데이터를 먼저 읽은 후, 나중 표기된 우측 테이블에서 JOIN 대상 데이터를 읽어 온다. 즉, Table A와 B가 있을 때, A와 B를 비교해서 B의 JOIN 칼럼에서 같은 값이 있을 때 그 해당 데이터를 가져오고, B의 JOIN 칼럼에서 같은 값이 없는 경우에는 B 테이블에서 가져오는 칼럼들은 NULL 값으로 채운다. 그리고 LEFT JOIN으로 OUTER 키워드를 생략해서 사용할 수 있다.



**나. RIGHT OUTER JOIN**

조인 수행시 LEFT JOIN과 반대로 우측 테이블이 기준이 되어 결과를 생성한다. 즉, TABLE A와 B가 있을 때, A와 B를 비교해서 A의 JOIN 칼럼에서 같은 값이 있을 때 그 해당 데이터를 가져오고, A의 JOIN 칼럼에서 같은 값이 없는 경우에는 A 테이블에서 가져오는 칼럼들은 NULL 값으로 채운다. 그리고 RIGHT JOIN으로 OUTER 키워드를 생략해서 사용할 수 있다.



**다. FULL OUTER JOIN**

조인 수행시 좌측, 우측 테이블의 모든 데이터를 읽어 JOIN하여 결과를 생성한다. 즉, TABLE A와 B가 있을 때, RIGHT OUTER JOIN과 LEFT OUTER JOIN의 결과를 합집합으로 처리한 결과와 동일하다. 단, UNION ALL이 아닌 UNION 기능과 같으므로 중복되는 데이터는 삭제한다. 그리고 FULL JOIN으로 OUTER 키워드를 생략해서 사용할 수 있다.

