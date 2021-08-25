## 1. WINDOW FUNCTION 개요

**WINDOW FUNCTION의 종류**

WINDOW FUNCTION의 종류는 크게 다섯 개의 그룹으로 분류할 수 있는데 벤더별로 지원하는 함수에는 차이가 있다.



- **그룹 내 순위(RANK) 관련 함수**
  - RANK, DENSE_RANK, ROW_NUMBER
- **그룹 내 집계(AGGREGATE) 관련 함수**
  - SUM, MAX, MIN, AVG, COUNT
- **그룹 내 행 순서 관련 함수**
  - FIRST_VALUE, LAST_VALUE, LAG, LEAD
- **그룹 내 비율 관련 함수**
  - CUME_DIST, PERCENT_RANK, NTILE, RATIO_TO_REPORT
- **선형 분석을 포함한 통계 분석 관련 함수**





**WINDOW FUNCTION SYNTAX**

WINDOW 함수에는 **OVER** 문구가 키워드로 필수 포함된다.

```sql
SELECT WINDOW_FUNCTION (ARGUMENTS) OVER ([PARTITION BY 칼럼] [ORDER BY 절] [WINDOWING 절]) FROM 테이블명
```

- ARGUMENTS: 함수에 따라 0~N 개의 인수가 지정될 수 있다.

  



## 2. 그룹 내 순위 함수

**가. RANK 함수**

RANK 함수는 ORDER BY를 포함한 QUERY 문에서 특정 항목(칼럼)에 대한 순위를 구하는 함수이다. 이때 특정 범위(PARTITION) 내에서 순위를 구할 수도 있고, 전체 데이터에 대한 순위를 구할 수도 있다. 또한 동일한 값에 대해서는 동일한 순위를 부여하게 된다.



**사원 데이터에서 급여가 높은 순서와 JOB 별로 급여가 높은 순서를 함께 출력**

```sql
SELECT job, ename, sal, 
       RANK () OVER (ORDER BY sal DESC) ALL_RANK, 
       RANK () OVER (PARTITION BY job ORDER BY sal DESC) JOB_RANK
FROM emp;
```





**나. DENSE_RANK 함수**

DENSE_RANK 함수는 RANK와 흡사하나, 동일한 순위를 하나의 건수로 취급한다.



```sql
SELECT job, ename, sal, 
       RANK () OVER (ORDER BY sal DESC) RANK,
       DENSE_RANK () OVER (ORDER BY sal DESC) DENSE_RANK
FROM emp;
```





**다. ROW_NUMBER 함수**

ROW_NUMBER 함수는 RANK나 DENSE_RANK 함수가 동일한 값에 대해서는 동일한 순위를 부여하는데 반해, 동일한 값이라도 고유한 순위를 부여한다.



```sql
SELECT job, ename, sal,
       RANK () OVER (ORDER BY sal DESC) RANK,
       ROW_NUMBER () OVER (ORDER BY sal DESC) ROW_NUMBER
FROM emp;
```





## 3. 일반 집계 함수

**가. SUM 함수**

**사원들의 급여와 같은 매니저를 두고 있는 사원들의 SALARY 합을 구한다.**

```sql
SELECT mgr, ename, sal, SUM(sal) OVER (PARTITION BY mgr) MGR_SUM
FROM emp;
```



**OVER 절 내에 ORDER BY 절을 추가해 파티션 내 데이터를 정렬하고 이전 SALARY 데이터까지의 누적값을 출력한다.**

```sql
SELECT mgr, ename, sal, 
       SUM(sal) OVER (PARTITION BY mgr ORDER BY sal RANGE UNBOUNDED PRECEDING) as MGR_SUM
FROM emp;
```





**나. MAX 함수**

**사원들의 급여와 같은 매니저를 두고 있는 사원들의 SALARY 중 최대값을 같이 구한다.**

```sql
SELECT mgr, ename, sal, MAX(sal) OVER (PARTITION BY mgr) AS MGR_MAX FROM emp;
```



**INLINE VIEW를 이용해 파티션별 최대값을 가진 행만 추출**

```sql
SELECT mgr, ename, sal 
FROM (SELECT mgr, ename, sal, MAX(sal) OVER (PARTITION BY mgr) as IV_MAX_SAL FROM emp)
WHERE sal = IV_MAX_SAL;
```



