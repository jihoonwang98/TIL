# [NestJS 한글 매뉴얼] TECHNIQUES - Configuration

> https://docs.nestjs.kr/techniques/configuration



## Configuration

- 애플리케이션은 종종 서로 다른 **환경**에서 실행됩니다. 

  - 환경에 따라 다른 구성 설정을 사용해야 합니다. 
    - 예를 들어, 일반적으로 로컬 환경은 로컬 DB 인스턴스에만 유효한 특정 데이터베이스 자격증명에 의존합니다. 
    - 프로덕션 환경은 별도의 DB 자격증명 세트를 사용합니다.
  - 구성 변수가 변경되므로 환경에서 [구성 변수 저장](https://12factor.net/config)을 사용하는 것이 가장 좋습니다.

  

- 외부에서 정의된 환경 변수는 `process.env` 전역을 통해 Node.js 내부에서 볼 수 있습니다.

  - 각 환경에서 개별적으로 환경 변수를 설정하여 여러 환경의 문제를 해결할 수 있습니다. 
  - 이는 특히 이러한 값을 쉽게 모의하거나 변경해야 하는 개발 및 테스트 환경에서 빠르게 다루기 어려울 수 있습니다.



- Node.js 애플리케이션에서는 각 환경을 나타내기 위해 각 키가 특정 값을 나타내는 키-값 쌍을 보유한 `.env` 파일을 사용하는 것이 일반적입니다.
  - 다른 환경에서 앱을 실행하는 것은 올바른 `.env` 파일에서 스와핑하면 됩니다.



- Nest에서 이 기술을 사용하는 좋은 방법은 적절한 `.env` 파일을 로드하는 `ConfigService`를 노출하는 `ConfigModule`을 만드는 것입니다. 
  - 이러한 모듈을 직접 작성하도록 선택할 수 있지만 편의를 위해 Nest는 즉시 사용 가능한 `@nestjs/config` 패키지를 제공합니다.
  - 이 패키지는 현재 장에서 다룰 것입니다.



## Installation

- 사용을 시작하려면 먼저 필요한 종속성을 설치합니다.

  ```shell
  $ npm i --save @nestjs/config
  ```

  > **힌트**
  >
  > `@nestjs/config` 패키지는 내부적으로 [dotenv](https://github.com/motdotla/dotenv)를 사용합니다.





## Getting started

- 설치 프로세스가 완료되면 `ConfigModule`을 가져올 수 있습니다.

- 일반적으로 루트 `AppModule`로 가져오고 `.forRoot()` 정적 메서드를 사용하여 동작을 제어합니다. 

  -  이 단계에서 환경 변수 키/값 쌍이 구문 분석되고 해결됩니다. 

- 나중에 다른 기능 모듈에서 `ConfigModule`의 `ConfigService` 클래스에 액세스하기 위한 몇가지 옵션을 볼 수 있습니다.

  ```typescript
  // app.module.ts
  import { Module } from '@nestjs/common';
  import { ConfigModule } from '@nestjs/config';
  
  @Module({
    imports: [ConfigModule.forRoot()],
  })
  export class AppModule {}
  ```

  - 위의 코드는 기본 위치(프로젝트 루트 디렉터리)에서 `.env` 파일을 로드 및 구문 분석하고 `.env` 파일의 키/값 쌍을 `process.env`에 할당된 환경 변수와 병합하고 결과적으로 `ConfigService`를 통해 액세스할 수 있는 개인 구조가 됩니다.

  - `forRoot()` 메서드는 이러한 파싱/병합된 구성 변수를 읽기 위한 `get()` 메서드를 제공하는 `ConfigService` 프로바이더를 등록합니다.
  - `@nestjs/config`는 [dotenv](https://github.com/motdotla/dotenv)에 의존하므로 환경 변수 이름의 충돌을 해결하기 위해 해당 패키지의 규칙을 사용합니다. 
  - 키가 런타임 환경에 환경 변수(예: `export DATABASE_USER=test`와 같은 OS 셸 내보내기를 통해)와 `.env` 파일로 모두 존재하는 경우 런타임 환경 변수가 우선합니다.



- 샘플 `.env` 파일

  ```
  DATABASE_USER=test
  DATABASE_PASSWORD=test
  ```

  



## Custom env file path

- 기본적으로 패키지는 애플리케이션의 루트 디렉토리에서 `.env` 파일을 찾습니다.

- `.env` 파일의 다른 경로를 지정하려면 아래 예제 코드와 같이 `forRoot()`에 전달하는(선택 사항) 옵션 객체의 `envFilePath` 속성을 설정합니다.

  ```typescript
  ConfigModule.forRoot({
    envFilePath: '.development.env',
  });
  ```



- 다음과 같이 `.env` 파일에 여러 경로를 지정할 수도 있습니다.

  ```typescript
  ConfigModule.forRoot({
    envFilePath: ['.env.development.local', '.env.development'],
  });
  ```

  - 변수가 여러 파일에서 발견되면 첫번째 파일이 우선합니다.





## Disable env variables loading

- `.env` 파일을 로드하지 않고 대신 런타임 환경에서 환경 변수에 액세스하려는 경우(`export DATABASE_USER=test`와 같은 OS 셸 내보내기와 마찬가지로) 옵션 객체의 `ignoreEnvFile`을 설정합니다. 

- 다음과 같이 속성을 `true`로 설정합니다.

  ```typescript
  ConfigModule.forRoot({
    ignoreEnvFile: true,
  });
  ```

  



## Use module globally

- 다른 모듈에서 `ConfigModule`을 사용하려면 가져와야합니다 (모든 Nest 모듈의 표준과 동일).

- 또는 아래와 같이 옵션 객체의 `isGlobal` 속성을 `true`로 설정하여 이를 [global module](https://docs.nestjs.kr/modules#global-modules)로 선언합니다. 

  ```typescript
  ConfigModule.forRoot({
    isGlobal: true,
  });
  ```

  - 이 경우 루트 모듈(예: `AppModule`)에 로드되면 다른 모듈에서 `ConfigModule`을 가져올 필요가 없습니다.





## Custom configuration files

- 더 복잡한 프로젝트의 경우 사용자 지정 구성 파일을 사용하여 중첩된 구성 객체를 반환할 수 있습니다. 
  - 이를 통해 관련 구성 설정을 기능(예: 데이터베이스 관련 설정)별로 그룹화하고 관련 설정을 개별 파일에 저장하여 독립적으로 관리할 수 있습니다.

- 사용자 지정 구성 파일은 구성 객체를 반환하는 팩토리 함수를 내보냅니다. 

  - 구성 객체는 임의로 중첩된 일반 JavaScript 객체일 수 있습니다. 

  - `process.env` 객체에는 [위](https://docs.nestjs.kr/techniques/configuration#getting-started)에서 설명한대로 해결 및 병합 된 `.env` 파일 및 외부 정의 변수와 함께 완전히 해결된 환경 변수 키/값 쌍이 포함됩니다. 

  - 반환된 구성 객체를 제어하므로 값을 적절한 유형으로 캐스팅하고 기본값을 설정하는 데 필요한 논리를 추가할 수 있습니다.

    - 예를 들면 다음과 같습니다.

    ```typescript
    // config/configurations.ts
    export default () => ({
      port: parseInt(process.env.PORT, 10) || 3000,
      database: {
        host: process.env.DATABASE_HOST,
        port: parseInt(process.env.DATABASE_PORT, 10) || 5432
      }
    });
    ```

    이 파일은 `ConfigModule.forRoot()` 메서드에 전달하는 옵션 객체의 `load` 속성을 사용하여 로드합니다.

    ```typescript
    import configuration from './config/configuration';
    
    @Module({
      imports: [
        ConfigModule.forRoot({
          load: [configuration],
        }),
      ],
    })
    export class AppModule {}
    ```

> **알림**
>
> `load` 속성에 할당된 값은 배열이므로 여러 구성 파일을 로드할 수 있습니다 (예: `load: [databaseConfig, authConfig]`).



- 사용자 지정 구성 파일을 사용하면 YAML 파일과 같은 사용자 지정 파일도 관리할 수 있습니다. 

  - 다음은 YAML 형식을 사용하는 구성의 예입니다.

    ```yaml
    http:
      host: 'localhost'
      port: 8080
    
    db:
      postgres:
        url: 'localhost'
        port: 5432
        database: 'yaml-db'
    
      sqlite:
        database: 'sqlite.db'
    ```

  - YAML 파일을 읽고 파싱하기 위해 `js-yaml` 패키지를 활용할 수 있습니다.

    ```shell
    $ npm i js-yaml
    $ npm i -D @types/js-yaml
    ```

  - 패키지가 설치되면 `yaml#load` 함수를 사용하여 방금 만든 YAML 파일을 로드합니다.

    ```typescript
    // config/configuration.ts
    import { readFileSync } from 'fs';
    import * as yaml from 'js-yaml';
    import { join } from 'path';
    
    const YAML_CONFIG_FILENAME = 'config.yaml';
    
    export default () => {
      return yaml.load(
        readFileSync(join(__dirname, YAML_CONFIG_FILENAME), 'utf8'),
      ) as Record<string, any>;
    };
    ```

> **참고**
>
> - Nest CLI는 빌드 프로세스 중에 "자산"(비 TS 파일)을 `dist`폴더로 자동으로 이동하지 않습니다. 
> - YAML 파일이 복사되었는지 확인하려면 `nest-cli.json` 파일의 `compilerOptions#assets` 객체에 이를 지정해야 합니다. 
>   - 예를 들어,`config` 폴더가 `src` 폴더와 동일한 수준에 있는 경우 `"assets": [{"include": "../config/*.yaml", "outDir": "./dist/config"}]`를 추가합니다. [여기](https://docs.nestjs.kr/cli/monorepo#assets)에서 자세히 알아보세요.





## Using the `ConfigService`

- `ConfigService`에서 구성 값에 액세스하려면 먼저 `ConfigService`를 삽입해야합니다. 

  - 다른 프로바이더와 마찬가지로 포함 모듈인 `ConfigModule`을 사용할 모듈로 가져와야 합니다(`ConfigModule.forRoot()` 메소드에 전달된 옵션 객체의 `isGlobal` 속성을 `true`로 설정하지 않는 한).

  - 아래와 같이 기능 모듈로 가져옵니다.

    ```typescript
    // feature.module.ts
    @Module({
      imports: [ConfigModule],
      // ...
    })
    ```

  - 그런 다음 표준 생성자 주입을 사용하여 주입할 수 있습니다.

    ```typescript
    constructor(private configService: ConfigService) {}
    ```

    

    > **힌트**
    >
    > `ConfigService`는 `@nestjs/config` 패키지에서 가져옵니다.

    

  - 그리고 우리 클래스에서 사용하십시오.

    ```typescript
    // get an environment variable
    const dbUser = this.configService.get<string>('DATABASE_USER');
    
    // get a custom configuration value
    const dbHost = this.configService.get<string>('database.host');
    ```

    - 위와 같이 `configService.get()` 메소드를 사용하여 변수 이름을 전달하여 간단한 환경 변수를 가져옵니다. 
    - 위에 표시된대로 타입을 전달하여 TypeScript 타입 힌트를 수행할 수 있습니다 (예: `get<string>(...)`).
    - `get()`메소드는 위의 두번째 예와 같이 중첩된 사용자 정의 구성 객체 ([사용자 정의 구성 파일](https://docs.nestjs.kr/techniques/configuration#custom-configuration-files)을 통해 생성됨)를 탐색할 수도 있습니다.

    

  - 인터페이스를 타입 힌트로 사용하여 전체 중첩된 사용자 지정 구성 객체를 가져올 수도 있습니다.

    ```typescript
    interface DatabaseConfig {
      host: string;
      port: number;
    }
    
    const dbConfig = this.configService.get<DatabaseConfig>('database');
    
    // you can now use `dbConfig.port` and `dbConfig.host`
    const port = dbConfig.port;
    ```

    

  - `get()` 메서드는 또한 기본값을 정의하는 두번째 인수를 선택적으로 받습니다. 기본값은 아래와 같이 키가 존재하지 않을 때 반환됩니다.

    ```typescript
    // use "localhost" when "database.host" is not defined
    const dbHost = this.configService.get<string>('database.host', 'localhost');
    ```

    

  - `ConfigService`에는 존재하지 않는 구성 속성에 대한 액세스를 방지하는 데 도움이 되는 선택적 일반(타입 인수)이 있습니다. 아래와 같이 사용하십시오.

    ```typescript
    interface EnvironmentVariables {
      PORT: number;
      TIMEOUT: string;
    }
    
    // somewhere in the code
    constructor(private configService: ConfigService<EnvironmentVariables>) {
      const port = this.configService.get('PORT', { infer: true });
    
      // Error: this is invalid as the URL property is not defined
      const url = this.configService.get('URL', { infer: true });
    }
    ```

    - `infer` 속성을 `true`로 설정하면 `ConfigService#get` 메서드가 인터페이스를 기반으로 속성 유형을 자동으로 추론합니다.
    - 예를 들어 `PORT`는 `EnvironmentVariables` 인터페이스에 `number` 유형을 가지므로 `typeof port === "number"`입니다.

  - 또한 `infer` 기능을 사용하면 다음과 같이 점 표기법을 사용하는 경우에도 중첩된 사용자 지정 구성 객체의 속성 유형을 유추할 수 있습니다.

    ```typescript
    constructor(private configService: ConfigService<{ database: { host: string } }>) {
      const dbHost = this.configService.get('database.host', { infer: true });
      // typeof dbHost === "string"
    }
    ```

    



## Configuration namespaces

... 나중에























