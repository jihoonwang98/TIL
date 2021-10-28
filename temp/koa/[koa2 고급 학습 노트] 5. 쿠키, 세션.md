# [koa2 고급 학습 노트] 5. 쿠키, 세션

> https://chenshenhai.com/koa2-note/



## 5.1 쿠키

### 도입

- Koa는 컨텍스트에서 직접 쿠키를 읽고 쓰는 방법을 제공합니다.
  - ctx.cookies.get(name, [options]) 컨텍스트 요청에서 쿠키를 읽습니다.
  - ctx.cookies.set(이름, 값, [옵션]) 컨텍스트에 쿠키 쓰기
- koa2에서 동작하는 쿠키는 npm의 쿠키 모듈을 사용하며 소스코드는 https://github.com/pillarjs/cookies 이므로 쿠키를 읽고 쓸 때 사용하는 매개변수는 이 모듈의 사용과 일치합니다.



### 예제

- 예제 코드

  ```javascript
  const Koa = require('koa')
  const app = new Koa()
  
  app.use( async ( ctx ) => {
  
    if ( ctx.url === '/index' ) {
      ctx.cookies.set(
        'cid', 
        'hello world',
        {
          domain: 'localhost',  // 写cookie所在的域名
          path: '/index',       // 写cookie所在的路径
          maxAge: 10 * 60 * 1000, // cookie有效时长
          expires: new Date('2017-02-15'),  // cookie失效时间
          httpOnly: false,  // 是否只用于http请求中获取
          overwrite: false  // 是否允许重写
        }
      )
      ctx.body = 'cookie is ok'
    } else {
      ctx.body = 'hello world' 
    }
  
  })
  
  app.listen(3000, () => {
    console.log('[demo] cookie is starting at port 3000')
  })
  ```

- 실행

  - http://localhost:3000/index 방문

  - 콘솔의 쿠키 목록에서 페이지에 작성된 쿠키를 볼 수 있습니다.

  - 콘솔 콘솔에서 document.cookie를 사용하여 페이지의 모든 쿠키를 인쇄합니다(표시하려면 httpOnly를 false로 설정해야 함).

    ![](https://chenshenhai.com/koa2-note/note/images/cookie-result-01.png)

  

  

  

## 5.2 세션

### 도입

- <u>koa2의 기본 기능은 쿠키 작업만 제공하며 세션 작업은 제공하지 않습니다.</u> 
  - 세션은 자체적으로 또는 타사 미들웨어를 통해서만 구현될 수 있습니다. 
  - koa2에서 세션을 구현하는 방법에는 여러 가지가 있습니다.
    - 세션 데이터의 양이 적으면 메모리에 직접 저장할 수 있음
    - 세션 데이터의 양이 많은 경우 세션 데이터를 저장할 저장 매체가 필요합니다.



### DB 저장 체계

- 세션을 MySQL 데이터베이스에 저장
- 미들웨어를 사용해야 함
  - koa-session-minimal은 저장 매체에 대한 읽기 및 쓰기 인터페이스를 제공하는 koa2 세션 미들웨어에 적합합니다.
  - koa-mysql-session은 koa-session-minimal 미들웨어에 MySQL 데이터베이스 세션 데이터 읽기 및 쓰기 작업을 제공합니다.
  - sessionId 및 해당 데이터를 데이터베이스에 저장
- 데이터베이스의 저장된 sessionId를 페이지의 쿠키에 저장
- 쿠키의 sessionId에 따라 세션 정보를 가져옵니다.



### Getting Started

> [데모 소스 코드](https://github.com/ChenShenhai/koa2-note/blob/master/demo/session/index.js)

- 예제 코드

  ```javascript
  const Koa = require('koa')
  const session = require('koa-session-minimal')
  const MysqlSession = require('koa-mysql-session')
  
  const app = new Koa()
  
  // 配置存储session信息的mysql
  let store = new MysqlSession({
    user: 'root',
    password: 'abc123',
    database: 'koa_demo',
    host: '127.0.0.1',
  })
  
  // 存放sessionId的cookie配置
  let cookie = {
    maxAge: '', // cookie有效时长
    expires: '',  // cookie失效时间
    path: '', // 写cookie所在的路径
    domain: '', // 写cookie所在的域名
    httpOnly: '', // 是否只用于http请求中获取
    overwrite: '',  // 是否允许重写
    secure: '',
    sameSite: '',
    signed: '',
  }
  
  // 使用session中间件
  app.use(session({
    key: 'SESSION_ID',
    store: store,
    cookie: cookie
  }))
  
  app.use( async ( ctx ) => {
  
    // 设置session
    if ( ctx.url === '/set' ) {
      ctx.session = {
        user_id: Math.random().toString(36).substr(2),
        count: 0
      }
      ctx.body = ctx.session
    } else if ( ctx.url === '/' ) {
  
      // 读取session信息
      ctx.session.count = ctx.session.count + 1
      ctx.body = ctx.session
    } 
  
  })
  
  app.listen(3000)
  console.log('[demo] session is starting at port 3000')
  ```

  - 실행

    **연결 설정 세션에 액세스**

    http://localhost:3000/set

    ![](https://chenshenhai.com/koa2-note/note/images/session-result-01.png)

    **DB 세션이 저장되었는지 확인**

    ![](https://chenshenhai.com/koa2-note/note/images/session-result-03.png)

    **sessionId가 쿠키에 심어졌는지 확인**

    http://localhost:3000

    ![](https://chenshenhai.com/koa2-note/note/images/session-result-02.png)
