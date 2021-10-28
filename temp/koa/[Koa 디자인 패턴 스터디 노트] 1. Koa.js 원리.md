# [Koa 디자인 패턴 스터디 노트] 1. Koa.js 원리

> https://chenshenhai.com/koajs-design-note/



## 1.1 공부 준비

### 공부 준비

- 지식 보유
  - Node.js에 대한 기본 지식
    - `http` 모듈 사용
    - `fs` 모듈 사용
    - `path` 모듈 사용
    - `Buffer` 유형
  - ES의 기본 지식
    - `Promise`
    - `async/await`
  - 기타 지식
    - `HTTP` 규약
    - `Cookie` 원칙





## 1.2 Promise의 주요 용도

- 네이티브 Call back 작성의 장단

  - callback 결과 상태의 불편한 관리

  - callback 방법은 자유롭고 느슨하며 규범적 제약이 없다.

  - 예제 

    ```javascript
    function func(num, callback) {
        setTimeout(() => {
            try {
                let result =  1/num;
                callback(result, null);
            } catch(err) {
              callback(null, err);
            }
        }, 10)
    }
    
    
    func(1, (result, err) => {
        if( err ) {
          console.log(err)
        } else {
          console.log(result)
        }   
    })
    ```

    - 위의 코드에서 콜백 결과 `result`및 오류 처리가 발견되면 `err`이후의 모든 처리는 콜백 함수 내부에 있어야 하며 자체 콜백 함수 이상 판단 프로세스도 필요합니다. 
    - `Promise`콜백 작업을 처리하는 데 사용 하는 경우 다음과 같이 처리할 수 있습니다.

    ```javascript
    function func(num, callback) {
        return new Promise((resolve) => {
            setTimeout(() => { 
                let result = 1/num;
                resolve(result); 
            }, 1000)
        })
    }
    
    func(1).then((result) => { 
      console.log(result)
    }).catch((err) => { 
      console.log(err)
    })
    ```

    

## 1.3 `async/await`의 사용

- 예제

  ```javascript
  // 封装原子任务
  function increase(num) {
    return new Promise((resolve, reject) => {
      setTimeout(() => {
        if( !(num >= 0) ) {
          reject(new Error('The parameters must be greater than zero'))
        } else {
          resolve(num + 1)
        }
  
      }, 100);
    }).catch(err => console.log(err))
  
  }
  
  // 声明任务环境
  async function envIncrease() {
    let num = 1;
    // 等待回调任务结果1返回
    let result1 = await increase(num);
    console.log(`result1 = ${result1}`);
  
    // 等待回调任务结果2返回
    let result2 = await increase(result1);
    console.log(`result2 = ${result2}`);
  
    // 等待回调任务结果3返回
    let result3 = await increase(result2);
    console.log(`result3 = ${result3}`);
  
    return result3
  }
  
  // 声明任务环境
  async function env() {
    // 等待 环境 Increase 的结果返回
    let result = await envIncrease()
    console.log(`result = ${result}`);
  }
  
  // 运行环境
  env()
  
  // 运行结果
  // "result1 = 2"
  // "result1 = 3"
  // "result1 = 4"
  ```

  



## 1.4 Node.js 네이티브 http 모듈

### 도입

- Koa.js는 미들웨어 모델을 기반으로 하는 HTTP 서비스 프레임워크 `http`로 Node.js의 기본 모듈과 분리할 수 없다는 것이 기본 원칙 입니다 .



### http 모듈 사용

```javascript
const http = require('http');
const PORT = 3001;
const router = (req, res) => {
  res.end(`this page url = ${req.url}`);
}
const server = http.createServer(router)
server.listen(PORT, function() {
  console.log(`the server is started at port ${PORT}`)
})
```



### http 서비스 구성

