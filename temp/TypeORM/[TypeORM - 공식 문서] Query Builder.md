# [TypeORM - 공식 문서] Query Builder 목차

> https://typeorm.io/#/select-query-builder



### 목차

- [What is `QueryBuilder`](https://typeorm.io/#/select-query-builder/what-is-querybuilder)
- [Important note when using the `QueryBuilder`](https://typeorm.io/#/select-query-builder/important-note-when-using-the-querybuilder)
- [How to create and use a `QueryBuilder`](https://typeorm.io/#/select-query-builder/how-to-create-and-use-a-querybuilder)
- [Getting values using QueryBuilder](https://typeorm.io/#/select-query-builder/getting-values-using-querybuilder)
- [What are aliases for?](https://typeorm.io/#/select-query-builder/what-are-aliases-for)
- [Using parameters to escape data](https://typeorm.io/#/select-query-builder/using-parameters-to-escape-data)
- [Adding `WHERE` expression](https://typeorm.io/#/select-query-builder/adding-where-expression)
- [Adding `HAVING` expression](https://typeorm.io/#/select-query-builder/adding-having-expression)
- [Adding `ORDER BY` expression](https://typeorm.io/#/select-query-builder/adding-order-by-expression)
- [Adding `GROUP BY` expression](https://typeorm.io/#/select-query-builder/adding-group-by-expression)
- [Adding `LIMIT` expression](https://typeorm.io/#/select-query-builder/adding-limit-expression)
- [Adding `OFFSET` expression](https://typeorm.io/#/select-query-builder/adding-offset-expression)
- [Joining relations](https://typeorm.io/#/select-query-builder/joining-relations)
- [Inner and left joins](https://typeorm.io/#/select-query-builder/inner-and-left-joins)
- [Join without selection](https://typeorm.io/#/select-query-builder/join-without-selection)
- [Joining any entity or table](https://typeorm.io/#/select-query-builder/joining-any-entity-or-table)
- [Joining and mapping functionality](https://typeorm.io/#/select-query-builder/joining-and-mapping-functionality)
- [Getting the generated query](https://typeorm.io/#/select-query-builder/getting-the-generated-query)
- [Getting raw results](https://typeorm.io/#/select-query-builder/getting-raw-results)
- [Streaming result data](https://typeorm.io/#/select-query-builder/streaming-result-data)
- [Using pagination](https://typeorm.io/#/select-query-builder/using-pagination)
- [Set locking](https://typeorm.io/#/select-query-builder/set-locking)
- [Max execution time](https://typeorm.io/#/select-query-builder/max-execution-time)
- [Partial selection](https://typeorm.io/#/select-query-builder/partial-selection)
- [Using subqueries](https://typeorm.io/#/select-query-builder/using-subqueries)
- [Hidden Columns](https://typeorm.io/#/select-query-builder/hidden-columns)
- [Querying Deleted rows](https://typeorm.io/#/select-query-builder/querying-deleted-rows)



## 1. `QueryBuilder`가 뭐냐??

- `QueryBuilder`란?

  - 우아하고 간편한 문법을 통해 SQL 쿼리를 build 해주는 놈
  - 그렇게 build된 쿼리를 수행해주고, 수행 결과를 자동으로 엔티티 형태로 가져올 수 있게 해준다.

- 예시

  ```ts
  const firstUser = await connection
      .getRepository(User)
      .createQueryBuilder("user")
      .where("user.id = :id", { id: 1 })
      .getOne();
  ```

  - 위의 statement는 아래와 같은 쿼리를 만들어 낸다.

  ```sql
  SELECT
      user.id as userId,
      user.firstName as userFirstName,
      user.lastName as userLastName
  FROM users user
  WHERE user.id = 1
  ```

  - 그리고 위의 쿼리 수행 결과를 `User`의 인스턴스 형태로 반환해준다.

  ```ts
  User {
      id: 1,
      firstName: "Timber",
      lastName: "Saw"
  }
  ```



## 2. `QueryBuilder`를 사용할 때 주의할 점

- `QueryBuilder`를 사용할 때, `WHERE`절에 unique parameter를 제공해줘야 한다.

- 아래의 예시는 작동하지 않는다.

  ```ts
  const result = await getConnection()
      .createQueryBuilder('user')
      .leftJoinAndSelect('user.linkedSheep', 'linkedSheep')
      .leftJoinAndSelect('user.linkedCow', 'linkedCow')
      .where('user.linkedSheep = :id', { id: sheepId })
      .andWhere('user.linkedCow = :id', { id: cowId });
  ```

  - `id`를 두 번 중복해서 써서 작동을 안함
  - 아래와 같이 이름을 구별해주자.

  ```ts
  const result = await getConnection()
      .createQueryBuilder('user')
      .leftJoinAndSelect('user.linkedSheep', 'linkedSheep')
      .leftJoinAndSelect('user.linkedCow', 'linkedCow')
      .where('user.linkedSheep = :sheepId', { sheepId })
      .andWhere('user.linkedCow = :cowId', { cowId });
  ```

  - 다른 파라미터에 대해 중복된 이름(`:id`)으로 두 번 쓰지 말고 각각 unique한 이름(`:sheepId`, `:cowId`)을 붙여주자.



## 3. `QueryBuilder`를 생성하고 사용하는 방법

- `QueryBuilder` 생성하는 3가지 방법

  - connection을 사용하는 방법

    - `await getConnection().createQueryBuilder()`

    ```ts
    import {getConnection} from "typeorm";
    
    const user = await getConnection()
        .createQueryBuilder()
        .select("user")
        .from(User, "user")
        .where("user.id = :id", { id: 1 })
        .getOne();
    ```

  - entity manager를 사용하는 방법

    - `await getManager().createQueryBuilder()`

    ```ts
    import {getManager} from "typeorm";
    
    const user = await getManager()
        .createQueryBuilder(User, "user")
        .where("user.id = :id", { id: 1 })
        .getOne();
    ```

  - repository를 사용하는 방법

    - `await getRepository(User).createQueryBuilder()`

    ```ts
    import {getRepository} from "typeorm";
    
    const user = await getRepository(User)
        .createQueryBuilder("user")
        .where("user.id = :id", { id: 1 })
        .getOne();
    ```

    

- `QueryBuilder`의 5가지 타입

  - `SelectQueryBuilder`: `SELECT` 쿼리 날릴때 사용

    ```ts
    import {getConnection} from "typeorm";
    
    const user = await getConnection()
        .createQueryBuilder()
        .select("user")
        .from(User, "user")
        .where("user.id = :id", { id: 1 })
        .getOne();
    ```

  - `InsertQueryBuilder`: `INSERT` 쿼리 날릴때 사용

    ```ts
    import {getConnection} from "typeorm";
    
    await getConnection()
        .createQueryBuilder()
        .insert()
        .into(User)
        .values([
            { firstName: "Timber", lastName: "Saw" },
            { firstName: "Phantom", lastName: "Lancer" }
         ])
        .execute();
    ```

  - `UpdateQueryBuilder`: `UPDATE` 쿼리 날릴때 사용

    ```ts
    import {getConnection} from "typeorm";
    
    await getConnection()
        .createQueryBuilder()
        .update(User)
        .set({ firstName: "Timber", lastName: "Saw" })
        .where("id = :id", { id: 1 })
        .execute();
    ```

  - `DeleteQueryBuilder`: `DELETE` 쿼리 날릴때 사용

    ```ts
    import {getConnection} from "typeorm";
    
    await getConnection()
        .createQueryBuilder()
        .delete()
        .from(User)
        .where("id = :id", { id: 1 })
        .execute();
    ```

  - `RelationQueryBuilder`: relation-specific한 연산[TBD]들을 빌드하고 수행할때 사용

- You can switch between different types of query builder within any of them, once you do, you will get a new instance of query builder (unlike all other methods).



## 4. Getting values using `QueryBuilder`

- DB에서 단건 조회하고 싶을때(예: get user by id or name), `getOne` 메서드를 사용하자.

  ```ts
  const timber = await getRepository(User)
      .createQueryBuilder("user")
      .where("user.id = :id OR user.name = :name", { id: 1, name: "Timber" })
      .getOne();
  ```

- `getOneOrFail` 메서드는 마찬가지로 단건 조회할때 사용하지만, 조회 결과가 존재하지 않을 때 `EntityNotFoundError`를 던져준다.

  ```ts
  const timber = await getRepository(User)
      .createQueryBuilder("user")
      .where("user.id = :id OR user.name = :name", { id: 1, name: "Timber" })
      .getOneOrFail();
  ```

- DB에서 데이터 여러개를 조회하고 싶을때(예: get all users) `getMany` 메서드를 사용하자.

  ```ts
  const users = await getRepository(User)
      .createQueryBuilder("user")
      .getMany();
  ```

  

- select query builder를 사용해서 얻을 수 있는 result의 종류는 두 가지다.

  - **entities**
  - **raw results**

- 대부분의 경우, DB로부터 엔티티 형태로 조회할 때가 많다.

  - 이때는 `getOne`이나 `getMany`를 쓰면 된다.

- 그런데 가끔 specific한 데이터를 가져와야할 때가 있다.

  - 예를 들면, 모든 user가 가진 money의 합을 구해야 하는 경우
  - 이럴때는 결과 값의 형태가 entity가 아니다.
  - 이런 데이터를 **raw data**라고 한다.
  - 이러한 raw data를 가져와야 하는 경우에는 `getRawOne`이나 `getRawMany`를 쓰면 된다.

  ```ts
  const { sum } = await getRepository(User)
      .createQueryBuilder("user")
      .select("SUM(user.photosCount)", "sum")
      .where("user.id = :id", { id: 1 })
      .getRawOne();
  ```

  

  ```ts
  const photosSums = await getRepository(User)
      .createQueryBuilder("user")
      .select("user.id")
      .addSelect("SUM(user.photosCount)", "sum")
      .groupBy("user.id")
      .getRawMany();
  
  // result will be like this: [{ id: 1, sum: 25 }, { id: 2, sum: 13 }, ...]
  ```

  

## 5. alias에 대해 알아보자.

- 위의 코드에서 `createQueryBuilder("user")`란 코드를 사용했다.

  - 근데 여기서 문자열 "user"가 의미하는게 뭘까?
  - 이것은 단지 일반 SQL 별칭일뿐입니다.

- `createQueryBuilder("user")`는 아래의 코드와 동등합니다.

  ```ts
  createQueryBuilder()
      .select("user")
      .from(User, "user")
  ```

  - 위의 코드는 아래와 같은 sql 쿼리를 만들어냅니다.

  ```sql
  SELECT ... FROM users user
  ```

- alias를 설정했으면, 테이블에 접근할 때 이 alias를 이용할 수 있다.

  ```ts
  createQueryBuilder()
      .select("user")
      .from(User, "user")
      .where("user.name = :name", { name: "Timber" })
  ```

  - 위의 statement는 아래와 같은 쿼리로 변환된다.

  ```sql
  SELECT ... FROM users user WHERE user.name = 'Timber'
  ```

  

## 6. Using parameters to escape data

- `where("user.name = :name", { name: "Timber" })`를 쓸때, `{ name: "Timber" }`가 의미하는것이 뭐냐?
  - `:name`이 파라미터명이고 이에 해당하는 값이 객체 `{ name: "Timber" }`안에 명시되는 것이다.



```ts
.where("user.name = :name", { name: "Timber" })
```

- 위 코드는 아래 코드의 shortcut이다.

```ts
.where("user.name = :name")
.setParameter("name", "Timber")
```





- IN 쿼리를 쓸때 special expansion syntax(`...`)를 사용하면 배열을 넘겨줄 수도 있다.

```ts
.where("user.name IN (:...names)", { names: [ "Timber", "Cristal", "Lina" ] })
```





## 7. Adding `WHERE` expression

- `WHERE`절을 추가하는 것은 쉽다.

  ```ts
  createQueryBuilder("user")
      .where("user.name = :name", { name: "Timber" })
  ```

  ```sql
  SELECT ... FROM users user WHERE user.name = 'Timber'
  ```

- 이미 존재하고 있는 `WHERE` 절에 `AND` 조건을 추가할 수도 있다.

  ```ts
  createQueryBuilder("user")
      .where("user.firstName = :firstName", { firstName: "Timber" })
      .andWhere("user.lastName = :lastName", { lastName: "Saw" });
  ```

  ```sql
  SELECT ... FROM users user WHERE user.firstName = 'Timber' AND user.lastName = '
  ```

- 이미 존재하고 있는 `WHERE` 절에 `OR` 조건을 추가할 수도 있다.

  ```ts
  createQueryBuilder("user")
      .where("user.firstName = :firstName", { firstName: "Timber" })
      .orWhere("user.lastName = :lastName", { lastName: "Saw" });
  ```

  ```sql
  SELECT ... FROM users user WHERE user.firstName = 'Timber' OR user.lastName = 'Saw'
  ```

- `WHERE`절에 `IN` 쿼리를 사용할 수도 있다.

  ```ts
  createQueryBuilder("user")
      .where("user.id IN (:...ids)", { ids: [1, 2, 3, 4] })
  ```

  ```sql
  SELECT ... FROM users user WHERE user.id IN (1, 2, 3, 4)
  ```

- 복잡한 `WHERE` 조건을 `Brackets`를 이용해서 만들 수도 있다.

  ```ts
  createQueryBuilder("user")
      .where("user.registered = :registered", { registered: true })
      .andWhere(new Brackets(qb => {
          qb.where("user.firstName = :firstName", { firstName: "Timber" })
            .orWhere("user.lastName = :lastName", { lastName: "Saw" })
      }))
  ```

  ```sql
  SELECT ... FROM users user WHERE user.registered = true AND (user.firstName = 'Timber' OR user.lastName = 'Saw')
  ```



- 만약에 `andWhere`나 `orWhere` 사용안하고 그냥 `where` 메서드를 체이닝해버리면 이전의 `where`절은 override 되어 버린다.





## 8. `HAVING`절

- `HAVING`절을 추가하는 것은 쉽다.

  ```ts
  createQueryBuilder("user")
      .having("user.name = :name", { name: "Timber" })
  ```

  ```sql
  SELECT ... FROM users user HAVING user.name = 'Timber'
  ```

- `AND` 추가 가능

  ```ts
  createQueryBuilder("user")
      .having("user.firstName = :firstName", { firstName: "Timber" })
      .andHaving("user.lastName = :lastName", { lastName: "Saw" });
  ```

  ```sql
  SELECT ... FROM users user HAVING user.firstName = 'Timber' AND user.lastName = 'Saw'
  ```

- `OR` 추가 가능

  ```ts
  createQueryBuilder("user")
      .having("user.firstName = :firstName", { firstName: "Timber" })
      .orHaving("user.lastName = :lastName", { lastName: "Saw" });
  ```

  ```sql
  SELECT ... FROM users user HAVING user.firstName = 'Timber' OR user.lastName = 'Saw'
  ```

  

- having도 한번 이상쓰면 전에거 override됨



## 9. `ORDER BY` 표현식 추가하기

- `ORDER BY` 

  ```ts
  createQueryBuilder("user")
      .orderBy("user.id")
  ```

  ```sql
  SELECT ... FROM users user ORDER BY user.id
  ```

- 오름차순, 내림차순

  ```ts
  createQueryBuilder("user")
      .orderBy("user.id", "DESC")
  
  createQueryBuilder("user")
      .orderBy("user.id", "ASC")
  ```

- multiple order-by criteria 

  `.addOrderBy()` 사용

  ```ts
  createQueryBuilder("user")
      .orderBy("user.name")
      .addOrderBy("user.id");
  ```

  또는 

  map of order-by fields 사용하기

  ```ts
  createQueryBuilder("user")
      .orderBy({
          "user.name": "ASC",
          "user.id": "DESC"
      });
  ```



- `.orderBy`도 마찬가지로 한번 이상 쓰면 override된다.



## 10. `DISTINCT ON` (Postgres only)

- distinct-on을 order-by와 함께 쓸때, distinct-on 표현식은 leftmost order-by와 반드시 매칭되어야 한다.
- distinct-on 표현식은 order-by와 같은 rule이 적용된다.
- **주의**
  - order-by 없이 distinct-on을 사용하는 것은 각 set의 first row를 예측할 수 없다.



- `DISTINCT ON`

  ```ts
  createQueryBuilder("user")
      .distinctOn(["user.id"])
      .orderBy("user.id")
  ```

  ```sql
  SELECT DISTINCT ON (user.id) ... FROM users user ORDER BY user.id
  ```

  



## 11. `GROUP BY`

- `GROUP BY`

  ```ts
  createQueryBuilder("user")
      .groupBy("user.id")
  ```

  ```sql
  SELECT ... FROM users user GROUP BY user.id
  ```

  

- 더 많은 group-by criteria를 추가하려면 `addGroupBy`를 사용한다.

  ```ts
  createQueryBuilder("user")
      .groupBy("user.name")
      .addGroupBy("user.id");
  ```



- 얘도 groupBy만 두번쓰면 이전거 override됨





## 12. `LIMIT`

- `LIMIT`

  ```ts
  createQueryBuilder("user")
      .limit(10)
  ```

  ```
  SELECT ... FROM users user LIMIT 10
  ```

- The resulting SQL query depends on the type of database (SQL, mySQL, Postgres, etc).

- Note: **<u>LIMIT may not work as you may expect if you are using complex queries with joins or subqueries.</u>** 

- If you are using pagination, it's recommended to use `take` instead.



## 13. `OFFSET`

- `OFFSET`

  ```ts
  createQueryBuilder("user")
      .offset(10)
  ```

  ```sql
  SELECT ... FROM users user OFFSET 10
  ```

- The resulting SQL query depends on the type of database (SQL, mySQL, Postgres, etc). 

- Note: OFFSET may not work as you may expect if you are using complex queries with joins or subqueries. 

- If you are using pagination, it's recommended to use `skip` instead.



## 14. Join













