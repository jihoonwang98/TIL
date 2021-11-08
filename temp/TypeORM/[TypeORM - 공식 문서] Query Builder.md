# [TypeORM - 공식 문서] Query Builder

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

```ts
import {Entity, PrimaryGeneratedColumn, Column, OneToMany} from "typeorm";
import {Photo} from "./Photo";

@Entity()
export class User {

    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    name: string;

    @OneToMany(type => Photo, photo => photo.user)
    photos: Photo[];
}
```

```ts
import {Entity, PrimaryGeneratedColumn, Column, ManyToOne} from "typeorm";
import {User} from "./User";

@Entity()
export class Photo {

    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    url: string;

    @ManyToOne(type => User, user => user.photos)
    user: User;
}
```



- "Timber"라는 유저를 그의 photo들과 함께 조회해보자.

  ```ts
  const user = await createQueryBuilder("user")
      .leftJoinAndSelect("user.photos", "photo")
      .where("user.name = :name", { name: "Timber" })
      .getOne();
  ```

  - 아래와 같은 결과를 얻는다.

  ```json
  {
      id: 1,
      name: "Timber",
      photos: [{
          id: 1,
          url: "me-with-chakram.jpg"
      }, {
          id: 2,
          url: "me-with-trees.jpg"
      }]
  }
  ```

  

- `leftJoinAndSelect(relation, alias)`
  - `relation`: `"user.photos"`
    - `user`의 연관 엔티티인 `photo`를 가져와라
    - The first argument is the relation you want to load
  - `alias`: `"photo"`
    - `photo` 테이블의 alias를 `"photo"`로 지정하자
    - the second argument is an alias you assign to this relation's table



- 예시

  ```ts
  const user = await createQueryBuilder("user")
      .leftJoinAndSelect("user.photos", "photo")
      .where("user.name = :name", { name: "Timber" })
      .andWhere("photo.isRemoved = :isRemoved", { isRemoved: false })
      .getOne();
  ```

  ```sql
  SELECT user.*, photo.* FROM users user
      LEFT JOIN photos photo ON photo.user = user.id
      WHERE user.name = 'Timber' AND photo.isRemoved = FALSE
  ```

  

- "where"를 쓰는 대신 join expression에 condition을 추가할 수도 있다.

  ```ts
  const user = await createQueryBuilder("user")
      .leftJoinAndSelect("user.photos", "photo", "photo.isRemoved = :isRemoved", { isRemoved: false })
      .where("user.name = :name", { name: "Timber" })
      .getOne();
  ```

  ```sql
  SELECT user.*, photo.* FROM users user
      LEFT JOIN photos photo ON photo.user = user.id AND photo.isRemoved = FALSE
      WHERE user.name = 'Timber'
  ```

  



## 15. Inner and left joins

- `LEFT JOIN` 대신 `INNER JOIN`을 사용하고 싶으면 `innerJoinAndSelect`를 사용하면 된다.

  ```ts
  const user = await createQueryBuilder("user")
      .innerJoinAndSelect("user.photos", "photo", "photo.isRemoved = :isRemoved", { isRemoved: false })
      .where("user.name = :name", { name: "Timber" })
      .getOne();
  ```

  ```sql
  SELECT user.*, photo.* FROM users user
      INNER JOIN photos photo ON photo.user = user.id AND photo.isRemoved = FALSE
      WHERE user.name = 'Timber'
  ```





## 16. Join without selection

- `select` 없이 data를 join 할 수 있다.

  - 이를 위해서, `leftJoin`이나 `innerJoin`을 사용하자.

  ```ts
  const user = await createQueryBuilder("user")
      .innerJoin("user.photos", "photo")
      .where("user.name = :name", { name: "Timber" })
      .getOne();
  ```

  ```sql
  SELECT user.* FROM users user
      INNER JOIN photos photo ON photo.user = user.id
      WHERE user.name = 'Timber'
  ```

  

## 17. Joining any entity or table (관계 없는 놈들끼리 조인하기)