- 서비스 컨테이너

  - 여기에 서비스 컨테이너는 전체 HTTP 서비스의 초석 `apache`과 `nginx`가 제공하는 기능과 일치한다.

    - 통신 연결 설정
    - 통신 포트 지정
    - 커스터마이징 가능한 컨텐츠 서비스 컨테이너, 즉 서비스 콜백 기능을 위한 컨테이너 제공

    ```javascript
    const http = require('http');
    const PORT = 3001;
    const server = http.createServer((req, res) => {
        // TODO 容器内容
        // TODO 服务回调内容
    })
    server.listen(PORT, function() {
      console.log(`the server is started at port ${PORT}`)
    })
    ```

- 서비스 콜백(컨텐츠)

  - 서비스 콜백은 주로 서비스 기능을 제공하는 서비스 콘텐츠로 이해할 수 있습니다.

    - 서비스 요청 분석 `req`
    - 요청 내용에 응답 `res`

    ```javascript
    const router = (req, res) => {
      res.end(`this page url = ${req.url}`);
    }
    ```

- 요청

  - 서비스 콜백의 첫 번째 파라미터로 HTTP 요청의 `request`내용과 그 내용을 운용하는 방식 을 주로 제공한다.

  - 더 많은 작업 제안은 공식 Node.js 문서를 참조하세요.

    > https://nodejs.org/dist/latest-v8.x/docs/api/http.html
    >
    > https://nodejs.org/dist/latest-v10.x/docs/api/http.html

- 응답

  - 서비스 콜백의 두 번째 매개변수로 HTTP 응답 `response`의 내용과 내용을 운용하는 방식 을 주로 제공한다.

  - 참고: 요청이 끝나면 응답을 실행해야 하고 `res.end()`, 그렇지 않으면 연결이 끊어지고 페이지가 충돌할 때까지 요청이 차단을 기다립니다.

  - 더 많은 작업 제안은 공식 Node.js 문서를 참조하세요.

    > https://nodejs.org/dist/latest-v8.x/docs/api/http.html
    >
    > https://nodejs.org/dist/latest-v10.x/docs/api/http.html

- 후속 조치

  - 위의 설명을 통해 주요 HTTP 서비스 콘텐츠를 서비스 콜백에서 처리한 다음,

    다른 연결에 따라 분할하여 경로를 형성 `router`하고,

    경로 콘텐츠의 분할에 따라 컨트롤러를 형성 `controller`합니다. 

  - 참조 코드는 다음과 같습니다.

    ```javascript
    onst http = require('http');
    const PORT = 3001;
    
    // 控制器
    const controller = {
      index(req, res) {
        res.end('This is index page')
      },
      home(req, res) {
        res.end('This is home page')
      },
      _404(req, res) {
        res.end('404 Not Found')
      }
    }
    
    // 路由器
    const router = (req, res) => {
      if( req.url === '/' ) {
        controller.index(req, res)
      } else if( req.url.startsWith('/home') ) {
        controller.home(req, res)
      } else {
        controller._404(req, res)
      }
    }
    
    // 服务
    const server = http.createServer(router)
    server.listen(PORT, function() {
      console.log(`the server is started at port ${PORT}`)
    })
    ```

    

## 1.5 미들웨어 엔진

### 도입

- Koa.js를 사용하는 과정에서 다음 코드와 같이 미들웨어의 사용이 이렇습니다.

  ```javascript
  const Koa = require('koa');
  let app = new Koa();
  
  const middleware1 = async (ctx, next) => { 
    console.log(1); 
    await next();  
    console.log(6);   
  }
  
  const middleware2 = async (ctx, next) => { 
    console.log(2); 
    await next();  
    console.log(5);   
  }
  
  const middleware3 = async (ctx, next) => { 
    console.log(3); 
    await next();  
    console.log(4);   
  }
  
  app.use(middleware1);
  app.use(middleware2);
  app.use(middleware3);
  app.use(async(ctx, next) => {
    ctx.body = 'hello world'
  })
  
  app.listen(3001)
  
  // 启动访问浏览器
  // 控制台会出现以下结果
  // 1
  // 2
  // 3
  // 4
  // 5
  // 6
  ```

  - 위와 같은 결과가 나오는 이유는 주로 Koa.js의 미들웨어 엔진 `koa-compose`모듈인 Koa.js `양파모델`코어 엔진 을 달성하기 위한 것 입니다.



