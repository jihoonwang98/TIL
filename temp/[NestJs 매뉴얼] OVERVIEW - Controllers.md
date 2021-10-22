# [NestJs 매뉴얼] OVERVIEW - Controllers

> https://docs.nestjs.kr/controllers



## Controllers

- 컨트롤러는 들어오는 **요청 request**을 처리하고 **응답 response**을 클라이언트에 반환할 책임이 있습니다.

![](https://docs.nestjs.kr/assets/Controllers_1.png)



- 컨트롤러의 목적은 애플리케이션에 대한 특정 요청을 수신하는 것입니다.
- **라우팅** 메커니즘은 어떤 컨트롤러가 어떤 요청을 수신하는지 제어합니다. 
  - 종종 각 컨트롤러에는 둘 이상의 라우트가 있으며 다른 라우트는 다른 작업을 수행할 수 있습니다.



- 기본 컨트롤러를 만들기 위해 클래스와 **데코레이터**를 사용합니다. 
  - 데코레이터는 클래스를 필수 메타데이터와 연결하고 Nest가 라우팅 맵을 만들 수 있도록 합니다(요청을 해당 컨트롤러에 연결).

> **힌트**
>
> [validation](https://docs.nestjs.kr/techniques/validation)이 내장된 CRUD 컨트롤러를 빠르게 생성하려면 CLI의 [CRUD 생성기](https://docs.nestjs.kr/recipes/crud-generator#crud-generator): `nest g resource [name]`을 사용할 수 있습니다.



### Routing

> **힌트**
>
> CLI를 사용하여 컨트롤러를 만들려면 `$ nest g controller cats` 명령을 실행하면 됩니다.

- 예제

  ```typescript
  // cats.controller.ts
  import { Controller, Get } from '@nestjs/common';
  
  @Controller('cats' /* 선택적(optional) 컨트롤러 경로 접두사 */)
  export class CatsController {
    @Get()
    findAll(): string {
      return 'This action returns all cats';
    }
  }
  ```

  - 위 예제에서는 기본 컨트롤러를 정의하는 데 **필수**인 `@Controller()` 데코레이터를 사용합니다.
  - 선택적 라우트 경로(path) 접두사 `cats`를 지정합니다. 
  - `@Controller()` 데코레이터에서 경로(path) 접두사를 사용하면 관련 라우트 집합을 쉽게 그룹화하고 반복코드를 최소화할 수 있습니다. 
    - 예를 들어 `/customers` 라우트 아래에서 고객 엔터티와의 상호작용을 관리하는 라우트 집합을 그룹화하도록 선택할 수 있습니다. 
    - 이 경우 `@Controller()` 데코레이터에서 경로(path) 접두사 `customers`를 지정하여 파일의 각 라우트에 대해 경로(path)의 해당부분을 반복할 필요가 없습니다.
  - `findAll()` 메서드 앞에 있는 `@Get()` HTTP 요청 메서드 데코레이터는 Nest에 HTTP 요청에 대한 특정 엔드포인트에 대한 핸들러를 생성하도록 지시합니다.
    - 엔드포인트란? 
      - 엔드포인트는 HTTP 요청 메서드(이 경우 GET) 및 라우트 경로에 해당합니다. 
    - 라우트 경로란?
      - 핸들러의 라우트 경로는 컨트롤러에 대해 선언된(선택 사항) 접두사와 메소드 데코레이터에 지정된 경로를 연결하여 결정됩니다.
  - 모든 라우트 (`cats`)에 대한 접두사를 선언하고 데코레이터에 경로 정보를 추가하지 않았으므로 Nest는 `GET /cats` 요청을 이 핸들러에 매핑합니다.
  - 언급했듯이 경로에는 선택적 컨트롤러 경로 접두사 **및** 요청 메서드 데코레이터에서 선언된 모든 경로 문자열이 모두 포함됩니다. 
    - 예를 들어, 데코레이터 `@Get('profile')`과 결합된 `customers`의 경로 접두사는 `GET /customers/profile`과 같은 요청에 대한 라우트 매핑을 생성합니다.
  - 위의 예에서 이 엔드포인트에 GET 요청이 있을 때 Nest는 요청을 커스텀 `findAll()` 메서드로 라우팅합니다. 
    - 여기서 선택한 메서드 이름은 완전히 임의적입니다.
    - 분명히 경로를 바인딩할 메서드를 선언해야 하지만 Nest는 선택한 메서드 이름에 어떤 의미도 부여하지 않습니다.
  - 이 메서드는 200 상태 코드와 관련 응답을 반환합니다. 
    - 이 경우에는 문자열일 뿐입니다. 
    - 왜 그럴까요? 
      - 설명하기 위해 먼저 <u>Nest가 응답을 조작하기 위해 **다른** 두가지 옵션을 사용한다</u>는 개념을 소개하겠습니다.



| 옵션                      | 설명                                                         |
| ------------------------- | ------------------------------------------------------------ |
| 표준 (권장)               | 이 내장 메서드를 사용하면 요청 핸들러가 자바스크립트 객체 또는 배열을 반환할 때 **자동**으로 JSON으로 직렬화됩니다. 그러나 자바스크립트 기본 타입(예: `string`, `number`, `boolean`)을 반환하면 Nest는 직렬화를 시도하지 않고 값만 보냅니다. 이렇게 하면 응답처리가 간단해집니다. 값을 반환하기만 하면 Nest가 나머지 작업을 처리합니다.  또한 응답의 **상태 코드**는 201을 사용하는 POST 요청을 제외하고는 항상 기본적으로 200입니다. 핸들러 수준에서 `@HttpCode(...)` 데코레이터를 추가하여 이 동작을 쉽게 변경할 수 있습니다(참조: [상태 코드](https://docs.nestjs.kr/controllers#status-code)). |
| 라이브별 Library-specific | 메소드 핸들러 시그니처(예: `findAll(@Res() response)`)에서 `@Res()` 데코레이터를 사용하여 삽입할 수 있는 라이브러리별(예: Express) [응답객체](https://expressjs.com/en/api.html#res)를 사용할 수 있습니다. 예를 들어 Express에서는 `response.status(200).send()`와 같은 코드를 사용하여 응답을 구성할 수 있습니다. |

> **경고**
>
> Nest는 핸들러가 `@Res()` 또는 `@Next()`를 사용할 때 이를 감지하여 라이브러리별 옵션을 선택했음을 나타냅니다. 두 접근 방식을 동시에 사용하는 경우 이 단일 라우트에 대해 표준 접근방식이 **자동으로 비활성화**되고 더 이상 예상대로 작동하지 않습니다. 두 접근 방식을 동시에 사용하려면 (예: 쿠키/헤더만 설정하고 나머지는 프레임워크에 남겨 두도록 응답객체를 삽입) `@Res({ passthrough: true })` 데코레이터에서 `passthrough` 옵션을 `true`로 설정해야 합니다.



### Request object

- 핸들러는 종종 클라이언트 **요청** 세부정보에 액세스해야 합니다. 
- Nest는 기본 플랫폼(기본적으로 Express)의 [요청객체](https://expressjs.com/en/api.html#req)에 대한 액세스를 제공합니다. 
- 핸들러의 시그니처에 `@Req()` 데코레이터를 추가하여 Nest에 주입하도록 지시하여 요청객체에 액세스할 수 있습니다.

```typescript
// cats.controller.ts
import { Controller, Get, Req } from '@nestjs/common';
import { Request } from 'express';

@Controller('cats')
export class CatsController {
  @Get()
  findAll(@Req() request: Request): string {
    return 'This action returns all cats';
  }
}
```



> **힌트**
>
> `express` 타입(위의 `request: Request` 매개변수 예제 참조)을 활용하려면 `@types/express` 패키지를 설치합니다.

- 요청객체는 HTTP 요청을 나타내며 요청 쿼리 문자열, 매개변수, HTTP 헤더 및 본문에 대한 속성을 포함합니다(자세한 내용은 [여기](https://expressjs.com/en/api.html#req) 참조). 
- 대부분의 경우 이러한 속성을 수동으로 가져올 필요가 없습니다. 
  - 바로 사용할 수 있는 `@Body()` 또는 `@Query()`와 같은 전용 데코레이터를 대신 사용할 수 있습니다. 

- 아래는 제공된 데코레이터와 이들이 나타내는 일반 플랫폼별 객체 목록입니다.

| 데코레이터                 | 일반 플랫폼별 객체 목록             |
| -------------------------- | ----------------------------------- |
| `@Request(), @Req()`       | `req`                               |
| `@Response(), @Res()`***** | `res`                               |
| `@Next()`                  | `next`                              |
| `@Session()`               | `req.session`                       |
| `@Param(key?: string)`     | `req.params` / `req.params[key]`    |
| `@Body(key?: string)`      | `req.body` / `req.body[key]`        |
| `@Query(key?: string)`     | `req.query` / `req.query[key]`      |
| `@Headers(name?: string)`  | `req.headers` / `req.headers[name]` |
| `@Ip()`                    | `req.ip`                            |
| `@HostParam()`             | `req.hosts`                         |





