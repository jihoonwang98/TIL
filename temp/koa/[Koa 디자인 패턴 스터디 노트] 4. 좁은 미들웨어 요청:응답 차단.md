# [Koa 디자인 패턴 스터디 노트] 4. 좁은 미들웨어 요청/응답 차단

> https://chenshenhai.com/koajs-design-note/note/chapter02/01
>
> https://github.com/koajs/logger
>
> https://github.com/koajs/send
>
> https://github.com/koajs/static



## 4.1 koa-logger 구현

### 도입

요청/차단의 가장 주목할만한 기능은 좁은 미들웨어입니다.

- 직접`app.use()`
- 가로채기 요청
- 작업 응답

가장 간단한 시나리오는 Koa.js가 정적 파일 전송 미들웨어 구현을 공식적으로 지원한다는 것입니다 `koa-logger`

> 이 섹션은 공식 `koa-logger`미들웨어 참조의 간단한 `koa-logger`구현과 원리 및 후속 보조 사용자 정의 개발 최적화를 설명하기 쉽게 달성합니다 .



### 구현 단계

- 01단계 요청을 가로채서 요청 URL을 인쇄합니다.
- 02단계 작업 응답, 응답 URL 인쇄

- 예제

  ```js
  const logger = async function(ctx, next) {
    let res = ctx.res;
  
    // 拦截操作请求 request
    console.log(`<-- ${ctx.method} ${ctx.url}`);
  
    await next();
  
    // 拦截操作响应 request
    res.on('finish', () => {
      console.log(`--> ${ctx.method} ${ctx.url}`);
    });
  };
  
  module.exports = logger
  ```

  사용

  ```js
  const Koa = require('koa');
  const logger = require('./index');
  const app = new Koa();
  
  app.use(logger);
  
  app.use(async(ctx, next) => {
    ctx.body = 'hello world';
  });
  
  app.listen(3000, () => {
    console.log('[demo] is starting at port 3000');
  });
  ```



## 4.2 koa-send 구현

### 도입

요청/차단의 가장 주목할만한 기능은 좁은 미들웨어입니다.

- 직접`app.use()`
- 가로채기 요청
- 작업 응답

가장 일반적인 시나리오는 Koa.js가 정적 파일 전송 미들웨어 구현을 공식적으로 지원한다는 것입니다 `koa-send`.

주요 실현 시나리오 프로세스는

- 요청을 가로채고 요청이 로컬 정적 리소스 파일을 요청하는지 확인합니다.
- 작업 응답, 해당 정적 파일 텍스트 콘텐츠 또는 오류 프롬프트 반환

> 이 섹션은 공식 `koa-send`미들웨어 참조의 간단한 `koa-end`구현과 원리 및 후속 보조 사용자 정의 개발 최적화를 설명하기 쉽게 달성합니다 .

### 구현 단계

- 01단계 정적 리소스의 절대 디렉터리 주소 구성
- 02단계 숨김 파일 지원 여부 결정
- 03단계 파일 또는 디렉터리 정보 가져오기
- 04단계 압축이 필요한지 판단
- 05단계 HTTP 헤더 정보 설정
- 06단계 정적 파일 읽기



### koa-send 소스 코드

