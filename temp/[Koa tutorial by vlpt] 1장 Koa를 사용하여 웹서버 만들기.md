[Koa tutorial by vlpt] 1장 Koa를 사용하여 웹서버 만들기

> https://backend-intro.vlpt.us/



### Koa란?

- Koa
  - Express 개발팀이 Koa라는 웹 프레임워크를 새로 만듦.
  - Express와의 큰 차이는, Koa는 훨씬 가볍고, Node.js v7의 `async/await` 기능을 아주 편하게 사용할 수 있다는 점이다.



## \#1. 프로젝트 생성 및 ESLint 설정

- 프로젝트 생성
  1. 웹서버 프로젝트를 위한 디렉토리 생성
  2. 해당 디렉토리에서 `yarn init` (패키지 정보 생성)
  3. 그 다음에, koa 설치 `yarn add koa`



- ESLint 설정

  - ESLint란?

    - JS 문법을 검토해주는 도구

  - ESLint 설치 (글로벌)

    ```shell
    $ npm install -g eslint
    ```

  - 설치한 후, 프로젝트 내에서 다음 명령어 실행

    ```shell
    $ eslint --init
    ```

    - 그 다음에 나오는 질문은 각자의 세팅에 맞게 답변 선택
    - 이 작업이 끝나면, 프로젝트 루트 디렉토리에 `.eslintrc.js`라는 파일이 생긴다.



## \#2. Koa 기본 사용법

