# NestJS + TypeORM + PostgreSQL 프로젝트 세팅



> [참고 velog](https://velog.io/@haileeyu21/NestJS-PostgreSQL%EA%B3%BC-%EC%97%B0%EB%8F%99-%EB%B0%8F-TypeORM-%EC%82%AC%EC%9A%A9%ED%95%B4%EC%84%9C-%ED%85%8C%EC%9D%B4%EB%B8%94-%EC%83%9D%EC%84%B1%ED%95%B4%EB%B3%B4%EA%B8%B0)





### 설치

```shell
//nestjs cli global module 설치
npm install -g @nestjs/cli

// nestjs 새로운 프로젝트 설치
nest new typeorm-test-project

cd typeorm-test-project

// typeorm 모듈 설치, postgresql 모듈 설치
yarn add @nestjs/typeorm typeorm pg
```



### TypeORM 모듈 import

- typeorm에 연결하기 위해서는 App module에서 typeorm module을 import 해주면 된다.
- module을 import 할 때는 TypeORM 모듈의 설정값이 동적이기 때문에 TypeORM module을 forRoot 메서드를 이용해서 import 해준다.

```typescript
import { TypeOrmModule } from '@nestjs/typeorm';

@Module({
  imports: [
    TypeOrmModule.forRoot({
      type: 'postgres',
      host: 'localhost',
      port: 5432,
      username: 'mojo',
      password: '',
      database: 'prac',
      entities: [],
      synchronize: false, // false가 안전함
    }),
  ],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

- synchronize 기능을 키면 작성된 entity와 database간의 sync를 맞춰준다.

  - 해당 기능을 사용하면 편할 수 있지만 원하지 않는 시점에 database 구조가 달라질 수 있으므로 주의해야 한다.

  - 그래서 entity 설정과 sync를 맞춰주기 위해서 typeorm cli를 이용해서 원하는 시점에 sync를 맞추는게 좋다.

    ```shell
    // entity와 sync를 맞춰줌
    yarn typeorm schema:sync
    ```







