# Pipe 만들기



```ts
@Injectable()
export default class UserByIdPipe implements PipeTransform {
  constructor(private readonly userRepository: UserRepository) {}

  async transform(value: string, metadata: ArgumentMetadata) {
    console.log('typeof value: ', typeof value);
    console.log('value: ', value);
    console.log('metadata: ', metadata);
    console.log('metadata.metatype: ', metadata.metatype);
    let metatype = new metadata.metatype();
    console.log(metatype);

    const id = value;
    const user = await this.userRepository.findOne(id);
    if (!user) {
      throw new BadRequestException(`id: ${id}인 유저를 찾을 수 없다.`);
    }

    return user;
  }
}
```

```ts
@Controller('/users')
export default class UserController {
  
  @Get('/:id')
  async getUserByIdParam(
    @Param('id', UserByIdPipe) user: UserEntity,
  ): Promise<UserEntity> {
    return user;
  }

  @Post('')
  async getUserByIdBody(
    @Body('id', UserByIdPipe) user: UserEntity,
  ): Promise<UserEntity> {
    return user;
  }

  @Get('')
  async getUserByIdQuery(
    @Query('id', UserByIdPipe) user: UserEntity,
  ): Promise<UserEntity> {
    return user;
  }
}
```

- `GET /users/2`

  - `value`: `"2"`
  - `metadata`:  `{ metatype: [class UserEntity], type: 'param', data: 'id' }`

- `POST /users/`

  - `body`: 

    ```ts
    {
    	"id": 2
    }
    ```

  - `value`: 2

    - `typeof value`: `number`

  - `metadata`: `{ metatype: [class UserEntity], type: 'body', data: 'id' }`

    

  - `body`: 

    ```ts
    {
    	"id": "2"
    }
    ```

  - `value`: "2"

    - `typeof value`: `string`

  - `metadata`: `{ metatype: [class UserEntity], type: 'body', data: 'id' }`

- `GET /users/?id=2`
  - `value`: `"2"`
  - `metadat`: `metadata:  { metatype: [class UserEntity], type: 'query', data: 'id' }`



- Pipe 만드려면 `PipeTransform` implements 해야함
- `transform` 메서드 구현
  - 인자 설명
    - `value`: 위의 예에서는 `id` Param으로 넘어오는 값.
    - `me



### Issue

```ts
@Injectable()
export default class UserByIdPipe implements PipeTransform {
  constructor(private readonly userRepository: UserRepository) {}

  async transform(id: number, metadata: ArgumentMetadata) {
    console.log('id: ', id);
    console.log('typeof id: ', typeof id);
    const user = await this.userRepository.findOne(id);
    if (!user) {
      throw new BadRequestException(`id: ${id}인 유저를 찾을 수 없다.`);
    }

    return user;
  }
}
```

- `id: number`여도 `@Param('id', UserByIdPipe) `같이 value로 `"2"`같이 string넘어오면 typeof id가 string 찍힘