```js
const fs = require('fs');
const path = require('path');
const {
  basename,
  extname
} = path;

const defaultOpts = {
  root: '',
  maxage: 0,
  immutable: false,
  extensions: false,
  hidden: false,
  brotli: false,
  gzip: false,
  setHeaders: () => {}
};

async function send(ctx, urlPath, opts = defaultOpts) {
  const { root, hidden, immutable, maxage, brotli, gzip, setHeaders } = opts;
  let filePath = urlPath;

  // step 01: normalize path
  // 정적 리소스의 절대 디렉터리 주소 구성
  try {
    filePath = decodeURIComponent(filePath);
    // check legal path
    if (/[\.]{2,}/ig.test(filePath)) {
      ctx.throw(403, 'Forbidden');
    }
  } catch (err) {
    ctx.throw(400, 'failed to decode');
  }

  filePath = path.join(root, urlPath);
  const fileBasename = basename(filePath);

  // step 02: check hidden file support
  // 숨김 파일 지원 여부 결정
  if (hidden !== true && fileBasename.startsWith('.')) {
    ctx.throw(404, '404 Not Found');
    return;
  }

  // step 03: stat
  // 파일 또는 디렉토리 정보 얻기
  let stats; 
  try { 
    stats = fs.statSync(filePath);
    if (stats.isDirectory()) {
      ctx.throw(404, '404 Not Found');
    }
  } catch (err) {
    const notfound = ['ENOENT', 'ENAMETOOLONG', 'ENOTDIR']
    if (notfound.includes(err.code)) {
      ctx.throw(404, '404 Not Found');
      return;
    }
    err.status = 500
    throw err
  }

  let encodingExt = '';
  // step 04 check zip
  // 압축이 필요한지 여부 결정
  if (ctx.acceptsEncodings('br', 'identity') === 'br' && brotli && (fs.existsSync(filePath + '.br'))) {
    filePath = filePath + '.br';
    ctx.set('Content-Encoding', 'br');
    ctx.res.removeHeader('Content-Length');
    encodingExt = '.br';
  } else if (ctx.acceptsEncodings('gzip', 'identity') === 'gzip' && gzip && (fs.existsSync(filePath + '.gz'))) {
    filePath = filePath + '.gz';
    ctx.set('Content-Encoding', 'gzip');
    ctx.res.removeHeader('Content-Length');
    encodingExt = '.gz';
  }

  // step 05 setHeaders
  // HTTP 헤더 정보 설정
  if (typeof setHeaders === 'function') {
    setHeaders(ctx.res, filePath, stats);
  }

  ctx.set('Content-Length', stats.size);
  if (!ctx.response.get('Last-Modified')) {
    ctx.set('Last-Modified', stats.mtime.toUTCString());
  }
  if (!ctx.response.get('Cache-Control')) {
    const directives = ['max-age=' + (maxage / 1000 | 0)];
    if (immutable) {
      directives.push('immutable');
    }
    ctx.set('Cache-Control', directives.join(','));
  }

  const ctxType = encodingExt !== '' ? extname(basename(filePath, encodingExt)) : extname(filePath);
  ctx.type = ctxType;

  // step 06 stream
  // 정적 파일 읽기
  ctx.body = fs.createReadStream(filePath);
}

module.exports = send;
```



### koa-send 사용

```js
const send = require('./index');
const Koa = require('koa');
const app = new Koa();


// public/ 현재 프로젝트의 정적 파일 디렉토리
app.use(async ctx => {
  await send(ctx, ctx.path, { root: `${__dirname}/public` });
});

app.listen(3000);
console.log('listening on port 3000');
```





## 4.3 koa-static 구현

### 도입

좁은 의미에서 미들웨어 요청/차단, 가장 일반적인 시나리오는 Koa.js 전송 정적 파일 미들웨어의 구현입니다 `koa-send`. Koa.js 공식 `koa-send`2차 패키징, `koa-static`미들웨어 도입 , 목표는 정적 자원 또는 프로젝트 관리를 위한 정적 서버를 만드는 것입니다.

> 이 섹션의 공식 `koa-static`미들웨어는 구현하기 쉬운 `koa-send`간단한 `koa-static`미들웨어 의 실현을 기반으로 원리를 설명하고 후속 2차 맞춤형 개발 최적화를 쉽게 설명합니다.



### 구현 단계

- 01단계 정적 리소스의 절대 디렉터리 주소 구성
- 02단계 다른 요청에 대한 대기 지원 여부 결정
- 03단계 GET 및 HEAD 유형 요청인지 확인
- 04단계 `koa-send`미들웨어를 통해 정적 파일 읽기 및 반환



### koa 정적 종속성

`koa-send` 미들웨어, 여기서는 이전 섹션의 가장 간단한 구현만 사용됩니다. `koa-send`



### koa-static 코드

```js
const {resolve} = require('path');
const send = require('./send');

function statics(opts = {
  root: ''
}) {
  opts.root = resolve(opts.root);

  // 다른 요청을 기다려야 합니까?
  if (opts.defer !== true) {
    // 다른 요청을 기다려야 하는 경우
    return async function statics(ctx, next) {
      let done = false;

      if (ctx.method === 'HEAD' || ctx.method === 'GET') {
        try {
          await send(ctx, ctx.path, opts);
          done = true;
        } catch (err) {
          if (err.status !== 404) {
            throw err;
          }
        }
      }

      if (!done) {
        await next();
      }
    };
  } else {
    // 다른 요청을 기다릴 필요가 없는 경우
    return async function statics(ctx, next) {
      await next();

      if (ctx.method !== 'HEAD' && ctx.method !== 'GET') {
        return;
      }

      if (ctx.body != null || ctx.status !== 404) {
        return;
      }

      try {
        await send(ctx, ctx.path, opts);
      } catch (err) {
        if (err.status !== 404) {
          throw err;
        }
      }
    };
  }
}

module.exports = statics;
```



### koa-static 사용

```js
const path = require('path');
const Koa = require('koa');
const statics = require('./index');

const app = new Koa();

const root = path.join(__dirname, './public');
app.use(statics({ root }));

app.use(async(ctx, next) => {
  if (ctx.path === '/hello') {
    ctx.body = 'hello world';
  }
});

app.listen(3000);
console.log('listening on port 3000');
```