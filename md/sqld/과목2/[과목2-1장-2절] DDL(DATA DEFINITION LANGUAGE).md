## 1. 데이터 유형



**데이터 유형**이란?

- DB의 테이블에 특정 자료를 입력할 때, 그 자료를 받아들일 공간을 자료의 유형별로 나누는 기준

  > 프로그래밍 언어의 자료형과 비슷한 개념







**대표적인 4가지 데이터 유형**

1. CHARACTER(s)
   - <u>고정</u> 길이 문자열 정보 (Oracle, SQL Server 모두 CHAR로 표현)
   
   - s만큼 최대 길이를 갖고 고정 길이를 가지고 있으므로 할당된 변수 값의 길이가 s보다 작을 경우 그 차이 길이만큼 <u>공간으로 채워진다</u>.
   
2. VARCHAR(s)
   - CHARACTER VARYING의 약자로 <u>가변</u> 길이 문자열 정보 (Oracle은 VARCHAR2로 표현, SQL Server는 VARCHAR로 표현)
   
   - s만큼의 최대 길이를 갖지만 가변 길이로 조정이 되기 때문에 할당된 변수값의 바이트만 적용된다.
   
3. NUMERIC
   - 정수, 실수 등 숫자 정보 (Oracle은 NUMBER로, SQL Server는 10가지 이상의 숫자 타입을 가지고 있음)
   
   - Oracle은 처음에 전체 자리수를 지정하고, 그 다음 소수 부분의 자리 수를 지정한다. (예를 들어, 정수 부분이 6자리이고 소수점 부분이 2자리인 경우에는 'NUMBER(8,2)'와 같이 된다.)
   
4. DATETIME
   
   - 날짜와 시각 정보 (Oracle은 DATE로 표현, SQL Server는 DATETIME으로 표현)





CHAR와 VARCHAR의 **문자열 비교 방법 차이**

- CHAR에서는 문자열을 비교할 때 <u>공백(BLANK)을 채워서 비교</u>한다. 두 CHAR 문자열을 비교할 때 길이가 짧은 쪽의 끝에 공백을 추가하여 2개의 문자열이 같은 길이가 되도록 한다. 그리고 앞에서부터 한 문자씩 비교한다. 그렇기 때문에 <u>끝의 공백만 다른 문자열은 같다</u>고 판단한다. (ex) 'AA' = 'AA ')

  

- VARCHAR에서는 맨 처음부터 한 문자씩 비교하고 <u>공백도 하나의 문자로 취급</u>하므로 끝의 <u>공백이 다르면 다른 문자로 판단</u>한다. (ex) 'AA' != 'AA ')







## 2. CREATE TABLE



### 가. 테이블과 칼럼 정의



- 테이블에 존재하는 모든 데이터를 <u>고유하게 식별</u>할 수 있으면서 반드시 <u>값이 존재</u>하는 단일 칼럼이나 칼럼의 조합들(후보키) 중에 하나를 선정하여 **기본키 칼럼**으로 지정한다.

  

- 테이블간 **관계**는 <u>기본키와 외부키</u>를 활용해서 설정한다.







### 나. CREATE TABLE

테이블 생성하는 구문 형식은 다음과 같다.

```sql
CREATE TABLE 테이블명 ( 
	칼럼명1 DATATYPE [DEFAULT 형식],
	칼럼명2 DATATYPE [DEFAULT 형식],
	칼럼명3 DATATYPE [DEFAULT 형식],
);
```





테이블 작성시 주의해야 할 규칙

- 테이블명은 <u>객체를 의미</u>하는 적절한 이름 사용. 가능한 <u>단수형</u>
- 테이블명이 다른 테이블명과 <u>중복 불가</u>
- 한 테이블 내에서 <u>칼럼명 중복 불가</u>
- 테이블명은 영어 대소문자, 0-9, _, $, # 문자만 허용된다. (**<u>특수 문자 '-' 허용X</u>**)
- 테이블명은 반드시 숫자가 아닌 문자로 시작해야 한다.





테이블 제약조건은 

