# NestJS + TypeORM + PostgreSQL 프로젝트 세팅



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