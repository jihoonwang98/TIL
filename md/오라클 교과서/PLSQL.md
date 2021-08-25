# PL/SQL

SQL 문 자체는 **비 절차성 언어**이므로 여러 개의 질의문 사이에 연결이나 절차가 있어야 할 때는 사용할 수 없다. 즉, 데이터를 등록하는 2개의 SQL 문을 연관지을 수는 없다는 의미이다. 그러나 실제 프로그래밍을 할 때에는 각각의 SQL 문들을 서로 연관되도록 하고 절차적(또는 순차적)인 단계를 가지고 SQL 문이 실행되도록 해야 하는 경우가 대부분이다. 이를 위해 오라클에서 지원하는 기능이 PL/SQL이다.

PL/SQL은 오라클에서 제공하는 프로그래밍 언어이며 다른 제품군(MSSQL, MySQL 등)에서는 사용할 수 없다.



## 1. PL/SQL 개요

**PL/SQL**은 Procedural Language extension to Structured Query Language의 약자로, SQL 언어에 절차적인 프로그래밍 언어를 가미해 만든 것이다.

PL/SQL은 주로 자료 내부에서 SQL 문만으로 처리하기에는 복잡한 자료의 저장이나 프로시저와 트리거 등을 작성하는 데 쓰인다.



### PL/SQL의 장점과 기능

- **응용 프로그램 성능**
- **생산성, 이식성 그리고 보안**
- **사전 정의된 패키지**
- **객체지향 프로그래밍**



### PL/SQL의 역사





## 2. PL/SQL 구조

PL/SQL은 **블록**으로 이루어진 구조이다. 이러한 블록(Block) 형태의 PL/SQL을 사용자가 실행하면 오라클 서버의 PL/SQL 엔진이 해당 블록을 받는다. 그 후 해당 블록에 있던 모든 SQL 문이 오라클 서버 프로세스에 전달되어 수행된다.

또한, PL/SQL 엔진은 해당 SQL 문장이 수행되어 결과가 돌아오면 그 결과를 받고 난 후 다음 PL/SQL 문을 실행한다. PL/SQL 문이 DB에서 처리된 데이터를 처리해야 하는 경우가 발생하면 반드시 변수를 선언하여 처리해야 하며 변수를 잘못 선언한 경우에는 PL/SQL 엔진에서 해당 내용을 사용할 수 없다.



즉, PL/SQL 엔진이 SQL 문을 발견하면 변환 과정을 거쳐 오라클 서버 프로세스에 전달하고 오라클 서버 프로세스가 SQL 문의 수행과정을 거쳐 쿼리를 수행한 결과(SELECT의 경우 값)를 반환해줘야 나머지 부분을 실행할 수 있다는 뜻이다. 또한, SELECT의 경우에 반환된 값을 직접적으로 사용할 수는 없고 변수를 통해서만 처리할 수 있다. 하나의 행의 하나의 칼럼이라면 해당 칼럼의 데이터 형식과 동일한 타입의 변수를 선언해야 하며 여러 행이라면 커서 타입의 변수를 통해서만 사용할 수 있다.



---

### PL/SQL 구조

PL/SQL 기본 구조인 **블록**은 

- **선언부 **(DECLARE)
- **실행부** (BEGIN)
- **예외 처리부 **(EXCEPTION)

으로 구성된다.



또한, 각 블록은 블록 안에 다른 블록을 포함할 수 있는데 이러한 형태를 중첩 블록(Nested Block)이라 부른다.



PL/SQL 블록의 유형에는 크게

- **익명 PL/SQL 블록** (Anonymous PL/SQL Block)
- **저장 PL/SQL 블록** (Stored PL/SQL Block)

이 있다.



익명 PL/SQL 블록은 주로 **일회성**으로 사용할 때가 잦고, 

저장된 블록은 서버에 저장해두고 **주기적**으로 반복해서 사용할 때가 잦다.

저장 PL/SQL 블록은 스키마를 구성하는 오브젝트로 구성되어 오라클 서버에 저장되거나 라이브러리 형태로 저장된다.





#### PL/SQL 기본 구조