- 칼럼의 데이터 유형 뒤에 제약조건을 정의하는 **칼럼 LEVEL 정의 방식**과,
- 테이블 생성 마지막에 모든 제약조건을 기술하는 **테이블 LEVEL 정의 방식**이 있다.







### 다. 제약조건(CONSTRAINT)



**제약조건**이란?

- 사용자가 원하는 조건의 데이터만 유지하기 위한 즉, 데이터의 무결성을 유지하기 위한 DB의 보편적인 방법으로 테이블의 특정 칼럼에 설정하는 제약





**제약조건의 종류**

- PRIMARY KEY (기본키): 테이블에 저장된 행 데이터를 고유하게 식별하기 위한 기본키 정의한다. <u>하나의 테이블에 하나의 기본키 제약만 정의할 수 있다</u>. (기본키 제약 = 고유키 제약 & NOT NULL 제약)

  ```sql
  CREATE TABLE tab(
  	col1 CHAR(8) PRIMARY KEY,
  	col2 CHAR(4),
  	col3 NUMBER CONSTRAINT tab_pk3 PRIMARY KEY,
  	CONSTRAINT tab_pk2 PRIMARY KEY(col2)
  );
  
  1) [컬럼명] [데이터타입] PRIMARY KEY
  2) [컬럼명] [데이터타입] CONSTRAINT 제약조건명 PRIMARY KEY
  3) CONSTRAINT 제약조건명 PRIMARY KEY ([컬럼명1], [컬럼명2], ...)
  4) ALTER TABLE 테이블명 ADD CONSTRAINT 제약조건명 PRIMARY KEY (컬럼명)
  ```

  

- UNIQUE KEY (고유키): 테이블에 저장된 행 데이터를 고유하게 식별하기 위한 고유키를 정의한다. <u>단, NULL은 고유키 제약의 대상이 아니므로, NULL 값을 가진 행이 여러 개가 있더라도 고유키 제약 위반이 되지 않는다</u>.

  

- NOT NULL: NULL 값의 입력 금지. 해당 칼럼 입력 필수.

  

- CHECK: 입력할 수 있는 값의 범위 등을 제한. CHECK 제약으로는 TRUE or FALSE로 평가할 수 있는 논리식을 지정한다.

  

- FOREIGN KEY (외래키): 관계형 DB에서 테이블 간 관계를 정의하기 위해 기본키를 다른 테이블의 외래키로 복사하는 경우 외래키가 생성된다. 





**NULL**이란?

- NULL(ASCII 코드 00번)은 공백(BLANK, ASCII 코드 32번)이나 숫자 0(ZERO, ASCII 48)과는 전혀 다른 값이며, 조건에 맞는 데이터가 없을 때의 공집합과도 다르다. 'NULL'은 아직 정의되지 않은 미지의 값이거나 현재 데이터를 입력하지 못하는 경우를 의미한다.





**DEFAULT**란?

- 데이터 입력 시에 칼럼의 값이 지정되어 있지 않을 경우 기본값(DEFAULT)을 사전에 설정할 수 있다. 데이터 입력시 명시된 값을 지정하지 않은 경우에 NULL 값이 입력되고, DEFAULT 값을 정의했다면 해당 칼럼에 NULL 값이 입력되지 않고 사전에 정의된 기본값이 자동으로 입력된다.







### 라. 생성된 테이블 구조 확인

- Oracle의 경우 
  - DESCRIBE 테이블명;
  
  - DESC 테이블명;
  
- SQL Server의 경우
  
  - sp_help dbo.테이블명;





### 마. SELECT 문장을 통한 테이블 생성

다음 절에서 배울 DML 문장 중에 SELECT 문장을 활용해서 테이블을 생성할 수 있는 방법(CTAS: Create Table ~ As Select ~)이 있다. 기존 테이블을 이용한 CTAS 방법을 이용할 수 있다면 <u>칼럼별로 데이터 유형을 다시 재정의 하지 않아도 되는</u> 장점이 있다.



