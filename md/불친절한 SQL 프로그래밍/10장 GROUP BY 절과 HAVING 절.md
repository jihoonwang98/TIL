# 10장 GROUP BY 절과 HAVING 절

GROUP BY 절은 행 그룹(groups of rows)을 생성하고, HAVING 절은 조회할 행 그룹을 선택한다.



GROUP BY 절은 WHERE 절 다음에 기술하며, WHERE 절이 수행된 후 수행된다. HAVING 절은 GROUP BY 절 다음에 기술하며, GROUP BY 절이 수행된 후 수행된다.

```sql
SELECT   절 -- (5)
    FROM 절 -- (1)
   WHERE 절 -- (2)
GROUP BY 절 -- (3)
  HAVING 절 -- (4)
ORDER BY 절 -- (6)
```



## 10.1 GROUP BY 절

GROUP BY 절은 expr로 행 그룹을 생성하고, 생성된 행 그룹을 하나의 행으로 그룹핑(grouping)한다.

```sql
GROUP BY expr [, expr]...
```



아래의 데이터로 예제를 수행하자.

```sql
SELECT deptno, job, hiredate, sal FROM emp WHERE sal > 2000 ORDER BY 1, 2, 3;

// output

    DEPTNO JOB                HIREDATE        SAL
---------- ------------------ -------- ----------
        10 MANAGER            81/06/09       2450
        10 PRESIDENT          81/11/17       5000
        20 ANALYST            81/12/03       3000
        20 ANALYST            87/04/19       3000
        20 MANAGER            81/04/02       2975
        30 MANAGER            81/05/01       2850
```



아래 쿼리는 하나의 행이 반환된다. GROUP BY 절을 기술하지 않거나 GROUP BY 절에 NULL이나 ()를 기술하면 전체 행이 하나의 행 그룹으로 처리된다.

```sql
SELECT SUM (sal) AS c1 FROM emp WHERE sal > 2000 GROUP BY ();

C1
--------------------
19275
```



아래 쿼리는 deptno별 sal의 합계 값을 집계한다.

```sql
SELECT deptno, SUM (sal) AS sal FROM emp WHERE sal > 2000 GROUP BY deptno ORDER BY 1;

// output

    DEPTNO        SAL
---------- ----------
        10       7450
        20       8975
        30       2850
```



위 쿼리는 아래의 행 그룹으로 그룹핑된다.

| DEPTNO | JOB       | SAL  | 행 그룹 | C1   |
| ------ | --------- | ---- | ------- | ---- |
| 10     | MANAGER   | 2450 | 10      | 7450 |
| 10     | PRESIDENT | 5000 | 10      | 7450 |
| 20     | ANALYST   | 3000 | 20      | 8975 |
| 20     | ANALYST   | 3000 | 20      | 8975 |
| 20     | MANAGER   | 2975 | 20      | 8975 |
| 30     | MANAGER   | 2850 | 30      | 2850 |



아래 쿼리는 deptno, job별 sal의 합계 값을 집계한다.

```sql
SELECT   deptno, job, SUM (sal) AS sal
    FROM emp
   WHERE sal > 2000
GROUP BY deptno, job
ORDER BY 1, 2;

// output

    DEPTNO JOB                       SAL
---------- ------------------ ----------
        10 MANAGER                  2450
        10 PRESIDENT                5000
        20 ANALYST                  6000
        20 MANAGER                  2975
        30 MANAGER                  2850
```



아래 쿼리는 에러가 발생한다. GROUP BY 절을 사용한 쿼리는 SELECT 절과 ORDER BY 절에 GROUP BY 절의 표현식이나 집계 함수를 사용한 표현식만 기술할 수 있다. 그렇지 않으면 결과 값을 결정할 수 없기 때문에 에러가 발생한다.

```sql
SELECT   deptno
       , sal
    FROM emp
   WHERE sal > 2000
GROUP BY deptno;

SELECT   deptno
    FROM emp
   WHERE sal > 2000
GROUP BY deptno
ORDER BY sal;
```



아래와 같이 sal 열에 집계 함수를 사용하면 에러가 발생하지 않는다.

```sql
SELECT   deptno
       , SUM (sal) AS sal
    FROM emp
   WHERE sal > 2000
GROUP BY deptno;

SELECT   deptno
    FROM emp
   WHERE sal > 2000
GROUP BY deptno
ORDER BY SUM (sal);
```



GROUP BY 절에 기술된 표현식(deptno)은 행 그룹으로 그룹핑되기 때문에 단일 행으로 처리된다. 아래 쿼리는 deptno 열에 DECODE 함수를 사용했다.

```sql
SELECT   DECODE (deptno, 10, 'ACCOUNTING', 20, 'RESEARCH', 30, 'SALES') AS dname
       , SUM (sal) AS sal
    FROM emp
   WHERE sal > 2000
GROUP BY deptno
ORDER BY 1;

// output

DNAME                       SAL
-------------------- ----------
ACCOUNTING                 7450
RESEARCH                   8975
SALES                      2850
```



집계 함수를 사용한 표현식도 단일 행으로 처리된다. 아래 쿼리는 SUM 함수의 결과 값에 GREATEST 함수를 사용했다.

```sql
SELECT   deptno, GREATEST (SUM (sal), 5000) AS sal
    FROM emp
   WHERE sal > 2000
GROUP BY deptno
ORDER BY 1;

// output

    DEPTNO        SAL
---------- ----------
        10       7450
        20       8975
        30       5000
```



































