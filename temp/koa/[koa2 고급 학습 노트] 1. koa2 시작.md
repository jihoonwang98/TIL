# [koa2 고급 학습 노트] 1. koa2 시작

> https://chenshenhai.com/koa2-note/



## 환경 준비 

- node.js v7.6.0은 async/await를 완벽하게 지원하므로 플래그가 필요하지 않으므로 node.js 환경은 7.6.0 이상이어야 합니다.
- Node.js 환경 버전 v7.6 이상
  - node.js 7.6 직접 설치: node.js 공식 웹사이트 주소 [https://nodejs.org](https://nodejs.org/)
  - nvm은 여러 버전의 node.js를 관리합니다. nvm을 사용하여 노드 버전을 관리할 수 있습니다.
- npm 버전 3.x 이상



## Getting Started

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

  