그러나 CTAS 기법 사용시 기존 테이블의 제약조건 중 <u>NOT NULL만 새로운 복제 테이블에 적용이 되고, 기본키, 고유키, 외래키, CHECK 등의 다른 제약 조건은 없어진다</u>. 제약 조건을 추가하기 위해서는 ALTER TABLE 기능을 사용해야 한다.



SQL Server에서는 Select ~ Into ~ 를 활용하여 위와 같은 결과를 얻을 수 있다. 단, 칼럼 속성에 Identity를 사용했다면 Identity 속성까지 같이 적용이 된다.







## 3. ALTER TABLE



### 가. ADD COLUMN

기존 테이블에 필요한 <u>칼럼을 추가</u>하는 명령

```sql
ALTER TABLE 테이블명 ADD 추가할_칼럼명 데이터_유형;
```

새롭게 추가된 칼럼은 테이블의 마지막 칼럼이 된다.





### 나. DROP COLUMN

기존 테이블에서 필요없는 <u>칼럼을 삭제</u>하는 명령

```sql
ALTER TABLE 테이블명 DROP COLUMN 삭제할_칼럼명;
```





### 다. MODIFY COLUMN

칼럼의 <u>데이터 유형, 디폴트 값, NOT NULL 제약조건</u>을 변경하는 명령



```sql
[Oracle]
ALTER TABLE 테이블명
MODIFY (칼럼명1 데이터유형 [DEFAULT 식] [NOT NULL],
        칼럼명2 데이터유형 [DEFAULT 식] [NOT NULL]...);
        
[SQL Server]
ALTER TABLE 테이블명
ALTER (칼럼명1 데이터유형 [DEFAULT 식] [NOT NULL],
       칼럼명2 데이터유형 [DEFAULT 식] [NOT NULL]...);
```



칼럼 변경시 주의 사항

- 해당 칼럼의 크기를 늘릴 수는 있지만 줄일 수는 없다 -> 기존 데이터의 훼손 방지
- 해당 칼럼이 NULL 값만 가지고 있거나 테이블에 아무 행도 없으면 칼럼의 폭을 줄일 수 있다.
- 해당 칼럼이 NULL 값만 가지고 있으면 데이터 유형을 변경할 수 있다.
- 해당 칼럼의 DEFAULT 값을 바꾸면 변경 작업 이후 발생하는 행 삽입에만 영향을 미친다.
- 해당 칼럼에 NULL 값이 없을 경우에만 NOT NULL 제약조건을 추가할 수 있다.







### 라. RENAME COLUMN

<u>칼럼명을 변경</u>하는 명령

```sql
ALTER TABLE 테이블명 RENAME COLUMN 변경해야_할_칼럼명 TO 새로운_칼럼명;
```







### 마. DROP CONSTRAINT

테이블 생성 시 부여했던 <u>제약조건을 삭제</u>하는 명령

```sql
ALTER TABLE 테이블명 DROP CONSTRAINT 제약조건명;
```







### 바. ADD CONSTRAINT

테이블 생성 이후에 <u>제약조건을 추가</u>하는 명령

```sql
ALTER TABLE 테이블명 ADD CONSTRAINT 제약조건명 제약조건 (칼럼명);
```







## 4. RENAME TABLE

RENAME 명령어를 사용하여 테이블의 이름을 변경할 수 있다.

```sql
[Oracle]
RENAME 변경_전_테이블명 TO 변경_후_테이블명;

[SQL Server]
sp_rename 변경_전_테이블명, 변경_후_테이블명;
```







## 5. DROP TABLE

테이블을 삭제하는 명령

```sql
DROP TABLE 테이블명 [CASCADE CONSTRAINT];
```



CASCADE CONSTRAINT 옵션은 해당 테이블과 관계가 있었던 참조되는 제약조건에 대해서도 삭제한다는 것을 의미한다.







## 6. TRUNCATE TABLE

TRUNCATE TABLE은 테이블 자체가 삭제되는 것은 아니고, 해당 테이블에 들어있던 모든 행들이 제거되고 저장 공간을 재사용 가능하도록 해제한다. 

```sql
TRUNCATE TABLE 테이블명;
```