- Hello Koa

  - 먼저 서버를 여는 방법부터 알아보자.

  - 다음 코드를 src 디렉토리를 생성하여 `index.js` 파일을 만든 다음에 그 내부에 작성하자.

  - `src/index.js`

    ```typescript
    const Koa = require('koa');
    const app = new Koa();
    
    app.use(ctx => {
        ctx.body = 'Hello Koa';
    });
    
    app.listen(4000, () => {
        console.log('heurm server is listening to port 4000');
    });
    ```

  - 서버 실행

    ```
    $ node src
    heurm server is listening to port 4000
    ```

  - 서버에 요청 보내기

    - 브라우저로 ` http://localhost:4000/` 접속

      ![](https://backend-intro.vlpt.us/images/hello-koa.png)



- 미들웨어

  - Koa 애플리케이션은 미들웨어의 배열로 구성되어있다.

  - 우리는 위의 예제 코드에서 `app.use`라는 함수를 사용했었다.

    - 이 코드는 미들웨어를 애플리케이션에 등록해준다.

      ```typescript
      app.use(ctx => {
      	ctx.body = 'Hello Koa';
      });
      ```

      - 위에서 `ctx => { ... }` 코드가, 하나의 미들웨어인 것이다.
      - koa의 미들웨어 함수에선, 두 가지의 파라미터를 받는다.
        - 첫째는 위 코드에서도 나오는 `ctx`이고,
        - 둘째는 `next`이다.

    - `ctx`는, 웹 요청과, 응답에 대한 정보를 지니고 있고, `next`는 다음 미들웨어를 실행시키는 함수입니다.

    - 만약에 미들웨어에서 `next`를 호출하지 않게 된다면, 그 부분에서 요청 처리를 완료시키고, 응답을 하게 됩니다.

    - 그리고, 미들웨어는, 등록하는 순서대로 실행을 하게 됩니다. 

      - 우선 다음과 같이 기록을 하는 미들웨어 두 개를 기존 미들웨어 상단에 넣어보자.

      - `src/index.js` 

        ```typescript
        const Koa = require('koa');
        const app = new Koa();
        
        app.use(ctx => {
          console.log(1);
        });
        
        app.use(ctx => {
          console.log(2);
        });
        
        app.use(ctx => {
          ctx.body = 'Hello Koa';
        });
        
        app.listen(4000, () => {
          console.log('heurm server is listening to port 4000');
        });
        ```

        - 브라우저에서 `http://localhost:4000/`에 들어가면, Not Found가 뜨고, 터미널쪽에선 다음과 같이 기록이 된다.

          ```shell
          $ node src
          heurm server is listening to port 4000
          1
          ```

        - 그 다음 미들웨어를 실행하지 않고, 첫번째 미들웨어에서 멈춰버렸다.

          - 이는 `next()`를 호출하지 않았기 때문이다.

          - 함수의 파라미터 부분에 `next`를 받아와서 호출 해보자.

            ```typescript
            (...)
             
            app.use((ctx, next) => {
                console.log(1);
                next();
            });
            
            
            app.use((ctx, next) => {
                console.log(2);
                next();
            });
            
            (...)
             
            [결과]
            $ node src
            heurm server is listening to port 4000
            1
            2
            ```

- `next()`는 프로미스

  - `next()`를 실행하면, 프로미스를 반환한다.

  - 따라서, 작업들이 끝나고 나서 할 작업들을 정해줄 수도 있다.

  - 첫번째 미들웨어의 코드를 다음과 같이 수정하고, 다시 페이지를 열어보자.

    ```typescript
    app.use((ctx, next) => {
        console.log(1);
        const started = new Date();
        next().then(() => {
            console.log(new Date() - started + 'ms');
        });
    });
    ```

    ```shell
    $ node src
    heurm server is listening to port 4000
    1
    2
    1ms
    ```

    - 코드를, Express처럼 콜백 위주로만 한다면, 이런걸 하기는 굉장히 번거롭다. 하지만 Koa에서는 매우 편하다.



- `async/await` 사용하기

  - Koa에서는 별도의 작업없이 `async/await`를 바로 사용할 수 있습니다.

  - 방금 작성했던 미들웨어를, `async/await`을 통하여 재작성해보자.

    ```typescript
    app.use(async (ctx, next) => {
    	console.log(1);
    	const started = new Date();
      await next();
      console.log(new Date() - started + 'ms');
    });
    ```

    - 이 코드는 아까랑 동일하게 작동한다.
    - 이렇게 미들웨어에서 `next()`를 기다리는 것 말고도, 서버측에서 `async/await`는 자주 사용된다.
    - 이는 DB에 요청을 할 때 매우 유용하게 사용된다.

    

## \#3. Nodemon 사용하기

- 서버 코드를 변경할 때마다 서버 재시작하기 귀찮다..

  - `nodemon`을 사용하자.

- `nodemon` 설치

  - `npm install -g nodemon` (글로벌)

- `nodemon` 실행

  - `nodemon --watch src/ src/index.js`

  - 위 명령어를 통해서 서버를 실행하면 코드가 바뀔때마다 자동으로 재시작 해준다.

  - 위 명령어는, `src/` 디렉토리에서 코드변화가 감지하면 재시작을 하도록 설정하고,

    `src/index.js`를 실행시켜준다.

  - 개발을 할 때마다 이 명령어를 작성하기 귀찮으니 이 명령어를 npm scripts에 추가하자.

    - `packages.json` 

      ```json
        (...)
        "scripts": {
          "start": "node src",
          "start:dev": "nodemon --watch src/ src/index.js"
        }
      }
      ```

    - 서버 실행은 `yarn start`

    - 개발 모드로 킬 때는 `yarn start:dev`



## \#4. koa-router 사용하기

- koa-router란?

  - 요청이 들어왔을 때, 경로에 따라 다른 작업을 할 수 있게 해주는 `koa-router`에 대해서 알아보자.
  - 이는 Koa에 내장되어 있는 것이 아니므로, 따로 모듈을 설치해주어야 한다.

- koa-router 설치

  - `yarn add koa-router`

- koa-router 기본 사용법

  - `src/index.js`

    ```typescript
    const Koa = require('koa');
    const Router = require('koa-router');
    
    const app = new Koa();
    const router = new Router();
    
    router.get('/', (ctx, next) => {
      ctx.body = '홈';
    });
    
    app.use(router.routes());
    app.use(router.allowedMethods());
    
    app.listen(4000, () => {
      console.log('heurm server is listening to port 4000');
    });
    ```

    - Router 인스턴스를 새로 생성하여 `router` 값에 넣었고, `/` 경로로 들어오면 홈이라는 내용으로 응답하도록 라우터를 설정하였다.

- 여러 개의 라우트, 라우트 파라미터

  - 예제

    ```typescript
    const Koa = require('koa');
    const Router = require('koa-router');
    
    const app = new Koa();
    const router = new Router();
    
    router.get('/', (ctx, next) => {
        ctx.body = '홈';
    });
    
    router.get('/about', (ctx, next) => {
        ctx.body = '소개';
    });
    
    router.get('/about/:name', (ctx, next) => {
        const { name } = ctx.params; // 라우트 경로에서 :파라미터명 으로 정의된 값이 ctx.params 안에 설정됩니다.
        ctx.body = name + '의 소개';
    });
    
    router.get('/post', (ctx, next) => {
        const { id } = ctx.request.query; // 주소 뒤에 ?id=10 이런식으로 작성된 쿼리는 ctx.request.query 에 파싱됩니다.
        if(id) {
            ctx.body = '포스트 #' + id;
        } else {
            ctx.body = '포스트 아이디가 없습니다.';
        }
    });
    
    app.use(router.routes()).use(router.allowedMethods());
    
    app.listen(4000, () => {
        console.log('heurm server is listening to port 4000');
    });
    ```

- 라우트 모듈화

  - 프로젝트에는 여러 개의 라우트를 만들게 된다.

  - 하지만, 각 라우트를 지금 `index.js`에서 다 작성하면 코드가 너무 길어진다.

  - 이번에는, 라우터를 다른 파일에 작성해서 불러오는 방법을 알아보자.

  - 우선, 라우트들을 저장할 디렉토리부터 만들도록 하자.

  - `src/api/` 디렉토리를 만들고, 이 내부에 `index.js` 파일을 생성하자.

    

    `src/api/index.js` 파일

    ```typescript
    // src/api/index.js
    const Router = require('koa-router')
    
    const api = new Router();
    
    api.get('/books', (ctx, next) => {
      ctx.body = 'GET ' + ctx.request.path;
    });
    
    module.exports = api;
    ```

    위에서 모듈로 만든 라우트를 서버 엔트리파일(`src/index.js`)에서 불러와서 사용해보자.

    

    `src/index.js` 파일

    ```typescript
    const Koa = require('koa');
    const Router = require('koa-router');
    
    const app = new Koa();
    const router = new Router();
    const api = require('./api');
    
    router.use('/api', api.routes()); // api 라우트를 /api 경로 하위 라우트로 설정
    
    app.use(router.routes()).use(router.allowedMethods());
    
    app.listen(4000, () => {
        console.log('heurm server is listening to port 4000');
    });
    ```

- 여러 메서드 사용하기

  - REST API에서는, 요청의 종류에 따라 다른 HTTP 메서드를 사용한다.

  - HTTP 메서드는 여러 종류가 있는데, 그 중 주로 사용되는 것들은 다음과 같다.

    - GET, POST, DELETE, PUT, PATCH

  - `src/api/books/index.js`

    ```typescript
    const Router = require('koa-router');
    
    const books = new Router();
    
    const handler = (ctx, next) => {
        ctx.body = `${ctx.request.method} ${ctx.request.path}`;
    };
    
    books.get('/', handler);
    
    books.post('/', handler);
    
    books.delete('/', handler);
    
    books.put('/', handler);
    
    books.patch('/', handler);
    
    module.exports = books;
    ```

    - 위 코드처럼, 각 메서드를 처리하는 함수를 따로 분리시켜서 작성할 수도 있다.
    - 라우트를 작성할 때에는, 각 라우트에 해당하는 핸들러를 따로 작성하는 것이 좋습니다.
      - 그 이유는, 그렇게 해야 라우트들을 한눈에 보기 쉽기 때문이다.
    - 이제, 각 라우트에 대한 핸들러를 books 디렉토리에 books.controller.js 파일로 분리시켜보도록 하자.

- `src/api/books/books.controller.js`

  ```typescript
  exports.list = (ctx) => {
      ctx.body = 'listed';
  };
  
  exports.create = (ctx) => {
      ctx.body = 'created';
  };
  
  exports.delete = (ctx) => {
      ctx.body = 'deleted';
  };
  
  exports.replace = (ctx) => {
      ctx.body = 'replaced';
  };
  
  exports.update = (ctx) => {
      ctx.body = 'updated';
  };
  ```

  - 이렇게, `exports.변수명 = ...`으로 내보내기 한 코드는, 파일을 불러올 때 다음과 같이 사용할 수 있다.

    ```typescript
    const 모듈명 = require('파일명');
    모듈명.변수명
    ```

  - 코드를 내보낼 때에는, 일반 변수값을 내보낼 수도 있고, 함수를 내보낼 수도 있습니다.

  - 그럼, 방금 작성한 코드에 맞춰서 books 인덱스 파일을 업데이트 해주자.

- `src/api/books/index.js`

  ```typescript
  const Router = require('koa-router');
  
  const books = new Router();
  const booksCtrl = require('./books.controller');
  
  books.get('/', booksCtrl.list);
  books.post('/', booksCtrl.create);
  books.delete('/', booksCtrl.delete);
  books.put('/', booksCtrl.replace);
  books.patch('/', booksCtrl.update);
  
  module.exports = books;
  ```

- Node.js에서는 코드가 복잡해질 것 같을 땐, 모듈화를 하여 파일을 분리시키는것이 좋습니다. 

  - 그렇게 할 수록, 코드의 가독성이 높아지고 유지보수 하기도 편해지기 때문이죠.