### 미들웨어 원리

- 양파 모델은 `await next()`시나리오의 스택 데이터 구조와 같은 전후 의 미들웨어 작업을 볼 수 있습니다. 동시에 통합 컨텍스트 관리 운영 데이터가 있습니다. 요약하면 특성을 요약할 수 있습니다.
  - 화합하다 `context`
  - 선입후출 작업
  - 선입 선출을 제어하는 메커니즘이 있습니다. `next`
  - 조기 종료 메커니즘이 있습니다.

- 이 방법을 사용 `Promise`하면 간단하게 다음을 달성할 수 있습니다.

  ```javascript
  let context = {
    data: []
  };
  
  async function middleware1(ctx, next) {
    console.log('action 001');
    ctx.data.push(1);
    await next();
    console.log('action 006');
    ctx.data.push(6);
  }
  
  async function middleware2(ctx, next) {
    console.log('action 002');
    ctx.data.push(2);
    await next();
    console.log('action 005');
    ctx.data.push(5);
  }
  
  async function middleware3(ctx, next) {
    console.log('action 003');
    ctx.data.push(3);
    await next();
    console.log('action 004');
    ctx.data.push(4);
  }
  
  Promise.resolve(middleware1(context, async() => {
    return Promise.resolve(middleware2(context, async() => {
      return Promise.resolve(middleware3(context, async() => {
        return Promise.resolve();
      }));
    }));
  }))
    .then(() => {
      console.log('end');
      console.log('context = ', context);
    });
  
  // 结果显示
  // "action 001"
  // "action 002"
  // "action 003"
  // "action 004"
  // "action 005"
  // "action 006"
  // "end"
  // "context = { data: [1, 2, 3, 4, 5, 6]}"
  ```

  

### 엔진 구현

- 앞 절의 미들웨어 원리를 통해 단순히 네 `Promise`스팅을 이용하여 미들웨어 프로세스를 직접 구현할 수 있음을 알 수 있다. 
- 실현될 수는 있지만, `Promise`중첩은 코드의 가독성 및 유지보수성의 문제를 야기할 뿐만 아니라 미들웨어 확장의 문제도 야기할 수 있다.
- 따라서 `Promise`사용자 정의할 수 있는 계층 수를 달성하려면 미들웨어 의 중첩 구현 을 고도로 추상화 해야 합니다. 
  - 이것은 이전 장에서 언급한 `Promise`중첩된 가공물 처리에 도움이 필요할 때 `async/await`입니다.
- 필요한 단계를 명확히 합시다
  - 미들웨어 대기열
  - 미들웨어 대기열을 처리하고 컨텍스트 `context`를 전달합니다.
  - 미들웨어 프로세스 컨트롤러`next`
  - 예외 처리
- 이전 섹션의 분석에 따르면 다음을 추상화할 수 있습니다.
  - 각 미들웨어는 하나의 미들웨어를 캡슐화해야 합니다. `Promise`
  - 양파 모델의 선입 선출 작업, 해당 `Promise.resolve`전면 및 후면 작업