```ts
const user = await createQueryBuilder("user")
    .leftJoinAndSelect(Photo, "photo", "photo.userId = user.id")
    .getMany();
```

```ts
const user = await createQueryBuilder("user")
    .leftJoinAndSelect("photos", "photo", "photo.userId = user.id")
    .getMany();
```





## 18. Joining and mapping functionality

- `leftJoinAndMapOne()` 메서드 

Add `profilePhoto` to `User` entity and you can map any data into that property using `QueryBuilder`:

```typescript
export class User {
    /// ...
    profilePhoto: Photo;

}
const user = await createQueryBuilder("user")
    .leftJoinAndMapOne("user.profilePhoto", "user.photos", "photo", "photo.isForProfile = TRUE")
    .where("user.name = :name", { name: "Timber" })
    .getOne();
```

This will load Timber's profile photo and set it to `user.profilePhoto`. If you want to load and map a single entity use `leftJoinAndMapOne`. If you want to load and map multiple entities use `leftJoinAndMapMany`.





## 19. Getting the generated query (쿼리 로깅)

- `QueryBuilder`에 의해 생성된 SQL 쿼리를 보고 싶으면 `getSql`을 쓰면 된다

  ```ts
  const sql = createQueryBuilder("user")
      .where("user.firstName = :firstName", { firstName: "Timber" })
      .orWhere("user.lastName = :lastName", { lastName: "Saw" })
      .getSql();
  ```

- 디버깅 하고 싶으면 `printSql`을 사용하면 된다.

  ```ts
  const users = await createQueryBuilder("user")
      .where("user.firstName = :firstName", { firstName: "Timber" })
      .orWhere("user.lastName = :lastName", { lastName: "Saw" })
      .printSql()
      .getMany();
  ```

  - This query will return users and print the used sql statement to the console.





## 20. Getting raw results

- select query builder를 사용하면 두가지 종류의 결과를 얻을 수 있다.

  - **entities**
  - **raw results**

- raw results를 얻어야 하는 경우

  - ex) sum of all user photos

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

    

## 21. Streaming result data

- You can use `stream` which returns you a stream.

- Streaming returns you raw data and you must handle entity transformation manually:

  ```ts
  const stream = await getRepository(User)
      .createQueryBuilder("user")
      .where("user.id = :id", { id: 1 })
      .stream();
  ```

  



## 22. Using pagination

- skip: 건너뛸 개수
- take: 가져올 개수



```ts
const users = await getRepository(User)
    .createQueryBuilder("user")
    .leftJoinAndSelect("user.photos", "photo")
    .take(10)
    .getMany();
```



- `take` and `skip` may look like we are using `limit` and `offset`, but they aren't.
- `limit` and `offset` may not work as you expect once you have more complicated queries with joins or subqueries. 
- <u>**Using `take` and `skip` will prevent those issues.**</u>





## 23. Set locking

- QueryBuilder supports both optimistic and pessimistic locking. 

- To use pessimistic read locking use the following method:

  ```typescript
  const users = await getRepository(User)
      .createQueryBuilder("user")
      .setLock("pessimistic_read")
      .getMany();
  ```

- 
  To use pessimistic write locking use the following method:

  ```typescript
  const users = await getRepository(User)
      .createQueryBuilder("user")
      .setLock("pessimistic_write")
      .getMany();
  ```

- To use dirty read locking use the following method:

  ```typescript
  const users = await getRepository(User)
      .createQueryBuilder("user")
      .setLock("dirty_read")
      .getMany();
  ```

- To use optimistic locking use the following method:

  ```typescript
  const users = await getRepository(User)
      .createQueryBuilder("user")
      .setLock("optimistic", existUser.version)
      .getMany();
  ```





## 24. Max execution time

```ts
const users = await getRepository(User)
    .createQueryBuilder("user")
    .maxExecutionTime(1000) // milliseconds.
    .getMany();
```

- We can drop slow query to avoid crashing the server. 
- Only MySQL driver is supported at the moment:





## 25. Partial selection

