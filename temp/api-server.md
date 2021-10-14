## api-server 프로젝트

- koa
- typeorm



### `package.json`

- "scripts"

  - nodemon

    - **node monitor**의 약자로, 노드가 실행하는 파일이 속한 디렉터리를 감시하고 있다가 파일이 수정되면 자동으로 노드 애플리케이션을 재시작하는 확장 모듈

    - view 파일은 수정한게 자동 반영되는데, node.js의 JS 파일들은 수정을 해도 반영이 안되기 때문에 실행시킨 노드 애플리케이션을 종료하고 다시 시작해야 한다. -> 불편함

    - nodemon은 프로젝트 폴더의 파일들을 모니터링하고 있다가 파일이 수정될 경우 자동으로 서버를 리스타트 시켜준다.

      > https://blog.outsider.ne.kr/649
      >
      > https://backend-intro.vlpt.us/1/03.html
      >
      > [구름 확장모듈 - nodemon](https://edu.goorm.io/learn/lecture/557/%ED%95%9C-%EB%88%88%EC%97%90-%EB%81%9D%EB%82%B4%EB%8A%94-node-js/lesson/382959/%ED%99%95%EC%9E%A5%EB%AA%A8%EB%93%88-nodemon)

      

  - ts-node/register/transpile-only

    - 트랜스파일링: 코드를 코드로 변환한다.
    - 실제로 node.js에서 실행할 때에는 js 파일만 해석할 수 있는데, 그래서 ts 파일을 모두 js 파일로 변경해야 한다.
      - 타입스크립트 컴파일러인 tsc가 그 역할을 한다.
        - typescript를 설치하면 함께 설치되는 컴파일러인 `tsc`는 문법 검사 및 코드 변환에 대한 책임만 있고 추가적인 작업을 하지 않는다.
        - 예를 들어 pug 파일이나 graphql 파일이 소스 폴더에 포함되어 있어도, 이것들을 전혀 건드리지 않기 때문에 `copyfiles` 등을 이용해 빌드가 완료된 폴더에 따로 복사를 해주어야 합니다.
    - `ts-node`는 node 명령어를 사용하는 것처럼 typescript를 쓸 수 있도록 하는 모듈이다.
      - 기존에 node 명령어로 실행하던 것을 그대로 ts-node로 바꿔주기만 하면 된다. 아주 편리하다!
      - 그런데, 아까 타입스크립트는 파일을 변환한 뒤에 실행해야 한다고 했다.
        - ts-node는 변환되는 js 파일의 디렉터리 경로를 그냥 임시 폴더 경로로 해둔다.
        - 그러므로 어디에 변환되는지는 사용자 입장에서 전혀 신경쓸 필요가 없다.
      - ts-node는 tsconfig.json의 설정을 읽어서 수행한다.
        - 다만, node와 동작 방법이 유사하기 때문에(index.js로부터 require로 필요한 소스 파일들을 검색해나감) tsconfig.json 파일 내의 include, exclude, files 옵션을 무시하는 게 기본 세팅이다.

    > [타입스크립트 프로젝트 세팅하기](https://elvanov.com/2524)

    

  - command-line options

    - `-`
      - stdin의 동의어
      - 다른 command-line utility에서의 `-` 사용법과 유사하다.
      - `-`의 의미는 stdin으로부터 script가 읽히고 있고, 나머지 옵션들이 해당 script로 넘겨지고 있음을 의미한다.
    - `-r`, `--require module`
      - startup 시에 미리 명시된 모듈을 불러온다.
    - `--`
      - node option의 마지막을 나타낸다.
      - 나머지 arguments들을 script로 념긴다.
      - 만약 `--` 옵션 전에 어떠한 script filename 또는 eval/print script가 명시되지 않으면, 다음 argument가 script filename으로 사용된다.

    > [node.js reference - cli](https://nodejs.org/api/cli.html#cli)





### `server.ts`

- `ìnitializeTransactionalContext();`

  - use typeorm CLS -> CLS가 뭐지

  - ```javascript
    export declare const initializeTransactionalContext: () => Namespace;
    ```

- `useContainer(Container);`

  - 그냥 DI 컨테이너 쓴다고 선언하는거 같다

- BlueBird

  - promise를 다른 패키지에서 사용하기 위해 사용

  - 콜백 지옥에서 구원해준다...

  - ```javascript
    Bluebird.config({
      longStackTraces: true,
    });
    // 그냥 설정인듯
    ```

- ```javascript
  process.on('unhandledRejection', (error: any) => {
    logger.error('unhandledRejection', { message: error.message, error, type: 'SYSTEM_ERROR' });
  
    sentry.withScope(scope => {
      scope.setTag('errorType', 'unhandledRejection');
      scope.setLevel(sentry.Severity.Critical);
      sentry.captureException(error);
    });
  });
  ```

  - sentry: 에러 로그 시스템
  - 에러 발생하면 로그 찍는 부분인듯

- ```javascript
  export async function createServer() {
    let server: https.Server | http.Server;
    let connectionOpts;
    const intervals: NodeJS.Timeout[] = [];
  
    logger.debug('createServer', `[NODE_CONFIG_ENV] ${process.env.NODE_CONFIG_ENV || 'default'}`);
    await chaiConfig.useChaiConfig();
  
    if (config.vault) {
      // Vault 설정을 가져와서 초기화
      logger.debug('createServer', `Loading configurations from Vault`);
  
      const conn = vault(config.vault);
  
      ({ data: connectionOpts } = await conn.read(`kv/api/orm/${config.orm}`));
    } else {
      // ormconfig 파일을 로드해서 초기화
      connectionOpts = await readConnectionOptions(config.orm);
    }
  
    const dbConn = await initORM(connectionOpts);
    await initAppPush();
  
    const application = Container.get(Application);
    await application.run();
  
    const app = await createApp();
  
    if (config.ssl) {
      const credential = {
        cert: fs.readFileSync(config.ssl.cert),
        key: fs.readFileSync(config.ssl.key),
      };
  
      logger.debug('createServer', `[SSL] cert ${config.ssl.cert}`);
      logger.debug('createServer', `[SSL] key ${config.ssl.key}`);
  
      server = https.createServer(credential, app.callback());
    } else {
      server = http.createServer(app.callback());
    }
  
    server.listen(config.port, () => {
      const { port, orm, ssl } = config;
      logger.debug('createServer', `🚀 Server ready(${orm}) at ${ssl ? 'https' : 'http'}://local.chai.finance:${port}`);
    });
  
    // Cache Policies
    await updateActivePolicies();
    const POLICY_UPDATE_TIME = 600000; // 10 minutes
    intervals.push(setInterval(updateActivePolicies, POLICY_UPDATE_TIME));
  
    return { server, intervals, dbConn, application };
  }
  ```

  - vault
    - hashicorp의 비밀정보 관리 도구 vault
    - vault는 비밀 정보 즉, 공개하면 안 되는 비밀번호, API 키, 토큰 등을 저장하고 관리하는 도구이다.

  



### `RootController.ts`

- 모든 컨트롤러 관리하는 컨트롤러 같음