PL/SQL을 구성하는 기본 구조는 블록 구조이며, 하나의 블록에는 여러 개의 블록이 포함될 수 있으며 각각의 블록은 문제를 해결하거나 하위에 포함된 기능으로 구성된다. 다시 말해 각 부분을 구성하는 블록은 프로그램을 구성하는 기본 단위로, 절차 또는 기능을 자체적으로 가지거나 다른 PL/SQL 블록을 포함하는 구조로 구성된다는 의미이다.



블록은 PL/SQL의 가장 기본적인 단위로, 모든 PL/SQL 프로그램은 블록으로 결합한다. 이러한 블록은 다른 블록 안에서 중첩될 수 있다. 일반적으로 PL/SQL 블록은 하나의 논리적 작업을 나타내는 문장으로 구성된다.또한, 하나의 논리적인 작업은 PL/SQL 문과 SQL 문의 조합으로 구성된다. 이렇게 구성된 블록은 클라이언트의 응용 프로그램을 통하여 서버의 PL/SQL 엔진에 전달된다. 전달된 블록은 PL/SQL 엔진에 의해 각각의 SQL 문으로 변환되며, 변환된 SQL 문은 각각의 처리 프로세스로 전달되어 처리된다.





#### PL/SQL 종류

PL/SQL은 이름을 정한 PL/SQL 블록과 이름을 정하지 않은 PL/SQL 블록 두 가지로 나눌 수 있다. 



**이름을 정한 블록**에는 주로 **저장 프로시저**(Stored Procedure), **사용자 함수**(User Function)가 대표적이며, PL/SQL 블록안에서 절차 및 기능으로 서브 루틴을 만들어 처리할 때에 주로 많이 사용한다.

**이름이 없는 블록**은 이름을 저장하지는 않고 한 번만 사용하므로 나중에 다시 참조하여 사용할 수 없다. 주로 이름을 저장하기 전의 테스트 용도나 한 번 사용하고 재사용하지 않을 때에 사용한다. 익명 PL/SQL 문은 대부분 **프로시저**의 형태이다.







PL/SQL의 실행은 



**익명 PL/SQL 블록**의 경우 서버의 PL/SQL 엔진으로 PL/SQL 블록을 보내 먼저 컴파일을 하고 실행된다.

**명명된 PL/SQL 블록**은 블록의 생성 시나 수정된 경우에만 컴파일을 한다.



컴파일은 PL/SQL의 문장을 확인하고 편집 오류에 대한 코드를 확인하는 작업을 의미한다. 오류가 발생하면 프로그래머들이 이러한 오류를 수정하여 완벽한 컴파일이 이루어지게 되며 다시 오라클 용 데이터를 저장하게 되면서 프로그램 변수에 저장 주소를 할당한다. 이를 **바인딩**이라고 한다. 실제의 사용은 바인딩이 되고 나서 가능하다.



컴파일이 완료되면 명명된 PL/SQL 블록의 상태가 VALID 상태로 표시되게 되며, 컴파일 과정에 실패하면 명명된 PL/SQL 블록의 상태는 INVALID로 설정된다.





#### PL/SQL 블록 구성

PL/SQL 블록은 선언부, 실행부, 예외 처리부의 세 부분으로 구성되어 있으며 실행부는 반드시 블록에 포함되어야 하지만 선언부와 예외 처리부는 선택 사항으로, 포함되지 않아도 구성에 문제가 발생하지는 않는다. 실제 PL/SQL 블록은 다음과 같이 구성된다.

```plsql
DECLARE
	선언부
BEGIN
	실행부
EXCEPTION
	예외 처리부
END;
```



- **선언부 (DECLARE)**

  선언부는 PL/SQL 블록의 첫 번째 부분으로, **실행부에서 참조할 변수, 상수 및 커서 등의 모든 식별자를 정의하는 데 사용**되며 DECLARE로 시작한다. 실행부에서 참조할 변수나 상수가 없다면 생략할 수 있다.



- **실행부 (BEGIN)**

  실행부는 PL/SQL 블록의 선언부 다음 부분으로, 데이터를 처리할 SQL 문과 블록 안의 데이터를 처리할 PL/SQL 문을 기술하여 **선언부에서 선언된 식별자를 조작하는 부분**이다. 즉 **제어문, 반복문, 함수 정의 등 절차적 형식으로 SQL 문을 실행할 수 있도록 로직을 기술**할 수 있는 부분으로, BEGIN으로 시작한다.



