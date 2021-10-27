# [TypeORM 이슈] Entity constructor 

- `Post`

  ```typescript
  @Entity()
  export class Post {
    @PrimaryGeneratedColumn()
    readonly id: number;
  
    @Column()
    readonly title: string;
  
    @Column("text")
    readonly text: string
  
    constructor(title: string, text: string) {
      console.log('constructor 호출됨.')
      console.log('title: ', title);
      console.log('text: ', text);
      this.title = title;
      this.text = text;
    }
  
  }
  ```



- `postGetAllAction`

  ```typescript
  export async function postGetAllAction(context: Context) {
  
    // get a post repository to perform operations with post
    const postRepository = getManager().getRepository(Post);
  
    console.log('postRepository.find() 전');
    const posts = await postRepository.find();  // load all posts
    console.log('posts: ', posts);
    console.log('postRepository.find() 후');
  
  
    // return loaded posts
    context.body = posts;
  }
  
  // post 두 개 있는 상태에서 
  // GET http://localhost:3000/posts 호출
  postRepository.find() 전
  constructor 호출됨.
  title:  undefined
  text:  undefined
  constructor 호출됨.
  title:  undefined
  text:  undefined
  posts:  [
    Post { title: 'title1', text: 'text1', id: 1 },
    Post { title: 'title2', text: 'text2', id: 2 }
  ]
  postRepository.find() 후
  ```







