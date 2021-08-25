# 15장 Top-N 쿼리

Top-N 쿼리는 말 그대로 상위(Top) N개의 행을 조회하는 쿼리다. 



## 15.1 기본 문법

오라클 DB는 세 가지 방식의 Top-N 쿼리를 사용할 수 있다. ROWNUM 방식, 분석 함수 방식, ROW LIMITING 절을 차례대로 살펴보자.



### 15.1.1 ROWNUM 방식

ROWNUM 방식은 오라클 DB의 전통적인 Top-N 쿼리 방식이다. ORDER BY 절로 행을 정렬하고, 정렬된 행을 ROWNUM 슈도 칼럼으로 제한한다.



ROWNUM 슈도 칼럼은 행이 반환되는 순서대로 순번을 반환한다. 1부터 시작하고 행이 반환될 때마다 순번이 증가한다.

```sql
SELECT empno, sal, ROWNUM AS rn FROM emp;
```



아래 쿼리는 결과가 반환되지 않는다. ROWNUM 슈도 칼럼은 1부터 시작하고 행이 반환될 때마다 순번이 증가하기 때문에 ROWNUM = 2 조건은 항상 FALSE다.

```sql
SELECT empno, sal, ROWNUM AS rn FROM emp WHERE ROWNUM = 2;
```



ROWNUM 슈도 칼럼은 < 조건이나 <= 조건을 사용해야 한다.

```sql
SELECT empno, sal, ROWNUM AS rn FROM emp WHERE ROWNUM <= 2;
```



sal를 오름차순으로 정렬하여 상위 5개의 행을 조회해보자. 아래 쿼리는 WHERE 절로 5개의 행을 무작위로 제한한 후, ORDER BY 절로 행을 정렬했기 때문에 의도한 결과가 반환되지 않는다.

```sql
SELECT empno, sal, ROWNUM AS rn FROM emp WHERE ROWNUM <= 5 ORDER BY sal;
```



의도한 결과를 얻기 위해서는 아래와 같이 인라인 뷰를 사용해야 한다. 아래 쿼리는 인라인 뷰에 의해 ORDER BY 절이 수행된 후 WHERE 절이 수행된다. 아래 쿼리가 ROWNUM 방식의 Top-N 쿼리다.

```sql
SELECT empno, sal, ROWNUM AS rn
  FROM (SELECT empno, sal FROM emp ORDER BY sal, empno)
 WHERE ROWNUM <= 5;
```



아래 쿼리는 상위 6행에서 10행까지의 행을 조회한다. 한 페이지(v_pr)를 5행으로 보면 두 번째 페이지(v_pn)를 조회한 것이다. 이런 쿼리를 페이징 쿼리(pagination query)라고 한다.

```sql
VAR v_pr NUMBER = 5;
VAR v_pn NUMBER = 2;

SELECT empno, sal, rn
  FROM (SELECT empno, sal, ROWNUM AS rn
          FROM (SELECT empno, sal FROM emp ORDER BY sal, empno)
         WHERE ROWNUM <= :v_pr * :v_pn)
 WHERE rn >= (:v_pr * (:v_pn - 1)) + 1;
```



아래 쿼리는 위 쿼리와 결과가 동일하지만 쿼리의 성능이 저하될 수 있다. ROWNUM 방식의 Top-N 쿼리는 ROWNUM 슈도 칼럼을 WHERE 절에 직접 기술해야 한다.

```sql
SELECT empno, sal, rn
  FROM (SELECT empno, sal, ROWNUM AS rn
          FROM (SELECT empno, sal FROM emp ORDER BY sal, empno))
 WHERE rn BETWEEN (:v_pr * (:v_pn - 1)) + 1 AND :v_pr * :v_pn;
```



경품 추첨 등 무작위로 n개의 행을 조회해야 하는 경우 ORDER BY 절에 DBMS_RANDOM.VALUE 함수를 사용할 수 있다. DBMS_RANDOM.VALUE 함수는 32장에서 살펴보자.

```sql
SELECT empno, sal
  FROM (SELECT empno, sal FROM emp ORDER BY DBMS_RANDOM.VALUE)
 WHERE ROWNUM <= 3;
```



(중략)



### 15.1.2 분석 함수 방식

분석 함수 방식은 14장에서 살펴본 순위 분석 함수를 사용한다.



아래는 ROW_NUMBER 함수를 사용한 Top-N 쿼리다.

