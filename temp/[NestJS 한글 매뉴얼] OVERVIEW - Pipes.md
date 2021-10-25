# [NestJS 한글 매뉴얼] OVERVIEW - Pipes

> https://docs.nestjs.kr/pipes



## Pipes

- 파이프는 `@Injectable()` 데코레이터로 주석이 달린 클래스입니다. 파이프는 `PipeTransform` 인터페이스를 구현해야 합니다.

![](https://docs.nestjs.kr/assets/Pipe_1.png)



- Pipe의 두가지 일반적인 사용 사례
  - **변환**(transformation): 입력 데이터를 원하는 형식으로 변환(예: 문자열에서 정수로)
  - **유효성 검사**(validation): 입력 데이터를 평가하고 유효한 경우 변경하지 않고 전달합니다. 그렇지 않으면 데이터가 올바르지 않을 때 예외를 발생시킵니다.
  - 두 경우 모두 파이프는 [컨트롤러 라우트 핸들러](https://docs.nestjs.kr/controllers#route-parameters)가 처리하는 `인수(arguments)`에서 작동합니다.
- Pipe 실행 시점
  - Nest는 **<u>메소드가 호출되기 직전</u>**에 파이프를 삽입하고 파이프는 메소드로 향하는 인수를 수신하고 이에 대해 작동합니다. 
  - 모든 변환 또는 유효성 검사 작업은 해당 시간에 발생하며 그 후 라우트 핸들러가(잠재적으로) 변환된 인수와 함께 호출됩니다.
  - 즉, Pipe 작동 -> 라우트 핸들러 호출



> **힌트**
>
> - 파이프는 예외 영역 내에서 실행됩니다. 
> - 이것은 파이프가 예외를 던질 때 예외 계층 (전역 예외필터 및 현재 컨텍스트에 적용되는 모든 [예외필터](https://docs.nestjs.kr/exception-filters))에 의해 처리된다는 것을 의미합니다. 
> - 위의 내용을 고려할 때 파이프에서 예외가 발생하면 이후에 컨트롤러 메서드가 실행되지 않음을 분명히해야 합니다. 이는 시스템 경계의 외부 소스에서 애플리케이션으로 들어오는 데이터를 검증하기 위한 모범사례 기술을 제공합니다.



## Built-in pipes

- Nest에는 즉시 사용할 수 있는 6 개의 파이프가 함께 제공됩니다.
  - `ValidationPipe`
  - `ParseIntPipe`
  - `ParseBoolPipe`
  - `ParseArrayPipe`
  - `ParseUUIDPipe`
  - `DefaultValuePipe`
  - 모두 `@nestjs/common` 패키지에서 내보내집니다.

- `ParseIntPipe`는 메서드 핸들러 매개변수가 자바스크립트 정수로 변환되도록 해준다. 또는 변환에 실패하면 예외가 발생함.



## Binding pipes

- 파이프를 사용하려면 파이프 클래스의 인스턴스를 적절한 컨텍스트에 바인딩해야 합니다.

- `ParseIntPipe` 예제에서 파이프를 특정 라우트 핸들러 메소드와 연관시키고 메소드가 호출되기 전에 실행되는지 확인하려고 합니다.

- 메서드 매개변수 수준에서 파이프를 바인딩하는 것으로 참조할 다음 구성으로 이를 수행합니다.

  ```typescript
  @Get(':id')
  async findOne(@Param('id', ParseIntPipe) id: number) {
    return this.catsService.findOne(id);
  }
  ```

  - 이렇게 하면 `findOne()` 메서드에서 수신하는 매개변수가 숫자거나 그렇지 않으면 무조건 예외가 발생하게 된다.

    - 예외가 발생하면 라우트 핸들러가 호출되기 전에 발생된다.

    - 예시

      ```json
      // GET localhost:3000/abc
      {
        "statusCode": 400,
        "message": "Validation failed (numeric string is expected)",
        "error": "Bad Request"
      }
      ```

    - 예외는 `findOne()` 메서드의 본문이 실행되지 않도록 한다.

  - 위의 예에서는 인스턴스가 아닌 클래스(`ParseIntPipe`)를 전달하여 인스턴스화를 프레임워크에 맡기고 종속성 주입을 활성화합니다.





- Pipe 및 가드와 마찬가지로 대신 <u>내부 인스턴스를 전달할 수 있습니다</u>.

  - 내부(in-place) 인스턴스를 전달하는 것은 옵션을 전달하여 내장 파이프의 동작을 커스텀하려는 경우 유용합니다.

  ```typescript
  @Get(':id')
  async findOne(
    @Param('id', new ParseIntPipe({ errorHttpStatusCode: HttpStatus.NOT_ACCEPTABLE }))
    id: number,
  ) {
    return this.catsService.findOne(id);
  }
  ```



- 다른 변환 파이프(모든 **Parse\*** 파이프)를 바인딩하는 방법도 비슷합니다. 

  - 이러한 파이프는 모두 라우트 매개변수, 쿼리 문자열 매개변수 및 요청 본문 값을 확인하는 컨텍스트에서 작동합니다.

    - `@Query()`
    - `@Param()`
    - `@Body()`

  - 예시 (Query String Parameter를 사용하는 경우)

    ```typescript
    @Get()
    async findOne(@Query('id', ParseIntPipe) id: number) {
      return this.catsService.findOne(id);
    }
    ```

  - 예시 (문자열 매개변수가 UUID인지 확인)

    ```typescript
    @Get(':uuid')
    async findOne(@Param('uuid', new ParseUUIDPipe()) uuid: string) {
      return this.catsService.findOne(uuid);
    }
    ```

    > **힌트**
    >
    > - `ParseUUIDPipe()`를 사용하는 경우 버전 3, 4 또는 5에서 UUID를 구문분석합니다. 
    > - 특정 버전의 UUID만 필요한 경우 파이프 옵션에서 버전을 전달할 수 있습니다.

  



## Custom pipes

- 언급했듯이 커스텀 파이프를 만들 수 있습니다.

- Nest는 강력한 내장 `ParseIntPipe` 및 `ValidationPipe`를 제공하지만, 각각의 간단한 커스텀 버전을 처음부터 만들어 커스텀 파이프가 어떻게 구성되는지 살펴 보겠습니다.

- 간단한 `ValidationPipe`를 만들어 봅시다.

  - 처음에는 단순히 입력값을 취하고 즉시 동일한 값을 반환하여 식별함수처럼 동작하도록 합니다.

    ```typescript
    // validation.pipe.ts
    import { PipeTransform, Injectable, ArgumentMetadata } from '@nestjs/common';
    
    @Injectable()
    export class ValidationPipe implements PipeTransform {
      transform(value: any, metadata: ArgumentMetadata) {
        return value;
      }
    }
    ```

> **힌트**
>
> - `PipeTransform<T, R>`은 파이프로 구현해야하는 일반 인터페이스입니다. 
> - 일반 인터페이스는 `T`를 사용하여 입력 `value`의 유형을 나타내고 `R`을 사용하여 `transform()`메서드의 반환유형을 나타냅니다.



- 모든 파이프는 `PipeTransform` 인터페이스 계약을 이행하기 위해 `transform()`메서드를 구현해야 합니다. 

- 이 메소드에는 두개의 매개변수가 있습니다. 

  - `value`
  - `metadata`

- `value` 매개변수는 현재 처리된 메서드 인수(라우트 처리 메서드에 의해 수신되기 전)이고,

- `metadata`는 현재 처리된 메서드 인수의 메타데이터입니다.

  - 메타데이터 객체에는 다음과 같은 속성이 있습니다.

    ```typescript
    export interface ArgumentMetadata {
      type: 'body' | 'query' | 'param' | 'custom';
      metatype?: Type<unknown>;
      data?: string;
    }
    ```

    

    | 프로퍼티   | 설명                                                         |
    | ---------- | ------------------------------------------------------------ |
    | `type`     | 인수가 본문 `@Body()`, 쿼리 `@Query()`, param `@Param()` 또는 커스텀 매개변수인지 여부를 나타냅니다(자세한 내용은 [여기](https://docs.nestjs.kr/custom-decorators)) 참조). |
    | `metatype` | 인수의 메타타입을 제공합니다(예: `String`). 참고: 라우트 핸들러 메소드 서명에서 타입 선언을 생략하거나 바닐라 자바스크립트를 사용하면 값이 `정의되지 않습니다`. |
    | `data`     | 데코레이터에 전달된 문자열(예: `@Body('string')`). 데코레이터 괄호를 비워두면 `정의되지 않습니다`. |



> **경고**
>
> - TypeScript 인터페이스는 변환중에 사라집니다. 
> - 따라서 메소드 매개변수의 타입이 클래스가 아닌 인터페이스로 선언되면 `metatype`값은 `Object`가 됩니다.



## Schema based validation







