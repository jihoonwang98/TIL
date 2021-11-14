# [TypeORM] EntityManager, Repository, CustomRepository

> [What is EntityManager](https://typeorm.io/#/working-with-entity-manager), [What is Repository](https://typeorm.io/#/working-with-repository), [Custom repositories](https://typeorm.io/#/custom-repository), [EntityManager API](https://typeorm.io/#/entity-manager-api)



- EntityManager: 모든 entity repository를 하나로 합쳐놓은 느낌

  - access 하는 방법

    - `getManager()` or `getConnection().manager`

    ```ts
    import {getManager} from "typeorm";
    import {User} from "./entity/User";
    
    const entityManager = getManager();  // or getConnection().manager
    const user = await entityManager.findOne(User, 1);
    user.name = 'Umed';
    await entityManager.save(user);
    ```

- Repository: 하나의 구체적인 엔티티에 종속된 연산들을 수행할 수 있는 EntityManager라고 생각하면 된다.

  - access 하는 방법

    - `getRepository(Entity)` or `Connection#getRepository` or `EntityManager#getRepository`

    ```ts
    import {getRepository} from "typeorm";
    import {User} from "./entity/User";
    
    const userRepository = getRepository(User) // or getConnection().getRepository() or getManager().getRepository
    const user = await userRepository.findOne(1);
    user.name = 'Umed';
    await userRepository.save(user);
    ```

  - Repository의 3가지 종류

    - `Repository` - 모든 엔티티를 위한 Regular repository
    - `TreeRepository` - tree-entity(`@Tree`로 marking된 엔티티)들을 위한 `Repository`의 extension 형태
    - `MongoRepository` - MongoDB에서만 사용되는 special function들을 가지고 있는 Repository



- Custom Repository

  - 그냥 Repository 있는데 왜 Custom Repository가 필요하냐??

    - 우리가 `findByName(firstName: string, lastName: string)`이라는 메서드가 필요하다고 해보자. 
    - 이런 메서드가 위치하기에 가장 좋은 곳은 `Repository` 안이다.
    - `userRepository.findbyName(...)` 형태로 호출하기를 원하는 것이다.

  - Custom Repository 생성하는 방법

    - `CustomRepository` extends `Repository`

      ```ts
      @EntityRepository(User)
      export class UserRepository extends Repository<User> {
        
        findByName(firstName: string, lastName: string) {
          return this.findOne({
            firstName, lastName
          });
        }
      }
      ```

      - `@EntityRepository(엔티티명)`, `extends Repository<엔티티명>` 두개만 선언
      - 위와 같이 선언하고 난 후에는 아래와 같은 방법(`getCustomRepository(엔티티명)`)으로 access 가능



### Repository API

- `manager` - 이 repository에 의해 사용되는 `EntityManager`

  ```ts
  const manager = repository.manager;
  ```

- `queryRunner` - The query runner used by `EntityManager`. Used only in transactional instances of EntityManager.



























