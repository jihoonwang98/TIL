# [Java] Functional Interface 정리



## 매개 변수 1개인 것들

### 1. `Supplier<T>`

supply가 '공급하다'니깐 아무것도 안받고 만들어내는 느낌임

따라서, 매개변수는 없고 T 타입의 반환값만 있음.

```java
@FunctionalInterface
public interface Supplier<T> {
    T get();
}
```



### 2. `Consumer<T>`

consume이 '소비하다'니까 매개변수로 받은 값을 소비하는 느낌임.

따라서, T 타입의 매개변수 하나에 반환값은 void임.

```java
@FunctionalInterface
public interface Consumer<T> {
    void accept(T t);

    default Consumer<T> andThen(Consumer<? super T> after) {
        Objects.requireNonNull(after);
        return (T t) -> { accept(t); after.accept(t); };
    }
}
```

- `andThen()` 메서드로 `Consumer<T>` 인터페이스의 `accept()`에 정의된 행동 후에 추가적인 행동을 하게 할 수 있다.





**예시**

```java
Consumer<String> stringConsumer = System.out::println;
Consumer<String> stringConsumer2 = stringConsumer.andThen(str -> System.out.println(str.length()));

stringConsumer2.accept("hello");

// stringConsumer -> 넘겨준 파라미터 프린트해라.
// stringConsumer2 -> 1. 넘겨준 파라미터 프린트하고,
//                    2. 넘겨준 파라미터 길이 프린트해라.
```





### 3. `Function<T, R>`

'함수'니깐 가장 일반적인 꼴을 생각하면 됨.

파라미터 하나 받아서 반환값 하나 리턴한다.

파라미터 타입을 T, 리턴 타입을 R로 지정한다.



```java
@FunctionalInterface
public interface Function<T, R> {

    R apply(T t);


    // Function은 T를 받아서 R를 리턴한다.
    // Function 인스턴스에 andThen메서드를 체이닝할 수 있다.
    // andThen 메서드는 R를 받아서 V를 리턴한다.
    // 따라서 andThen 메서드를 체이닝하면 T를 받아서 V를 리턴하는 함수를 만들 수 있다.
    // 함수 수행 과정: T를 받아서 R를 리턴 -> 나온 R값을 받아서 V를 리턴
    default <V> Function<T, V> andThen(Function<? super R, ? extends V> after) {
        Objects.requireNonNull(after);
        return (T t) -> after.apply(apply(t));
    }
    
    
    // andThen이랑 비슷한데, 순서가 반대임
    // compose는 V를 받아서 T를 리턴하는 함수를 파라미터로 받는다.
    default <V> Function<V, R> compose(Function<? super V, ? extends T> before) {
        Objects.requireNonNull(before);
        return (V v) -> apply(before.apply(v));
    }

    // input 값을 그대로 돌려주는 Function을 반환한다.
    static <T> Function<T, T> identity() {
        return t -> t;
    }
}
```





**예시**



**identity()**

```java
Function<String, String> identity = Function.<String>identity();
String output = identity.apply("input");
System.out.println(output);  // input
```



**andThen()**

```java
Function<String, Integer> strLength = String::length;
Function<Integer, Integer> doubleVal = i -> i*2;

Function<String, Integer> doubleLength = strLength.andThen(doubleVal);
System.out.println(doubleLength.apply("hello"));  // 10
```

- `strLength`는 String의 Length를 돌려주는 함수
- `doubleVal`는 int 값을 두배로 돌려주는 함수
- `doubleLength`는 `strLength`와 `doubleVal`을 합친 함수.
  - String을 받아서 `strLength`를 수행하고 난 후 반환된 값을 다시 `doubleVal`에 넣어서 반환된 값을 돌려주는 함수다.



**compose()**

위의 `doubleLength`와 똑같은 함수를 `andThen()`이 아닌 `compose()`로 만들 수 있다.

```java
Function<String, Integer> strLength = String::length;
Function<Integer, Integer> doubleVal = i -> i*2;

Function<String, Integer> doubleLength = doubleVal.compose(strLength);
System.out.println(doubleLength.apply("hello"));
```





**비교**

```java
strLength.andThen(doubleVal);  // strLength 함수 실행 후 doubleVal 함수 실행
doubleVal.compose(strLength);  // dobuleVal 함수 실행 하기 전에 strLength 함수 실행
```

같은 말이긴 한데 직관적으로 `andThen()`이 더 잘 와닿는다.









### 4. `Predicate<T>`

조건식을 표현할 때 사용한다.

T 타입의 파라미터를 받아서 boolean 타입으로 반환한다.

