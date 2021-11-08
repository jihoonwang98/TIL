# [TypeORM - 공식 문서] Find Options

> https://typeorm.io/#/find-options



## Basic Options

- `find` 메서드의 옵션들 정리

  - `select`: 엔티티의 어떤 프로퍼티를 select 할지를 나타냄

    ```ts
    userRepository.find({ select: ["firstName", "lastName"] });
    ```

    ```ts
    SELECT "firstName", "lastName" FROM "user"
    ```

  - `relations`: 메인 엔티티와 함께 불러올 relation들을 지정함. Sub-relation들도 불러와질 수 있다. (shorthand for `join` and `leftJoinAndSelect`)

    ```ts
    userRepository.find({ relations: ["profile", "photos", "videos"] });
    userRepository.find({
        relations: ["profile", "photos", "videos", "videos.video_attributes"],
    });
    ```

    ```sql
    SELECT * FROM "user"
    LEFT JOIN "profile" ON "profile"."id" = "user"."profileId"
    LEFT JOIN "photos" ON "photos"."id" = "user"."photoId"
    LEFT JOIN "videos" ON "videos"."id" = "user"."videoId"
    
    SELECT * FROM "user"
    LEFT JOIN "profile" ON "profile"."id" = "user"."profileId"
    LEFT JOIN "photos" ON "photos"."id" = "user"."photoId"
    LEFT JOIN "videos" ON "videos"."id" = "user"."videoId"
    LEFT JOIN "video_attributes" ON "video_attributes"."id" = "videos"."video_attributesId"
    ```

  - `join`: `relations`를 좀 더 구체적으로 쓸 수 있는 버전이다.

    ```ts
    userRepository.find({
        join: {
            alias: "user",
            leftJoinAndSelect: {
                profile: "user.profile",
                photo: "user.photos",
                video: "user.videos",
            },
        },
    });
    ```

    ```sql
    SELECT * FROM "user" "user"
    LEFT JOIN "profile" ON "profile"."id" = "user"."profile"
    LEFT JOIN "photo" ON "photo"."id" = "user"."photos"
    LEFT JOIN "video" ON "video"."id" = "user"."videos"
    ```

  - `where`: simple conditions by which entity should be queried.

    ```ts
    userRepository.find({ where: { firstName: "Timber", lastName: "Saw" } });
    ```

    ```sql
    SELECT * FROM "user"
    WHERE "firstName" = 'Timber' AND "lastName" = 'Saw'
    ```

    

    - embedded entity의 컬럼에 대해 where 조건을 추가하는 경우

    ```ts
    userRepository.find({
        where: {
            project: { name: "TypeORM", initials: "TORM" },
        },
        relations: ["project"],
    });
    ```

    ```sql
    SELECT * FROM "user"
    WHERE "project"."name" = 'TypeORM' AND "project"."initials" = 'TORM'
    LEFT JOIN "project" ON "project"."id" = "user"."projectId"
    ```

    

    - OR 연산자로 쿼리하는 경우

    ```ts
    userRepository.find({
        where: [
            { firstName: "Timber", lastName: "Saw" },
            { firstName: "Stan", lastName: "Lee" },
        ],
    });
    ```

    ```sql
    SELECT * FROM "user" WHERE ("firstName" = 'Timber' AND "lastName" = 'Saw') OR ("firstName" = 'Stan' AND "lastName" = 'Lee')
    ```

  - `order`: 순서 정하기

    ```ts
    userRepository.find({
        order: {
            name: "ASC",
            id: "DESC",
        },
    });
    ```

    ```sql
    SELECT * FROM "user"
    ORDER BY "name" ASC, "id" DESC
    ```

  - `withDeleted`:  include entities which have been soft deleted with `softDelete` or `softRemove`, e.g. have their `@DeleteDateColumn` column set. By default, soft deleted entities are not included.

    ```ts
    userRepository.find({
        withDeleted: true,
    });
    ```

    

- 여러 개의 entity를 리턴하는 조회 메서드들 (`find`, `findAndCount`, `findByIds`)은 다음과 같은 옵션들도 지원한다.

  - `skip`: offset (paginated) from where entities should be taken.

    ```ts
    userRepository.find({
        skip: 5,
    });
    ```

    ```sql
    SELECT * FROM "user"
    OFFSET 5
    ```

  - `take`: limit (paginated) - max number of entities that should be taken.

    ```ts
    userRepository.find({
        take: 10,
    });
    ```

    ```sql
    SELECT * FROM "user"
    LIMIT 10
    ```

    

- `cache` - Enables or disables query result caching. See [caching](https://typeorm.io/#/caching/) for more information and options.

  ```ts
  userRepository.find({
      cache: true,
  });
  ```

- `lock` - Enables locking mechanism for query. Can be used only in `findOne` method. `lock` is an object which can be defined as:

  ```ts
  { mode: "optimistic", version: number|Date }
  ```

  or

  ```ts
  {
      mode: "pessimistic_read" |
          "pessimistic_write" |
          "dirty_read" |
          "pessimistic_partial_write" |
          "pessimistic_write_or_fail" |
          "for_no_key_update";
  }
  ```

  