# 17장 PIVOT 절과 UNPIVOT 절

PIVOT은 회전시킨다는 의미를 가지고 있다. PIVOT 절은 행을 열로 회전시키고, UNPIVOT 절은 열을 행으로 회전시킨다.



## 17.1 PIVOT 절

PIVOT 절은 행을 열로 전환한다. 



### 17.1.1 기본 문법

PIVOT 절의 구문은 아래와 같다.

```sql
PIVOT [XML]
       (aggregate_function (expr) [[AS] alias]
     [, aggregate_function (expr) [[AS] alias]]...
        FOR {column | (column [, column] ...)}
        IN ({{{expr | (expr [, expr]...)} [[AS] alias]}...
            | subquery
            | ANY [, ANY]...
            })
       )
```



| 항목               | 설명                 |
| ------------------ | -------------------- |
| aggregate_function | 집계할 열을 지정     |
| FOR 절             | PIVOT할 열을 지정    |
| IN 절              | PIVOT할 열 값을 지정 |



아래는 PIVOT 절을 사용한 쿼리다. PIVOT 절은 집계 함수와 FOR 절에 지정되지 않은 열을 기준으로 집계되기 때문에 인라인 뷰를 통해 사용할 열을 지정해야 한다.

```sql
SELECT   *
    FROM (SELECT job, deptno, sal FROM emp)
   PIVOT (SUM (sal) FOR deptno IN (10, 20, 30))
ORDER BY 1;
```



아래 쿼리는 인라인 뷰에 yyyy 표현식을 추가했다. 행 그룹에 yyyy 표현식이 추가된 것을 확인할 수 있다.

```sql
SELECT   *
    FROM (SELECT TO_CHAR (hiredate, 'YYYY') AS yyyy, job, deptno, sal FROM emp)
   PIVOT (SUM (sal) FOR deptno IN (10, 20, 30))
ORDER BY 1, 2;
```



아래 쿼리는 집계 함수와 IN 절에 별칭을 지정했다. 별칭을 지정하면 결과 집합의 열명이 변경된다.

```sql
SELECT   *
    FROM (SELECT job, deptno, sal FROM emp)
   PIVOT (SUM (sal) AS sal FOR deptno IN (10 AS d10, 20 AS d20, 30 AS d30))
ORDER BY 1;
```



집계 함수와 IN 절에 지정한 별칭에 따라 아래와 같은 규칙으로 열명이 부여된다. 집계 함수와 IN 절 모두 별칭을 지정하는 편이 바람직하다.

|                      | **10** | **10 AS d10** |
| -------------------- | ------ | ------------- |
| **SUM (sal)**        | 10     | D10           |
| **SUM (sal) AS sal** | 10_SAL | D10_SAL       |



PIVOT 절은 다수의 집계 함수를 지원한다. 아래 쿼리는 SUM 함수와 COUNT 함수를 함께 사용했다.

```sql
SELECT   *
    FROM (SELECT job, deptno, sal FROM emp)
   PIVOT (SUM (sal) AS sal, COUNT (*) AS cnt FOR deptno IN (10 AS d10, 20 AS d20))
ORDER BY 1;
```



FOR 절에도 다수의 열을 기술할 수 있다. 아래와 같이 IN 절에 다중 열을 사용해야 한다.

```sql
SELECT   *
    FROM (SELECT TO_CHAR (hiredate, 'YYYY') AS yyyy, job, deptno, sal FROM emp)
   PIVOT (SUM (sal) AS sal, COUNT (*) AS cnt
          FOR (deptno, job) IN ((10, 'ANALYST') AS d10a, (10, 'CLERK') AS d10c
                              , (20, 'ANALYST') AS d20a, (20, 'CLERK') AS d20c))
ORDER BY 1;
```





## 17.2 UNPIVOT 절

UNPIVOT 절은 PIVOT 절과 반대로 동작한다. 열이 행으로 전환된다.























