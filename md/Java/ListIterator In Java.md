# ListIterator In Java



> 참고: https://www.scientecheasy.com/2020/09/listiterator-in-java.html/



![](https://www.scientecheasy.com/wp-content/uploads/2020/09/java-listiterator.png)



**ListIterator란?**

- Iterator의 한계(forward로만 순회 가능)를 깨기 위해 만들어짐.
- 단, List에서만 사용가능 (Set은 순서가 없어서 앞,뒤가 없음)

- ListIterator를 통해 

  조회, 삭제, 변경, 추가 연산을 할 수 있다.





## ListIterator 만들기

List 인터페이스의 메서드인 `listIterator()` 메서드를 호출함으로써 

ListIterator 객체를 얻을 수 있다.



**문법)**

```java
public ListIterator<Type> listIterator();

ex)
ArrayList<String> list = new ArrayList<>();

// list에 대한 listIterator 생성(full list iterator)
ListIterator<String> iter = list.listIterator();  

// index 3(앞에서 셈)에서 시작하는 listIterator 생성
ListIterator<String> iter = list.listIterator(3);
```



- ListIterator의 `next()` 메서드의 반환 타입은 ListIterator의 type parameter와 일치한다.
  - 위의 예에서는 `next()` 메서드의 반환 타입은 String이 된다.







## ListIterator의 메서드

```java
public interface ListIterator<E> extends Iterator<E> 
```



위와 같이 ListIterator 인터페이스는 양방향 순회 기능을 추가하기 위해

Iterator 인터페이스를 상속받고 거기다가 기능을 더 추가했다.

예를 들면, hasPrevious(), previous() 같은 메서드들이 추가됐다.

또한, Iterator 인터페이스를 상속받았기 때문에 Iterator의 모든 기능을 ListIterator에서도 사용할 수 있다.







ListIterator의 메서드들은 아래와 같다.



### Forward direction (앞방향 순회 기능 메서드)

```java
public interface ListIterator<E> extends Iterator<E> {
    
    // 다음 element가 존재하면(앞 방향에 element가 존재하면) true 리턴
	boolean hasNext();
    
    // 다음 element를 return하면서 커서를 옮긴다.
	E next();
    
    // 다음 element의 index를 반환한다.
	int nextIndex();
	...
}
```





### Backward direction (뒷방향 순회 기능 메서드)

```java
public interface ListIterator<E> extends Iterator<E> {
    
    // 이전 element가 존재하면(이전 방향에 element가 존재하면) true 리턴
	boolean hasPrevious();
    
    // 이전 element를 return하면서 커서를 옮긴다.
	E previous();
    
    // 이전 element의 index를 반환한다.
	int nextIndex();
	...
}
```



방향만 뒷 방향이지 기능은 똑같다...





### Other capability methods

```java
public interface ListIterator<E> extends Iterator<E> {
    
    // next()나 previous()로 return된 마지막 element를 제거한다.
	void remove();
    
    // next()나 previous()로 return된 마지막 element를 변경한다.
	void set(E e);
    
    // 현재 cursor가 가리키는 위치에 element를 넣고,
    // cursor를 순방향(오른쪽)으로 한칸 이동 시킨다.
	void add(E e);
	...
}
```









### 예제로 살펴보기



#### 순방향 순회

```java
List<String> list = 
    new ArrayList<>(Arrays.asList("zero", "first", "second", "third", "fourth"));

ListIterator<String> iter = list.listIterator();
while (iter.hasNext()) {
    System.out.print(iter.next() + " ");
}

// zero first second third fourth
```



#### 역방향 순회

```java
List<String> list = 
    new ArrayList<>(Arrays.asList("zero", "first", "second", "third", "fourth"));

ListIterator<String> iter = list.listIterator(list.size());
while (iter.hasPrevious()) {
    System.out.print(iter.previous() + " ");
}

// fourth third second first zero
```



#### 인덱스와 함께 출력 (순방향)

```java
List<String> list = 
    new ArrayList<>(Arrays.asList("zero", "first", "second", "third", "fourth"));

ListIterator<String> iter = list.listIterator();
while (iter.hasNext()) {
    int idx = iter.nextIndex();
    System.out.println("idx = " + idx + ", " + iter.next());
}

idx = 0, zero
idx = 1, first
idx = 2, second
idx = 3, third
idx = 4, fourth
```





#### Add, Set

그냥 iterator랑 똑같다.

```java
List<String> list = new ArrayList<>();
list.add("A");
list.add("B");
list.add("C");
list.add("D");
System.out.println("List: "+list);  // List: [A, B, C, D]

ListIterator<String> iter = list.listIterator(list.size());
iter.add("E");
System.out.println("List: " + list);   // List: [A, B, C, D, E]
```





#### Remove

`remove()` 메서드는 `next()`나 `previous()` 메서드가 호출된 후 오직 한 번만 수행될 수 있다!!!

위의 메서드들을 호출 안하거나 두번 호출 뒤 `remove()` 하면 **IllegalStateException**이 터진다.



```java
List<String> list = 
    new ArrayList<>(Arrays.asList("A", "B", "C", "D"));

ListIterator<String> iter = list.listIterator(1);
// iter.remove();  -> IllegalStateException
iter.next();
iter.remove();
System.out.println("List: "+list);  // List: [A, C, D]
// iter.remove();  -> IllegalStateException
```