- If you want to select only some entity properties, you can use the following syntax:

```typescript
const users = await getRepository(User)
    .createQueryBuilder("user")
    .select([
        "user.id",
        "user.name"
    ])
    .getMany();
```

This will only select the `id` and `name` of `User`.





## 26. Using subqueries

- You can easily create subqueries. Subqueries are supported in `FROM`, `WHERE` and `JOIN` expressions. 

```ts
const qb = await getRepository(Post).createQueryBuilder("post");
const posts = qb
    .where("post.title IN " + qb.subQuery().select("user.name").from(User, "user").where("user.registered = :registered").getQuery())
    .setParameter("registered", true)
    .getMany();
```

- 좀 더 우아한 방법

```ts
const posts = await connection.getRepository(Post)
    .createQueryBuilder("post")
    .where(qb => {
        const subQuery = qb.subQuery()
            .select("user.name")
            .from(User, "user")
            .where("user.registered = :registered")
            .getQuery();
        return "post.title IN " + subQuery;
    })
    .setParameter("registered", true)
    .getMany();
```

- Alternatively, you can create a separate query builder and use its generated SQL:

```ts
const userQb = await connection.getRepository(User)
    .createQueryBuilder("user")
    .select("user.name")
    .where("user.registered = :registered", { registered: true });

const posts = await connection.getRepository(Post)
    .createQueryBuilder("post")
    .where("post.title IN (" + userQb.getQuery() + ")")
    .setParameters(userQb.getParameters())
    .getMany();
```

- `FROM`에서도 subquery를 쓸 수 있다.

```ts
const userQb = await connection.getRepository(User)
    .createQueryBuilder("user")
    .select("user.name", "name")
    .where("user.registered = :registered", { registered: true });

const posts = await connection
    .createQueryBuilder()
    .select("user.name", "name")
    .from("(" + userQb.getQuery() + ")", "user")
    .setParameters(userQb.getParameters())
    .getRawMany();
```

- 더 우아한 방법도 있다.

```ts
const posts = await connection
    .createQueryBuilder()
    .select("user.name", "name")
    .from(subQuery => {
        return subQuery
            .select("user.name", "name")
            .from(User, "user")
            .where("user.registered = :registered", { registered: true });
    }, "user")
    .getRawMany();
```



- If you want to add a subselect as a "second from" use `addFrom`.

  You can use subselects in `SELECT` statements as well:

  ```typescript
  const posts = await connection
      .createQueryBuilder()
      .select("post.id", "id")
      .addSelect(subQuery => {
          return subQuery
              .select("user.name", "name")
              .from(User, "user")
              .limit(1);
      }, "name")
      .from(Post, "post")
      .getRawMany();
  ```



## 27. Hidden Columns

If the model you are querying has a column with a `select: false` column, you must use the `addSelect` function in order to retrieve the information from the column.

Let's say you have the following entity:

```typescript
import {Entity, PrimaryGeneratedColumn, Column} from "typeorm";

@Entity()
export class User {

    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    name: string;

    @Column({select: false})
    password: string;
}
```

Using a standard `find` or query, you will not receive the `password` property for the model. However, if you do the following:

```typescript
const users = await connection.getRepository(User)
    .createQueryBuilder()
    .select("user.id", "id")
    .addSelect("user.password")
    .getMany();
```

You will get the property `password` in your query.



## 28. Querying Deleted rows

If the model you are querying has a column with the attribute `@DeleteDateColumn` set, the query builder will automatically query rows which are 'soft deleted'.

Let's say you have the following entity:

```typescript
import {Entity, PrimaryGeneratedColumn, Column} from "typeorm";

@Entity()
export class User {

    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    name: string;

    @DeleteDateColumn()
    deletedAt?: Date;
}
```

Using a standard `find` or query, you will not receive the rows which have a value in that row. However, if you do the following:

```typescript
const users = await connection.getRepository(User)
    .createQueryBuilder()
    .select("user.id", "id")
    .withDeleted()
    .getMany();
```

You will get all the rows, including the ones which are deleted.

