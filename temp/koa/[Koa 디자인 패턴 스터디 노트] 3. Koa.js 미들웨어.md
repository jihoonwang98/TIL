# [Koa 디자인 패턴 스터디 노트] 3. Koa.js 미들웨어

> https://chenshenhai.com/koajs-design-note/note/chapter02/01



## 3.1 미들웨어 분류

### 머리말

예를 들어, 시장에 나와 있는 대부분의 웹 프레임워크는 많은 웹 관련 기능 지원을 제공합니다.

- HTTP 서비스
- 경로 관리
- 템플릿 렌더링
- 통나무
- 플러그인/미들웨어 및 기타 AOP 기능
- 기타 능력

웹 프레임워크인 Koa.js는 요약하면 두 가지 기능만 제공합니다.

- HTTP 서비스
- 미들웨어 메커니즘(AOP 측면)

요약하면 대부분의 웹 기능을 Koa.js로 구현하려면 관련 기능을 통합하는 미들웨어가 필요합니다. 즉, Koa.js는 미들웨어의 큰 컨테이너이며 웹에서 요구하는 모든 기능은 미들웨어를 통해 구현됩니다.

Koa.js 미들웨어의 분류는 제가 알기로는 다음 두 가지로 나눌 수 있습니다.

- 좁은 미들웨어
- 일반화된 미들웨어

### 좁은 미들웨어

좁은 미들웨어의 특징:

- 미들웨어에서 작업 요청 `request`
- 미들웨어에서의 동작 응답 `response`
- 미들웨어의 작업 컨텍스트 `context`
- 대부분 직접 `app.use()`로드됨

예를 들어 미들웨어는 `koa-static`주로 요청과 응답을 가로채서 정적 자원을 로드하는데 , 미들웨어는 주로 요청을 가로채서 요청 가중치의 POST 데이터 `koa-bodyparser`를 파싱 `HTTP`하여 탑재 `ctx`한다.

### 일반화된 미들웨어

일반화된 미들웨어 특성

- 미들웨어를 직접 제공하지 마십시오
- 간접적인 방식으로 미들웨어 또는 서브미들웨어 제공
- 간접적으로 `app.use()`로드됨
- Koa 측면에 액세스하는 다른 방법

예를 들어, 중간 `koa-router`에 여러 경로를 먼저 등록한 `서브 미들웨어`다음 로드를 `상위 미들웨어`위해 하나로 캡슐화하여 `app.use()`모든 하위 미들웨어를 `Koa.js`요청에 로드할 수 있습니다 `양파 모델`.





## 3.2 좁은 미들웨어

### 머리말

내로우 미들웨어의 공통 요소는 다음과 같습니다.

- 모든 것이 미들웨어
- 미들웨어에서 작업 요청 `request`
  - 가로채기 요청
- 미들웨어에서의 동작 응답 `response`
  - 응답 가로채기
- 미들웨어의 작업 컨텍스트 `context`
  - 에 장착 직접 컨텍스트 에이전트, 에이전트 초기화 인스턴스`app.context`
  - 요청 프로세스에서 컨텍스트 프록시, 요청에 프록시가 탑재 `ctx`됨
- 대부분 직접 `app.use()` 로드됨
  - 참고 : 초기 인스턴스는 에이전트가됩니다 마운트 `context`하지`app.use()`



### 가로채기 요청

```javascript
const Koa = require('koa');
let app = new Koa();

const middleware = async function(ctx, next) {
  // 미들웨어 가로채기 요청
  // /page/로 시작하지 않는 모든 경로에 대해 500 오류 발생
  const reqPath = ctx.request.path;
  if( reqPath.indexOf('/page/') !== 0 ) {
    ctx.throw(500)
  }
  await next();
}

const page = async function(ctx, next) {
  ctx.body = `
      <html>
        <head></head>
        <body>
          <h1>${ctx.request.path}</h1>
        </body>
      </html>
    `; 
}

app.use(middleware);
app.use(page);

app.listen(3001, function(){
  console.log('the demo is start at port 3001');
})
```



### 응답 가로채기

```javascript
const Koa = require('koa');
let app = new Koa();

const middleware = async function(ctx, next) {
  ctx.response.type = 'text/plain';
  await next();
}

const page = async function(ctx, next) {
  ctx.body = `
      <html>
        <head></head>
        <body>
          <h1>${ctx.path}</h1>
        </body>
      </html>
    `; 
}

app.use(middleware);
app.use(page);

app.listen(3001, function(){
  console.log('the demo is start at port 3001');
})
```



### 컨텍스트 마운트 에이전트

- 프록시 주입 요청
  - app.use에서 직접 요청 시에만 주입
  - 각 요청의 주입이 다릅니다.
- 인스턴스(애플리케이션) 프록시 인젝션 초기화
  - app.context에 직접 주입
  - 애플리케이션이 초기화될 때만 주입
  - 한 번만 주입되며 모든 요청에 사용할 수 있습니다.



### 요청 시 마운트 프록시 컨텍스트

