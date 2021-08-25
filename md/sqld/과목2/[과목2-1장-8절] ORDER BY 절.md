## 1. ORDER BY 정렬

**ORDER BY 절**은 SQL 문장으로 조회된 데이터들을 다양한 목적에 맞게 특정 칼럼을 기준으로 **정렬**하여 출력하는데 사용한다.



ORDER BY 절에 칼럼(Column)명 대신에 SELECT 절에서 사용한 **ALIAS 명**이나 칼럼 순서를 나타내는 **정수**도 사용 가능하다. 그리고 별도로 정렬 방식을 지정하지 않으면 기본적으로 오름차순이 적용되며, SQL 문장의 제일 마지막에 위치한다.



```sql
SELECT 칼럼명 [ALIAS명]
FROM 테이블명
[WHERE 조건식]
[GROUP BY 칼럼(Column)이나 표현식]
[HAVING 그룹조건식]
[ORDER BY 칼럼(Column)이나 표현식 [ASC 또는 DESC]];
```





**ORDER BY 절 사용 특징**은 다음과 같다.

- 기본적인 정렬 순서는 오름차순(ASC)이다.
- Oracle에서는 NULL 값을 가장 큰 값으로 간주하여 오름차순으로 정렬했을 경우 가장 마지막에 위치한다.
- 반면, SQL Server에서는 NULL 값을 가장 작은 값으로 간주하여 오름차순으로 정렬했을 경우 가장 먼저 위치한다.
- ORDER BY 절에서 칼럼명, ALIAS명, 칼럼 순서를 같이 혼용하는 것도 가능하다.







## 2. SELECT 문장 실행 순서

GROUP BY 절과 ORDER BY가 같이 사용될 때 SELECT 문장은 6개의 절로 구성이 되고, SELECT 문장의 수행 단계는 아래와 같다.

```sql
5. SELECT 칼럼명 [ALIAS명]
1. FROM 테이블명
2. WHERE 조건식
3. GROUP BY 칼럼(Column)이나 표현식
4. HAVING 그룹조건식
6. ORDER BY 칼럼(Column)이나 표현식;
```







## 3. Top N 쿼리

- ROWNUM

  Oracle에서 순위가 높은 N개의 row를 추출하기 위해 ORDER BY 절과 WHERE 절의 ROWNUM 조건을 같이 사용하는 경우가 있는데 이 두 조건으로는 원하는 결과를 얻을 수 없다. Oracle의 경우 정렬이 완료된 후 데이터의 일부가 출력되는 것이 아니라, 데이터의 일부가 먼저 추출된 후 데이터에 대한 정렬 작업이 일어난다.



```sql
SELECT ename, sal 
FROM emp
WHERE ROWNUM < 4
ORDER BY sal DESC;

// 정렬X
```



```sql
SELECT ename, sal
FROM (SELECT ename, sal
      FROM emp
      ORDER BY sal DESC)
WHERE ROWNUM < 4;

// 정렬 O
```





- TOP ()

  반면 SQL Server는 TOP 조건을 사용하게 되면 별도 처리 없이 관련 ORDER BY 절의 데이터 정렬 후 원하는 일부 데이터만 쉽게 출력할 수 있다. 



```sql
TOP (Expression) [PERCENT] [WITH TIES]
```



TOP 절을 사용하여 결과 집합으로 반환되는 행 수를 제한할 수 있다. WITH TIES 옵션은 ORDER BY 절의 조건 기준으로 TOP N의 마지막 행으로 표시되는 추가 행의 데이터가 같을 경우 N+ 동일 정렬 순서 데이터를 추가 반환하도록 지정하는 옵션이다.



**[사원 테이블에서 급여가 높은 2명을 내림차순 출력]**

```sql
SELECT TOP(2) ename, sal
FROM emp
ORDER BY sal DESC;
```