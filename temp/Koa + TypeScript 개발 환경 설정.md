# Koa + TypeScript 개발 환경 설정

> https://medium.com/@masnun/typescript-with-koa-part-1-c4843f16a4ad



- `package.json` 생성

  ```js
  npm init -y
  ```

- Koa랑 관련된 패키지 설치

  ```js
  npm i -S koa koa-bodyparser koa-json koa-logger koa-router
  ```

- 디렉토리 두 개 만들기

  - `src`: Typescript 소스 파일 넣는 곳
  - `dist`: 컴파일된 JS 파일 넣을 곳

- `src/index.ts` 만들기

  ```js
  import Koa from 'koa';
  import Router from 'koa-router';
  import logger from 'koa-logger';
  import json from 'koa-json';
  import bodyParser from "koa-bodyparser";
  
  const app = new Koa();
  const router = new Router();
  
  router.get('/', async (ctx, next) => {
    ctx.body = {
      msg: 'hello, world!'
    };
  
    await next();
  });
  
  // Middlewares
  app.use(json());
  app.use(logger());
  app.use(bodyParser());
  
  // Routes
  app.use(router.routes()).use(router.allowedMethods());
  
  app.listen(3000, () => {
    console.log('koa started');
  })
  ```

- TS로 작성할거면 JS로 컴파일 하는 과정이 필요함

  - TS 컴파일러 설치해야 함

  ```
  npm i -g typescript
  ```

  - 위와 같이 글로벌로 설치하면 `tsc` 커맨드를 어디에서나 사용할 수 있다.

- `tsconfig.json` 작성

  - `tsc --init`: tsconfig.json 생성해주는거
  - `tsconfig.json`를 통해 컴파일러가 어떻게 작동하게 할지 설정할 수 있다.

  ```json
  {
    "version": "2.0.2",
    "compilerOptions": {
      "outDir": "./dist",
      "lib": ["es5", "es6"],
      "target": "es6",
      "module": "commonjs",
      "moduleResolution": "node",
      "emitDecoratorMetadata": true,
      "experimentalDecorators": true
    },
    "include": ["./src/**/*"]
  }
  ```

  - 위 설정을 통해 `src` 디렉토리 안에 있는 모든 파일을 TS 컴파일러에게 컴파일하라고 한 뒤, 그 결과물을 `dist` 폴더에 산출하라고 할 수 있다.

- nodejs specific type definintions 추가

  ```shell
  npm i -D @types/node
  ```

  - 이거 안 해주면 TS 컴파일러가 글로벌 객체를 모른다고 불평함. (`console` 같은 거)

- koa declaration 파일 for 'koa' 추가 (Adding Type Definitions)

  ```
  npm i --save-dev @types/koa @types/koa-router @types/koa-logger @types/koa-json @types/koa-bodyparser
  ```

- 코드 컴파일

  ```
  tsc
  ```

  - 위 명령 입력하면 `dist` 디렉토리에 `index.js` 파일이 생김

- 실행

  ```shell
  nodemon dist/index
  ```

  

- TSC watch 옵션 

  ```shell
  tsc -w
  ```

  - 파일 변화 감지해서 바로 자동 컴파일 해줌

- nodemon

  - 파일 변화 감지해서 서버 다시 띄워줌

- `tsc -w`, `nodemon .` 같이하기

  - TS 파일 변화 -> 자동 JS 파일로 컴파일
  - JS파일 컴파일(변화) -> nodemon이 서버 자동으로 다시 띄움















