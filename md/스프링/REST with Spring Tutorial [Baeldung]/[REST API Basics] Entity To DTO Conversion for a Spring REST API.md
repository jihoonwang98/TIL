# Entity를 DTO 객체로 변환하기 in Spring REST API :cow: (Model Mapper)



> **[참고]:** Entity To DTO Conversion for a Spring REST API
>
> https://www.baeldung.com/entity-to-and-from-dto-for-a-java-spring-application
>
> 의역과 오역에 주의...





# 1. Overview :page_with_curl:

이번 포스팅에서는, 

Spring 애플리케이션 내에서 사용되는 내부 **Entity**와

Client에게 data를 전달할 때 사용되는 외부 **DTO(Data Transfer Object)** 간의 

**<u>변환 방법</u>**에 대해 살펴본다.







## 2. Model Mapper :electric_plug:

우리가 이번 포스팅에서 사용할 

**Entity-DTO 변환 라이브러리**, **<u>ModelMapper</u>**를 maven 의존성에 추가해주자.

```xml
<dependency>
    <groupId>org.modelmapper</groupId>
    <artifactId>modelmapper</artifactId>
    <version>2.3.5</version>
</dependency>
```



그다음 우리의 Spring configuration 파일에 ***ModelMapper*** 빈을 정의해주자.

```java
@Bean
public ModelMapper modelMapper() {
    return new ModelMapper();
}
```







## 3. The DTO :dash:

다음으로, Entity-DTO간의 변환시 발생하는 two-sided problem의 문제를 DTO 측면에서 살펴보자.



*PostDto*는 아래와 같다.

```java
public class PostDto {
    private static final SimpleDateFormat dateFormat
      = new SimpleDateFormat("yyyy-MM-dd HH:mm");

    private Long id;

    private String title;

    private String url;

    private String date;

    private UserDto user;

    public Date getSubmissionDateConverted(String timezone) throws ParseException {
        dateFormat.setTimeZone(TimeZone.getTimeZone(timezone));
        return dateFormat.parse(this.date);
    }

    public void setSubmissionDate(Date date, String timezone) {
        dateFormat.setTimeZone(TimeZone.getTimeZone(timezone));
        this.date = dateFormat.format(date);
    }

    // standard getters and setters
}
```



위 코드에서 두개의 Date 처리와 관련한 method들(*getSubmisstionDateConverted, setSubmissionDate*)이

Client와 Server 사이의 Date 변환을 처리하는 method들임을 주목하라.

- *getSubmissionDateConverted()* 메서드는 String 형태의 Date를 server의 timezone 형태의 Date 타입으로  바꾸는 역할을 한다. 
- *setSubmissionDate()* 메서드는 DTO의 *date* 필드를 현재 user timezone의 Post의 Date로 설정하는 역할을 한다.





## 4. The Service Layer :seat:

이제 DTO가 아니라 **Entity**를 가지고 역할을 수행하는

**Service Layer** 계층의 코드를 살펴보자.

```java
public List<Post> getPostsList(
  int page, int size, String sortDir, String sort) {
 
    PageRequest pageReq
     = PageRequest.of(page, size, Sort.Direction.fromString(sortDir), sort);
 
    Page<Post> posts = postRepository
      .findByUser(userService.getCurrentUser(), pageReq);
    return posts.getContent();
}
```







## 5. The Controller Layer :control_knobs:

다음으로 Entity-DTO conversion이 실제로 일어나는 **Controller Layer**에 대해 살펴보자.



아래의 예에서는 *Post* 자원에 대한 간단한 REST API를 가지고 있는 표준 controller 구현을 살펴보자.

우리는 여기서 몇가지의 간단한 CRUD 연산들(: create, update, get one, get all)을 살펴볼 것이다.

또한, **<u>Entity-DTO Conversion</u>**이 어떻게 일어나는지도 살펴보자.



```java
@Controller
class PostRestController {

    @Autowired
    private IPostService postService;

    @Autowired
    private IUserService userService;

    @Autowired
    private ModelMapper modelMapper;

    @GetMapping
    @ResponseBody
    public List<PostDto> getPosts(...) {
        //...
        List<Post> posts = postService.getPostsList(page, size, sortDir, sort);
        return posts.stream()
          .map(this::convertToDto)
          .collect(Collectors.toList());
    }

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    @ResponseBody
    public PostDto createPost(@RequestBody PostDto postDto) {
        Post post = convertToEntity(postDto);
        Post postCreated = postService.createPost(post));
        return convertToDto(postCreated);
    }

    @GetMapping(value = "/{id}")
    @ResponseBody
    public PostDto getPost(@PathVariable("id") Long id) {
        return convertToDto(postService.getPostById(id));
    }

    @PutMapping(value = "/{id}")
    @ResponseStatus(HttpStatus.OK)
    public void updatePost(@RequestBody PostDto postDto) {
        Post post = convertToEntity(postDto);
        postService.updatePost(post);
    }
}
```



아래는 ***Post* Entity -> *PostDTO***로의 변환 메서드다.

```java
private PostDto convertToDto(Post post) {
    PostDto postDto = modelMapper.map(post, PostDto.class);
    postDto.setSubmissionDate(post.getSubmissionDate(), 
        userService.getCurrentUser().getPreference().getTimezone());
    return postDto;
}
```



아래는 ***PostDTO* -> *Post* Entity**로의 변환 메서드다.

```java
private Post convertToEntity(PostDto postDto) throws ParseException {
    Post post = modelMapper.map(postDto, Post.class);
    post.setSubmissionDate(postDto.getSubmissionDateConverted(
      userService.getCurrentUser().getPreference().getTimezone()));
 
    if (postDto.getId() != null) {
        Post oldPost = postService.getPostById(postDto.getId());
        post.setRedditID(oldPost.getRedditID());
        post.setSent(oldPost.isSent());
    }
    return post;
}
```



보다싶이, *model mapper*의 도움으로 변환 로직이 아주 빠르고 간편하다.

우리는 ***mapper*의 *map()***를 사용해서 

한 줄의 conversion 로직도 작성하지 않고

변환된 data를 얻을 수 있다.





## 6. Unit Testing :memo:

마지막으로, Entity-DTO 간 변환이 잘 일어나는지 간단한 테스트를 작성해보자.



```java
public class PostDtoUnitTest {

    private ModelMapper modelMapper = new ModelMapper();

    @Test
    public void whenConvertPostEntityToPostDto_thenCorrect() {
        Post post = new Post();
        post.setId(1L);
        post.setTitle(randomAlphabetic(6));
        post.setUrl("www.test.com");

        PostDto postDto = modelMapper.map(post, PostDto.class);
        assertEquals(post.getId(), postDto.getId());
        assertEquals(post.getTitle(), postDto.getTitle());
        assertEquals(post.getUrl(), postDto.getUrl());
    }

    @Test
    public void whenConvertPostDtoToPostEntity_thenCorrect() {
        PostDto postDto = new PostDto();
        postDto.setId(1L);
        postDto.setTitle(randomAlphabetic(6));
        postDto.setUrl("www.test.com");

        Post post = modelMapper.map(postDto, Post.class);
        assertEquals(postDto.getId(), post.getId());
        assertEquals(postDto.getTitle(), post.getTitle());
        assertEquals(postDto.getUrl(), post.getUrl());
    }
}
```