```javascript
const Koa = require('koa');
let app = new Koa();

const middleware = async function(ctx, next) {
  // 미들웨어 프록시/마운트 컨텍스트
  // ctx에서 모든 현재 서비스 프로세스 PID 및 메모리 사용 방법을 프록시/마운트합니다.
  ctx.getServerInfo = function() {
    function parseMem( mem = 0 ) {
      let memVal = mem / 1024 / 1024;
      memVal = memVal.toFixed(2) + 'MB';
      return memVal;
    }

    function getMemInfo() {
      let memUsage = process.memoryUsage();
      let rss = parseMem(memUsage.rss);
      let heapTotal = parseMem(memUsage.heapTotal);
      let heapUsed =  parseMem(memUsage.heapUsed);
      return {
        pid: process.pid,
        rss,
        heapTotal,
        heapUsed
      }
    }
    return getMemInfo()
  };
  await next();
}

const page = async function(ctx, next) {
  const serverInfo = ctx.getServerInfo();
  ctx.body = `
      <html>
        <head></head>
        <body>
          <p>${JSON.stringify(serverInfo)}</p>
        </body>
      </html>
    `; 
}

app.use(middleware);
app.use(page);

app.listen(3001, function(){
  console.log('the demo is start at port 3001');
})
```



### 인스턴스 마운트 프록시 컨텍스트 초기화

```js
const Koa = require('koa');
let app = new Koa();

const middleware = function(app) {
  // 미들웨어가 인스턴스를 초기화하고 있고 getServerInfo 메소드가 프록시를 컨텍스트에 마운트합니다.
  app.context.getServerInfo = function() {
    function parseMem( mem = 0 ) {
      let memVal = mem / 1024 / 1024;
      memVal = memVal.toFixed(2) + 'MB';
      return memVal;
    }

    function getMemInfo() {
      let memUsage = process.memoryUsage();
      let rss = parseMem(memUsage.rss);
      let heapTotal = parseMem(memUsage.heapTotal);
      let heapUsed =  parseMem(memUsage.heapUsed);
      return {
        pid: process.pid,
        rss,
        heapTotal,
        heapUsed
      }
    }
    return getMemInfo()
  };
}

middleware(app);

const page = async function(ctx, next) {
  const serverInfo = ctx.getServerInfo();
  ctx.body = `
      <html>
        <head></head>
        <body>
          <p>${JSON.stringify(serverInfo)}</p>
        </body>
      </html>
    `; 
}

app.use(page);

app.listen(3001, function(){
  console.log('the demo is start at port 3001');
})
```





## 3.3 일반화된 미들웨어

### 머리말

- 미들웨어를 직접 제공하지 마십시오
- 미들웨어는 간접적인 방식으로 제공되며 가장 일반적인 것은 다음 `간접 미들웨어`과 같습니다.`서브 미들웨어`
- 간접적으로 `app.use()`로드됨
- Koa 측면에 액세스하는 다른 방법



### 간접 미들웨어

```js
const Koa = require('koa');
let app = new Koa();

function indirectMiddleware(path, middleware) {
  return async function(ctx, next) {
    console.log(ctx.path === path, middleware);
    if (ctx.path === path) {
      await middleware(ctx, next);
    } else {
      await next();
    }
  };
}

const index = async function(ctx, next) {
  ctx.body = 'this is index page';
};

const hello = async function(ctx, next) {
  ctx.body = 'this is hello page';
};

const world = async function(ctx, next) {
  ctx.body = 'this is world page';
};

app.use(indirectMiddleware('/', index));
app.use(indirectMiddleware('/hello', hello));
app.use(indirectMiddleware('/world', world));

app.listen(3001, () => {
  console.log('the demo is start at port 3001');
});
```





### 서브 미들웨어

서브 미들웨어는 일반화된 미들웨어의 가장 대표적인 시나리오 중 하나이며 주요 기능은 다음과 같습니다.

- 미들웨어 초기화 시 내장된 서브미들웨어 목록
- 하위 미들웨어 목록에 하위 미들웨어 요소 추가
- 하위 미들웨어 목록은 간접 미들웨어에 캡슐화되어 나중에 `app.use()`로드 됩니다.

```js
const Koa = require('koa');
let app = new Koa();

class Middleware{
  constructor() {
    this.stack = [];
  }

  get(path, childMiddleware) {
    this.stack.push({ path, middleware: childMiddleware })
  }

  middlewares() {
    let stack = this.stack;
    return async function(ctx, next) {
      let path = ctx.path;
      for( let i=0; i<stack.length; i++ ) {
        const child = stack[i];
        if( child && child.path === path && child.middleware ) {
          await child.middleware(ctx, next);
        }
      }
      await next();
    }
  }
}

const middleware = new Middleware();
middleware.get('/page/001', async(ctx, next) => { ctx.body = 'page 001' })
middleware.get('/page/002', async(ctx, next) => { ctx.body = 'page 002' })
middleware.get('/page/003', async(ctx, next) => { ctx.body = 'page 003' })

app.use(middleware.middlewares());

app.listen(3001, function(){
  console.log('the demo is start at port 3001');
})
```