```java
@FunctionalInterface
public interface Predicate<T> {

    boolean test(T t);

    default Predicate<T> and(Predicate<? super T> other) {
        Objects.requireNonNull(other);
        return (t) -> test(t) && other.test(t);
    }
    
    default Predicate<T> negate() {
        return (t) -> !test(t);
    }
   
    default Predicate<T> or(Predicate<? super T> other) {
        Objects.requireNonNull(other);
        return (t) -> test(t) || other.test(t);
    }

    // 주어진 targetRef와 동등한지 검사하는 Predicate을 리턴한다.
    static <T> Predicate<T> isEqual(Object targetRef) {
        return (null == targetRef)
                ? Objects::isNull
                : object -> targetRef.equals(object);
    }
}
```





## 매개 변수가 2개인 것들

### 1. `BiConsumer<T, U>`

일단, `Bi`가 붙으니까 매개 변수가 2개임.

근데 `Consumer`니까 매개 변수만 있고 반환값은 void임.

따라서 `BiConsumer<T, U>`는 T 타입과 U 타입의 파라미터 두 개를 받아서 아무것도 반환 안함.

```java
@FunctionalInterface
public interface BiConsumer<T, U> {

    void accept(T t, U u);

    
    // 그냥 Consumer의 andThen() 메서드와 비슷하다.
    default BiConsumer<T, U> andThen(BiConsumer<? super T, ? super U> after) {
        Objects.requireNonNull(after);

        return (l, r) -> {
            accept(l, r);
            after.accept(l, r);
        };
    }
}
```





### 2. `BiFunction<T, U, R>`

`Bi`니까 매개변수가 두 개임.

그런데 `Function`이니까 매개변수를 두 개 받고, 반환 값이 한 개 있어야 함.

`BiFunction<T, U, R>`는 T 타입과, U 타입의 파라미터 두 개를 받아서 R 타입의 반환 값을 리턴한다.

```java
@FunctionalInterface
public interface BiFunction<T, U, R> {

    R apply(T t, U u);

    
    // BiFunction 수행 후 after라는 이름의 Function을 수행 하는 새로운 BiFunction을 만든다.
    // 1. BiFunction -> T,U 받아서 R 리턴
    // 2. Function(after) -> R 받아서 V 리턴
    // 따라서 이때 만들어지는 BiFunction은 T,U 받아서 V 리턴하는 함수
    default <V> BiFunction<T, U, V> andThen(Function<? super R, ? extends V> after) {
        Objects.requireNonNull(after);
        return (T t, U u) -> after.apply(apply(t, u));
    }
}
```





### 3. `BiPredicate<T, U>`

`Bi`니까 파라미터가 두 개.

`Predicate`니까 파라미터를 받아서 조건식을 평가해서 boolean 리턴.

따라서, `BiPredicate`는 T 타입과 U 타입 파라미터 두 개를 받아서 boolean 리턴.

```java
@FunctionalInterface
public interface BiPredicate<T, U> {
    
    boolean test(T t, U u);

    default BiPredicate<T, U> and(BiPredicate<? super T, ? super U> other) {
        Objects.requireNonNull(other);
        return (T t, U u) -> test(t, u) && other.test(t, u);
    }
    
    default BiPredicate<T, U> negate() {
        return (T t, U u) -> !test(t, u);
    }
    
    default BiPredicate<T, U> or(BiPredicate<? super T, ? super U> other) {
        Objects.requireNonNull(other);
        return (T t, U u) -> test(t, u) || other.test(t, u);
    }
}
```







## 그 외

### 1. `Runnable`

thread를 생성할 때 주로 사용한다.

파라미터도 없고 반환 값도 void다.

```java
@FunctionalInterface
public interface Runnable {
    public abstract void run();
}
```



### 2. `Comparator<T>`

T 타입 파라미터 두 개를 서로 비교하는 인터페이스

주의할 점은 반환 타입이 `int`라는 것.

