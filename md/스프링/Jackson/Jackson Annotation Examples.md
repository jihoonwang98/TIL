# Jackson Annotation Examples :dog:



> **[참고]**: https://www.baeldung.com/jackson-annotations
>
> 의역과 오역에 주의...





## 1. Overview :mag:

이번 포스팅에서는, **Jackson Annotation**들에 대해 살펴본다.





## 2. Jackson Serialization Annotations :sun_with_face:

먼저, **serialization(직렬화)** 어노테이션들부터 살펴보자.





### 2.1 @JsonAnyGetter :yum:

**@JsonAnyGetter** 어노테이션은 **Map** 형태 필드의 모든 **key-value 쌍**을 *표준 일반 속성*으로 가져올 수 있게 해준다.



예를 들어, 아래와 같은 *ExtendableBean* 엔터티가 **String** 형태인 *name* 속성과 **Map** 형태인 *properties* 속성으로 구성되어 있다고 하자.

```java
public class ExtendableBean {
    public String name;
    private Map<String, String> properties;

    @JsonAnyGetter
    public Map<String, String> getProperties() {
        return properties;
    }
}
```





위처럼, **Map** 형태의 속성인 *properties*의 **getter**에 **@JsonAnyGetter** 어노테이션을 붙여주면,

이 객체의 인스턴스를 *serialize* 했을 때, 우리는 아래와 같이 **Map** 내에 있는 모든 **key-value 쌍**들을 *standard*하고 *plain*한 속성처럼 얻을 수 있다. 

```json
{
    "name":"My bean",
    "attr2":"val2",
    "attr1":"val1"
}
```





> [참고] **@JsonAnyGetter** 안 붙여주거나 **@JsonAnyGetter(enabled = false)** 속성을 지정하면, 아래와 같이 나온다.
>
> ```json
> {
>     "name":"My bean",
>     "properties": {
>         "attr2":"val2",
>         "attr1":"val1"
>     }
> }
> ```









### 2.2 @JsonGetter :clown_face:

**@JsonGetter** 어노테이션은 **@JsonProperty**과 기능이 비슷하다. **getter** 메서드 위에 붙여서 **<u>getter 이름 기반</u>**으로 키값이 정해지는 것을 어노테이션을 통해 바꿀 수 있다.



무슨 말인지 모르겠으면 아래 예를 보자. 



```java
public class MyBean {
    public int id;
    private String name;
    
    public MyBean(int id, String name) {
        this.id = id;
        this.name = name;
    }

    @JsonGetter("name")
    public String getTheName() {
        return name;
    }
}
```

위와 같이 MyBean 객체를 정의하자.

getter method로 get'<u>**TheName**</u>'을 정의했는데, 여기서 **@JsonGetter("name")** 어노테이션을 붙이지 않았다면 저 밑줄친 'TheName'을 키 값으로 인식한다. 



즉, 아래와 같이 나온다.

```java
public class MyBean {
    public int id;
    private String name;

    public MyBean(int id, String name) {
        this.id = id;
        this.name = name;
    }

    public String getTheName() {
        return name;
    }
    
    public static void main(String[] args) throws Exception{
        MyBean bean = new MyBean(1, "MyBean");
        String result = new ObjectMapper().writeValueAsString(bean);
        System.out.println(result);
    }
}


// output
{"id":1,"theName":"MyBean"}
```



그런데 여기서 저 결과의 **키 값**을 내가 원하는 값으로 지정해주기 위해서 **@JsonGetter** 어노테이션을 이용해 지정해주면, 아래와 같이 바뀐다. **getTheName** 메서드 위에 어노테이션만 붙여줬다.

```java
public class MyBean {
    public int id;
    private String name;

    public MyBean(int id, String name) {
        this.id = id;
        this.name = name;
    }


    @JsonGetter("name")
    public String getTheName() {
        return name;
    }

    public static void main(String[] args) throws Exception{
        MyBean bean = new MyBean(1, "MyBean");
        String result = new ObjectMapper().writeValueAsString(bean);
        System.out.println(result);
    }
}


// output
{"id":1,"name":"MyBean"}
```





