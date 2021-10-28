# [koa2 고급 학습 노트] 3. 데이터 수집 요청

> https://chenshenhai.com/koa2-note/



## 3.1 GET 요청 데이터 수집

### 원칙

- koa에서 GET 요청 데이터의 소스는 koa에서 `request` 객체의 `query` 또는 `querystring`입니다. 

- 쿼리 반환은 형식이 지정된 매개 변수 객체이고 쿼리 문자열은 요청 문자열을 반환합니다. 

- ctx에는 요청에 대한 직접 참조가 있기 때문입니다. 

- API 방식이므로 GET 요청 데이터를 얻는 두 가지 방법이 있습니다.

  1. ctx에서 직접 얻음

  - 요청 객체 ctx.query, {a:1, b:2}와 같은 반환
  - 요청 문자열 ctx.querystring, a=1&b=2와 같은 반환

  2. ctx의 request 객체에서 획득

  - 요청 객체 ctx.request.query, {a:1, b:2}와 같은 반환
  - 요청 문자열 ctx.request.querystring, a=1&b=2와 같은 반환



### 예제

> [데모 소스](https://github.com/ChenShenhai/koa2-note/blob/master/demo/request/get.js)

- 코드

  ```javascript
  const Koa = require('koa')
  const app = new Koa()
  
  app.use( async ( ctx ) => {
    let url = ctx.url
    // 从上下文的request对象中获取
    let request = ctx.request
    let req_query = request.query
    let req_querystring = request.querystring
  
    // 从上下文中直接获取
    let ctx_query = ctx.query
    let ctx_querystring = ctx.querystring
  
    ctx.body = {
      url,
      req_query,
      req_querystring,
      ctx_query,
      ctx_querystring
    }
  })
  
  app.listen(3000, () => {
    console.log('[demo] request get is starting at port 3000')
  })
  ```

  - 프로그램 실행 후 크롬을 이용하여 http://localhost:3000/page/user?a=1&b=2 에 접속하면 다음과 같은 상황이 발생합니다.

    ![](https://chenshenhai.com/koa2-note/note/images/request-get.png)





## 3.2 POST 요청 데이터 수집

### 원칙

- POST 요청 처리를 위해 <u>koa2는 매개변수를 캡슐화하고 획득하는 메소드가 없으며</u> 컨텍스트에서 기본 node.js 요청 객체 req를 구문 분석하여 POST 양식 데이터를 쿼리 문자열(예:)로 구문 분석 `a=1&b=2&c=3`한 다음 구문 분석해야 합니다. 
- 쿼리 문자열을 JSON 형식으로 변환 `{"a":"1", "b":"2", "c":"3"}`(예: )

> **[참고]**
>
> - ctx.request는 컨텍스트에 의해 캡슐화된 요청 객체이고,
> - ctx.req는 컨텍스트에 의해 제공되는 node.js 기본 HTTP 요청 객체이며, 
> - 유사하게 ctx.response는 컨텍스트에 의해 캡슐화된 응답 객체이고, 
> - ctx.res는 context.js 네이티브 HTTP 응답 객체에서 제공하는 노드입니다.

> 특정 koa2 API 문서는 https://github.com/koajs/koa/blob/master/docs/api/context.md#ctxreq 에서 찾을 수 있습니다.



### POST 요청의 ctx에서 데이터 파싱하기

> [데모 소스](https://github.com/ChenShenhai/koa2-note/blob/master/demo/request/post.js)

- 예제 코드

  ```javascript
  const Koa = require('koa')
  const app = new Koa()
  
  app.use( async ( ctx ) => {
  
    if ( ctx.url === '/' && ctx.method === 'GET' ) {
      // 当GET请求时候返回表单页面
      let html = `
        <h1>koa2 request post demo</h1>
        <form method="POST" action="/">
          <p>userName</p>
          <input name="userName" /><br/>
          <p>nickName</p>
          <input name="nickName" /><br/>
          <p>email</p>
          <input name="email" /><br/>
          <button type="submit">submit</button>
        </form>
      `
      ctx.body = html
    } else if ( ctx.url === '/' && ctx.method === 'POST' ) {
      // 当POST请求的时候，解析POST表单里的数据，并显示出来
      let postData = await parsePostData( ctx )
      ctx.body = postData
    } else {
      // 其他请求显示404
      ctx.body = '<h1>404！！！ o(╯□╰)o</h1>'
    }
  })
  
  // 解析上下文里node原生请求的POST参数
  function parsePostData( ctx ) {
    return new Promise((resolve, reject) => {
      try {
        let postdata = "";
        ctx.req.addListener('data', (data) => {
          postdata += data
        })
        ctx.req.addListener("end",function(){
          let parseData = parseQueryStr( postdata )
          resolve( parseData )
        })
      } catch ( err ) {
        reject(err)
      }
    })
  }
  
  // 将POST请求参数字符串解析成JSON
  function parseQueryStr( queryStr ) {
    let queryData = {}
    let queryStrList = queryStr.split('&')
    console.log( queryStrList )
    for (  let [ index, queryStr ] of queryStrList.entries()  ) {
      let itemList = queryStr.split('=')
      queryData[ itemList[0] ] = decodeURIComponent(itemList[1])
    }
    return queryData
  }
  
  app.listen(3000, () => {
    console.log('[demo] request post is starting at port 3000')
  })
  ```

  - 페이지 방문

    ![](https://chenshenhai.com/koa2-note/note/images/request-post-form.png)

    양식 제출 시 

    ![](https://chenshenhai.com/koa2-note/note/images/request-post-result.png)



## 3.3 koa-bodyparser 미들웨어

### 원칙

- POST 요청 처리를 위해 `koa-bodyparser` 미들웨어는 koa2 컨텍스트의 `formData`를 `ctx.request.body`로 parsing할 수 있습니다.
  - 3.2절에서 본 것처럼 일일이 처리하지 않아도 됩니다!!



- `koa-bodyparser` 미들웨어 설치

  ```shell
  npm install --save koa-bodyparser
  ```



## 예제

> [데모 소스 코드](https://github.com/ChenShenhai/koa2-note/blob/master/demo/request/post-middleware.js)

- 예제

  ```javascript
  const Koa = require('koa')
  const app = new Koa()
  const bodyParser = require('koa-bodyparser')
  
  // 使用ctx.body解析中间件
  app.use(bodyParser())
  
  app.use( async ( ctx ) => {
  
    if ( ctx.url === '/' && ctx.method === 'GET' ) {
      // 当GET请求时候返回表单页面
      let html = `
        <h1>koa2 request post demo</h1>
        <form method="POST" action="/">
          <p>userName</p>
          <input name="userName" /><br/>
          <p>nickName</p>
          <input name="nickName" /><br/>
          <p>email</p>
          <input name="email" /><br/>
          <button type="submit">submit</button>
        </form>
      `
      ctx.body = html
    } else if ( ctx.url === '/' && ctx.method === 'POST' ) {
      // 当POST请求的时候，中间件koa-bodyparser解析POST表单里的数据，并显示出来
      let postData = ctx.request.body
      ctx.body = postData
    } else {
      // 其他请求显示404
      ctx.body = '<h1>404！！！ o(╯□╰)o</h1>'
    }
  })
  
  app.listen(3000, () => {
    console.log('[demo] request post is starting at port 3000')
  })
  ```

  - 페이지 방문

    ![](https://chenshenhai.com/koa2-note/note/images/request-post-form.png)

    submit 하면,

    ![](https://chenshenhai.com/koa2-note/note/images/request-post-result.png)