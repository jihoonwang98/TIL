## 1. 데이터 분석 개요

ANSI/ISO SQL 표준은 데이터 분석을 위해서 다음 세 가지 함수를 정의하고 있다.

- **AGGREGATE FUNCTION(집계 함수)**

  GROUP AGGREGATE FUNCTION이라고도 부르며, GROUP FUNCTION의 한 부분으로 분류할 수 있다. 1장 7절에서 설명한 **COUNT, SUM, AVG, MAX, MIN** 외 각종 집계 함수들이 포함되어 있다.



- **GROUP FUNCTION(그룹 함수)**

  소계, 중계, 합계, 총 합계 등 여러 레벨의 결산 보고서를 만드는 데 도움을 주는 함수들이다. 그룹 함수로는 **ROLL UP, CUBE, GROUPING SETS** 함수가 있다.



- **WINDOW FUNCTION(윈도우 함수)**



## 2. ROLLUP 함수

ROLLUP에 지정된 Grouping Columns의 List는 Subtotal을 생성하기 위해 사용되어지며, Grouping Columns의 수를 N이라고 했을 때 N+1 Level의 Subtotal이 생성된다. 중요한 것은, ROLLUP의 인수는 계층 구조이므로 인수 순서가 바뀌면 수행 결과가 바뀌게 되므로 인수의 순서에도 주의해야 한다.  ROLLUP과 CUBE의 효과를 알아보기 위해 단계별로 데이터를 출력해보자.



**STEP 1. GROUP BY 절 + ORDER BY 절 사용**

```sql
SELECT dname, job, COUNT(*) "Total empl", SUM(sal) "Total sal" 
FROM emp, dept
WHERE dept.deptno = emp.deptno
GROUP BY dname, job
ORDER BY dname, job;

DNAME                        JOB                Total empl  Total sal
---------------------------- ------------------ ---------- ----------
ACCOUNTING                   CLERK                       1       1300
ACCOUNTING                   MANAGER                     1       2450
ACCOUNTING                   PRESIDENT                   1       5000
RESEARCH                     ANALYST                     2       6000
RESEARCH                     CLERK                       2       1900
RESEARCH                     MANAGER                     1       2975
SALES                        CLERK                       1        950
SALES                        MANAGER                     1       2850
SALES                        SALESMAN                    4       5600
```



**STEP 2. ROLLUP 함수 사용 + ORDER BY 절 사용**

```sql
SELECT dname, job, COUNT(*) "Total empl", SUM(sal) "Total sal" 
FROM emp, dept
WHERE dept.deptno = emp.deptno 
GROUP BY ROLLUP(dname, job)
ORDER BY dname, job;

DNAME                        JOB                Total empl  Total sal
---------------------------- ------------------ ---------- ----------
ACCOUNTING                   CLERK                       1       1300
ACCOUNTING                   MANAGER                     1       2450
ACCOUNTING                   PRESIDENT                   1       5000
ACCOUNTING                                               3       8750
RESEARCH                     ANALYST                     2       6000
RESEARCH                     CLERK                       2       1900
RESEARCH                     MANAGER                     1       2975
RESEARCH                                                 5      10875
SALES                        CLERK                       1        950
SALES                        MANAGER                     1       2850
SALES                        SALESMAN                    4       5600
SALES                                                    6       9400
                                                        14      29025
```



실행 결과에서 2개의 GROUPING COLUMNS(dname, job)에 대하여 다음과 같은 추가 LEVEL의 집계가 생성된 것을 볼 수 있다.



- L1 - GROUP BY 수행시 생성되는 표준 집계 (9건)
- L2 - DNAME 별 모든 JOB의 SUBTOTAL (3건)
- L3 - GRAND TOTAL (마지막 행, 1건)



추가로 ROLLUP의 경우 계층 간 집계에 대해서는 LEVEL 별 순서(L1 -> L2 -> L3)를 정렬하지만, 계층 내 GROUP BY 수행시 생성되는 표준 집계에는 별도의 정렬을 지원하지 않는다. L1, L2, L3 계층 내 정렬을 위해서는 별도의 ORDER BY 절을 사용해야 한다.



**STEP 3. GROUPING 함수 사용**

ROLLUP, CUBE, GROUPING SETS 등 새로운 그룹 함수를 지원하기 위해 GROUPING 함수가 추가되었다.



- ROLLUP이나 CUBE에 의한 **소계가 계산된 결과**에는 **GROUPING(EXPR) = 1**이 표시되고,

  **그 외의 결과**에는 **GROUPING(EXPR) = 0**이 표시된다.



