# [Best Practices 시리즈 - Node.js Best Practices] 

> [원문](https://github.com/goldbergyoni/nodebestpractices), [한국어](https://github.com/goldbergyoni/nodebestpractices/blob/master/README.korean.md)



## 목차

1. 프로젝트 구조 설계 (5)
2. 에러 처리 방법 (11)
3. 코드 스타일 (12)
4. 테스트 및 전체 품질 관리 (13)
5. 운영 환경으로 전환하기 (19)
6. 보안 (25)
7. 성능 (2) (현재진행형 ✍️)





## 1. 프로젝트 구조 설계

### 1.1 컴포넌트 기반으로 설계하라

- **핵심요약:** 
  - 큰 프로젝트에서 빠지기 쉬운 최악의 함정은 수백개의 의존성을 가진 커다란 소스코드를 유지보수하는 것이다. 
  - 그렇게 단일로 통째로 짜여진 monolith 코드는 개발자가 새로운 기능을 추가하는 속도가 느려지게 한다. 
  - 그 대신에 코드를 컴포넌트로 나누고, 각각의 컴포넌트가 자신의 폴더 혹은 할당된 코드베이스를 안에서 작고 간단한 단위로 유지되도록 해라. 
  - 아래의 '자세히 보기'를 눌러 올바른 프로젝트 구조의 예시를 확인해라.

- **그렇게 하지 않을 경우:** 새로운 기능을 작성하는 개발자가 변경사항이 어떤 영향을 미치는지 알기가 힘들면 의존하고 있는 다른 컴포넌트를 망칠까 두려워 하게 되고, 이는 배포를 더 느리고 더 불안전하게 만든다. 비지니스 단위가 나눠져 있지 않으면 확장(scale-out)하기도 쉽지 않게 된다.



#### 자세히보기: 컴포넌트로 구조화하기

**한문단 설명**

- 중소규모 이상의 앱부터는 monolith는 정말 좋지 않다 - 의존성이 여럿 있는 커다란 하나의 소프트웨어를 쓰는 것은 꼬일대로 꼬인 스파게티 코드를 초래한다. 
- 이 괴물을 '모듈화'하여 길들일만큼 숙련된 똑똑한 설계자들조차 디자인에 엄청난 정신력을 쏟아붓고, 모든 변화는 다른 의존하는 프로젝트에 어떤 영향을 미치는지 신중히 감정할것을 필요로 한다. 
- 궁극적인 해결책은 소규모의 소프트웨어를 개발하는 것이다: 스택 전체를 다른이들과 파일을 공유하지 않는 자족적(self-contained)인 컴포넌트로 나누고, 추론하기 쉽게 극소수의 파일로만 구성되어야 한다 (예: API, 서비스, 데이터 접근, 테스트 등). 
- 이것을 "마이크로서비스" 설계라고 부르기도 하는데, 마이크로서비스는 따라야 하는 사양이 아니라 원칙들이란 것을 아는 것이 중요하다.
- 원칙을 여럿 받아들여 완전한 마이크로서비스 설계를 할 수도 있고, 아니면 그 중 소수만 받아들일 수도 있다.
  - 소프트웨어의 복잡하게 하지 않는 한 둘 다 좋은 방법이다. 
- **<u>최소한 컴포넌트 사이에 경계를 나누고, 프로젝트 최상위 루트에 각각의 비지니스 컴포넌트를 각기 다른 폴더에 배치하여 다른 컴포넌트들은 공용 인터페이스나 API로만 기능을 소비할 수 있게 자족적으로 만들 것.</u>** 
  - 이것이 컴포넌트를 간단하게 하고, 의존성 지옥을 방지하며 미래에 앱이 완전한 마이크로서비스로 자라나는 길을 닦는 기반이다.



**블로그 인용: "확장하려면 애플리케이션 전체를 확장해야 한다"**

MartinFowler.com 블로그로부터

> - monolith 애플리케이션도 성공적일 수 있지만, 점점 더 많은 사람들이 불만을 느끼고 있다 - 특히 더 많은 애플리케이션들이 클라우드로 전개될수록. 
> - 변화 주기는 다 같이 묶여 있다 - 애플리케이션의 조그마한 부분을 바꾸면 monolith 전체를 재건하고 재배치하여야 한다. 
> - 시간이 흐를수록 좋은 모듈식의 구조를 유지하는것이 힘들어지고, 모듈 하나에만 작용해야 할 변화가 그 모듈 이내에서만 작용하도록 하는것이 힘들어진다. 
> - 더 많은 자원을 필요로 하는 부분만 확장하는 것이 아니라, 확장하려면 애플리케이션 전체를 확장해야한다.



**블로그 인용: "그러니 당신의 어플리케이션의 설계를 보면 어떤 감이 오는가?"**

[uncle-bob](https://8thlight.com/blog/uncle-bob/2011/09/30/Screaming-Architecture.html)블로그로부터

> ...도서관 설계도를 보면, 아마도 커다란 입구, 체크인/체크아웃 구역, 독서실, 소규모 회의실들, 도서관의 모든 책을 수용할 수 있게 책꽂이들을 놓을 만한 공간들이 보일 것이다. 설계도를 보면 도서관이라고 바로 감이 올 것이다.

- 그러니 당신의 어플리케이션의 설계를 보면 어떤 감이 오는가? 
- 최상위 디렉토리 구조와 최고위 레벨의 패키지의 소스 파일을 보면 건강 관리 시스템인지, 회계 시스템인지, 재고관리 시스템인지 바로 감이 오는가? 
- 아니면 Rails이나 Spring/Hibernate, 혹은 ASP라는 감이 오는가?



**좋은예: 자족적인 컴포넌트 기반으로 설계하라**

![](https://github.com/goldbergyoni/nodebestpractices/raw/master/assets/images/structurebycomponents.PNG)



**나쁜예: 파일을 기술적인 역할별로 모아라**

![](https://github.com/goldbergyoni/nodebestpractices/raw/master/assets/images/structurebyroles.PNG)



### 1.2 컴포넌트를 계층화(layer)하고, Express를 그 경계 안에 둬라

- **핵심요약:** 
  - 각각의 컴포넌트는 웹, 로직, 데이터 접근 코드을 위한 객체인 '계층'을 포함해야 한다. 
  - 이것은 우려할 만한 요소들을 깨끗하게 분리할 뿐만 아니라 모의 객체(mock)를 만들어 테스트하기 굉장히 쉽게 만든다. 
  - 이것이 굉장히 일반적인 패턴임에도, API 개발자들은 웹 계층의 객체 (Express req, res)를 비지니스 로직과 데이터 계층으로 보내서 계층을 뒤섞어버리는 경향이 있다. 
  - 그렇게 하는것은 당신의 어플리케이션에 의존성을 만들고 Express에서만 접근 가능하도록 만든다.

- **그렇게 하지 않을 경우:** 
  - 웹 객체와 다른 계층을 뒤섞은 앱은 테스트 코드, CRON 작업이나 Express가 아닌 다른 곳에서의 접근을 불가능하게 한다.



#### 자세히 보기: 앱을 계층화하기

**컴포넌트 코드를 웹(Controller), 서비스, 데이터 접근 언어(DAL, Repository) 계층으로 나누어라**

![](https://github.com/goldbergyoni/nodebestpractices/raw/master/assets/images/structurebycomponents.PNG)



**1분 설명: 계층을 섞으면 불리한 점**

![](https://github.com/goldbergyoni/nodebestpractices/raw/master/assets/images/keepexpressinweb.gif)

- `Express`를 Web Layer에서만 접근 가능하게 유지하라.





### 1.3 공유 유틸리티들은 NPM 패키지로 감싸라 (wrap)

- **핵심요약:** 
  - 커다란 코드 기반으로 구성되어있는 커다란 앱에서는 로깅, 암호화 같은 횡단 관심사(cross-cutting-concern)가 있는 유틸의 경우 당신이 쓴 코드로 감싸진 private NPM package의 형태로 노출이 되어야 한다. 
  - 이것은 여러 코드 기반과 프로젝트들에게 그것들을 공유할 수 있도록 해준다.
- **그렇게 하지 않을 경우:** 
  - 당신 자신만의 배포 및 종속 바퀴(dependency wheel)를 새로이 발명해야 할 것이다.



#### 자세히 보기: 기능으로 구조화 하기

**한문단 설명**

- 자라나기 시작하면서 비슷한 유틸리티들을 소비하는 다른 서버의 다른 컴포넌트들이 생겨나면 의존성을 관리하기 시작해야 한다 - 유틸리티 코드 한 부를 어떻게 소비자 컴포넌트 여럿이서 같이 쓰고 배치할 수 있게 하는가? 
- 자, 여기 쓸만한 도구가 여기 있다...npm이라 불리는. 
- 먼저 제삼자 유틸리티 패키지를 자신만의 코드로 감싸 미래에 대체하기 쉽게 하고 그 코드를 private npm 패키지로 publish해라. 이제 당신의 모든 코드 기반은 그 코드를 수입하여 무료 의존성 관리 도구의 혜택을 볼 수 있다. 
- [private 모듈](https://docs.npmjs.com/private-modules/intro)이나 [private 레지스트리](https://npme.npmjs.com/docs/tutorials/npm-enterprise-with-nexus.html), 혹은 [로컬 npm 패키지](https://medium.com/@arnaudrinquin/build-modular-application-with-npm-local-modules-dfc5ff047bcc)를 사용하면 npm 패키지를 공개적으로 공유하지 않고도 자용으로 쓸 수 있게 출판할 수 있다.



#### 당신만의 공유 유틸리티들을 환경과 컴포넌츠에 공유하기

![](https://github.com/goldbergyoni/nodebestpractices/raw/master/assets/images/Privatenpm.png)





### 1.4 Express의 app과 server를 분리하라

- **핵심요약:**
  - [Express](https://expressjs.com/) 앱을 통째로 하나의 큰 파일에 정의하는 나쁜 습관은 피해라 - 'Express' 정의를 최소한 둘로는 나누자: 
    - API 선언(app.js)과 
    - 네트워크 부분(WWW)으로. 
  - 더 좋은 구조는 API 선언을 컴포넌트 안에 놓는 것이다.
- **그렇게 하지 않을 경우:** 
  - HTTP 요청으로만 API 테스트가 가능하게 된다 (커버리지 보고서를 생성하기가 더 느려지고 훨씬 힘들어진다). 
  - 수백줄의 코드를 하나의 파일에서 관리하는 것이 크게 즐겁지는 않을 것이다.



#### 자세히 보기: Express를 'app'과 'server'로 분리하기

**한문단 설명**

- The latest Express generator comes with a great practice that is worth to keep - the API declaration is separated from the network related configuration (port, protocol, etc). 
  - 최신의 Express generator가 지니고 있는 greate practice - API declaration이 네트워크 관련 설정(포트, 프로토콜, 기타 등등)과 분리되어 있다는 것
- This allows testing the API in-process, without performing network calls, with all the benefits that it brings to the table: fast testing execution and getting coverage metrics of the code. 
  - 이것은 제조 중(in-process, 개발 중?)에 API 테스팅을 할 수 있게 해준다. 
    - network 호출 없이,
    - 빠른 테스트 실행 및 Test Coverage 지표를 얻을 수 있음
- It also allows deploying the same API under flexible and different network conditions. Bonus: better separation of concerns and cleaner code
  - 이것은 또한 동일한 API를 서로 다른 network condition 하에서 배포할 수 있게 해줍니다.
  - 보너스로 얻게 되는 것: Separaction of Concerns(관심사의 분리)와 더 깨끗한 코드



**Code example: API declaration, should reside in app.js**

```javascript
var app = express();
app.use(bodyParser.json());
app.use("/api/events", events.API);
app.use("/api/forms", forms);
```



**Code example: Server network declaration, should reside in `/bin/www`**

```javascript
var app = require('../app');
var http = require('http');

/**
 * Get port from environment and store in Express.
 */

var port = normalizePort(process.env.PORT || '3000');
app.set('port', port);

/**
 * Create HTTP server.
 */

var server = http.createServer(app);
```



**Example: test your API in-process using supertest (popular testing package)**

```javascript
const app = express();

app.get('/user', function(req, res) {
  res.status(200).json({ name: 'tobi' });
});

request(app)
  .get('/user')
  .expect('Content-Type', /json/)
  .expect('Content-Length', '15')
  .expect(200)
  .end(function(err, res) {
    if (err) throw err;
  });
```





### 1.5 환경을 인식하는, 보안적인, 계층적인 설정을 사용하라

- **핵심요약:** 
  - 완벽하고 결점이 없는 구성 설정은 
  - (a) 파일과 환경변수 모두에서 키 값을 읽을 수 있어야하고 
  - (b) 보안 값들은 커밋된 코드 밖에서 관리되어야하며 
  - (c) 설정은 좀 더 쉽게 찾을 수 있도록 계층적으로 관리해야 한다. 
  - [rc](https://www.npmjs.com/package/rc), [nconf](https://www.npmjs.com/package/nconf), [config](https://www.npmjs.com/package/config), [convict](https://www.npmjs.com/package/convict)와 같이 이러한 요구사항을 동작하게 해주는 몇가지 패키지가 존재한다.

- **그렇게 하지 않을 경우:** 
  - 위의 구성 요구사항 중 하나라도 만족시키지 못한다면 개발팀이나 데브옵스팀을 늪으로 몰아갈 수 있다. 십중팔구 둘 다.



#### 자세히 보기: 구성 모범 사례

**한문단 요약**

- When dealing with configuration data, many things can just annoy and slow down:
  - 설정 값들을 다룰 때, 많은 것들이 짜증나고 느려질 수 있다.

1. setting all the keys using process environment variables becomes very tedious when in need to inject 100 keys (instead of just committing those in a config file), however when dealing with files only the DevOps admins cannot alter the behavior without changing the code. A reliable config solution must combine both configuration files + overrides from the process variables
   - 만약 100개의 key 값들에 대해 설정해야 할 때, 이러한 값들을 config file 내에 적지 않고 프로세스 환경 변수를 사용해 모든 키를 설정하는 것은 매우 지루한 작업이다.
   - 그러나 config file을 통해서만 이러한 값들을 설정하면 DevOps 관리자들은 코드를 바꾸지 않으면 behaviour를 바꿀 수 없다.
   - **<u>신뢰할만한 설정 solution은 반드시 configuration files을 통한 방법 + process variable로부터 override 받는 방법 두 가지를 결합해야 한다.</u>**
2. when specifying all keys in a flat JSON, it becomes frustrating to find and modify entries when the list grows bigger. A hierarchical JSON file that is grouped into sections can overcome this issue + few config libraries allow to store the configuration in multiple files and take care to union all at runtime. See example below
   - 모든 key 값들을 하나의 flat한 JSON 파일에 명시할 때, 설정할 key 값들의 리스트가 커질수록 항목을 찾고 수정하기가 어려워진다.
   - 섹션별로 그룹화된 계층화된 JSON 파일 형태로 설정 값들을 작성하면 이런 이슈를 극복할 수 있다.
   - 또한, 설정값들을 여러 개의 파일에 나눠서 저장할 수 있게 도와주는 config 라이브러리를 이용해 runtime시에만 이들을 합쳐서 사용할 수도 있다.
3. storing sensitive information like DB password is obviously not recommended but no quick and handy solution exists for this challenge. Some configuration libraries allow to encrypt files, others encrypt those entries during GIT commits or simply don't store real values for those entries and specify the actual value during deployment via environment variables.
   - DB 비밀번호와 같은 민감한 정보를 저장하는 것은 분명히 권장되는 방법은 아니지만 이 방법만큼 이 문제에 대한 빠르고 편리한 솔루션은 없습니다.
   - 몇몇 config 라이브러리들은 파일을 암호화할 수 있게 해주고, 다른 라이브러리들은 git 커밋 중에 해당 항목을 암호화하거나 단순히 해당 항목에 대한 실제 값을 저장하지 않고 환경 변수를 통해 배포하는 동안 실제 값을 지정합니다.
4. some advanced configuration scenarios demand to inject configuration values via command line (vargs) or sync configuration info via a centralized cache like Redis so multiple servers will use the same configuration data.
   - 몇몇 advanced한 configuration 시나리오에서는 커맨드 라인(vargs)를 통해 설정값들을 주입하거나 Redis와 같은 중앙 집중식 캐시를 통해 설정 정보를 동기화해야 여러 서버에서 동일한 구성 데이터를 사용할 수 있습니다.

Some configuration libraries can provide most of these features for free, have a look at npm libraries like [rc](https://www.npmjs.com/package/rc), [nconf](https://www.npmjs.com/package/nconf), [config](https://www.npmjs.com/package/config), and [convict](https://www.npmjs.com/package/convict) which tick many of these requirements.

- 몇몇 configuration 라이브러리들은 위와 같은 기능들을 무료로 제공한다.
- rc, nconf, config, convict와 같은 라이브러리들을 살펴보라.



**Code example - 계층화된 config 파일 형식으로 작성해야 entry들을 찾고 거대한 설정 파일들을 관리하기가 쉽다**

```json
{
  // Customer module configs 
  "Customer": {
    "dbConfig": {
      "host": "localhost",
      "port": 5984,
      "dbName": "customers"
    },
    "credit": {
      "initialLimit": 100,
      // Set low for development 
      "initialDays": 1
    }
  }
}
```





## 2. 에러 처리 방법

### 2.1 비동기 에러 처리시에는 `async-await` 혹은 `promise`를 사용하라

- **핵심요약:** 
  - 비동기 에러를 콜백 스타일로 처리하는 것은 지옥으로 가는 급행열차나 마찬가지 (혹은 파멸의 피라미드). 
  - 당신이 코드에 줄 수 있는 가장 큰 선물은 평판이 좋은 promise 라이브러리를 사용하거나 훨신 작고 친숙한 코드 문법인 try-catch를 사용하게 해주는 async-await를 사용하는 것이다.
- **그렇게 하지 않을 경우:** 
  - Node.js의 function(err, response) 콜백 스타일은 에러 처리와 일반 코드의 혼합, 코드의 과도한 중첩, 어색한 코딩 패턴 때문에 유지보수가 불가능한 코드로 가는 확실한 길이다.



#### 자세히보기: 콜백 피하기

**한문단 요약**

- Callbacks don’t scale well since most programmers are not familiar with them. 
  - 대부분의 프로그래머들이 콜백에 익숙하지 않기 때문에 콜백을 확장하기는 어렵다.
- They force to check errors all over, deal with nasty code nesting and make it difficult to reason about the code flow. 
  - 콜백은 곳곳의 에러들을 체크하고, 중첩된(nesting) 불쾌한 코드를 사용하도록 강요합니다.
  - 그리고 콜백은 코드 흐름에 대해 추론하기 어렵게 만듭니다.
- Promise libraries like BlueBird, async, and Q pack a standard code style using RETURN and THROW to control the program flow. 
  - `BlueBird`, `async`, `Q`와 같은 Promise 라이브러리들은 RETURN 문과 THROW를 사용하여 프로그램 흐름을 제어하는 표준 코드 스타일을 구비하고 있습니다.
- Specifically, they support the favorite try-catch error handling style which allows freeing the main code path from dealing with errors in every function
  - 특히, 이 라이브러리들은 가장 선호되는 방식의 try-catch error handling style을 지원하고 있습니다.
    - 이 style은 main code path에서 모든 함수가 error를 처리하지 않아도 되게 해주는 style을 말합니다.



**Code Example - using promises to catch errors**

```js
doWork()
 .then(doWork)
 .then(doOtherWork)
 .then((result) => doWork)
 .catch((error) => {throw error;})
 .then(verify);
```



**Anti pattern code example – callback style error handling**

```js
getData(someParameter, function(err, result) {
    if(err !== null) {
        // do something like calling the given callback function and pass the error
        getMoreData(a, function(err, result) {
            if(err !== null) {
                // do something like calling the given callback function and pass the error
                getMoreData(b, function(c) {
                    getMoreData(d, function(e) {
                        if(err !== null ) {
                            // you get the idea? 
                        }
                    })
                });
            }
        });
    }
});
```



### Blog Quote: "We have a problem with promises"

From the blog pouchdb.com

> ……And in fact, callbacks do something even more sinister: they deprive us of the stack, which is something we usually take for granted in programming languages. Writing code without a stack is a lot like driving a car without a brake pedal: you don’t realize how badly you need it until you reach for it and it’s not there. The whole point of promises is to give us back the language fundamentals we lost when we went async: return, throw, and the stack. But you have to know how to use promises correctly in order to take advantage of them.
>
> - ... 사실, 콜백들은 훨씬 더 사악한 것들을 합니다:
>   - 콜백들은 우리에게서 프로그래밍 언어에서 당연하게 여기는 stack을 빼앗아 갑니다.
>   - stack 없이 코드를 작성하는 것은 브레이크 없이 차를 모는 것과 같습니다
> - promise의 핵심 point는 우리가 비동기화되었을 때 잃어버린 언어 기본 사항인 return, throw 및 스택을 다시 제공하는 것입니다.



### Blog Quote: "The promises method is much more compact"

From the blog gosquared.com

> ………The promises method is much more compact, clearer and quicker to write. If an error or exception occurs within any of the ops it is handled by the single .catch() handler. Having this single place to handle all errors means you don’t need to write error checking for each stage of the work.
>
> - promise 메서드는 훨씬 더 컴팩트하고, 명확하고 빠르게 쓸 수 있다.
> - 작업 내에서 에러나 예외가 발생하면 단일 `.catch()` 핸들러에 의해 처리됩니다.
> - 모든 에러들을 처리할 single place를 가지고 있다는 것은 작업들의 각 단계마다 error checking 로직을 작성할 필요가 없다는 것을 의미합니다.



### Blog Quote: "Promises are native ES6, can be used with generators"

From the blog StrongLoop

> ….Callbacks have a lousy error-handling story. Promises are better. Marry the built-in error handling in Express with promises and significantly lower the chances of an uncaught exception. Promises are native ES6, can be used with generators, and ES7 proposals like async/await through compilers like Babel
>
> - 콜백은 형편없는 에러 핸들링 이야기를 가지고 있습니다.
>   - Promises를 쓰는 것이 더 낫습니다.
> - Express의 built-in 에러 핸들링을 promise를 이용하세요. 그리고 처리되지 않은 exception이 발생할 가능성을 현저히 줄이세요.
> - Promise들은 native ES6이고, generator와 함께 사용될 수 있고, Babel과 같은 컴파일러를 통해 `async/await`와 같은 ES7 제안을 사용할 수 있습니다.



### Blog Quote: "All those regular flow control constructs you are used to are completely broken"

From the blog Benno’s

> ……One of the best things about asynchronous, callback-based programming is that basically all those regular flow control constructs you are used to are completely broken. However, the one I find most broken is the handling of exceptions. Javascript provides a fairly familiar try…catch construct for dealing with exceptions. The problem with exceptions is that they provide a great way of short-cutting errors up a call stack, but end up being completely useless if the error happens on a different stack…
>
> - 비동기식 콜백 기반 프로그래밍의 가장 좋은 점 중 하나는 기본적으로 익숙한 모든 regular flow 제어 구조가 완전히 무너졌다는 것입니다.
> - 그러나 내가 찾아낸 가장 바뀐(broken) 것은 예외 처리입니다.
> - JS는 예외를 처리하기 위해 상당히 친숙한 `try-catch` 구문을 제공합니다.
> - 예외의 문제는 호출 스택에서 오류를 단축하는 훌륭한 방법을 제공하지만 오류가 다른 스택에서 발생하면 완전히 쓸모 없게 된다는 것입니다.



### 2.2 내장된 Error 객체만 사용하라

- **핵심요약:** 
  - 많은 사람들이 문자열이나 사용자가 임의로 정의한 타입으로 에러를 던지곤 하는데(throw), 이것은 에러처리 로직과 모듈 사이의 상호운영성을 복잡하게 한다. 
  - 당신이 promise를 거부(reject)하든, 예외를 던지든, 에러를 내던간에, 내장된 Error 객체만 이용하는 것이 균일성을 향상하고 정보의 손실을 방지해준다.
- **그렇게 하지 않을 경우:** 
  - 일부 컴포넌트를 호출할때 어떤 에러의 타입이 반환될지 불확실해져서 적절한 에러처리가 매우 어려워진다. 
  - 게다가 임의적인 타입으로 에러를 나타내는 것은 스택 정보(stack trace)와 같은 중요한 에러 관련 정보 손실을 일으킬 수 있다!



#### 자세히보기: 내장된 Error 객체 사용하기

**한문단 설명**

- 다양한 코드 흐름 옵션(예:EventEmitter, Callbacks, Promises 등)과 함께 JS의 허용 가능한 특성은 개발자가 오류를 발생시키는 방법에 큰 차이를 둔다. – 일부 개발자들은 문자열을 사용하고, 일부는 그들만의 커스텀 타입을 정의한다. 
- Node.js 내장 Error 객체를 사용하면 당신의 코드와 제 3자 라이브러리의 균일성을 유지할 수 있도록 도와주며, 또한 스택정보(StrackTrace)와 같은 중요한 정보도 보존할 수 있다. 
- 예외가 발생할 때, 일반적으로 오류 이름이나 관련 HTTP 오류 코드같은 상황별로 추가적인 속성으로 채우는 것이 좋은 방법이다. 이 균일성과 관행을 얻을려면, Error 객체를 추가 속성으로 확장하라, 아래 코드 예 참조

**코드 예시 – 제대로 하기**

```js
// throwing an Error from typical function, whether sync or async
if(!productToAdd)
    throw new Error("How can I add new product when no value provided?");

// 'throwing' an Error from EventEmitter
const myEmitter = new MyEmitter();
myEmitter.emit('error', new Error('whoops!'));

// 'throwing' an Error from a Promise
const addProduct = async (productToAdd) => {
  try {
    const existingProduct = await DAL.getProduct(productToAdd.id);
    if (existingProduct !== null) {
      throw new Error("Product already exists!");
    }
  } catch (err) {
    // ...
  }
}
```

**코드 예시 - 좋지않은 패턴**

```js
// throwing a string lacks any stack trace information and other important data properties
if(!productToAdd)
    throw ("How can I add new product when no value provided?");
```

**코드 예시 - 훨씬 더 좋다**

```js
// centralized error object that derives from Node’s Error
function AppError(name, httpCode, description, isOperational) {
    Error.call(this);
    Error.captureStackTrace(this);
    this.name = name;
    //...other properties assigned here
};

AppError.prototype = Object.create(Error.prototype);
AppError.prototype.constructor = AppError;

module.exports.AppError = AppError;

// client throwing an exception
if(user == null)
    throw new AppError(commonErrors.resourceNotFound, commonHTTPErrors.notFound, "further explanation", true)
```

**블로그 인용: "여러 가지 유형이 있다는 것은 가치가 없다고 본다"**

Ben Nadel 블로그에서 "Node.js error 객체" 키워드 5위에 위치했다.

> …”개인적으로, 여러 가지 유형이 있다는 것은 가치가 없다고 본다 – 언어로서의 자바스크립트는 생성자 기반의 오류 파악에 적합하지 않은 것 같다. 따라서, 객체 속성에서 구별하는 것이 생성자 유형에서 구분하는 것보다 훨씬 쉬운 것 같다…

**블로그 인용: "문자열은 오류가 아니다"**

devthought.com 블로그에서 "Node.js error 객체" 키워드 6위에 위치했다.

> …오류 대신 문자열을 전달함으로써 모듈 간의 상호운용성이 줄어들었다. 문자열을 사용하면 `instanceof` 에러로 검사하는 API 계약을 깨뜨리는 데다가 오류의 자세한 정보도 알기 어렵다. 뒤에서 설명하겠지만, 최신 자바스크립트 엔진의 Error 객체에는 유용한 속성을 많이 있으며 생성자에 전달된 메시지도 포함된다…

**블로그 인용: "에러로 부터 상속해도 많은 값을 추가하지 않는다"**

machadogj 블로그에서

> …Error 클래스와 관련된 한 가지 문제는 확장하기가 그리 간단하지 않다는 것이다. 물론 클래스를 상속하고 HttpError, DbError 등 자신만의 에러 클래스를 만들 수 있다. 그러나, 당신이 타입으로 무엇인가를 하지 않는 한 이것은 시간이 걸리며 많은 값을 추가하지 않는다. 때로 당신은 메시지 추가와 내부 에러를 유지하길 원하고 때로는 매개 변수를 사용하여 에러를 확장하길 원할 것이다, 등등…

**블로그 인용: "모든 자바스크립트와 시스템 에러들은 node.js이 상속한 에러로 부터 발생한다"**

Node.js 공식문서에서

> …Node.js에 의해 발생된 모든 자바스크립트 및 시스템 에러는 표준 자바스크립트 에러 클래스에서 상속되거나 표준 자바스크립트 에러 클래스의 인스턴스이며 최소한 그 클래스에서 사용할 수 있는 속성을 제공할 수 있도록 보장된다. 에러가 발생한 이유에 대한 특정 상황을 나타내지 않는 일반 자바스크립트 에러 객체. 에러 객체는 에러가 인스턴스화된 코드에서 포인트를 자세히 설명하는 "스택 추적"을 캡처(capture)하며, 에러에 대한 텍스트 설명을 제공할 수도 있다. 모든 시스템과 자바스크립트 에러를 포함한 Node.js에 의해, 생성된 모든 에러는, 에러 클래스의 인스턴스이거나, 상속 받은 것이다.…











