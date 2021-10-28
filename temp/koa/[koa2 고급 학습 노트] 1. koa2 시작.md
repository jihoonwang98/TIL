# [koa2 고급 학습 노트] 1. koa2 시작

> https://chenshenhai.com/koa2-note/



## 1.1 빠른 시작

### 환경 준비 

- node.js v7.6.0은 async/await를 완벽하게 지원하므로 플래그가 필요하지 않으므로 node.js 환경은 7.6.0 이상이어야 합니다.
- Node.js 환경 버전 v7.6 이상
  - node.js 7.6 직접 설치: node.js 공식 웹사이트 주소 [https://nodejs.org](https://nodejs.org/)
  - nvm은 여러 버전의 node.js를 관리합니다. nvm을 사용하여 노드 버전을 관리할 수 있습니다.
- npm 버전 3.x 이상



### Getting Started

- koa2 설치

  ```shell
  mkdir koa-practice
  cd koa-practice
  npm init
  npm install koa
  ```

- Hello, world 코드

  ```javascript
  // index.js
  const Koa = require('koa')
  const app = new Koa()
  
  app.use( async ( ctx ) => {
    ctx.body = 'hello koa2'
  })
  
  app.listen(3000)
  console.log('[demo] start-quick is starting at port 3000')
  ```

- 데모 시작

  ```shell
  node index.js
  ```

- `http://localhost:3000`에 접속하면 아래와 같은 화면이 뜬다.

  ![img](https://chenshenhai.com/koa2-note/note/images/start-result-01.png)

  



## 1.2 `async/await`의 사용

### Getting started

- 다음 코드를 크롬 콘솔에 붙여넣고 실행해보자.

  ```javascript
  function getSyncTime() {
    return new Promise((resolve, reject) => {
      try {
        let startTime = new Date().getTime()
        setTimeout(() => {
          let endTime = new Date().getTime()
          let data = endTime - startTime
          resolve( data )
        }, 500)
      } catch ( err ) {
        reject( err )
      }
    })
  }
  
  async function getSyncData() {
    let time = await getSyncTime()
    let data = `endTime - startTime = ${time}`
    return data
  }
  
  async function getData() {
    let data = await getSyncData()
    console.log( data )
  }
  
  getData()
  ```

  - 실행 결과

    ![](https://chenshenhai.com/koa2-note/note/images/async.png)



## 1.3 koa2 구조에 대한 간략한 분석

### 소스 파일

```
├── lib
│   ├── application.js
│   ├── context.js
│   ├── request.js
│   └── response.js
└── package.json
```

- 이것은 오픈소스 koa2 소스 코드의 폴더 구조입니다.
  - 핵심 코드는 `lib` 디렉토리에 있는 네 개의 파일입니다.
  - `application.js`는 컨텍스트, 요청, 응답 및 핵심 미들웨어 처리 흐름을 캡슐화하는 전체 koa2의 항목 파일입니다.
  - `context.js`는 request.js 및 response.js 메소드의 일부를 직접 캡슐화하는 애플리케이션 컨텍스트를 처리합니다.
  - `request.js`는 http 요청을 처리합니다.
  - `response.js`는 http 응답을 처리합니다.



### koa2 특성

- 캡슐화된 http 컨텍스트, 요청, 응답 및 이를 기반으로 하는 `async/await`미들웨어 컨테이너 만 제공하십시오 .
- ES7 `async/await`은 koa Generator @ 1의 문제와 중첩 위치를 처리하기 위해 전통적인 콜백을 사용 하지만 node.js 조화 모드 7.x에서 지원해야 합니다 `async/await`.
- 미들웨어 `async/await`는 패키지 만 지원 하며, koa @ 1 제너레이터 기반 미들웨어를 사용하려면 미들웨어 koa-convert 패키지 사용을 살펴봐야 합니다.





## 1.4 koa 미들웨어 개발 및 사용

- koa v1, v2에서 사용되는 미들웨어 개발 및 활용
- Generator 미들웨어 개발은 koa v1 및 v2에서 사용됩니다.
- async는 미들웨어 개발을 기다리고 koa v2에서만 사용할 수 있습니다.



### Generator 미들웨어 개발

- Generator middleware 개발

  > generator middleware 함수는 `* ()` 함수를 반환해야 합니다.

  ```javascript
  /* ./middleware/logger-generator.js */
  function log( ctx ) {
      console.log( ctx.method, ctx.header.host + ctx.url )
  }
  
  module.exports = function () {
      return function * ( next ) {
  
          log( this )
  
          if ( next ) {
              yield next
          }
      }
  }
  ```



- 생성한 미들웨어 적용해보기

  - koa@1에서 사용

    > Generator 미들웨어는 koa v1에서 직접 사용가능함

    ```javascript
    const koa = require('koa')  // koa v1
    const loggerGenerator  = require('./middleware/logger-generator')
    const app = koa()
    
    app.use(loggerGenerator())
    
    app.use(function *( ) {
        this.body = 'hello world!'
    })
    
    app.listen(3000)
    console.log('the server is starting at port 3000')
    ```

  - koa@2에서 사용

    > Generator 미들웨어를 사용하려면 `koa-convert`로 캡슐화해야 한다.

    ```javascript
    const Koa = require('koa') // koa v2
    const convert = require('koa-convert')
    const loggerGenerator  = require('./middleware/logger-generator')
    const app = new Koa()
    
    app.use(convert(loggerGenerator()))
    
    app.use(( ctx ) => {
        ctx.body = 'hello world!'
    })
    
    app.listen(3000)
    console.log('the server is starting at port 3000')
    ```





### 비동기 미들웨어 개발

- 비동기 미들웨어 개발

  ```javascript
  /* ./middleware/logger-async.js */
  
  function log( ctx ) {
      console.log( ctx.method, ctx.header.host + ctx.url )
  }
  
  module.exports = function () {
    return async function ( ctx, next ) {
      log(ctx);
      await next()
    }
  }
  ```

- 비동기 미들웨어는 koa@2에서 사용됩니다.

  > 비동기 미들웨어는 koa v2에서만 사용할 수 있습니다.

  ```javascript
  const Koa = require('koa') // koa v2
  const loggerAsync  = require('./middleware/logger-async')
  const app = new Koa()
  
  app.use(loggerAsync())
  
  app.use(( ctx ) => {
      ctx.body = 'hello world!'
  })
  
  app.listen(3000)
  console.log('the server is starting at port 3000'
  ```

  









