# Java 8 Stream API 정리



> 참고: 
>
> https://howtodoinjava.com/java8/java-streams-by-examples/
>
> https://beomseok95.tistory.com/217





## 1. Java Stream vs Collection

Youtube에서 동영상을 보기 시작하면, 동영상 파일의 앞 부분만 부분적으로 먼저 로딩되면서 동영상이 재생된다.

우리는 동영상을 재생하기 전에 완전한 동영상 파일을 준비할 필요가 없다. 

이것이 바로 **Streaming**이다.



아주 추상적인 레벨에서 접근하면,

**Stream**을 동영상 파일의 작은 부분들을 모아 놓은 것으로 생각할 수 있고,

**Collection**은 하나의 완전한 동영상 파일로 생각할 수 있다.



좀 더 구체적인 레벨에서 **Stream**과 **Collection**의 차이점을 살펴보자.

**Collection**

- in-memory 자료구조
- 현재 Collection이 갖고 있는 모든 값을 메모리에 올린다.
- 어떤 값을 Collection에 추가하고 싶다면, 반드시 해당 값을 evaluation한 후에 추가할 수 있다.
- Collection에 어떤 요소를 추가하거나 삭제할 때마다 Collection의 모든 요소를 메모리에 올려야 하며, Collection에 추가하려는 값은 미리 evaluation이 되어야 한다.



**Stream**

- Collection과 달리 <u>요청할 때만 요소를 계산</u>하는 고정된 자료구조다.
  - Stream에 요소를 추가하거나 삭제할 수 없다.

- Stream을 사용하는 User는 그들이 원하는 값만 쏙 뽑아서 사용할 수 있다. 그리고, User로부터 요청이 들어온 값들만 Stream에서 그 즉시 생산한다.
  - 이런 형태를 producer-consumer 관계라고 부른다.





### Stream의 특징

- Not a data structure
- 람다를 위해 설계됨
- index를 통한 접근이 불가능함
- 쉽게 array나 list로 전환된다
- Lazy access가 지원된다
- Parallelizable하다 (병렬 연산 가능)







## 2. Creating Streams



### (1) `Stream.of()`

아래의 예시에서는 integer로 이루어진 stream을 생성한다.

```java
Stream<Integer> stream = Stream.of(1,2,3,4,5,6,7,8,9);
stream.forEach(p -> System.out.println(p));
```





### (2) `Stream.of(array)`

아래의 예시에서는 array를 통해 stream을 생성한다.

```java
Stream<Integer> stream = Stream.of(new Integer[]{1,2,3,4,5,6,7,8,9});
stream.forEach(p -> System.out.println(p));
```





### (3) `List.stream()`

아래의 예시에서는 `List`로부터 stream을 생성한다.

```java
List<Integer> list = Arrays.asList(1,2,3,4,5);
Stream<Integer> stream = list.stream();
stream.forEach(p -> System.out.println(p));
```





### (4) `Stream.generate()` 

아래의 예시에서는 `Stream`의 `generate()` 메서드를 통해 `Stream`을 생성한다.

아래의 예시에서는 20개의 난수로 이루어진 stream을 생성한다.

`limit()` 메서드를 통해 요소의 개수를 제한했다.

```java
Stream<Integer> randomNumbers = Stream.generate(() -> (new Random()).nextInt(100));
randomNumbers.limit(20).forEach(System.out::println);
```



**메서드 형태**

```java
public static<T> Stream<T> generate(Supplier<T> s);
```

- `generate()`를 통해 생성되는 stream은 크기가 무한하기 때문에 `limit(int)`를 호출하여 특정 사이즈로 최대 크기를 제한해야 한다.





### (5) `Stream.iterate()`

아래의 예시에서는 `Stream`의 `iterate()` 메서드를 통해 `Stream`을 생성한다.

아래의 예시는 초깃값 0부터 2씩 더해나가면서 총 10개의 요소를 stream으로 만든다.

```java
Stream<Integer> stream = Stream.iterate(0, a -> a + 2).limit(10);
stream.forEach(i -> System.out.print(i + " "));

// output
0 2 4 6 8 10 12 14 16 18 
```



**메서드 형태**

```java
public static<T> Stream<T> iterate(final T seed, final UnaryOperator<T> f);
```





### (6) String chars 또는 tokens로 `Stream` 생성하기

