# [NestJs] Create Param Decorator to access Body, Params and Query in single Pipe implementation

> https://cloudreports.net/nestjs-create-param-decorator-to-access-body-params-and-query-in-single-pipe-implementation/



### 도입

- Parameter Decorator에 대해서 알아보자
- NestJS 컨트롤러에서는 request body, params 또는 query string의 값을 편리하게 받아올 수 있는 fundamental decorator들을 구비하고 있다.
  - `@Body()`
    - whole request body를 받을 때 (`@Body() body`)
    - field name을 같이 넘겨줘서 single body field를 받을 때 (`@Body('title') title: string`)
  - `@Query()`
    - whole query string을 받을 때
    - field name을 같이 넘겨줘서 single query field를 받을 때
  - `@Param()`
    - route configuration에 정의된 route parameter들에 접근할 때



## 1. Using `@Body` decorator

```typescript
import { Controller, Post, Body } from '@nestjs/common';

@Controller('/api/post')
export class PostController {
  @Post()
  async createPost(@Body('title') title: string) {
    return `Post ${title} created`;
  }
}
```

- `@Body('title')`
  - Request body 안의 **title** 필드에 접근할 때 사용됨



## 2. Using `@Param` decorator

```typescript
import { Controller, Get, Param } from '@nestjs/common';

@Controller('/api/post')
export class PostController {
  @Get('/detail/:id')
  async createPost(@Param('id') id: number) {
    return `Post ${id} fetched`;
  }
}
```

- `@Param('id')`
  - 경로 변수로 정의된 `id` 값을 가져옴



## 3. Using `@Query` decorator

```typescript
import { Controller, Get, Query } from '@nestjs/common';

@Controller('api/post')
export class PostController {
  @Get('/filter')
  async createPost(@Query('name') name: string) {
    return `Filter for ${name}`;
  }
}
```

- 정보를 전달할 때, Body나 Param을 유지하지 않고 **Query String**이라는 전통적인 방법을 사용할 수도 있다.
  - 당신이 `/api/post/filter?name=foo&otherFilterField=1`으로 접근하면 `foo`라는 이름을 `@Query`로 접근 가능하다.



## 4. Pipe

- Pipe

  - Pipe는 우리의 로직을 컨트롤러에서 분리시켜서 한 곳에 모아 놓을 수 있게 해주는 NestJS의 기능이다.
  - Pipe를 쓰면 좋은 점
    - 때때로 우리는 **<u>요청을 처리하기 전</u>**에 authentication, authorization, request validating 같은 전처리를 해야 할 때가 있다.
    - 만약 우리가 Pipe를 쓰지 않으면, auth/validate 로직을 route handler 안에 모두 집어 넣어야 한다. (가독성 down)

- 예제 1 (Simple pipe)

  ```typescript
  import { PipeTransform, Injectable, ArgumentMetadata } from '@nestjs/common';
  
  @Injectable()
  export class ValidationPipe implements PipeTransform {
    transform(value: any, metadata: ArgumentMetadata) {
      return value;
    }
  }
  ```

- 예제 2 (post ID를 체크하는 Pipe)

  ```typescript
  import { PipeTransform, Injectable, ArgumentMetadata, BadRequestException } from '@nestjs/common';
  
  @Injectable()
  export class PostIdValidationPipe implements PipeTransform {
    transform(id: number, metadata: ArgumentMetadata) {
      if (id > 10) {
        throw new BadRequestException('Validation failed');
      }
      return id;
    }
  }
  ```

  - id가 10보다 크면 에러 throw

  ```typescript
  import { Body, Controller, Get, Param, Query } from '@nestjs/common';
  import { BoardsService } from './boards.service';
  import { PostIdValidationPipe } from './pipes/PostIdValidationPipe';
  
  @Controller('/boards')
  export class BoardsController {
    constructor(private boardsService: BoardsService) {}
  
    @Get('/details/:id')
    async getPost(
      @Param('id', PostIdValidationPipe) id: number,
    ): Promise<string> {
      console.log('id: ', id);
      return `Post ${id} fetched.`;
    }
  }
  ```

  - 위에서 만든 pipe 컨트롤러에 적용하기
    - ` @Param('id', PostIdValidationPipe)`: 두번째 인자에 pipe를 넘겨주자

  

  - Pipe 컨셉은 유용하지만, 우리는 각각의 pipe implement에서 오직 한 종류의 정보만을 가질 수 있다는 한계점이 있다.
    - 예를 들어, 위의 예제에서 ` @Param('id', PostIdValidationPipe)`에서는 `PostIdValidationPipe`를 사용했는데, 이 Pipe는 Params 객체만을 가져올 수 있었다.
    - 그런데 만약 **<u>우리가 Param과 Body를 하나의 Pipe에서 접근하고 싶다</u>**면 어떻게 해야할까?
    - 이에 대한 답은 NestJS는 이 기능을 지원하는 builtin method가 없다는 것이다.
    - 따라서 우리는 우리 스스로 이 기능을 만들어야 한다.

  

  

  

  ## 5. Create Param Decorator to access Body, Params and Query in single Pipe 

  - 우리의 own Param Decorator를 만들어보자.

    ```typescript
    import { createParamDecorator } from '@nestjs/common';
    
    interface IObject {
      [key: string]: any;
    }
    
    export interface IIncomingData {
      body: IObject;
      params: IObject;
      query: IObject;
      req?: any;
      user?: any;
    }
    
    export const IncomingData = createParamDecorator(async (req): Promise<IIncomingData | void> => {
      const body = req.body;
      const params = req.params;
      const query = req.query;
      const user = req.user;
    
      const result: IIncomingData = {
        body,
        params,
        query,
        user,
      };
    
      return result;
    });
    ```

  - 이제 모든 request information을 한 곳에서 얻어낼 수 있다. 아래 처럼...

    ```typescript
    @Put(':id')
    async update(@IncomingData(PostDataPipe) post: PostDto) {
      const createdPost = await this.postService.create(post);
      return createdPost;
    }
    ```

    - You can implement PostDataPipe with using both **id** param and the whole Post Body to do validation…

  

  

  

  

  

  

  

  

  