- **예외 처리부 (EXCEPTION)**

  예외 처리부는 PL/SQL 블록의 마지막 부분으로 PL/SQL 문이 실행되는 중에 에러가 발생할 수 있는데 이를 예외 사항이라 하며 **이러한 예외 사항이 발생이 되었을 때에 문제를 해결**하기 위한 구문으로 구성되며 EXCEPTION으로 시작된다. 예외 사항을 처리할 필요가 없다면 생략할 수 있다.



또한, PL/SQL 블록 내의 각 부분에 포함되는 문장 중 <u>DECLARE, BEGIN, EXCEPTION과 같은 예약어들은 세미콜론으로 끝나지 않지만</u>, 나머지 문장들은 SQL 문처럼 세미콜론으로 끝난다.



```plsql
DECLARE
	V_NAME		VARCHAR2(30) := ''; -- 변수에 할당할 값
BEGIN
	V_NAME := 'LEE';
	DBMS_OUTPUT.PUT_LINE('MY NAME IS ' || V_NAME);
END;
/
```



선언부에서는 V_NAME이라는 변수를 VARCHAR2 형식 30자리로 초깃값을 선언하였고 실행부에서는 V_NAME 변수에 LEE라는 값을 할당하여 'MY NAME IS'와 결합하는 예제이다. DBMS_OUTPUT.PUT_LINE은 모니터에 표시를 해주는 출력문이다. 







---

### PL/SQL 용어의 기본



#### 개요

실행 파일은 파일이나 컴퓨터 프로그래밍 언어 중 하나를 사용하여 작성된 프로그램의 이름이다. 프로그램은 명령 프롬프트에서 실행 파일의 이름을 입력할 때 사용된다. 예를 들어, 명령 프롬프트에서 'sqlplus'를 입력하면 SQL*Plus를 호출할 수 있다.



#### PL/SQL 문자 집합

PL/SQL 문자 집합이란 PL/SQL 안에서 사용되는 문자 집합들의 의미하며 식별자, 구분자, 리터럴, 주석 등으로 구성된다. PL/SQL 프로그램은 문자, 숫자, 기호, 공간 등의 네 가지 유형을 사용하여 코드를 기록한다.



문자는 a~z, A~Z를 포함하고 숫자는 0~9, 기호는 `(, ), -, +, /, <, >, =` 등을, 마지막 공간은 탭, 공백 등을 포함한다. 이러한 문자 유형들이 서로 결합하여 PL/SQL의 단어가 만들어진다. 이러한 PL/SQL의 단어들은 식별자, 예약어, 연산자, 리터럴, 주석의 다섯 가지 그룹 중 하나에 속하게 된다.



기본적으로 SQL과 마찬가지로 PL/SQL 문에서 명령은 대소문자를 구분하지 않지만, 문자열과 문자 리터럴 내의 문자는 대소문자를 구분한다.



#### 식별자



#### 예약어



#### 구분자(연산자)



#### 리터럴



#### 주석





---

### 공통 데이터 형식

PL/SQL에서 사용되는 데이터 유형은 거의 SQL에서 사용되는 데이터 형식과 비슷하다.



**[PL/SQL의 데이터 형식]**

| 데이터 형식                    | 설명                                                         |
| ------------------------------ | ------------------------------------------------------------ |
| VARCHAR2, NVARCHAR2            | 가변 길이 문자열                                             |
| CHAR, NCHAR                    | 고정 길이 문자열                                             |
| NUMBER                         | 수치를 저장하기 위한 가변 길이의 표준 내부 형식 (부동 소수점, 고정 소수점, 정수 표현 가능) |
| PLS_INTEGER (BINARY_INTEGER)   | 정수                                                         |
| DATE                           | 날짜                                                         |
| BOOLEAN                        | TRUE, FALSE, NULL                                            |
| LOB (BFILE, BLOB, CLOB, NCLOB) | 대용량 객체                                                  |
| %TYPE                          | 데이터베이스 필드의 데이터 형식을 가정                       |
| %ROWTYPE                       | 데이터베이스 행(ROW)의 데이터 형식을 가정                    |



#### TYPE과 ROWTYPE