**다. MIN 함수**

**사원들의 급여와 같은 매니저를 두고 있는 사원들을 입사일자를 기준으로 정렬하고, SALARY 최소값을 같이 구한다.**

```sql
SELECT mgr, ename, hiredate, sal, 
       MIN(sal) OVER (PARTITION BY mgr ORDER BY hiredate) as MGR_MIN 
FROM emp;
```



**라. AVG 함수**

**EMP 테이블에서 같은 매니저를 두고 있는 사원들의 평균 SALARY를 구하는데, 조건은 같은 매니저 내에서 자기 바로 앞의 사번과 바로 뒤의 사번인 직원만을 대상으로 한다.**

```sql
SELECT mgr, ename, hiredate, sal, 
       ROUND(AVG(sal) OVER (PARTITION BY mgr 
                            ORDER BY hiredate 
                            ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING)) AS MGR_AVG
FROM emp;
```





**마. COUNT 함수**

**사원들을 급여 기준으로 정렬하고, 본인의 급여보다 50 이하가 적거나 150 이하로 많은 급여를 받는 인원수를 출력하라.**

```sql
SELECT ename, sal, 
       COUNT(*) OVER (ORDER BY SAL 
                      RANGE BETWEEN 50 PRECEDING AND 150 FOLLOWING) AS SIM_CNT
FROM emp;
```





## 4. 그룹 내 행 순서 함수

**가. FIRST_VALUE 함수**

**부서별 직원들을 연봉이 높은 순서부터 정렬하고, 파티션 내에서 가장 먼저 나온 값을 출력한다.**

```sql
SELECT deptno, ename, sal, 
       FIRST_VALUE(ename) OVER (PARTITION BY deptno 
                                ORDER BY sal DESC
                                ROWS UNBOUNDED PRECEDING) as DEPT_RICH
FROM emp;
```



**나. LAST_VALUE 함수**

**부서별 직원들을 연봉이 높은 순서부터 정렬하고, 파티션 내에서 가장 마지막에 나온 값을 출력한다.**

```sql
SELECT deptno, ename, sal, 
       LAST_VALUE(ename) OVER (PARTITION BY deptno 
                               ORDER BY sal DESC
                               ROWS BETWEEN CURRENT ROW AND UNBOUNDED FOLLOWING) as DEPT_POOR
FROM emp;
```



**다. LAG 함수**

LAG 함수를 이용해 파티션별 윈도우에서 이전 몇 번째 행의 값을 가져올 수 있다.

```sql
SELECT ename, hiredate, sal, 
       LAG(sal) OVER (ORDER BY hiredate) AS PREV_SAL 
FROM emp
WHERE job='SALESMAN';
```



LAG 함수는 3개의 ARGUMENTS 까지 사용할 수 있는데, 두 번째 인자는 몇 번째 앞의 행을 가져올지 결정하는 것이고 (DEFAULT 1), 세 번째 인자는 예를 들어 파티션의 첫 번째 행의 경우 가져올 데이터가 없어 NULL 값이 들어오는데 이 경우 다른 값으로 바꾸어 줄 수 있다. 결과적으로 NVL이나 ISNULL 기능과 같다.



**라. LEAD 함수**

LEAD 함수를 이용해 파티션별 윈도우에서 이후 몇 번째 행의 값을 가져올 수 있다.



```sql
SELECT ename, hiredate, 
       LEAD(hiredate, 1) OVER (ORDER BY hiredate) AS "NEXTHIRED"
FROM emp;
```



LEAD 함수는 3개의 ARGUMENTS 까지 사용할 수 있는데, 두 번째 인자는 몇 번째 후의 행을 가져올지 결정하는 것이고 (DEFAULT 1), 세 번째 인자는 예를 들어 파티션의 마지막행의 경우 가져올 데이터가 없어 NULL 값이 들어오는데 이 경우 다른 값으로 바꾸어 줄 수 있다. 결과적으로 NVL이나 ISNULL 기능과 같다.





## 5. 그룹 내 비율 함수

**가. RATIO_TO_REPORT 함수**

RATIO_TO_REPORT 함수를 이용해 파티션 내에 전체 SUM(칼럼)값에 대한 행별 칼럼 값의 



















**나. PERCENT_RANK 함수**

**다. CUME_DIST 함수**

**라. NTILE 함수**