```java
@FunctionalInterface
public interface Comparator<T> {
    
    int compare(T o1, T o2);
    
    // 참고로 bool equals(Object obj) 메소드는 
    // default나 static이 안붙어있음에도 구현이 강제되지 않는 이유는 
    // 모든 객체의 최상위 타입(객체)인 Object 클래스에서 정의되어있기 때문이다.
    boolean equals(Object obj);

    default Comparator<T> reversed() {
        return Collections.reverseOrder(this);
    }
    
    default Comparator<T> thenComparing(Comparator<? super T> other) {
        Objects.requireNonNull(other);
        return (Comparator<T> & Serializable) (c1, c2) -> {
            int res = compare(c1, c2);
            return (res != 0) ? res : other.compare(c1, c2);
        };
    }
    
    default <U> Comparator<T> thenComparing(
            Function<? super T, ? extends U> keyExtractor,
            Comparator<? super U> keyComparator)
    {
        return thenComparing(comparing(keyExtractor, keyComparator));
    }

    
    default <U extends Comparable<? super U>> Comparator<T> thenComparing(
            Function<? super T, ? extends U> keyExtractor)
    {
        return thenComparing(comparing(keyExtractor));
    }

    
    default Comparator<T> thenComparingInt(ToIntFunction<? super T> keyExtractor) {
        return thenComparing(comparingInt(keyExtractor));
    }
    
    default Comparator<T> thenComparingLong(ToLongFunction<? super T> keyExtractor) {
        return thenComparing(comparingLong(keyExtractor));
    }

    
    default Comparator<T> thenComparingDouble(ToDoubleFunction<? super T> keyExtractor) {
        return thenComparing(comparingDouble(keyExtractor));
    }

    
    public static <T extends Comparable<? super T>> Comparator<T> reverseOrder() {
        return Collections.reverseOrder();
    }
    
    @SuppressWarnings("unchecked")
    public static <T extends Comparable<? super T>> Comparator<T> naturalOrder() {
        return (Comparator<T>) Comparators.NaturalOrderComparator.INSTANCE;
    }
    
    public static <T> Comparator<T> nullsFirst(Comparator<? super T> comparator) {
        return new Comparators.NullComparator<>(true, comparator);
    }
    
    public static <T> Comparator<T> nullsLast(Comparator<? super T> comparator) {
        return new Comparators.NullComparator<>(false, comparator);
    }

    
    public static <T, U> Comparator<T> comparing(
            Function<? super T, ? extends U> keyExtractor,
            Comparator<? super U> keyComparator)
    {
        Objects.requireNonNull(keyExtractor);
        Objects.requireNonNull(keyComparator);
        return (Comparator<T> & Serializable)
            (c1, c2) -> keyComparator.compare(keyExtractor.apply(c1),
                                              keyExtractor.apply(c2));
    }
    
    public static <T, U extends Comparable<? super U>> Comparator<T> comparing(
            Function<? super T, ? extends U> keyExtractor)
    {
        Objects.requireNonNull(keyExtractor);
        return (Comparator<T> & Serializable)
            (c1, c2) -> keyExtractor.apply(c1).compareTo(keyExtractor.apply(c2));
    }
    
    public static <T> Comparator<T> comparingInt(ToIntFunction<? super T> keyExtractor) {
        Objects.requireNonNull(keyExtractor);
        return (Comparator<T> & Serializable)
            (c1, c2) -> Integer.compare(keyExtractor.applyAsInt(c1), keyExtractor.applyAsInt(c2));
    }

    public static <T> Comparator<T> comparingLong(ToLongFunction<? super T> keyExtractor) {
        Objects.requireNonNull(keyExtractor);
        return (Comparator<T> & Serializable)
            (c1, c2) -> Long.compare(keyExtractor.applyAsLong(c1), keyExtractor.applyAsLong(c2));
    }

    public static<T> Comparator<T> comparingDouble(ToDoubleFunction<? super T> keyExtractor) {
        Objects.requireNonNull(keyExtractor);
        return (Comparator<T> & Serializable)
            (c1, c2) -> Double.compare(keyExtractor.applyAsDouble(c1), keyExtractor.applyAsDouble(c2));
    }
}
```





### 3. `UnaryOperator<T>`

UnaryOperator란, 단항 연산자를 뜻한다.

즉, 피연산자가 한 개라는 뜻이다.

근데, 만약에 T 타입 파라미터를 받아서 다른 타입인 R 타입 반환 값을 리턴하면, `Function`이랑 컨셉이 겹친다.

그래서 이 `UnaryOperator<T>`는 T 타입 파라미터를 받아서 똑같은 타입인 T 타입 반환 값을 리턴한다.

```java
@FunctionalInterface
public interface UnaryOperator<T> extends Function<T, T> {

    static <T> UnaryOperator<T> identity() {
        return t -> t;
    }
}
```





### 4. `BinaryOperator<T>`

BinaryOperator란, 이항 연산자를 뜻한다.

얘도 마찬가지로 피연산자가 두 개인데,

T 타입, U 타입을 받아서 R 타입을 리턴하면 `BiFunction`이랑 겹친다.

얘는 T 타입 파라미터 두 개를 받아서 T 타입의 반환 값을 리턴한다.

```java
@FunctionalInterface
public interface BinaryOperator<T> extends BiFunction<T,T,T> {
    
    // 둘 중에 작은놈 리턴하는 함수
    public static <T> BinaryOperator<T> minBy(Comparator<? super T> comparator) {
        Objects.requireNonNull(comparator);
        return (a, b) -> comparator.compare(a, b) <= 0 ? a : b;
    }

    
    // 둘 중에 큰놈 리턴하는 함수
    public static <T> BinaryOperator<T> maxBy(Comparator<? super T> comparator) {
        Objects.requireNonNull(comparator);
        return (a, b) -> comparator.compare(a, b) >= 0 ? a : b;
    }
}
```



>  **※ Operator가 붙은 인터페이스들은 파라미터 타입과 리턴 타입이 똑같다.**