TYPE과 ROWTYPE 형은 객체지향 프로그래밍이 가능한 유형이다. 기존의 자료형에는 없었던 유형으로, PL/SQL 문에서 사용할 수 있다. TYPE은 여러 가지의 형태로 사용할 수 있고 ROWTYPE은 ROW의 형태로 사용된다. 



- **ROWTYPE**

  ROWTYPE은 ROW 또는 레코드(테이블 또는 조회된 내용에서의 가로축) 라고 부르는 개념에 대응되는 형이다. 선언을 하는 방법은 테이블 기반의 선언, 커서 기반의 선언으로 구분된다.



```plsql
변수명 테이블명%ROWTYPE;
변수명 커서명%ROWTYPE;
```

테이블의 필드명으로 대응되는 한 건의 ROW를 담을 수 있는 공간을 선언한다. 또는 SELECT 하여 가져올 내용의 커서의 ROW를 선언한다.



실제 데이터를 처리하기 위한 중요한 내용으로 레코드 단위를 저장할 수 있는 공간으로 구분된다. 이러한 레코드 안에는 칼럼(또는 필드)으로 데이터가 구분이 되어 저장된다.



```plsql
V_EMP_ROW EMPLOYEES%ROWTYPE;
CURSOR V_EMP_CUR IS
	ELECT EMPLOYEE_ID, FIRST_NAME, LAST_NAME
	FROM EMPLOYEES
	WHERE JOB_ID = 'IT_PROG';
V_EMP_CUR_ROW V_EMP_CUR%ROWTYPE;
```



ROWTYPE 형은 ROW 자체를 담을 수 있고 하위 개념으로 칼럼을 포함하고 있다. 즉, 객체의 개념으로 저장하고 사용할 수 있다는 의미이다. 앞의 예문에서 살펴보면 V_EMP_ROW는 EMPLOYEES 테이블의 모든 칼럼을 가지는 하나의 ROW를 의미하는 내용이고 V_EMP_CUR_ROW는 커서에서 사용된 EMPLOYEE_ID, FIRST_NAME, LAST_NAME의 칼럼만을 가지는 하나의 ROW를 의미한다.



ROWTYPE을 사용하여 특정 칼럼의 데이터를 사용하려 한다면 '변수이름.칼럼명'으로 사용할 수 있다. 



- **TYPE**

  TYPE은 사용자 정의로 객체를 만들거나 기존의 객체를 사용할 때 사용하는 형이다. TYPE ~ RECORD를 이용하여 ROWTYPE처럼 사용할 수도 있으며 기존의 테이블이나 뷰의 특정 칼럼의 내용을 사용할 수도 있다.



```plsql
TYPE 변수명 IS RECORD (칼럼명 자료형, ...);
변수명 테이블명.칼럼명%TYPE;
```

새로운 사용자 레코드를 만들거나 기존의 객체들의 형식을 사용할 수 있게 하며 새로운 형식의 객체를 만들 수도 있게 한다.



```plsql
TYPE V_NEW_EMP_RECODE IS RECORD (
	EMP_ID EMPLOYEES.EMPLOYEE_ID%TYPE,
	MOBILE_NO VARCHAR2(20) )
```

앞의 예문을 살펴보면 TYPE을 레코드로 만들면서 EMP_ID는 EMPLOYEES 테이블의 EMPLOYEE_ID의 데이터 타입을 사용하고, MOBILE_NO는 VARCHAR2(20)으로 하여 새로운 타입인 레코드 V_NEW_EMP_RECORD를 만든다.







## 3. PL/SQL 실행

PL/SQL을 작성할 때는 기본적으로 SQL 문을 실행할 때와 같은 도구를 사용하면 된다. 



### PL/SQL 작성과 실행

PL/SQL 프로그램은 저장 프로서지나 사용자 함수처럼 오라클 데이터베이스에 저장하여 다시 실행하여 사용할 수도 있지만, 오라클 DB에 저장하지 않고 그냥 클라이언트에서 파일 형태로 가지고 있으면서 필요에 따라 실행할 수도 있다. 전자의 경우를 명명된 PL/SQL 블록이라 하며 앞으로 우리가 자세히 배워 나갈 부분이고, 후자의 경우를 익명 PL/SQL 블록이라 한다.



SET SERVEROUTPUT ON



### PL/SQL 문의 반복 처리