```sql
SELECT   *
    FROM (SELECT empno, sal
               , ROW_NUMBER () OVER (ORDER BY sal, empno) AS rn
            FROM emp)
   WHERE rn <= 5
ORDER BY sal, empno;
```



아래는 ROW_NUMBER 함수를 사용한 페이징 쿼리다. ROWNUM 방식에 비해 간결하게 작성할 수 있다.

```sql
SELECT   *
    FROM (SELECT empno, sal
               , ROW_NUMBER () OVER (ORDER BY sal, empno) AS rn
            FROM emp)
   WHERE rn BETWEEN (:v_pr * (:v_pn - 1)) + 1 AND :v_pr * :v_pn
ORDER BY sal, empno;
```



전체 건수가 필요한 경우 아래와 같이 쿼리를 작성할 수 있다. COUNT 분석 함수로 전체 행을 조회하기 때문에 쿼리의 성능이 저하될 수 있다. 성능을 개선하기 위해 첫 페이지 로딩 시에만 전체 건수를 조회하는 쿼리를 별도로 수행하기도 한다.

```sql
SELECT   empno, sal, rn, cn
    FROM (SELECT empno, sal
               , COUNT (*) OVER () AS cn
               , ROW_NUMBER () OVER (ORDER BY sal, empno) AS rn
            FROM emp)
   WHERE CEIL (rn / :v_pr) = :v_pn
ORDER BY sal, empno;
```



PERCENT_RANK 함수를 사용하면 백분율에 의한 Top-N 쿼리를 작성할 수 있다. 아래 쿼리는 상위 25%의 행을 반환한다. 위 쿼리와 동일한 이유로 쿼리의 성능이 저하될 수 있다.

```sql
SELECT   empno, sal, pr
    FROM (SELECT empno, sal
               , PERCENT_RANK () OVER (ORDER BY sal, empno) AS pr
            FROM emp)
   WHERE pr <= 0.25
ORDER BY sal, empno;
```



### 15.1.3 ROW LIMITING 절

12.1 버전부터 ROW LIMITING 절로 Top-N 쿼리를 작성할 수 있다. ROW LIMITING 절은 ANSI 표준 SQL 문법이다.



아래는 ROW LIMITING 절의 구문이다. ROW LIMITING 절은 ORDER BY 절 다음에 기술하며, ORDER BY 절과 함께 수행된다. ROW와 ROWS는 구분하지 않아도 된다.

```sql
[OFFSET offset {ROW | ROWS}]
[FETCH {FIRST | NEXT} [{rowcount | percent PERCENT}] {ROW | ROWS} {ONLY | WITH TIES}]
```



| 항목          | 설명                                    |
| ------------- | --------------------------------------- |
| OFFSET offset | 건너뛸 행의 개수를 지정                 |
| FETCH         | 반환할 행의 개수나 백분율을 지정        |
| ONLY          | 지정된 행의 개수나 백분율만큼 행을 반환 |
| WITH TIES     | 마지막 행에 대한 동순위를 포함해서 반환 |



아래는 ROW LIMITING 절을 사용한 Top-N 쿼리다.

```sql
SELECT empno, sal FROM emp ORDER BY sal, empno FETCH FIRST 5 ROWS ONLY;
```



아래는 ROW LIMITING 절을 사용한 페이징 쿼리다. 

```sql
SELECT   empno, sal
    FROM emp
ORDER BY sal, empno
  OFFSET :v_pr * (:v_pn - 1) ROWS FETCH NEXT :v_pr ROWS ONLY;
```



아래와 같이 OFFSET만 기술하면 건너뛴 행 이후의 전체 행이 반환된다.

```sql
SELECT empno, sal FROM emp ORDER BY sal, empno OFFSET 5 ROWS;
```



PERCENT 키워드를 사용하면 반환할 행의 백분율을 지정할 수 있다. 아래 쿼리는 상위 25%의 행을 반환한다.

```sql
SELECT empno, sal FROM emp ORDER BY sal, empno FETCH FIRST 25 PERCENT ROWS ONLY;
```



WITH TIES 키워드를 사용하면 동순위를 처리할 수 있다. 아래 쿼리는 RANK 함수를 사용한 Top-N 쿼리와 동일하게 동작한다.

```sql
SELECT empno, sal FROM emp ORDER BY sal FETCH FIRST 6 ROWS WITH TIES;
```



## 15.2 고급 주제

Top-N 쿼리에 대한 고급 주제를 살펴보자.

(나중에...)















