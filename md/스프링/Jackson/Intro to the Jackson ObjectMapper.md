# Intro to the Jackson ObjectMapper



> **[참고]**: https://www.baeldung.com/jackson-object-mapper-tutorial
>
> 의역과 오역에 주의...





## 1. Overview :elephant:

이번 포스팅은 Jackson 라이브러리의 **ObjectMapper** 클래스를 이해하고 

어떻게 Java 객체를 JSON으로 serialize(직렬화)하고 

JSON string을 Java 객체로 deserialize(역직렬화)하는지를 이해하는데 집중합니다. 







## 2. Dependencies

먼저, 메이븐에 다음과 같이 의존성을 추가합니다.

```xml
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.11.1</version>
</dependency>
```



`jackson-databind`만 추가해도

`jackson-annotations`와 `jackson-core`가 함께 추가됩니다.







## 3. Reading and Writing Using ObjectMapper

먼저 기본적인 읽기와 쓰기 연산부터 시작해봅시다.



**ObjectMapper**의 *readValue* API를 사용하면 

**JSON 형식 ->  Java 객체**로 parsing 하거나 deserialize 할 수 있습니다.



반대로, **ObjectMapper**의 *writeValue* API를 사용하면

**Java 객체 -> JSON 형식**으로 serialize 할 수 있습니다.





일단, 아래와 같이 *Car* 클래스를 작성해봅시다.

```java
public class Car {

    private String color;
    private String type;

    // standard getters setters
}
```







### 3.1 Java Object -> JSON (writeValue)

먼저, **ObjectMapper**의 *writeValue* 메서드를 이용해 **Java 객체 -> JSON**으로 serialize하는 예를 살펴봅시다.



```java
ObjectMapper objectMapper = new ObjectMapper();
Car car = new Car("yellow", "renault");
objectMapper.writeValue(new File("target/car.json"), car);
```



위 코드를 실행하고 생성된 파일의 결과는 아래와 같습니다.

```json
{"color":"yellow","type":"renault"}
```





**ObjectMapper**의 *writeValueAsString*과 *writeValueAsBytes* 메서드들은 

**Java 객체 -> JSON**으로 변환한 뒤, 

변환된 JSON 값을 *String*이나 *Byte Array* 형태로 리턴합니다.

```java
String carAsString = objectMapper.writeValueAsString(car);
```







### 3.2 JSON -> Java Object (readValue)

아래는 JSON String을 Java 객체로 변환하는 간단한 예제입니다.

```java
String json = "{ \"color\" : \"Black\", \"type\" : \"BMW\" }";
Car car = objectMapper.readValue(json, Car.class);
```



*readValue()* 메서드는 JSON 문자열을 포함한 file이나 URL같이 

다양한 형태의 input들을 처리할 수 있습니다.

```java
Car car = objectMapper.readValue(new File("src/test/resources/json_car.json"), Car.class);
```

```java
Car car = 
  objectMapper.readValue(new URL("file:src/test/resources/json_car.json"), Car.class);
```







### 3.3 JSON -> Jackson *JsonNode*

JSON은 *JsonNode* 객체로 parsing 될 수도 있습니다.



```java
String json = "{ \"color\" : \"Black\", \"type\" : \"FIAT\" }";
JsonNode jsonNode = objectMapper.readTree(json);
String color = jsonNode.get("color").asText();
// Output: color -> Black
```





### 3.4 JSON Array String -> Java List

TypeReference를 이용해 JSON 배열 객체를 Java object List로 변환할 수도 있습니다.



```java
String jsonCarArray = 
  "[{ \"color\" : \"Black\", \"type\" : \"BMW\" }, { \"color\" : \"Red\", \"type\" : \"FIAT\" }]";
List<Car> listCar = objectMapper.readValue(jsonCarArray, new TypeReference<List<Car>>(){});
```





### 3.5 JSON String -> Java Map

JSON을 Java Map으로 parsing 할 수도 있습니다.



```java
String json = "{ \"color\" : \"Black\", \"type\" : \"BMW\" }";
Map<String, Object> map 
  = objectMapper.readValue(json, new TypeReference<Map<String,Object>>(){});
```





## 4. Advanced Features





