> **[참고] @JsonProperty**
>
> **@JsonProperty**는 필드에도 붙일 수 있고, 메서드에도 붙일 수 있다.
>
> 
>
> - **필드**에 붙인 경우
>
> ```java
> public class MyBean {
>     
>     @JsonProperty("myId")
>     public int id;
>     private String name;
> 
>     public MyBean(int id, String name) {
>         this.id = id;
>         this.name = name;
>     }
> 
>     public String getTheName() {
>         return name;
>     }
> 
>     public static void main(String[] args) throws Exception{
>         MyBean bean = new MyBean(1, "MyBean");
>         String result = new ObjectMapper().writeValueAsString(bean);
>         System.out.println(result);
>     }
> }
> 
> // output
> {"theName":"MyBean","myId":1}
> ```
>
> 
>
> - **gettter** 메서드에 붙인 경우
>
> ```java
> public class MyBean {
>     public int id;
>     private String name;
> 
>     public MyBean(int id, String name) {
>         this.id = id;
>         this.name = name;
>     }
> 
>     @JsonProperty("name")
>     public String getTheName() {
>         return name;
>     }
> 
>     public static void main(String[] args) throws Exception{
>         MyBean bean = new MyBean(1, "MyBean");
>         String result = new ObjectMapper().writeValueAsString(bean);
>         System.out.println(result);
>     }
> }
> 
> // output
> {"id":1,"name":"MyBean"}
> ```







### 2.3 @JsonPropertyOrder :crab:

**@JsonPropertyOrder** 어노테이션을 통해 글자 그대로 **serialization**할때  **<u>property들의 순서</u>**를 지정할 수 있다.



**MyBean**을 아래와 같이 작성하고,

```java
@JsonPropertyOrder({ "name", "id" })
public class MyBean {
    public int id;
    public String name;
}
```



**serialization**을 해주면,

```java
{
    "name":"My bean",
    "id":1
}
```



어노테이션에 지정한 순서대로 결과가 나온다! :+1:



>  참고로, **@JsonPropertyOrder(alphabetic=true)**라는 어노테이션을 사용하면 property들의 순서를 alphabetical하게 정렬할 수도 있다고 한다...





### 2.4 @JsonRaw Value :deer:

**@JsonRawValue** 어노테이션은 속성 값을 생긴 그대로 **serialize** 할 수 있게 해준다.

역시나 무슨 말인지 모르겠다면 :dizzy_face: 아래 예시를 보자.. 



우선, **@JsonRawValue** 어노테이션을 붙이지 않은 경우 어떻게 되는지 보자.

```java
public class RawBean {
    public String name;
    
    public String json;

    public RawBean(String name, String json) {
        this.name = name;
        this.json = json;
    }

    public static void main(String[] args) throws JsonProcessingException {
        RawBean bean = new RawBean("name", "{\"attr\":true}");
        String res = new ObjectMapper().writeValueAsString(bean);
        System.out.println(res);
    }
}

// output
{"name":"name","json":"{\"attr\":true}"}
```

*name* 속성에는 "name" 문자열을,

*json* 속성에는 "{\\"attr\\":true}" 문자열을 넣어서 객체를 만들고 **serialize** 했더니



**json** 키 값의 **value**가 **<u>문자열 형태</u>**가 나왔다.

만약, 우리가 넘겨준 문자열을 json 객체 형태로 **serialize** 하고 싶은 경우에는 어떻게 해야할까?





바로 이때 **json property** 위에 **@JsonRawValue** 어노테이션을 붙여준다. :muscle:

```java
public class RawBean {
    public String name;

    @JsonRawValue
    public String json;

    public RawBean(String name, String json) {
        this.name = name;
        this.json = json;
    }

    public static void main(String[] args) throws JsonProcessingException {
        RawBean bean = new RawBean("name", "{\"attr\":true}");
        String res = new ObjectMapper().writeValueAsString(bean);
        System.out.println(res);
    }
}

// output
{"name":"name","json":{"attr":true}}
```



어노테이션을 붙여주니 **serialize**된 값이 문자열 형태가 아니라 Json 객체 형태로 잘 변환됐다! :full_moon_with_face:





### 2.5 @JsonValue



### 2.6 @JsonRootName



### 2.7 @JsonSerialize





## 3. Jackson Deserialization Annotations



### 3.1 @JsonCreator



### 3.2 @JacksonInject



### 3.3 @JsonAnySetter



### 3.4 @JsonSetter



### 3.5 @JsonDeserialize



### 3.6 @JsonAlias



## 4. Jackson Property Inclusion Annotations

### 4.1 @JsonIgnoreProperties

### 4.2 @JsonIgnore

### 4.3 @JsonIgnoreType

### 4.4 @JsonInclude

### 4.5 @JsonAutoDetect



## 5. Jackson Polymorphic Type Handling Annotations





## 6. Jackson General Annotations

### 6.1 @JsonProperty

### 6.2 @JsonFormat

### 6.3 @JsonUnwrapped

### 6.4 @JsonView

### 6.5 @JsonManagedReference, @JsonBackReference

### 6.6 @JsonIdentityInfo

### 6.7 @JsonFilter



## 7. Custom Jackson Annotation



## 8. Jackson MixIn Annotatinos



## 9. Disable Jackson Annotation









