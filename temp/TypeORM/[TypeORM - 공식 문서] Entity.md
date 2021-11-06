# [TypeORM - 공식 문서] Entity

> https://typeorm.io/#/entities



## What is Entity?

- Entity란?

  - Entity는 DB 테이블(또는 MongoDB의 collection)에 매핑되는 클래스다.

- Entity 클래스 생성

  - 새로운 클래스를 정의하고 `@Entity()`를 붙여주면 Entity 클래스를 만들 수 있다.

  - 예시

    ```typescript
    import {Entity, PrimaryGeneratedColumn, Column} from "typeorm";
    
    @Entity()
    export class User {
    
        @PrimaryGeneratedColumn()
        id: number;
    
        @Column()
        firstName: string;
      
        @Column()
        lastName: string;
      
        @Column()
        isActive: boolean;
    }
    ```

    위의 코드는 아래와 같은 DB 테이블을 생성한다.

    ```
    +-------------+--------------+----------------------------+
    |                          user                           |
    +-------------+--------------+----------------------------+
    | id          | int(11)      | PRIMARY KEY AUTO_INCREMENT |
    | firstName   | varchar(255) |                            |
    | lastName    | varchar(255) |                            |
    | isActive    | boolean      |                            |
    +-------------+--------------+----------------------------+
    ```



- 기본적으로 엔티티들은 columns와 relations들을 가지고 있다.

- 모든 엔티티는 **<u>반드시</u>** primary column(또는 MongoDB의 경우에는 ObjectId)를 가져야만 한다.

- 모든 엔티티는 반드시 connection options에 등록되어야만 한다.

  ```typescript
  import {createConnection, Connection} from "typeorm";
  import {User} from "./entity/User";
  
  const connection: Connection = await createConnection({
      type: "mysql",
      host: "localhost",
      port: 3306,
      username: "test",
      password: "test",
      database: "test",
      entities: [User]  // 여기에 추가되어야 함
  });
  ```

  - 근데 위 예시처럼 일일이 등록하기 귀찮으니까 아래처럼 지정해줄 수도 있따.

  ```typescript
  import {createConnection, Connection} from "typeorm";
  
  const connection: Connection = await createConnection({
      type: "mysql",
      host: "localhost",
      port: 3306,
      username: "test",
      password: "test",
      database: "test",
      entities: ["entity/*.js"]  
  });
  ```

  - entity 폴더 내에 있는 모든 js 파일들이 엔티티로 등록된다.

  - 또는 아래와 같이도 작성할 수 있다.

    ```typescript
    entities: [__dirname + '/**/*.entity{.ts,.js}'],
    ```

    

- 엔티티에 대응되는 테이블 이름
  - `@Entity("my_users")`: 테이블 이름이 `my_users`가 된다.
  - 또는 모든 DB 테이블에 prefix를 붙여주고 싶다면, connection options에 `entityPrefix`를 추가해줄 수도 있다.



- 주의
  - 원문: When using an entity constructor its arguments **must be optional**. Since ORM creates instances of entity classes when loading from the database, therefore it is not aware of your constructor arguments.
  - 엔티티 클래스의 생성자를 이용할 때 arguments들은 optional이어야만 한다. 왜냐하면 ORM이 DB로부터 entity를 불러올 때 해당 entity의 인스턴스를 만들기 때문에, 생성자 인수를 인식하지 못하게 된다. (생성자 인수에 undefined가 들어감).
    - 참고: [Entity로 테이블 만들기 - 호돌록](https://log.hodol.dev/typescript/typeorm/entity)
  - Learn more about parameters `@Entity` in [Decorators reference](https://typeorm.io/#/decorator-reference/).





## Entity Columns

- DB 테이블이 컬럼으로 구성되기 때문에 엔티티 역시 컬럼으로 구성되어야 한다.
  - `@Column`으로 표시한 엔티티 클래스 프로퍼티들은 DB 테이블 컬럼으로 매핑된다.



### Primary Columns

### Special Columns

### Spatial Columns





## C











