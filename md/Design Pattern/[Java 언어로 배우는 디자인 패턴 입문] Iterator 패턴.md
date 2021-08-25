# [Java 언어로 배우는 디자인 패턴 입문] Iterator 패턴



## 1. 도입

**자바의 배열 반복문 예시**

```java
for(int i = 0; i < arr.length; i++) {
	System.out.println(arr[i]);
}
```

- 위 예시의 변수 `i`의 기능을 추상화해서 일반화한 것을 디자인 패턴에서는 **Iterator 패턴**이라고 한다.
- 아니 그럼 그냥 배열 던져주고 알아서 반복문 쓰라 하면 되지 뭣하러 귀찮게 패턴까지 만들어서 쓰냐??라고 하면
  - 클라이언트에게 컬렉션 구현 방법을 노출시키지 않을 수 있다는 장점이 있다.
  - 클라이언트에게 구현 방법을 알려주지 않으면 구현 방법을 자유롭게 바꿀 수 있다.





**Iterator 패턴**이란?

- 무엇인가 많이 모여있는 것들을 순서대로 지정하면서 전체를 검색하는 처리를 실행하기 위한 것이다.
- 컬렉션 구현 방법을 노출시키지 않으면서도 그 집합체 안에 들어있는 모든 항목에 접근할 수 있는 추상화된 방법을 제공하는 패턴







## 2. `Iterator` 패턴을 활용한 예제 프로그램

서가(BookShelf) 안에 책(Book)을 넣고, 그 책의 이름을 차례대로 표시하는 프로그램을 `Iterator` 패턴을 활용해 만들어 보자.





### `Aggregate` 인터페이스

**aggregate란?**

- 한국말로 '모으다', '모이다', '집합'이라는 뜻
- 요소들이 나열되어 있는 집합체를 의미한다.
- 이 인터페이스를 구현하고 있는 클래스는 배열이나 리스트같은 컬렉션을 들고 있다.





### 예제 프로그램 클래스 다이어그램



![iter1](https://lh6.googleusercontent.com/ITTXxO3gii-L4Vyh-O12sfg4XUBFoux9bp2xaw1Hd3Hi3jc7R16Vc5f8iHuc9xB-pkr-u61qmvZ2TKqsamKYC_HDHVYrUU8nkzakm25KMFYlh5Q14Vnw3trO744U5i6Ru3cJye7i)





**클래스와 인터페이스**

| 이름              | 설명                                         |
| ----------------- | -------------------------------------------- |
| Aggregate         | 집합체를 나타내는 인터페이스                 |
| Iterator          | 하나씩 나열하면서 검색을 실행하는 인터페이스 |
| Book              | 책을 나타내는 클래스                         |
| BookShelf         | 서가를 나타내는 클래스                       |
| BookShelfIterator | 서가를 검색하는 클래스                       |





### 코드

**`Aggregate` 인터페이스**

```java
public interface Aggregate<T> {
    Iterator<T> iterator();
}
```



**`Iterator` 인터페이스**

```java
public interface Iterator<T> {
    boolean hasNext();
    T next();
}
```



**`Book` 클래스**

```java
@Getter
@AllArgsConstructor
@ToString
public class Book {
    private String name;
}
```



**`BookShelf` 클래스**

```java
@AllArgsConstructor
public class BookShelf implements Aggregate<Book> {

    private List<Book> books;
    
    @Override
    public Iterator<Book> iterator() {
        return new BookShelfIterator(this);
    }

    public int getLength() {
        return books.size();
    }

    public Book getBookAt(int index) {
        return books.get(index);
    }
}
```



**`BookShelfIterator` 클래스**

```java
public class BookShelfIterator implements Iterator<Book> {

    private final BookShelf bookShelf;
    private int length;
    private int index;

    public BookShelfIterator(BookShelf bookShelf) {
        this.bookShelf = bookShelf;
        this.index = 0;
        this.length = bookShelf.getLength();
    }

    @Override
    public boolean hasNext() {
        return index < length;
    }

    @Override
    public Book next() {
        return bookShelf.getBookAt(index++);
    }
}
```







## 3. `Iterator` 패턴의 등장 인물



![iter2](https://lh5.googleusercontent.com/2b9N-rdNHcbuL3SejbQuC6EtHvW7H0lxXcxIoo8OOM_JVRNp_dWrVKk1rQqiqvlWLQ9bw0ETfLB1OHUMFdCWzERN2oI0KzsFAnlxp04OB7LskOvfiwvXpSR0LutU-1_mu9mGfsOI)

### Aggregate(집합체)의 역할

- Iterator를 만들어내는 인터페이스(API)를 정의한다.
  - `iterator()` 메서드는 Aggregate 자신이 갖고 있는 요소를 순서대로 검색해 주는 iterator를 만들어내는 메서드다.



### ConcreteAggregate의 역할

- Aggregate 역할이 결정한 인터페이스(API)를 실제로 구현하는 일을 한다.
- ConcreteIterator 인스턴스를 만들어내는 일을 한다.



### Iterator(반복자)의 역할

- 요소를 순서대로 검색해가는 인터페이스(API)를 정의한다.



### ConcreteIterator의 역할

- Iterator가 결정한 인터페이스(API)를 실제로 구현한다.
- 이 역할은 <u>검색하기 위해 필요한 정보를 가지고 있어야 한다</u>.





## 4. `Aggregate`와 `Iterator`의 대응

`BookShelf`  클래스에 대응하는 `ConcreteIterator` 역할로서 `BookShelfIterator` 클래스를 정의했던 예시를 다시 떠올려보자.

<u>`BookShelfIterator`는 `BookShelf`가 어떻게 구현되고 있는지 알고 있기 때문에, '다음 책'을 얻기 위해 `getBookAt()` 메서드를 호출할 수 있었다.</u>



이것은 만약 `BookShelf`의 구현을 전부 변경하고, `getBookAt()` 메서드라는 인터페이스(API)도 변경했을 때에는 `BookShelfIterator`의 수정이 필요하게 된다.



`Aggregate`와 `Iterator`라는 두 개의 인터페이스가 쌍을 이루듯이, `BookShelf`와 `BookShelfIterator`라는 두 개의 클래스도 쌍을 이루고 있다.