아래의 예시에서는 첫 번째로, 주어진 String의 character들로 stream을 생성한다.

두 번째는, string을 `split()` 메서드를 통해 쪼개서 얻은 token들로 이루어진 stream을 생성한다.

```java
IntStream stream = "12345_abcdefg".chars();
stream.forEach(p -> System.out.print(p + " "));
System.out.println();

Stream<String> stream2 = Stream.of("A$B$C".split("\\$"));
stream2.forEach(p -> System.out.print(p + " "));

// output
49 50 51 52 53 95 97 98 99 100 101 102 103 
A B C 
```







### (7) `Arrays.stream(array)`

```java
String[] arr = new String[]{"a", "b", "c"};
Stream<String> stream = Arrays.stream(arr);
Stream<String> streamOfArrayPart = Arrays.stream(arr, 1, 3); // 1~2 요소 [b,c]
```





### (8) `Collection.stream()`

```java
List<String> list = Arrays.asList("a", "b", "c");
Stream<String> stream = list.stream();
Stream<String> parallelStream = list.parallelStream(); // 병렬 처리 스트림
```





### (9) `Stream.empty()`

비어있는 스트림을 생성한다.





### (10) `Stream.<T>builder()`

**메서드 형태**

```java
public static<T> Builder<T> builder() {
    return new Streams.StreamBuilderImpl<>();
}
```



**예시**

```java
Stream<String> builderStream =
                Stream.<String>builder()
                        .add("Eric")
                        .add("Elena")
                        .add("Java")
                        .build(); // [Eric, Elena, Java]
```





### (11) 기본 타입형 Stream

```java
public IntStream generateIntStream() {
    IntStream is = IntStream.range(1, 5);
    //1,2,3,4
    return is;
}

public LongStream generateLongStream() {
    LongStream ls = LongStream.rangeClosed(1, 5);
    //1,2,3,4,5
    return ls;
}

public void generateRandomStream() {
    DoubleStream ds = new Random().doubles(3);
    IntStream is = new Random().ints(3);
    LongStream ls = new Random().longs(3);
    // 난수 3개 생성, double, int, long 가능
}
```





## 3. Stream Collectors



> **intermediate operation** vs **terminal operation**
>
> - **intermediate operation**(중간 연산)
>   - stream을 리턴하는 연산
>   - stream을 리턴하기 때문에 여러 가지 메서드들을 chaining해서 다양한 operation을 수행해 나갈 수 있다.
> - **terminal operation**(마지막 연산)
>   - stream이 아니라 특정 타입의 결과를 리턴하는 연산



intermediate operation으로 stream에 대해 연산을 수행한 후,

우리는 `Stream`의 `Collector` 메서드들을 이용해 stream의 요소들을 `Collection` 형태로 모을 수 있다.





### (1) Collect Stream elements to a List

```java
List<Integer> results = Stream.iterate(1, i -> i + 1).limit(10)
        .filter(i -> i % 2 == 0)
        .collect(Collectors.toList());

System.out.println(results);  // [2, 4, 6, 8, 10]
```





### (2) Collect Stream elements to a Array

```java
Integer[] results = Stream.iterate(1, i -> i + 1).limit(10)
        .filter(i -> i % 2 == 0)
        .toArray(Integer[]::new);

for (Integer result : results) {
    System.out.print(result + " ");
}
```



이외에 [Collectors](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Collectors.html)의 메서드를 살펴보면 Stream의 요소를 Collection으로 만드는 메서드가 아주 많다.





## 4. Stream Operations

Stream Operation은 위에서 언급했듯이 2가지로 구분할 수 있다.

- Intermediate Operations
- Terminal Operations



### (1) Intermediate Operations

Intermediate Operation들은 stream을 리턴하므로, 여러 메서드들을 chaining해서 연속적으로 호출할 수 있다.



#### - `filter()`

`filter()` 메서드는 `Predicate`를 파라미터로 받아서 stream의 모든 요소들을 `Predicate`로 검사해서 참인 것만 걸러낸다.



**예시**

```java
list.stream().filter(s -> s.startsWith("A"))
    .forEach(System.out::println);
```





#### - `map()`

`map()` 메서드는 stream 안의 각각의 요소들을 파라미터로 넘어온 함수로 변환시킨다.



**예시**

아래의 예시는 stream 내의 모든 string들을 대문자로 변환하는 예제다.