PL/SQL 문의 흐름은 기본적으로 위에서 아래쪽으로 차례대로 실행된다. 그런데 프로그램을 하다 보면 특정 부분을 반복해야 할 때가 생기는데, 이때 필요한 것이 반복 처리이다. PL/SQL에서는 LOOP를 이용하여 반복 처리를 지원하는데 LOOP는 단순 LOOP, FOR LOOP, WHILE LOOP 등의 3가지로 사용할 수 있다.



**(1) 단순 LOOP**

가장 단순한 반복문은 LOOP ~ END LOOP의 구조이다. EXIT WHEN 조건문을 통하여 반복문을 종료하는 구조로 되어 있다.

```plsql
LOOP
	EXIT WHEN 종료 조건
			반복문
END LOOP;
```

종료 조건을 만족할 때까지 LOOP와 END LOOP 사이를 반복한다.



```plsql
DECLARE 
	V_NUMBER NUMBER := 2;
	V_CNT    NUMBER := 1;
BEGIN
	LOOP
		EXIT WHEN V_CNT > 9;
		DBMS_OUTPUT.PUT_LINE(TO_CHAR(V_NUMBER) || ' * ' || TO_CHAR(V_CNT)
		 || ' = ' || TO_CHAR(V_NUMBER * V_CNT));
		V_CNT := V_CNT + 1;
	END LOOP;
END;
/
```



```plsql
DECLARE
	V_NUMBER NUMBER := 2;
	V_CNT	 NUMBER := 1;
BEGIN
	LOOP
		IF(V_CNT > 9) THEN 
			EXIT;
		END IF;
		DBMS_OUTPUT.PUT_LINE(TO_CHAR(V_NUMBER) || ' * ' || TO_CHAR(V_CNT)
		 || ' = ' || TO_CHAR(V_NUMBER * V_CNT));
		V_CNT := V_CNT + 1;
	END LOOP;
END;
/
```





**(2) FOR LOOP**

FOR LOOP는 정확한 계산을 유추하기 가장 좋은 반복문이면서 무한 반복문이 발생할 수 없는 구조이다. FOR 변수 IN 1 ~  N LOOP ~ END LOOP의 구조로 되어 있다.

```plsql
FOR 변수 IN [REVERSE] 시작 .. 종료
LOOP
	반복문
END LOOP;
```

시작 숫자부터 1씩 증가하여 종료 숫자까지 LOOP와 END LOOP를 반복한다.

시작과 종료는 정수만 인식하며 무조건 1씩 증가한다. REVERSE 명령을 사용하면 종료부터 시작 순으로 값이 사용된다.



```plsql
DECLARE
	V_NUMBER NUMBER := 2;
	V_CNT    NUMBER;
BEGIN
	FOR V_CNT IN 1 .. 9
	LOOP 
		DBMS_OUTPUT.PUT_LINE(TO_CHAR(V_NUMBER) || ' * ' || TO_CHAR(V_CNT)
		 || ' = ' || TO_CHAR(V_NUMBER * V_CNT));
	END LOOP;
END;
/
```







**(3) WHILE LOOP**

WHILE LOOP는 WHILE 종료 조건 LOOP ~ END LOOP의 구조로 되어 있다. 단순 LOOP와 같이 무한 반복이 가능하기 때문에 주의해야 한다.



```plsql
WHILE 종료 조건
LOOP
	반복문
END LOOP;
```

WHILE 조건이 만족할 때만 LOOP와 END LOOP를 반복한다.



```plsql
DECLARE 
	V_NUMBER    NUMBER := 2;
	V_CNT       NUMBER := 1;
BEGIN
	WHILE (V_CNT < 10)
	LOOP
		DBMS_OUTPUT.PUT_LINE(TO_CHAR(V_NUMBER) || ' * ' || TO_CHAR(V_CNT)
		 || ' = ' || TO_CHAR(V_NUMBER * V_CNT));
		 V_CNT := V_CNT + 1;
	END LOOP;
END;
/
```





### PL/SQL 문의 제어 처리

프로그램 흐름의 제어 처리는 IF 문, CASE 문, GOTO 문을 통하여 이루어진다.



**(1) IF 문**

