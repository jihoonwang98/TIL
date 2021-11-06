# [NestJS 한국어 매뉴얼 - OVERVIEW] First steps

> https://docs.nestjs.kr/first-steps



## Language

- Nest는 TS도 지원하고 바닐라 JS도 지원함.
- 근데 Nest는 최신 언어를 활용하므로 바닐라 JS와 함께 사용하려면 [Babel](https://babeljs.io/) 컴파일러가 필요



## Node.js 버전

- Node.js (> 10.13.0, v13은 제외)

  

## 설치

- [Nest CLI](https://docs.nestjs.kr/cli/overview)를 사용하면 프로젝트 구성 쉽게 가능

  ```shell
  npm i -g @nestjs/cli
  nest new [project-name]
  ```

- 위와 같이 명령어를 입력하면 `project-name`이라는 폴더 안에 다음과 같은 파일들이 생성된다.

  ```shell
  mojo@MOJO-EX-MAC nest-prac % tree -I "node_modules"
  .
  ├── README.md
  ├── nest-cli.json  
  ├── package-lock.json
  ├── package.json  
  ├── src
  │   ├── app.controller.spec.ts
  │   ├── app.controller.ts
  │   ├── app.module.ts
  │   ├── app.service.ts
  │   └── main.ts
  ├── test
  │   ├── app.e2e-spec.ts
  │   └── jest-e2e.json
  ├── tsconfig.build.json
  └── tsconfig.json
  ```

  - `src/app.controller.spec.ts` 

    - 컨트롤러 유닛 테스트 파일

      ```ts
      import { Test, TestingModule } from '@nestjs/testing';
      import { AppController } from './app.controller';
      import { AppService } from './app.service';
      
      describe('AppController', () => {
        let appController: AppController;
      
        beforeEach(async () => {
          const app: TestingModule = await Test.createTestingModule({
            controllers: [AppController],
            providers: [AppService],
          }).compile();
      
          appController = app.get<AppController>(AppController);
        });
      
        describe('root', () => {
          it('should return "Hello World!"', () => {
            expect(appController.getHello()).toBe('Hello World!');
          });
        });
      });
      ```

  - `src/app.controller.ts` 

    - 하나의 라우트가 있는 기본 컨트롤러

      ```ts
      import { Controller, Get } from '@nestjs/common';
      import { AppService } from './app.service';
      
      @Controller()
      export class AppController {
        constructor(private readonly appService: AppService) {}
      
        @Get()
        getHello(): string {
          return this.appService.getHello();
        }
      }
      ```

  - `app.module.ts`

    - 애플리케이션의 root 모듈

      ```ts
      import { Module } from '@nestjs/common';
      import { AppController } from './app.controller';
      import { AppService } from './app.service';
      
      @Module({
        imports: [],
        controllers: [AppController],
        providers: [AppService],
      })
      export class AppModule {}
      ```

  - `app.service.ts`

    - 단일 메서드를 사용하는 기본 서비스

      ```ts
      import { Injectable } from '@nestjs/common';
      
      @Injectable()
      export class AppService {
        getHello(): string {
          return 'Hello World!';
        }
      }
      ```

  - `main.ts`

    - 핵심 기능 `NestFactory`를 사용하여 Nest 애플리케이션 인스턴스를 생성하는 애플리케이션의 엔트리 파일

    - 애플리케이션을 **bootstrap**하는 비동기 함수가 포함되어 있다.

      ```ts
      // main.ts
      import { NestFactory } from '@nestjs/core';
      import { AppModule } from './app.module';
      
      async function bootstrap() {
        const app = await NestFactory.create(AppModule);
        await app.listen(3000);
      }
      bootstrap();
      ```

      - `NestFactory.create()` 메서드는 `INestApplication` 인터페이스를 충족하는 애플리케이션 객체를 반환한다.



## Platform

- Nest는 플랫폼에 구애받지 않는 프레임워크를 목표로 합니다. 
  - 플랫폼 독립성을 통해 개발자가 여러 유형의 애플리케이션에서 활용할 수 있는 재사용 가능한 논리적 부분을 만들 수 있습니다. 
  - 기술적으로 Nest는 어댑터가 생성되면 모든 Node HTTP 프레임워크에서 작동할 수 있습니다. 
  - 기본적으로 지원되는 HTTP 플랫폼은 [express](https://expressjs.com/) 및 [fastify](https://www.fastify.io/)의 두가지입니다. 

|                    | 설명                                                         |
| ------------------ | ------------------------------------------------------------ |
| `platform-express` | [Express](https://expressjs.com/)는 노드용으로 잘 알려진 미니멀리스트 웹 프레임워크입니다. 커뮤니티에서 구현한 많은 리소스를 갖춘 수많은 테스트를 거친 프로덕션에 사용할 수 있도록 준비된 라이브러리입니다. 기본적으로 `@nestjs/platform-express` 패키지가 사용됩니다. 많은 사용자가 Express를 잘 사용하고 있으며 이를 활성화하기 위해 조치를 취할 필요가 없습니다. |
| `platform-fastify` | [Fastify](https://www.fastify.io/)는 최대의 효율성과 속도를 제공하는데 중점을 둔 고성능 및 낮은 오버헤드 프레임워크입니다. [여기](https://docs.nestjs.kr/techniques/performance)에서 사용 방법을 읽어보십시오. |

- 어떤 플랫폼을 사용하든 자체 애플리케이션 인터페이스를 노출합니다. 

  - 이들은 각각 `NestExpressApplication` 및 `NestFastifyApplication`으로 표시됩니다.

- 아래 예제와 같이 `NestFactory.create()` 메소드에 유형을 전달하면 `app` 객체는 해당 특정 플랫폼에서만 사용할 수 있는 메소드를 갖게 됩니다. 

  - 그러나 실제로 기본 플랫폼 API에 액세스하려는 경우를 **제외**하고는 유형을 지정할 **필요**는 없습니다.

  ```ts
  const app = await NestFactory.create<NestExpressApplication>(AppModule);
  ```