- 예제

  ```javascript
  function compose(middleware) {
  
    if (!Array.isArray(middleware)) {
      throw new TypeError('Middleware stack must be an array!');
    }
  
    return function(ctx, next) {
      let index = -1;
  
      return dispatch(0);
  
      function dispatch(i) {
        if (i < index) {
          return Promise.reject(new Error('next() called multiple times'));
        }
        index = i;
  
        let fn = middleware[i];
  
        if (i === middleware.length) {
          fn = next;
        }
  
        if (!fn) {
          return Promise.resolve();
        }
  
        try {
          return Promise.resolve(fn(ctx, () => {
            return dispatch(i + 1);
          }));
        } catch (err) {
          return Promise.reject(err);
        }
      }
    };
  }
  ```

  미들웨어 엔진 사용해보기

  ```javascript
  let middleware = [];
  let context = {
    data: []
  };
  
  middleware.push(async(ctx, next) => {
    console.log('action 001');
    ctx.data.push(2);
    await next();
    console.log('action 006');
    ctx.data.push(5);
  });
  
  middleware.push(async(ctx, next) => {
    console.log('action 002');
    ctx.data.push(2);
    await next();
    console.log('action 005');
    ctx.data.push(5);
  });
  
  middleware.push(async(ctx, next) => {
    console.log('action 003');
    ctx.data.push(2);
    await next();
    console.log('action 004');
    ctx.data.push(5);
  });
  
  const fn = compose(middleware);
  
  fn(context)
    .then(() => {
      console.log('end');
      console.log('context = ', context);
    });
  
  // 结果显示
  // "action 001"
  // "action 002"
  // "action 003"
  // "action 004"
  // "action 005"
  // "action 006"
  // "end"
  // "context = { data: [1, 2, 3, 4, 5, 6]}"
  ```

  

## 1.6 일반적인 미들웨어 스타일의 HTTP 서비스 구현

- 이 장에서는 네이티브 Node.js를 사용하여 순수한 콜백 미들웨어 HTTP 서비스를 구현합니다.



### 필요조건

- 내장 미들웨어 대기열
- 미들웨어 순회 메커니즘
- 예외 처리 메커니즘



### 가장 간단한 구현