IF ~ THEN ~ END IF 또는 IF ~ THEN ~ ELSE ~ END IF 또는 IF ~ THEN ~ ELSIF ~ ELSE ~ END IF 형식으로 사용된다. 즉, 조건이 참이면 THEN 조건을, 그렇지 않으면 ELSE 조건을 처리하는 식이다. 거기에 변형으로 CASE 문과 비슷하게 사용할 수 있는 ELSIF 문과 IF 문 안에 다시 IF 문을 사용하는 중첩 IF 문도 가능하다.



```plsql
IF 조건 1 THEN
	제어 처리 1
[ELSIF 조건2 THEN 제어 처리 2] [...]
[ELSE 조건에 맞지 않는 경우]
END IF;
```





```plsql
DECLARE 
	V_GENDER  VARCHAR2(1);
BEGIN
	V_GENDER := 'M';
	IF V_GENDER = 'M' THEN
		DBMS_OUTPUT.PUT_LINE('I AM MAN');
	ELSE 
		DBMS_OUTPUT.PUT_LINE('I AM WOMAN');
	END IF;
END;
/	
```





**(2) CASE 문**

CASE 문은 **단순 CASE 문**과 **조건 CASE 문**으로 나누어진다. CASE 문 역시 IF 문과 마찬가지로 CASE 문 안에 CASE 문을 사용하는 중첩 CASE 문이 가능하다.



```plsql
CASE 표현식
WHEN 결괏값1 THEN
	처리문1
WHEN 결괏값2 THEN
	처리문2
...
ELSE
	만족하지 않는 경우 처리문
END CASE;


CASE 
WHEN 표현식1 THEN
	처리문1
WHEN 표현식2 THEN
	처리문 2
...
ELSE
	만족하지 않는 경우 처리문
END CASE;
```



다음은 단순 CASE 문을 이용한 예제이다.

```plsql
DECLARE 
	V_GENDER VARCHAR2(1);
BEGIN 
	V_GENDER := 'M';
	CASE V_GENDER
	WHEN 'M' THEN
		DBMS_OUTPUT.PUT_LINE('I AM MAN');
	WHEN 'W' THEN
		DBMS_OUTPUT.PUT_LINE('I AM WOMAN');
	ELSE
		DBMS_OUTPUT.PUT_LINE('I AM ETC');
	END CASE;
END;
/
```



다음은 조건 CASE 문을 이용한 예제이다.

```plsql
DECLARE
	V_GENDER VARCHAR2(1);
BEGIN
	V_GENDER := 'M';
	CASE
	WHEN V_GENDER = 'M' THEN
		DBMS_OUTPUT.PUT_LINE('I AM MAN.');
	WHEN V_GENDER = 'W' THEN
		DBMS_OUTPUT.PUT_LINE('I AM WOMAN.');
	ELSE 
		DBMS_OUTPUT.PUT_LINE('I AM ETC.');
	END CASE;
END;
/
```





**(3) GOTO 문**

GOTO 문은 특정 레이블로 무조건 보내는 명령이다. GOTO 문은 단독으로 사용되기보다 반복문이나 다른 제어문인 IF 문이나 CASE 문과 같이 많이 사용된다. 레이블은 '<<' 와 '>>'로 둘러싸여야 한다.



```plsql
GOTO 레이블;
<<레이블>>
```



```plsql
DECLARE 
	V_GENDER VARCHAR2(1);
	V_RETURN VARCHAR2(10);
BEGIN
	V_GENDER := 'M';
	V_RETURN := '';
	CASE
	WHEN V_GENDER = 'M' THEN
		GOTO M_LABEL;
	WHEN V_GENDER = 'W' THEN
		GOTO W_LABEL;
	ELSE
		GOTO ETC_LABEL;
	END CASE;

<<M_LABEL>>
	V_RETURN := 'MAN';
	GOTO END_PROGRAM;
<<W_LABEL>>
	V_RETURN := 'WOMAN';
	GOTO END_PROGRAM;
<<ETC_LABEL>>
	V_RETURN := 'ETC';
	GOTO END_PROGRAM;
<<END_PROGRAM>>
	DBMS_OUTPUT.PUT_LINE('I AM ' || V_RETURN);
END;
/
```





### 대체 변수를 이용한 PL/SQL 실행

