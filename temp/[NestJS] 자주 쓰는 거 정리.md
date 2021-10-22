# [NestJS] 자주 쓰는 거 정리





### 입출력

- HTTP Body 값 DTO로 매핑받기 (`@Body()`)

  ```typescript
    @Post()
    async create(@Body() createCatDto: CreateCatDto) {
      console.log('name: ', createCatDto.name);
      console.log('age: ', createCatDto.age);
      console.log('breed: ', createCatDto.breed);
  
      return 'This action adds a new cat';
    }
  ```

  - 요청 방식이 `x-www-form-urlencoded`든 `JSON`이든 다 받을 수 있음

- 경로변수 받기 (`@Param()`)

  ```typescript
    @Get('/:id')
    getId(@Param('id') id: number): number {
      console.log('id: ', id);
      return id;
    }
  ```

- 쿼리 파라미터 받기 in Get method (`@Query()`)

  - ```typescript
      @Get('/query')
      getQueries(@Query() query: any): any {
        console.log('query: ', query);
        return query;
      }
    ```

    - 요청: `http://localhost:3000/cats/query?key1=val1&key2=val2&key3=val3`

    - 결과

      ```typescript
      query:  { key1: 'val1', key2: 'val2', key3: 'val3' }
      
      응답:
      {
          "key1": "val1",
          "key2": "val2",
          "key3": "val3"
      }
      ```

  - DTO로 받기

    ```typescript
      @Get('/query')
      getQueries(@Query() dto: CreateCatDto): any {
        console.log('getQueries method /query');
    
        console.log('name: ', dto.name);
        console.log('age: ', dto.age);
        console.log('breed: ', dto.breed);
    
        return dto;
      }
    ```

    - 요청: `http://localhost:3000/cats/query?name=val1&age=val2&breed=val3`

    - 결과

      ```typescript
      name:  val1
      age:  val2
      breed:  val3
      
      응답:
      {
          "name": "val1",
          "age": "val2",
          "breed": "val3"
      }
      ```



- 중첩 객체 받기

```typescript
// wheel.dto.ts
export class WheelDTO {
  name: string;
  age: number;
}

// car.dto.ts
import { WheelDTO } from './wheel.dto';
export class CarDto {
  wheels: WheelDTO[];
  name: string;
}

// cars.controller.ts
@Controller('/cars')
export class CarsController {
  @Post()
  method1(@Body() carDto: CarDto): void {
    console.log('carDto: ', carDto);
  }
}


[요청]
POST http://localhost:3000/cars
HTTP Body:
{
    "wheels": [
        {
            "name": "wheel1",
            "age": 1
        },
        {
            "name": "wheel2",
            "age": 2
        },
        {
            "name": "wheel3",
            "age": 3
        },
        {
            "name": "wheel4",
            "age": 4
        }
    ],
    "name": "my-car"
}

[출력결과]
carDto:  {
  wheels: [
    { name: 'wheel1', age: 1 },
    { name: 'wheel2', age: 2 },
    { name: 'wheel3', age: 3 },
    { name: 'wheel4', age: 4 }
  ],
  name: 'my-car'
}
```









