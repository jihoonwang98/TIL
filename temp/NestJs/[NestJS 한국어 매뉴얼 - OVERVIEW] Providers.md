# [NestJS 한국어 매뉴얼 - OVERVIEW] Providers

> https://docs.nestjs.kr/providers



### 목차

- Providers
- Services
- Dependency Injection
- Scopes
- Custom providers
- Optional providers
- Property-based injection
- Provider registration
- Manual instantiation



## Providers

- Provider는 그냥 Nest 클래스일 뿐이다.
- Provider의 종류
  - Service
  - Repository
  - Factory
  - Helper
  - 등등
- Provider의 핵심 아이디어
  - Provider는 **Injectable**하다.
  - 즉, 객체는 서로 다양한 관계를 만들 수 있으며 객체의 인스턴스를 "연결"하는 책임은 대부분 Nest 런타임 시스템에게 떠넘길 수 있다.

![](https://docs.nestjs.kr/assets/Components_1.png)



- 컨트롤러는 HTTP 요청을 처리하고 더 복잡한 작업을 **Provider**에게 위임해야 합니다. 

  - Provider란, [모듈](https://docs.nestjs.kr/modules)에서 `provider`로 선언된 일반 자바스크립트 클래스입니다.

    ```ts
    @Module({
      providers: [CatsService, ... ],
    })
    ```

> **힌트**
>
> Nest를 사용하면 보다 객체지향적인 방식(OO-way)으로 종속성을 설계하고 구성할 수 있으므로 [SOLID](https://en.wikipedia.org/wiki/SOLID) 원칙을 따르는 것이 좋습니다.



## Services

- `CatsService` 예제

  ```ts
  // cats.service.ts
  import { Injectable } from '@nestjs/common';
  import { Cat } from './interfaces/cat.interface';
  
  @Injectable()
  export class CatsService {
    private readonly cats: Cat[] = [];
  
    create(cat: Cat) {
      this.cats.push(cat);
    }
  
    findAll(): Cat[] {
      return this.cats;
    }
  }
  
  // interfaces/cat.interface.ts
  export interface Cat {
    name: string;
    age: number;
    breed: string;
  }
  ```

  - 우리의 `CatsService`는 하나의 속성과 두개의 메서드가 있는 기본 클래스입니다. 
  - 유일한 새로운 기능은 `@Injectable()` 데코레이터를 사용한다는 것입니다.
    - `@Injectable()` 데코레이터는 `CatsService`가 Nest IoC 컨테이너에서 관리할 수 있는 클래스임을 선언하는 메타데이터를 첨부합니다.

- 이제 이 `CatsService` 클래스를 `CatsController` 내에서 사용해봅시다.

  ```ts
  // cats.controller.ts
  import { Controller, Get, Post, Body } from '@nestjs/common';
  import { CreateCatDto } from './dto/create-cat.dto';
  import { CatsService } from './cats.service';
  import { Cat } from './interfaces/cat.interface';
  
  @Controller('cats')
  export class CatsController {
    constructor(private catsService: CatsService) {}
  
    @Post()
    async create(@Body() createCatDto: CreateCatDto) {
      this.catsService.create(createCatDto);
    }
  
    @Get()
    async findAll(): Promise<Cat[]> {
      return this.catsService.findAll();
    }
  }
  ```

  - `CatsService`는 클래스 생성자(constructor)를 통해 DI됩니다. 
  - `private` 구문의 사용에 주목하십시오. 
    - 이 약식(shorthand)을 사용하면 동일한 위치에서 즉시 `catsService` 멤버를 선언하고 초기화할 수 있습니다.



## DI

- Nest는 일반적으로 **종속성 주입(DI)**으로 알려진 강력한 디자인 패턴을 기반으로 구축되었습니다.

- Nest에서는 TypeScript 기능 덕분에 종속성이 유형별로 해결되기 때문에 매우 쉽게 관리할 수 있습니다. 

  - 아래 예에서 Nest는 `CatsService`의 인스턴스를 만들고 반환하여 `catsService`를 해결합니다(또는 일반적인 경우 싱글톤의 경우 기존 인스턴스가 이미 다른 곳에서 요청된 경우 반환). 이 종속성은 해결되어 컨트롤러의 생성자(또는 표시된 속성에 할당됨)로 전달됩니다.

    ```ts
    constructor(private catsService: CatsService) {}
    ```



## Provider의 Scopes(수명, 범위)

- Provider는 일반적으로 <u>애플리케이션 수명주기와 동기화된</u> 수명("범위(scope)")을 갖습니다. 
  - 애플리케이션이 부트스트랩되면 모든 종속성을 해결해야하므로 모든 프로바이더를 인스턴스화해야 합니다. 
  - 마찬가지로 애플리케이션이 종료되면 각 프로바이더가 삭제됩니다. 
- 그러나 프로바이더 수명을 **요청 범위**로 만드는 방법도 있습니다. 이러한 기술에 대한 자세한 내용은 [여기](https://docs.nestjs.kr/fundamentals/injection-scopes)를 참조하세요.



## Custom providers

- Nest에는 Provider 간의 관계를 해결하는 내장된 제어반전("IoC") 컨테이너가 있습니다. 
- 이 기능은 위에서 설명한 종속성 주입기능의 기본이되지만 실제로 지금까지 설명한 것보다 훨씬 강력합니다. 
- 프로바이더를 정의하는 방법에는 여러가지가 있습니다. 
  - 일반 값, 클래스 및 비동기 또는 동기 팩토리를 사용할 수 있습니다. 
  - 더 많은 예가 [여기](https://docs.nestjs.kr/fundamentals/dependency-injection)에 제공됩니다.



## Optional providers

- 경우에 따라 반드시 해결하지 않아도 되는 종속성이 있을 수 있습니다. 

  - 예를 들어, 클래스는 **구성 객체**에 종속될 수 있지만 전달되는 것이 없으면 기본값을 사용해야 합니다. 
  - 이러한 경우 구성 프로바이더가 없어도 오류가 발생하지 않으므로 종속성이 선택사항이 됩니다.

- 프로바이더가 선택사항임을 나타내려면 생성자의 서명에 `@Optional()` 데코레이터를 사용하십시오.

  ```ts
  import { Injectable, Optional, Inject } from '@nestjs/common';
  
  @Injectable()
  export class HttpService<T> {
    constructor(@Optional() @Inject('HTTP_OPTIONS') private httpClient: T) {}
  }
  ```

  - 위의 예에서는 커스텀 provider를 사용하고 있으므로 `HTTP_OPTIONS` 커스텀 토큰을 포함합니다.
  - 이전 예제에서 생성자의 클래스를 통해 종속성을 나타내는 생성자 기반 주입을 보여주었습니다.
    - 커스텀 프로바이더 및 관련 토큰에 대한 자세한 내용은 [여기](https://docs.nestjs.kr/fundamentals/custom-providers)를 참조하세요.



## Property-based injection

- 지금까지 사용한 기술을 **생성자 기반 주입**이라고 합니다. 

  - 프로바이더는 생성자 메서드를 통해 주입되기 때문입니다. 

- 매우 특정한 경우 **속성 기반 주입**이 유용할 수 있습니다. 

  - 예를 들어 최상위 클래스가 하나 또는 여러 프로바이더에 의존하는 경우 생성자의 하위 클래스에서 `super()`를 호출하여 모든 프로바이더를 전달하는 것은 매우 지루할 수 있습니다. 
  - 이를 방지하기 위해 property 수준에서 `@Inject()` 데코레이터를 사용할 수 있습니다.

  ```js
  import { Injectable, Inject } from '@nestjs/common';
  
  @Injectable()
  export class HttpService<T> {
    @Inject('HTTP_OPTIONS')
    private readonly httpClient: T;
  }
  ```

> **경고**
>
> - 클래스가 다른 프로바이더를 확장하지 않는 경우 항상 **생성자 기반** 주입 사용을 선호해야 합니다.



## Provider registration

- 이제 프로바이더(`CatsService`)를 정의하고, 해당 서비스의 소비자(`CatsController`)를 확보했으므로 주입을 수행할 수 있도록 서비스를 Nest에 등록해야 합니다. 

- 모듈 파일(`app.module.ts`)을 편집하고 `@Module()` 데코레이터의 `providers` 배열에 서비스를 추가하여 이를 수행합니다.

  ```ts
  // app.module.ts
  import { Module } from '@nestjs/common';
  import { CatsController } from './cats/cats.controller';
  import { CatsService } from './cats/cats.service';
  
  @Module({
    controllers: [CatsController],
    providers: [CatsService],
  })
  export class AppModule {}
  ```

  - Nest는 이제 `CatsController` 클래스의 종속성을 해결할 수 있습니다.



## Manual instantiation

- 지금까지 Nest가 종속성 해결에 대한 대부분의 세부사항을 자동으로 처리하는 방법에 대해 논의했습니다. 
- 특정 상황에서는 내장된 종속성 주입 시스템을 벗어나 <u>**프로바이더를 수동으로 검색하거나 인스턴스화해야할 수 있습니다.**</u> 
- 아래에서 이러한 두 가지 주제에 대해 간략하게 설명합니다.
  - 기존 인스턴스를 가져오거나 프로바이더를 동적으로 인스턴스화하려면 [모듈 참조](https://docs.nestjs.kr/fundamentals/module-ref)를 사용할 수 있습니다.
  - `bootstrap()` 함수내에서 프로바이더를 가져오려면(예: 컨트롤러가 없는 독립형 애플리케이션 또는 부트스트랩중에 구성 서비스를 활용하려면) [독립형 애플리케이션](https://docs.nestjs.kr/standalone-applications)을 참조하세요.

