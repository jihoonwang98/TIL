# DDL (Data Definition Language)

CREATE, ALTER, DROP, RENAME, TRUNCATE / 테이블 단위 변경



- **CREATE TABLE**

  ```sql
  CREATE TABLE tab1 (
      id NUMBER NOT NULL,
      name VARCHAR2(50)
      
      CONSTRAINT TAB1_PK PRIMARY KEY (id)
  );
  ```

  

- **ALTER TABLE**

  - ADD COLUMN

    ```sql
    ALTER TABLE tab1 ADD birthday DATE
    ```

  - DROP COLUMN

    ```sql
    ALTER TABLE tab1 DROP COLUMN birthday;
    ```

  - MODIFY  COLUMN

    ```sql
    ALTER TABLE tab1
    MODIFY (name VARCHAR(100) NOT NULL);
    ```

  - RENAME COLUMN

    ```sql
    ALTER TABLE tab1 RENAME COLUMN id TO tab1Id
    ```

  - DROP CONSTRAINT

    ```sql
    ALTER TABLE tab1 DROP CONSTRAINT TAB1_PK;
    ```

  - ADD CONSTRAINT

    ```sql
    ALTER TABLE tab1 ADD CONSTRAINT TAB1_PK PRIMARY KEY (id);
    ```

- **DROP TABLE**

  ```
  DROP TABLE tab1 
  ```

- **RENAME TABLE**

  ```sql
  RENAME tab1 TO tab2;
  ```

- TRUNCATE TABLE

  ```sql
  TRUNCATE TABLE tab1;
  ```





# DML (Data Manipulation Language)

INSERT, UPDATE, DELETE, SELECT  / 행 단위 변경



- **INSERT**

  ```sql
  INSERT INTO tab1 (id, name) VALUES (1, 'name1');
  INSERT INTO tab1 VALUES (2, 'name2');
  ```

- **UPDATE**

  ```sql
  UPDATE tab1 SET name = '홍길동'; -- 전체 변경
  UPDATE tab1 SET name = '김첨지' WHERE id = 8;
  ```

- **DELETE**

  ```sql
  DELETE [FROM] tab1; -- 전체 삭제
  DELETE [FROM] tab1 WHERE id = 3;
  ```

- **SELECT**

  ```sql
  SELECT id AS c1, name AS c2 FROM tab1;
  ```

  



# TCL (Transaction Control Language)

COMMIT, ROLLBACK, SAVEPOINT

























