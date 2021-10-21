# Koajs.com 정리

> https://koajs.com/



### 도입

- Koa는 Express 팀이 설계한 새로운 웹 프레임워크입니다.
- 웹 애플리케이션 및 API를 위한 더 작고, 표현력이 뛰어난 프레임워크를 지향합니다.
- async function들을 빌려옴으로써, Koa는 callback들을 버리고 error-handling을 향상시킬 수 있었습니다.
- Koa는 코어 내에 미들웨어를 번들로 제공하지 않습니다.



### 애플리케이션

- Koa 애플리케이션은 요청이 들어오면 스택과 같은 방식으로 구성되고 실행되는 미들웨어 함수 배열을 포함하는 객체입니다.



### Cascading

- Koa invoke "downstream", then control flows back "upstream".
  - Koa는 다운스트림을 호출한 다음, 제어 흐름을 업스트림으로 다시 호출합니다.

- 예제

  ```javascript
  const Koa = require('koa');
  const app = new Koa();
  
  // logger
  
  app.use(async (ctx, next) => {
    console.log('1.');
    await next();
    console.log('6.');
    const rt = ctx.response.get('X-Response-Time');
    console.log(`${ctx.method} ${ctx.url} - ${rt}`);
  });
  
  // x-response-time
  
  app.use(async (ctx, next) => {
    console.log('2.');
    const start = Date.now();
    await next();
    console.log('5.');
    const ms = Date.now() - start;
    ctx.set('X-Response-Time', `${ms}ms`);
  });
  
  // response
  
  app.use(async ctx => {
    console.log('3.');
    ctx.body = 'Hello World';
    console.log('4.');
  });
  
  app.listen(3000);
  
  [출력 결과]
  1.
  2.
  3.
  4.
  5.
  6.
  GET / - 2ms
  ```

  - 위 예제는 "Hello World"로 응답을 보내준다.
  - 먼저 요청은 x-response-time 및 로깅 미들웨어를 통해 흐르고 요청이 시작된 시간을 표시한 다음, 응답 미들웨어를 통해 계속해서 제어를 양보합니다.
  - 미들웨어가 `next()`를 호출하면 함수는 중지되고 다음 미들웨어로 제어 흐름이 넘어간다. 
  - downstream으로 실행할 미들웨어가 더 이상 없으면 스택이 해제되고 각 미들웨어가 다시 시작되어 업스트림 동작을 수행합니다.

  

- `app.use(function)`
  - 전달받은 middleware function을 이 애플리케이션에 추가한다.



- `app.context` 

  - `app.context`는 ctx가 생성되는 prototype입니다. 

  - 당신은 `app.context`를 수정함으로써 `ctx`에 추가적인 property들을 추가할 수 있습니다.

    - <u>이는 `ctx`에 속성이나 메서드를 추가하여 전체 앱에서 사용할 수 있도록 하는 데 유용합니다.</u>
    - 이는 `ctx`에 더 많이 의존하는 대신 성능이 더 높거나(미들웨어가 없어서) 더 쉬울 수 있다.
    - **<u>하지만 `ctx`에 많이 의존하게 되는 것은 anti-pattern이다.</u>**

  - 예제 (`ctx`에다가 db에 대한 참조를 추가하기)

    ```javascript
    app.context.db = db();
    
    app.use(async ctx => {
    	console.log(ctx.db);
    });
    ```

    

> Note:
>
> - Many properties on `ctx` are defined using getters, setters, and `Object.defineProperty()`. You can only edit these properties (not recommended) by using `Object.defineProperty()` on `app.context`. See https://github.com/koajs/koa/issues/652.
> - Mounted apps currently use their parent's `ctx` and settings. Thus, mounted apps are really just groups of middleware.







- Error Handling

By default outputs all errors to stderr unless `app.silent` is `true`. The default error handler also won't output errors when `err.status` is `404` or `err.expose` is `true`. To perform custom error-handling logic such as centralized logging you can add an "error" event listener:

```js
app.on('error', err => {
  log.error('server error', err)
});
```

If an error is in the req/res cycle and it is *not* possible to respond to the client, the `Context` instance is also passed:

```js
app.on('error', (err, ctx) => {
  log.error('server error', err, ctx)
});
```

When an error occurs *and* it is still possible to respond to the client, aka no data has been written to the socket, Koa will respond appropriately with a 500 "Internal Server Error". In either case an app-level "error" is emitted for logging purposes.





### Context

- Koa Context는 node의 `request`와 `response`객체들을 한 개의 객체에 캡슐화하고 있다. 이 객체는 web application과 API를 작성하는데 많은 도움이 되는 메서드들을 제공하고 있다.

  - 이러한 메서드들은 HTTP 서버 개발에서 너무 자주 사용되기 때문에 미리 다 구현되어 있다

- 한 개의 `Context`는 한 개의 요청마다 생성되며, 미들웨어에서 receiver 또는 `ctx`  식별자로 참조됩니다.

  ```javascript
  app.use(async ctx => {
    ctx; // is the Context
    ctx.request; // is a Koa Request
    ctx.response; // is a Koa Response
  });
  ```

- Context의 접근자와 메서드들 중 다수는 단순히 `ctx.request` 또는 `ctx.response`에 해당하는 항목에 위임하고 그 외에는 동일하다.

  - 예를 들면, `ctx.type`과 `ctx.length`는 `response` 객체에 위임해버리고, `ctx.path`와 `ctx.method`는 `request`에 위임해버린다.





### API

- `Context`의 method들과 accessor들

- `ctx.req`: Node의 `request` 객체

- `ctx.res`: Node의 `response` 객체

- Koa의 응답 처리 우회는 지원되지 않습니다. (Bypassing Koa's response handling is not supported.)

  - 따라서 아래와 같은 node property들은 사용을 피하세요
  - `res.statusCode` 
  - `res.writeHead()`
  - `res.write()`
  - `res.end()`

- `ctx.request`: Koa의 `request` 객체

- `ctx.response`: Koa의 `response` 객체

- `ctx.state`: 미들웨어를 통해 정보를 전달하거나 frontend view에 정보를 전달할 때 사용이 권장되는 namespace

  ```javascript
  ctx.state.user = await User.find(id);
  ```

- `ctx.app`: Application instance reference

- `ctx.cookies.get(name, [options])`:

....