```java
list.stream().map(String::toUpperCase)
	.forEach(System.out::println);
```





#### - `sorted()`

`sorted()` 메서드는 stream을 정렬해서 리턴한다.

만약 custom `Comparator`를 넘기지 않으면 natural order로 정렬된다.



**예시**

```java
list.stream().sorted()
	.forEach(System.out::println);
```

- 주의할 점은 여기서 stream의 source인 list는 전혀 변형되지 않는다. 오직 stream만이 정렬된다.







### (2) Terminal Operations

Terminal Operation들은 모든 stream의 요소들을 처리하고 난뒤의 최종 결과로 stream이 아닌 특정 타입을 리턴한다.



Stream에 terminal operation이 호출될 때, 그제서야 Stream의 iteration과 chaining된 stream operation들이 시작된다.

iteration이 끝나면, terminal operatoin의 결과가 반환된다.



#### - `forEach()`

`forEach()` 메서드는 stream 내의 모든 요소들을 iterating 하면서 각각의 요소에 operation을 수행하는 것을 도와주는 메서드다.

각각의 요소에 수행되어야 할 연산들은 lambda 표현식으로 전달된다.



**예시**

```java
list.forEach(System.out::println);
list.forEach(elem -> System.out.println(elem));
```





#### - `collect()`

`collect()` 메서드는 stream으로부터 요소들을 받아서 Collection에 저장하는 메서드다.

`collect()` 메서드의 파라미터로 `Collectors` 클래스의 여러 메서드를 이용할 수 있다.



**예시**

```java
List<String> namesInUppercase = list.stream.sorted()
    						.map(String::toUpperCase)
    						.collect(Collectors.toList());
```







#### - `match()`

stream 요소들에 대해 주어진 Predicate이 match되는지 체크한다.

matching operation들은 terminal 연산이고, `boolean`을 리턴한다.



**예시**

```java
// 요소들 중 하나라도 "A"로 시작하면 true 리턴
list.stream().anyMatch(s -> s.startsWith("A"));

// 모든 요소들이 "A"로 시작해야 true 리턴
list.stream().allMatch(s -> s.startsWith("A"));

// 한 개의 요소도 "A"로 시작하지 않아야 true 리턴
list.stream().noneMatch(s -> s.startsWith("A"));
```





#### - `count()`

`count()`는 stream에 들어있는 요소의 개수를 `long` 타입으로 리턴해주는 terminal operation이다.



**예시**

```java
// stream의 요소 개수 반환
list.stream().count();
```





#### - `reduce()` 

> reduce: 축소&middot;삭감 또는 **환원**
>
> 위 메서드의 뜻은 환원으로 쓰인다.



![](https://image.slidesharecdn.com/java8ws-141128011442-conversion-gate01/95/java-8-workshop-24-638.jpg?cb=1417137382)



`reduce()` 메서드는 stream의 요소들에 대해 파라미터로 넘어온 함수로 reduction(환원)을 수행하는 메서드다.

reduced value는 `Optional`로 감싸져서 반환된다.



**예시**

```java
Optional<String> reduced = list.stream().reduce((s1, s2) -> s1 + "#" + s2);

reduced.ifPresent(System.out::println);

// a#b#c#d#e
```





## 5. Stream Short-circuit Operations

Stream operation들은 `Predicate`을 만족하는 collection 내의 모든 요소들에 대하여 수행되지만,

종종 iteration 중 matching되는 요소를 만났을 때 operation을 중단하고 싶을 때도 있을 것이다.

마치 if-else 문에서의 break처럼 말이다.

stream에서는 이를 지원하기 위해 몇가지 메서드들을 지원하고 있다.





### (1) `anyMatch()`

`anyMatch()` 메서드는 넘겨받은 `Predicate`을 만족하는 요소를 한번이라도 만나면 `true`를 리턴하고 끝난다.



**예시**

아래의 예시에서, 'A'로 시작하는 String을 만난다면, stream은 그 즉시 연산을 중단하고 반환값을 반환한다.

```java
list.stream().anyMatch(s -> s.startsWith("A"));
```





### (2) `findFirst()`

`findFirst()` 메서드는 stream에서 첫번째 요소를 반환하고 더 이상은 연산을 진행하지 않는다.

```java
list.stream().filter(s -> s.startsWith("L"))
			 .findFirst()
			 .get();
```