이전의 PL/SQL 문에서 다른 경우를 실행하려 한다면 변수에 지정 값을 바꿔서 할당해야 하는데, 매번 프로그램을 변경하는 대신에 대체 변수를 사용하면 간단하게 처리할 수 있다. 이전의 구구단 PL/SQL 문을 대체 변수를 이용하여 처리하면서 대체 변수에 대해 살펴보도록 하자.



**(1) 대체 변수**

변수에 값을 할당하는 방법의 하나로 대체 변수를 사용하는데, 대체 변수를 지정하면 해당 문장이 수행될 때에 값을 묻고 값을 입력하면 입력된 값으로 대체 변수의 값을 변경시키고 실행되게 한다. 대체 변수는 접두사로 하나의 또표(&) 또는 두 개의 또표(&&)를 사용하여 지정한다.



```plsql
&변수명
&&변수명
```



대체 변수는 모두 입력을 받지만 두 개의 또표(&&)로 입력받은 내용은 한 번 입력되면 세션 안에서는 계속 입력된 내용으로 사용되고 또표 하나(&)로 지정된 대체 변수는 PL/SQL 문이 실행될 때마다 입력을 받는다.



```plsql
DECLARE
	V_NUMBER NUMBER;
	V_PUBLIC NUMBER;
    V_CNT    NUMBER;
BEGIN
	V_NUMBER := &IN_NUMBER;
	V_PUBLIC := &&IN_PUBLIC;
	
	FOR V_CNT IN 1 .. V_PUBLIC
	LOOP
		DBMS_OUTPUT.PUT_LINE(TO_CHAR(V_NUMBER) || ' * ' || TO_CHAR(V_CNT) 
						|| ' = ' || TO_CHAR(V_NUMBER * V_CNT));
	END LOOP;
END;
/
```





**(2) 대체 변수 사용 중지**

특정한 경우에 또표(&)와 두 개의 또표(&&)가 문자열로 사용되어야 할 때가 발생한다. 이럴 때는 잠시 대체 변수의 사용을 중지해야 하는데, 이때 사용하는 명령이 SET DEFINE 명령이다. SET DEFINE 명령을 사용하여 대체 변수를 다른 문자로 변경하든지 대체 변수의 사용을 중지하든지 대체 변수를 다시 사용할 수 있게 지정하든지 한다.



```sql
INSERT INTO TEST_TABLE (COL_01)
VALUES ('C&C');
```



위의 문장을 실행하려면 SET DEFINE OFF로 대체 변수 기능을 일시 정지한 다음 실행하면 된다.

```sql
SET DEFINE OFF;

INSERT INTO TEST_TABLE (COL_01)
VALUES ('C&C');
```



다시 SET DEFINE ON을 실행하면 대체 변수 기능이 활성화된다. 

PL/SQL에서 &를 대체 변수로 사용할 수 없을 때는 `SET DEFINE 문자`를 지정하여 변경하여 사용하고 다시 `SET DEFINE &`를 실행하여 처리하면 된다.





### PL/SQL 문에서 SQL 문 사용

PL/SQL 문에서도 SQL 문을 사용하여 테이블의 데이터를 검색하거나 조작할 수 있다. 하지만, SQL 문을 사용할 때 다음과 같은 사항에 주의하여야 한다.

- END 문은 트랜잭션의 끝이 아닌 PL/SQL의 끝이다. 특히 트리거 같은 경우에는 트랜잭션이 외부에서 발생하기 때문에 트리거 안에서 END를 만난다고 하더라도 트랜잭션이 완료된 것이 아니라는 것을 명심해야 한다.
- DDL은 사용할 수 없다.
- GRANT, REVOKE도 사용할 수 없다.
- 직접적으로 데이터를 검색하여 사용할 수 없다.



**(1) SELECT 사용**

```plsql
SELECT select_list
INTO {variable_name[, variable_name]..|record_name}
FROM table
WHERE condition;
```



데이터를 조회한 후 INTO 절에 있는 변수에 조회한 데이터를 저장하여 사용할 수 있다. 그러므로 SELECT 문에서 나열된 항목과 INTO 절의 변수의 수와 데이터 타입이 같아야 한다. 또, SELECT로 추출되어야 하는 ROW의 수도 역시 1건이어야만 처리할 수 있다. 따라서 WHERE 조건으로 반드시 데이터가 1건이 될 수 있도록 해야 한다.