> [데모 소스 코드](https://github.com/chenshenhai/koajs-design-note/tree/master/demo/chapter-01-06)

- 서비스 클래스 캡슐화

  ```javascript
  const http = require('http');
  const Emitter = require('events');
  
  class WebServer extends Emitter {
  
    constructor() {
      super();
      this.middleware = [];
      this.context = Object.create({});
    }
  
    /**
     * 서비스 이벤트 모니터링
     * @param {*} args 
     */
    listen(...args) {
      const server = http.createServer(this.callback());
      return server.listen(...args);
    }
  
    /**
     * 미들웨어 사용 등록
     * @param {Function} fn 
     */
    use(fn) {
      if (typeof fn === 'function') {
        this.middleware.push(fn);
      }
    }
  
    /**
     * 미들웨어 토탈 콜백 방식
     */
    callback() {
      let that = this;
  
      if (this.listeners('error').length === 0) {
        this.on('error', this.onerror);
      }
  
      const handleRequest = (req, res) => {
        let context = that.createContext(req, res);
        this.middleware.forEach((cb, idx) => {
          try {
            cb(context);
          } catch (err) {
            that.onerror(err);
          }
  
          if (idx + 1 >= this.middleware.length) {
            if (res && typeof res.end === 'function') {
              res.end();
            }
          }
        });
      };
      return handleRequest;
    }
  
    /**
     * 예외 처리 모니터
     * @param {EndOfStreamError} err 
     */
    onerror(err) {
      console.log(err);
    }
  
    /**
     * 공통 컨텍스트 만들기
     * @param {Object} req 
     * @param {Object} res 
     */
    createContext(req, res) {
      let context = Object.create(this.context);
      context.req = req;
      context.res = res;
      return context;
    }
  }
  
  module.exports = WebServer;
  ```

- 서비스 이용

  ```javascript
  const WebServer = require('./index');
  
  const app = new WebServer();
  const PORT = 3001;
  
  app.use(ctx => {
    ctx.res.write('<p>line 1</p>');
  });
  
  app.use(ctx => {
    ctx.res.write('<p>line 2</p>');
  });
  
  app.use(ctx => {
    ctx.res.write('<p>line 3</p>');
  });
  
  app.listen(PORT, () => {
    console.log(`the web server is starting at port ${PORT}`);
  });
  ```

  

## 1.7 아주 간단한 Koa.js 구현

### 도입

- 이전 장에서 미들웨어 스타일의 HTTP 서비스의 가장 간단한 구현을 볼 수 있습니다.

- 하위 계층은 미들웨어 큐를 처리하기 위해 콜백 중첩을 기반으로 합니다.

  ```javascript
    /**
     * 미들웨어 토탈 콜백 방식
     */
    callback() {
      let that = this;
  
      if (this.listeners('error').length === 0) {
        this.on('error', this.onerror);
      }
  
      const handleRequest = (req, res) => {
        let context = that.createContext(req, res);
        this.middleware.forEach((cb, idx) => {
          try {
            cb(context);
          } catch (err) {
            that.onerror(err);
          }
  
          if (idx + 1 >= this.middleware.length) {
            if (res && typeof res.end === 'function') {
              res.end();
            }
          }
        });
      };
      return handleRequest;
    }
  ```

- 하지만 미들웨어가 많을수록 콜백 중첩이 깊을수록 코드의 가독성과 확장성이 떨어지므로 이 때 콜백 중첩을 `Promise`+ `async/await`로 변환하면 이번에는 가장 간단한 `Koa.js`구현이 됩니다.



### 필요조건

- 컨텍스트 할당으로 대체 가능 `res.end()`
- 양파 모델의 미들웨어 메커니즘



### 소스 코드 구현

> [데모 소스 코드](https://github.com/chenshenhai/koajs-design-note/tree/master/demo/chapter-01-07)

- 가장 간단한 Koa.js 구현

  ```javascript
  onst http = require('http');
  const Emitter = require('events');
  // 注意：这里的compose是前几章的中间件引擎源码
  const compose = require('./../compose');
  
  /**
   * 일반 컨텍스트
   */
  const context = {
    _body: null,
  
    get body() {
      return this._body;
    },
  
    set body(val) {
      this._body = val;
      this.res.end(this._body);
    }
  };
  
  class SimpleKoa extends Emitter {
    constructor() {
      super();
      this.middleware = [];
      this.context = Object.create(context);
    }
  
    /**
     * 서비스 이벤트 모니터링
     * @param {*} args
     */
    listen(...args) {
      const server = http.createServer(this.callback());
      return server.listen(...args);
    }
  
    /**
     * 미들웨어 사용 등록
     * @param {Function} fn
     */
    use(fn) {
      if (typeof fn === 'function') {
        this.middleware.push(fn);
      }
    }
  
    /**
     * 미들웨어 토탈 콜백 방식
     */
    callback() {
  
      if (this.listeners('error').length === 0) {
        this.on('error', this.onerror);
      }
  
      const handleRequest = (req, res) => {
        let context = this.createContext(req, res);
        let middleware = this.middleware;
        // 执行中间件
        compose(middleware)(context).catch(err => this.onerror(err))
      };
      return handleRequest;
    }
  
    /**
     * 예외 처리 모니터
     * @param {EndOfStreamError} err
     */
    onerror(err) {
      console.log(err);
    }
  
    /**
     * 공통 컨텍스트 만들기
     * @param {Object} req
     * @param {Object} res
     */
    createContext(req, res) {
      let context = Object.create(this.context);
      context.req = req;
      context.res = res;
      return context;
    }
  }
  
  module.exports = SimpleKoa;
  ```

  - 구현 예

  ```javascript
  const SimpleKoa = require('./index');
  
  const app = new SimpleKoa();
  const PORT = 3001;
  
  app.use(async ctx => {
    ctx.body = '<p>this is a body</p>';
  });
  
  app.listen(PORT, () => {
    console.log(`the web server is starting at port ${PORT}`);
  });
  ```

  

