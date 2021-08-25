# 8장 ORDER BY 절

ORDER BY 절을 사용하면 SELECT 문의 결과를 정렬할 수 있다. ORDER BY 절을 기술하지 않으면 임의의 순서로 결과가 반환된다.



ORDER BY 절은 SELECT 문의 마지막에 기술하며, SELECT 절이 수행된 후 수행된다.

```
SELECT   절 -- (3)
  FROM   절 -- (1)
 WHERE   절 -- (2)
ORDER BY 절 -- (4)
```



## 8.1 기본 문법

ORDER BY 절의 구문은 아래와 같다.

```sql
ORDER BY {expr | position | c_alias} [ASC | DESC] [NULLS FIRST | NULLS LAST] 
      [, {expr | position | c_alias} [ASC | DESC] [NULLS FIRST | NULLS LAST]]...
```



| 항목        | 설명                                         |
| ----------- | -------------------------------------------- |
| ASC         | 오름차순으로 정렬 (기본값)                   |
| DESC        | 내림차순으로 정렬                            |
| NULLS FIRST | 널을 앞쪽으로 정렬 (내림차순 정렬 시 기본값) |
| NULLS LAST  | 널을 뒤쪽으로 정렬 (오름차순 정렬 시 기본값) |



아래 쿼리는 sal의 오름차순으로 결과를 정렬한다. MARTIN과 WARD는 sal가 1250으로 동일하기 때문에 정렬 순서가 변경될 수 있다. ORDER BY 절에 고유한 값을 가진 표현식을 기술해야 한다.

```sql
SELECT ename, sal FROM emp WHERE deptno = 30 ORDER BY sal;

// output

ENAME                       SAL
-------------------- ----------
JAMES                       950
WARD                       1250
MARTIN                     1250
TURNER                     1500
ALLEN                      1600
BLAKE                      2850
```



아래 쿼리는 ORDER BY 절에 DESC 키워드를 기술했다. 결과가 sal의 내림차순으로 정렬된다.

```sql
SELECT ename, sal FROM emp WHERE deptno = 30 ORDER BY sal DESC;
```



아래 쿼리는 sal의 내림차순, comm의 오름차순으로 결과를 정렬한다.

```sql
SELECT ename, sal, comm FROM emp WHERE deptno = 30 ORDER BY sal DESC, comm;

// output

ENAME                       SAL       COMM
-------------------- ---------- ----------
BLAKE                      2850
ALLEN                      1600        300
TURNER                     1500          0
WARD                       1250        500
MARTIN                     1250       1400
JAMES                       950
```



아래 쿼리는 ORDER BY 절에 SELECT 절의 열 위치(column position)를 지정했다. SELECT 절이 ORDER BY 절보다 먼저 수행되기 때문에 열 위치를 참조할 수 있는 것이다. 위의 쿼리와 동일하다.

```sql
SELECT ename, sal, comm FROM emp WHERE deptno = 30 ORDER BY 2 DESC, 3;
```



아래 쿼리는 SELECT 절의 열 위치를 지정했다. 입사 연도와 sal로 결과가 정렬된다.

```sql
SELECT   ename, TO_CHAR (hiredate, 'YYYY') AS hireyear, sal 
    FROM emp
   WHERE deptno = 20
ORDER BY 2, sal;
```



위 쿼리는 아래 쿼리로 해석된다. ORDER BY 절의 열 위치는 위치에 해당하는 SELECT 절의 표현식으로 대체된다.

```sql
SELECT   ename, TO_CHAR (hiredate, 'YYYY') AS hireyear, sal 
    FROM emp
   WHERE deptno = 20
ORDER BY TO_CHAR (hiredate, 'YYYY'), sal;
```



아래 쿼리는 ORDER BY 절에 열 별칭(sal)을 사용했기 때문에 문자 값으로 결과가 정렬된다.

```sql
SELECT   ename, TO_CHAR (sal, 'FM999,999') AS sal
    FROM emp
   WHERE deptno = 20
ORDER BY sal;

// output

ENAME                SAL
-------------------- ----------------
ADAMS                1,100
JONES                2,975
FORD                 3,000
SCOTT                3,000
SMITH                800
```



ORDER BY 절의 열을 테이블로 한정하면 열 별칭이 아닌 열(emp.sal)로 결과를 정렬할 수 있다.

```sql
SELECT   ename, TO_CHAR (sal, 'FM999,999') AS sal
    FROM emp
   WHERE deptno = 20
ORDER BY emp.sal;

// output

ENAME                SAL
-------------------- ----------------
SMITH                800
ADAMS                1,100
JONES                2,975
FORD                 3,000
SCOTT                3,000
```



NULLS FIRST 키워드를 기술하면 널을 앞쪽으로 정렬할 수 있다.

```sql
SELECT ename, comm FROM emp WHERE deptno = 30 ORDER BY comm NULLS FIRST;

// output

ENAME                      COMM
-------------------- ----------
JAMES
BLAKE
TURNER                        0
ALLEN                       300
WARD                        500
MARTIN                     1400
```



## 8.2 활용 예제

(일단 생략)

