```plsql
DECLARE 
	V_NAME employees.first_name%TYPE;
	V_SALARY employees.salary%TYPE;
BEGIN
	SELECT first_name, salary
	INTO V_NAME, V_SALARY
	FROM employees
	WHERE employee_id = &IN_EMPLOYEE_ID;
	DBMS_OUTPUT.PUT_LINE(V_NAME || '의 급여는 ' || TO_CHAR(V_SALARY) || ' 이다.');
END;
/
```





```plsql
DECLARE
	V_NAME employees.first_name%TYPE;
	V_SALARY employees.salary%TYPE;
BEGIN
	SELECT first_name, salary
	INTO V_NAME, V_SALARY
	FROM employees
	WHERE job_id = '&IN_JOB_ID';
	DBMS_OUTPUT.PUT_LINE(V_NAME || ' 의 급여는 ' || TO_CHAR(V_SALARY) || '이다.');
END;
/
```





**(2) 커서를 이용한 SELECT 사용**

PL/SQL 문에서 데이터 조회는 1건만을 조회해야 하는데, 여러 건의 데이터를 조회하여 처리해야 할 때가 있다. 이럴 때 사용하는 것이 커서이다. 커서는 SELECT한 내용을 커서에 담고서 FETCH 명령을 이용하여 1건씩 추출하여 변수에 담아서 사용하는 방법이다.



```plsql
CURSOR 커서 선언
OPEN 커서
FETCH 커서 INTO 변수
CLOSE 커서
```



커서 선언에 수행할 SELECT 문을 실행시키면 해당 내용이 커서로 선언된다.

```plsql
DECLARE
	CURSOR V_EMP_CUR
	IS
	SELECT first_name, salary
	FROM employees
	WHERE job_id = '&IN_JOB_ID';
	V_NAME employees.first_name%TYPE;
	V_SALARY employees.salary%TYPE;
BEGIN
	OPEN V_EMP_CUR;
	LOOP
		FETCH V_EMP_CUR INTO V_NAME, V_SALARY;
		EXIT WHEN V_EMP_CUR%NOTFOUND;
		DBMS_OUTPUT.PUT_LINE(V_NAME || ' 의 급여는 ' || TO_CHAR(V_SALARY) || ' 이다.');
	END LOOP;
	CLOSE V_EMP_CUR;
END;
/
```



FETCH 명령을 통해서 다음 행으로 이동할 수 있으므로 커서를 사용하면 반복문을 통해 전체를 처리할 수 있다. 반복문을 빠져나갈 때는 커서의 속성을 사용하였다. 



**[커서 속성]**

| 커서 속성        | 설명                                                      |
| ---------------- | --------------------------------------------------------- |
| %FOUND           | FETCH가 성공하면 TRUE 아니면 FALSE                        |
| %NOTFOUND        | FETCH가 실패하면 TRUE 아니면 FALSE                        |
| %ROWCOUNT        | 커서에서 FETCH된 레코드 수                                |
| %ISOPEN          | 커서가 열리면 TRUE 아니면 FALSE                           |
| %BULK_ROWCOUNT   | FORALL 문에 의해서 수정되는 레코드 수                     |
| %BULK_EXCEPTIONS | FORALL 문에 의해서 수정되는 레코드에서 발생하는 예외 정보 |





**(3) DML 사용**

PL/SQL 문에서 DML을 사용할 수 있도록 새로운 테이블을 만들고 구구단을 저장하는 PL/SQL 문을 만들어 보도록 하자. 우선 구구단을 저장할 테이블을 만든다.

```plsql
CREATE TABLE GOOGOODAN (
	NUM1 NUMBER,
	NUM2 NUMBER,
	CAL  NUMBER
);
```





**데이터 입력**

```plsql
DECLARE
	V_NUMBER NUMBER;
	V_PUBLIC NUMBER;
	V_CNT    NUMBER;
BEGIN
	V_NUMBER := &IN_NUMBER;
	V_PUBLIC := &&IN_PUBLIC;
	FOR V_CNT IN 1 .. V_PUBLIC
	LOOP
    	INSERT INTO GOOGOODAN (NUM1, NUM2, CAL)
    	VALUES (V_NUMBER, V_CNT, V_NUMBER * V_CNT);
    END LOOP;
END;
/
```