```sql
SELECT dname, grouping(dname), job, grouping(job), COUNT(*) "total emp", SUM(sal) "total sal" 
FROM emp, dept 
WHERE dept.deptno = emp.deptno
GROUP BY ROLLUP(dname, job);
```





**STEP 4-1. GROUPING 함수 + CASE 사용**

```sql
SELECT 
	CASE GROUPING(dname) 
	WHEN 1 THEN 'All Departments'
	ELSE dname 
	END AS dname,
	CASE GROUPING(job)
	WHEN 1 THEN 'All Jobs'
	ELSE job
	END as job,
	COUNT(*) "total emp", SUM(sal) "total sal" 
FROM emp, dept
WHERE dept.deptno = emp.deptno
GROUP BY ROLLUP(dname, job);

[Oracle]
SELECT DECODE(GROUPING(dname), 1, 'All Departments', dname) AS dname,
       DECODE(GROUPING(job), 1, 'All jobs', job) AS job,
       COUNT(*) "total emp",
       SUM(sal) "total sal"
FROM emp, dept
WHERE dept.deptno = emp.deptno
GROUP BY ROLLUP(dname, job);
```





**STEP 4-2. ROLLUP 함수 일부 사용**

```sql
SELECT 
	CASE GROUPING(dname) WHEN 1 THEN 'All Departments' ELSE dname END AS dname,
	CASE GROUPING(job) WHEN 1 THEN 'All Jobs' ELSE job END AS job,
	COUNT(*) "total emp",
	SUM(sal) "total sal"
FROM emp, dept
WHERE dept.deptno = emp.deptno
GROUP BY dname, ROLLUP(job);

DNAME                          JOB                 total emp  total sal
------------------------------ ------------------ ---------- ----------
SALES                          CLERK                       1        950
SALES                          MANAGER                     1       2850
SALES                          SALESMAN                    4       5600
SALES                          All Jobs                    6       9400
RESEARCH                       CLERK                       2       1900
RESEARCH                       ANALYST                     2       6000
RESEARCH                       MANAGER                     1       2975
RESEARCH                       All Jobs                    5      10875
ACCOUNTING                     CLERK                       1       1300
ACCOUNTING                     MANAGER                     1       2450
ACCOUNTING                     PRESIDENT                   1       5000
ACCOUNTING                     All Jobs                    3       8750
```





## 3. CUBE 함수

ROLLUP에서는 단지 가능한 Subtotal 만을 생성하였지만, CUBE는 결합 가능한 모든 값에 대하여 다차원 집계를 생성한다. CUBE를 사용할 경우에는 내부적으로 Grouping Columns의 순서를 바꾸어서 또 한 번의 Query를 추가 수행 해야 한다.



```sql
SELECT 
	CASE GROUPING(dname) WHEN 1 THEN 'All Departments' ELSE dname END AS dname,
	CASE GROUPING(job) WHEN 1 THEN 'All Jobs' ELSE job END AS job,
	COUNT(*) "total emp",
	SUM(sal) "total sal"
FROM emp, dept
WHERE dept.deptno = emp.deptno
GROUP BY CUBE(dname, job);
```





## 4. GROUPING SETS 함수

GROUPING SET를 이용해 더욱 다양한 소계 집합을 만들 수 있는데, GROUP BY SQL 문장을 여러 번 반복하지 않아도 원하는 결과를 쉽게 얻을 수 있게 되었다. GROUPING SETS에 표시된 인수들에 대한 개별 집계를 구할 수 있으며, 이때 표시된 인수들 간에는 계층 구조인 ROLLUP과는 달리 평등한 관계이므로 인수의 순서가 바뀌어도 결과는 같다. 그리고 GROUPING SETS 함수도 결과에 대한 정렬이 필요한 경우는 ORDER BY 절에 명시적으로 정렬 칼럼이 표시가 되어야 한다.



**[일반 그룹함수를 이용한 SQL]**

일반 그룹함수를 이용하여 부서별, JOB 별 인원수와 급여 합을 구하라.

