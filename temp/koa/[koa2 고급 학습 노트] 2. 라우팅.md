# [koa2 고급 학습 노트] 2. 라우팅

> https://chenshenhai.com/koa2-note/



## 2.1 koa2 네이티브 라우팅 구현

### 간단한 예

```javascript
const Koa = require('koa')
const app = new Koa()

app.use( async ( ctx ) => {
  let url = ctx.request.url
  ctx.body = url
})
app.listen(3000)
```

- http://localhost:3000/hello/world 페이지를 방문하면 [/hello/world](http://localhost:3000/hello/world) 가 출력되는데, 이는 컨텍스트의 요청 객체에 있는 url이 현재 접속된 경로의 이름임을 의미하며, 이는 ctx로 판단할 수 있다. 
- request.url 또는 일반 일치는 필요한 라우팅을 사용자 정의할 수 있습니다.



### 맞춤형 라우팅

> [데모 소스 코드](https://github.com/ChenShenhai/koa2-note/tree/master/demo/route-simple)

- 소스 파일 디렉토리

  ```
  .
  ├── index.js
  ├── package.json
  └── view
      ├── 404.html
      ├── index.html
      └── todo.html
  ```

- 데모 소스 코드

  ```javascript
  const Koa = require('koa')
  const fs = require('fs')
  const app = new Koa()
  
  /**
   * 用Promise封装异步读取文件方法
   * @param  {string} page html文件名称
   * @return {promise}      
   */
  function render( page ) {
    return new Promise(( resolve, reject ) => {
      let viewUrl = `./view/${page}`
      fs.readFile(viewUrl, "binary", ( err, data ) => {
        if ( err ) {
          reject( err )
        } else {
          resolve( data )
        }
      })
    })
  }
  
  /**
   * 根据URL获取HTML内容
   * @param  {string} url koa2上下文的url，ctx.url
   * @return {string}     获取HTML文件内容
   */
  async function route( url ) {
    let view = '404.html'
    switch ( url ) {
      case '/':
        view = 'index.html'
        break
      case '/index':
        view = 'index.html'
        break
      case '/todo':
        view = 'todo.html'
        break
      case '/404':
        view = '404.html'
        break
      default:
        break
    }
    let html = await render( view )
    return html
  }
  
  app.use( async ( ctx ) => {
    let url = ctx.request.url
    let html = await route( url )
    ctx.body = html
  })
  
  app.listen(3000)
  console.log('[demo] route-simple is starting at port 3000')
  ```

- 데모 실행

  - 스크립트 실행

    ```shell
    node -harmony index.js
    ```

  - 작동 효과

    http://localhost:3000/index 방문

    ![](https://chenshenhai.com/koa2-note/note/images/route-result-01.png)





## 2.2 `koa-router` 미들웨어

- ctx.request.url에 의존하여 라우팅을 수동으로 처리하면 많은 처리 코드를 작성하게 됩니다. (귀찮습니다.)
  - 이때 라우팅을 제어하기 위해 라우팅 미들웨어를 작성하면 훨씬 편리하게 처리할 수 있습니다.
  - 여기에 `koa-router`라는 사용하기 쉬운 라우팅 미들웨어가 있습니다.

- `koa-router` 미들웨어 설치

  ```shell
  npm install --save koa-router
  ```

- `koa-router` getting started

  > [데모 소스 코드](https://github.com/ChenShenhai/koa2-note/tree/master/demo/route-use-middleware)

  ```javascript
  const Koa = require('koa')
  const fs = require('fs')
  const app = new Koa()
  
  const Router = require('koa-router')
  
  let home = new Router()
  
  // 子路由1
  home.get('/', async ( ctx )=>{
    let html = `
      <ul>
        <li><a href="/page/helloworld">/page/helloworld</a></li>
        <li><a href="/page/404">/page/404</a></li>
      </ul>
    `
    ctx.body = html
  })
  
  // 子路由2
  let page = new Router()
  page.get('/404', async ( ctx )=>{
    ctx.body = '404 page!'
  }).get('/helloworld', async ( ctx )=>{
    ctx.body = 'helloworld page!'
  })
  
  // 装载所有子路由
  let router = new Router()
  router.use('/', home.routes(), home.allowedMethods())
  router.use('/page', page.routes(), page.allowedMethods())
  
  // 加载路由中间件
  app.use(router.routes()).use(router.allowedMethods())
  
  app.listen(3000, () => {
    console.log('[demo] route-use-middleware is starting at port 3000')
  })
  ```

  





