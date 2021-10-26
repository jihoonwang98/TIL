# [NestJS 한글 매뉴얼] OVERVIEW - Exception filters

> https://docs.nestjs.kr/exception-filters





## Exception filters

- Nest에는 애플리케이션 전체에서 처리되지 않은 모든 예외를 처리하는 **예외 레이어**가 내장되어 있습니다.
- 애플리케이션 코드에서 예외를 처리하지 않으면 이 레이어에서 예외를 포착하여 적절한 사용자 친화적인 응답을 자동으로 보냅니다.

![](https://docs.nestjs.kr/assets/Filter_1.png)



- 기본적으로 이 작업은 `HttpException` 유형의 예외(및 그 하위 클래스)를 처리하는 내장 **전역 예외필터**에 의해 수행됩니다. 

- 예외가 **인식되지 않음**(`HttpException`도 `HttpException`에서 상속되는 클래스도 아님)이면 내장된 예외필터가 다음과 같은 기본 JSON 응답을 생성합니다.

  ```json
  {
    "statusCode": 500,
    "message": "Internal server error"
  }
  ```

> **힌트**
>
> - 전역 예외 필터는 `http-errors` 라이브러리를 부분적으로 지원합니다. 
> - 기본적으로 `statusCode` 및 `message` 속성을 포함하는 모든 던져진 예외는 제대로 채워지고 응답으로 다시 전송됩니다(인식되지 않는 예외에 대한 기본 `InternalServerErrorException` 대신).



## Throwing standard exceptions

- Nest는 `@nestjs/common` 패키지에서 노출된 내장 `HttpException` 클래스를 제공합니다. 
- 일반적인 HTTP REST/GraphQL API 기반 애플리케이션의 경우 특정 오류 조건이 발생할 때 표준 HTTP 응답객체를 보내는 것이 가장 좋습니다.



- 예제

  - 예를 들어 `CatsController`에는 `findAll()` 메서드(`GET` 라우트 핸들러)가 있습니다. 

  - 이 라우트 핸들러가 어떤 이유로 예외를 던진다고 가정해 봅시다. 

    ```typescript
    // cats.controller.ts
    @Get()
    async findAll() {
      throw new HttpException('Forbidden', HttpStatus.FORBIDDEN);
    }
    ```

    클라이언트가 이 엔드포인트를 호출하면 응답은 다음과 같습니다.

    ```json
    {
      "statusCode": 403,
      "message": "Forbidden"
    }
    ```

    

> **힌트**
>
> - 여기서는 `HttpStatus`를 사용했습니다. 이것은 `@nestjs/common` 패키지에서 가져온 헬퍼 열거(enum)형입니다.



- `HttpException` 생성자(constructor)는 응답을 결정하는 두개의 필수인수를 사용합니다.
  - `response` 인수는 JSON 응답 본문을 정의합니다. 아래에 설명된대로 `string`또는 `object`일 수 있습니다.
  - `status` 인수는 [HTTP 상태코드](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status)를 정의합니다.
- 기본적으로 JSON 응답 본문에는 두가지 속성이 포함됩니다.
  - `statusCode`: `status` 인수에 제공된 HTTP 상태코드가 기본값입니다.
  - `message`: `status`에 따른 HTTP 오류에 대한 간단한 설명

- JSON 응답 본문의 메시지 부분만 재정의하려면 `response` 인수에 문자열을 제공하세요.
- 전체 JSON 응답 본문을 재정의하려면 `response` 인수에 객체를 전달하세요. 
  - Nest는 객체를 직렬화하고 JSON 응답 본문으로 반환합니다.

- 두번째 생성자 인수 `status`는 유효한 HTTP 상태코드여야합니다. 
  - 모범사례는 `@nestjs/common`에서 가져온 `HttpStatus` 열거(enum)형을 사용하는 것입니다.



- 예제 (전체 응답 본문을 재정의하는 예제)

  ```typescript
  // cats.controller.ts
  @Get()
  async findAll() {
    throw new HttpException({
      status: HttpStatus.FORBIDDEN,
      error: 'This is a custom message',
    }, HttpStatus.FORBIDDEN);
  }
  ```

  위의 내용을 사용하면 응답이 다음과 같이 표시됩니다.

  ```json
  {
    "status": 403,
    "error": "This is a custom message"
  }
  ```

  



## Custom exceptions

- 대부분의 경우 커스텀 예외를 작성할 필요가 없으며 다음 섹션에 설명된대로 기본제공 Nest HTTP 예외를 사용할 수 있습니다.

- 커스텀 예외를 만들어야 하는 경우 커스텀 예외가 기본 `HttpException` 클래스에서 상속되는 고유한 **예외 계층**을 만드는 것이 좋습니다.

- 이 접근방식을 사용하면 Nest가 예외를 인식하고 오류 응답을 자동으로 처리합니다. 이러한 커스텀 예외를 구현해 보겠습니다.

  ```typescript
  // forbidden.exception.ts
  export class ForbiddenException extends HttpException {
    constructor() {
      super('Forbidden', HttpStatus.FORBIDDEN);
    }
  }
  ```

  - `ForbiddenException`은 기본 `HttpException`을 확장하므로 내장된 예외 핸들러와 원활하게 작동하므로 `findAll()` 메서드 내에서 사용할 수 있습니다.

    ```typescript
    // cats.controller.ts
    @Get()
    async findAll() {
      throw new ForbiddenException();
    }
    ```

    

## Build-in HTTP exceptions

- Nest는 기본 `HttpException`에서 상속되는 표준 예외 집합을 제공합니다. 
- 이들은 `@nestjs/common` 패키지에서 노출되며 가장 일반적인 HTTP 예외중 대부분을 나타냅니다.
  - `BadRequestException`
  - `UnauthorizedException`
  - `NotFoundException`
  - `ForbiddenException`
  - `NotAcceptableException`
  - `RequestTimeoutException`
  - `ConflictException`
  - `GoneException`
  - `HttpVersionNotSupportedException`
  - `PayloadTooLargeException`
  - `UnsupportedMediaTypeException`
  - `UnprocessableEntityException`
  - `InternalServerErrorException`
  - `NotImplementedException`
  - `ImATeapotException`
  - `MethodNotAllowedException`
  - `BadGatewayException`
  - `ServiceUnavailableException`
  - `GatewayTimeoutException`
  - `PreconditionFailedException`



## Exception filters

- 기본(내장) 예외필터가 자동으로 많은 경우를 처리할 수 있지만 예외 레이어에 대한 **완전한 제어**를 원할 수 있습니다. 
  - 예를 들어 로깅을 추가하거나 일부 동적요인을 기반으로 다른 JSON 스키마를 사용할 수 있습니다.
  - **예외필터**는 정확히 이러한 목적을 위해 설계되었습니다. 
  - 이를 통해 정확한 제어 흐름과 클라이언트로 다시 전송되는 응답 내용을 제어할 수 있습니다.



- `HttpException` 클래스의 인스턴스인 예외를 포착하고 이에 대한 커스텀 응답 로직을 구현하는 예외필터를 만들어 보겠습니다. 

  - 이렇게 하려면 기본플랫폼 `Request`및 `Response`객체에 액세스해야 합니다. 

  - `Request` 객체에 액세스하여 원래 `url`을 추출하여 로깅 정보에 포함할 수 있습니다.

  - `Response`객체를 사용하여 `response.json()`메서드를 사용하여 전송되는 응답을 직접 제어합니다.

    ```typescript
    // http-exception.filter.ts
    import { ExceptionFilter, Catch, ArgumentsHost, HttpException } from '@nestjs/common';
    import { Request, Response } from 'express';
    
    @Catch(HttpException)
    export class HttpExceptionFilter implements ExceptionFilter {
      catch(exception: HttpException, host: ArgumentsHost) {
        const ctx = host.switchToHttp();
        const response = ctx.getResponse<Response>();
        const request = ctx.getRequest<Request>();
        const status = exception.getStatus();
    
        response
          .status(status)
          .json({
            statusCode: status,
            timestamp: new Date().toISOString(),
            path: request.url,
          });
      }
    }
    ```

    - `@Catch(HttpException)` 데코레이터는 필요한 메타데이터를 예외필터에 바인딩하여 이 특정 필터가 `HttpException` 타입의 예외만 찾고있다는 것을 Nest에 알립니다.
    - `@Catch()` 데코레이터는 단일 매개변수 또는 쉼표로 구분된 목록을 사용할 수 있습니다. 
      - 이를 통해 한번에 여러 타입의 예외에 대한 필터를 설정할 수 있습니다.

> **힌트**
>
> - 모든 예외필터는 일반 `ExceptionFilter<T>` 인터페이스를 구현해야 합니다. 
> - 이를 위해서는 표시된 서명과 함께 `catch(exception: T, host: ArgumentsHost)` 메서드를 제공해야 합니다. `T`는 예외 타입을 나타냅니다.



## Arguments host

- `catch()` 메서드의 매개변수를 살펴 보겠습니다. 
  - `exception` 매개변수는 현재 처리중인 예외 객체입니다.
  - `host` 매개변수는 `ArgumentsHost` 객체입니다. 
    - `ArgumentsHost`는 [실행 컨텍스트 장](https://docs.nestjs.kr/fundamentals/execution-context)* 에서 자세히 살펴볼 강력한 유틸리티 객체입니다.
  - 위의 코드 샘플에서는 이를 사용하여 원래 요청 핸들러(예외가 발생한 컨트롤러에서)로 전달되는 `Request` 및 `Response` 객체에 대한 참조를 얻습니다. 