```sql
SELECT dname, 'All Jobs' job, COUNT(*) "total emp", SUM(sal) "total sal" 
FROM emp, dept
WHERE dept.deptno = emp.deptno
GROUP BY dname 
UNION ALL
SELECT 'All Departments' dname, job, COUNT(*) "total emp", SUM(sal) "total sal"
FROM emp, dept
WHERE dept.deptno = emp.deptno
GROUP BY job;

DNAME                          JOB                 total emp  total sal
------------------------------ ------------------ ---------- ----------
ACCOUNTING                     All Jobs                    3       8750
RESEARCH                       All Jobs                    5      10875
SALES                          All Jobs                    6       9400
All Departments                CLERK                       4       4150
All Departments                SALESMAN                    4       5600
All Departments                PRESIDENT                   1       5000
All Departments                MANAGER                     3       8275
All Departments                ANALYST                     2       6000
```





**[GROUPING SETS 사용 SQL]**

일반 그룹함수를 GROUPING SETS 함수로 변경하여 부서별, JOB별 인원수와 급여 합을 구하라.

```sql
SELECT GROUPING(dname), GROUPING(job), dname, job 
FROM emp, dept 
WHERE dept.deptno = emp.deptno 
GROUP BY GROUPING SETS (dname, job);




GROUPING(DNAME) GROUPING(JOB) DNAME                        JOB
--------------- ------------- ---------------------------- ------------------
              1             0                              CLERK
              1             0                              SALESMAN
              1             0                              PRESIDENT
              1             0                              MANAGER
              1             0                              ANALYST
              0             1 ACCOUNTING
              0             1 RESEARCH
              0             1 SALES


SELECT DECODE(GROUPING(dname), 1, 'All Departments', dname) AS dname,
       DECODE(GROUPING(job), 1, 'All Jobs', job) AS job,
       COUNT(*) "total emp", 
       SUM(sal) "total sal"
FROM emp, dept
WHERE dept.deptno = emp.deptno
GROUP BY GROUPING SETS (dname, job);

DNAME                          JOB                 total emp  total sal
------------------------------ ------------------ ---------- ----------
All Departments                CLERK                       4       4150
All Departments                SALESMAN                    4       5600
All Departments                PRESIDENT                   1       5000
All Departments                MANAGER                     3       8275
All Departments                ANALYST                     2       6000
ACCOUNTING                     All Jobs                    3       8750
RESEARCH                       All Jobs                    5      10875
SALES                          All Jobs                    6       9400
```



GROUPING SETS 함수 사용시 UNION ALL을 사용한 일반 그룹함수를 사용한 SQL과 같은 결과를 얻을 수 있으며, 괄호로 묶은 집합 별로 집계를 구할 수 있다.





**[3개의 인수를 사용한 GROUPING SETS 이용]**

```sql
SELECT dname, job, mgr, SUM(sal) "total sal"
FROM emp, dept
WHERE dept.deptno = emp.deptno
GROUP BY GROUPING SETS ((dname, job, mgr), (dname, job), (job, mgr));

DNAME                        JOB                       MGR  total sal
---------------------------- ------------------ ---------- ----------
SALES                        CLERK                    7698        950
ACCOUNTING                   CLERK                    7782       1300
RESEARCH                     CLERK                    7788       1100
RESEARCH                     CLERK                    7902        800
RESEARCH                     ANALYST                  7566       6000
SALES                        MANAGER                  7839       2850
RESEARCH                     MANAGER                  7839       2975
ACCOUNTING                   MANAGER                  7839       2450
SALES                        SALESMAN                 7698       5600
ACCOUNTING                   PRESIDENT                           5000


                             CLERK                    7698        950
                             CLERK                    7782       1300
                             CLERK                    7788       1100
                             CLERK                    7902        800
                             ANALYST                  7566       6000
                             MANAGER                  7839       8275
                             SALESMAN                 7698       5600
                             PRESIDENT                           5000
                             
                             
SALES                        MANAGER                             2850
SALES                        CLERK                                950
ACCOUNTING                   CLERK                               1300
ACCOUNTING                   MANAGER                             2450
ACCOUNTING                   PRESIDENT                           5000
RESEARCH                     MANAGER                             2975
SALES                        SALESMAN                            5600
RESEARCH                     ANALYST                             6000
RESEARCH                     CLERK                               1900
```

GROUPING SETS 함수 사용시 괄호로 묶은 집합별로(괄호 내는 계층구조가 아닌 하나의 데이터로 간주) 집계를 구할 수 있다.



실행 결과에서 첫 번째 10 건의 데이터는 (DNAME+JOB+MGR) 기준의 집계이며, 두 번째 8 건의 데이터는 (JOB+MGR) 기준의 집계이며, 세 번째 9 건의 데이터는 (DNAME+JOB) 기준의 집계이다.