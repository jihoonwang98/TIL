## 1. 집계 함수(Aggregate Function)

여러 행들의 그룹이 모여서 그룹당 단 하나의 결과를 돌려주는 다중행 함수 중 집계 함수(Aggregate Function)의 특성은 다음과 같다.

- 여러 행들의 그룹이 모여서 그룹당 단 하나의 결과를 돌려주는 함수이다.
- GROUP BY 절은 행들을 소그룹화 한다.
- SELECT 절, HAVING 절, ORDER BY 절에 사용할 수 있다.



ANSI/ISO에서 데이터 분석 기능으로 분류한 함수 중 기본적인 집계 함수는 본 절에서 설명하고, ROLLUP, CUBE, GROUPING SETS 같은 GROUP 함수는 2장 5절에서, 다양한 분석 기능을 가진 WINDOW 함수는 2장 6절에서 설명한다.



```sql
집계함수명 ([DISTINCT | ALL] 칼럼이나 표현식)

-- ALL: Default 옵션이므로 생략 가능함
-- DISTINCT: 같은 값을 하나의 데이터로 간주할 때 사용하는 옵션임
```



자주 사용되는 주요 집계 함수들은 다음과 같다. 집계 함수는 그룹에 대한 정보를 제공하므로 주로 숫자 유형에 사용되지만, MAX, MIN, COUNT 함수는 문자, 날짜 유형에도 적용 가능하다.



**집계 함수의 종류**

| 집계 함수                        | 사용 목적                                          |
| -------------------------------- | -------------------------------------------------- |
| COUNT(*)                         | NULL 값을 포함한 행의 수를 출력                    |
| COUNT(표현식)                    | 표현식의 값이 NULL 값인 것을 제외한 행의 수를 출력 |
| SUM([DISTINCT \| ALL] 표현식)    | 표현식의 NULL 값을 제외한 합계를 출력              |
| AVG([DISTINCT \| ALL] 표현식)    | 표현식의 NULL 값을 제외한 평균을 출력              |
| MAX([DISTINCT \| ALL] 표현식)    | 표현식의 최대값을 출력                             |
| MIN([DISTINCT \| ALL] 표현식)    | 표현식의 최소값을 출력                             |
| STDDEV([DISTINCT \| ALL] 표현식) | 표현식의 표준 편차를 출력                          |
| VARIAN([DISTINCT \| ALL] 표현식) | 표현식의 분산을 출력                               |
| 기타 통계 함수                   | 벤더별로 다양한 통계식을 제공한다.                 |







## 2. GROUP BY 절

GROUP BY 절은 SQL 문에서 FROM 절과 WHERE 절 뒤에 오며, 데이터들을 작은 그룹으로 분류하여 소그룹에 대한 항목별로 통계 정보를 얻을 때 추가로 사용된다.

```sql
SELECT [DISTINCT] 칼럼명 [ALIAS명]
FROM 테이블명
[WHERE 조건식]
[GROUP BY 칼럼이나 표현식]
[HAVING 그룹조건식];
```



GROUP BY 절과 HAVING 절은 다음과 같은 특성을 가진다.

- GROUP BY 절을 통해 소그룹별 기준을 정한 후, SELECT 절에 집계 함수를 사용한다.
- 집계 함수의 통계 정보는 NULL 값을 가진 행을 제외하고 수행한다.
- GROUP BY 절에서는 SELECT 절과는 달리 ALIAS 명을 사용할 수 없다.
- 집계 함수는 WHERE 절에 올 수 없다. (집계 함수를 사용할 수 있는 GROUP BY 절보다 WHERE 절이 먼저 수행된다)
- WHERE 절은 전체 데이터를 GROUP으로 나누기 전에 행들을 미리 제거시킨다.
- HAVING 절은 GROUP BY 절의 기준 항목이나 소그룹의 집계 함수를 이용한 조건을 표시할 수 있다.
- GROUP BY 절에 의한 소그룹별로 만들어진 집계 데이터 중, HAVING 절에서 제한 조건을 두어 조건을 만족하는 내용만 출력한다.



## 3. HAVING 절





## 4. CASE 표현을 활용한 월별 데이터 집계





## 5. 집계 함수와 NULL

리포트의 빈칸을 NULL이 아닌 ZERO로 표현하기 위해 NVL(Oracle) / ISNULL(SQL Server) 함수를 사용하는 경우가 많은데, 다중 행 함수를 사용하는 경우는 오히려 불필요한 부하가 발생하므로 굳이 NVL 함수를 다중 행 함수 안에 사용할 필요가 없다.



다중 행 함수는 입력 값으로 전체 건수가 NULL 값인 경우만 함수의 결과가 NULL이 나오고 전체 건수 중에서 일부만 NULL인 경우는 NULL인 행을 다중 행 함수의 대상에서 제외한다. 예를 들면 100명 중 10명의 성적이 NULL 값일 때 평균을 구하는 다중 행 함수 AVG를 사용하면 NULL 값이 아닌 90명의 성적에 대해서 평균값을 구하게 된다.



CASE 표현 사용시 ELSE 절을 생략하면 Default 값이 NULL이다. NULL은 연산의 대상이 아닌 반면, SUM(CASE MONTH WHEN 1 THEN SAL ELSE 0 END)처럼 ELSE 절에서 0을 지정하면 불필요하게 0이 SUM 연산에 사용되므로 자원의 사용이 많아진다. 같은 결과를 얻을 수 있다면 가능한 ELSE 절의 상수값을 지정하지 않거나 ELSE 절을 작성하지 않도록 한다.



많이 실수하는 것 중 하나가 Oracle의 SUM( NVL( SAL, 0 )), SQL Server의 SUM( ISNULL( SAL, 0 )) 연산이다. 개별 데이터의 급여(SAL)가 NULL인 경우는 NULL의 특성으로 자동적으로 SUM 연산에서 빠지는데, 불필요하게 NVL/ISNULL 함수를 사용해 0으로 변환시켜 데이터 건수만큼의 연산이 일어나게 하는 것은 시스템의 자원을 낭비하는 일이다.







