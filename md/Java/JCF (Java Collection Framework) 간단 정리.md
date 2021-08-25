# JCF (Java Collection Framework) 간단 정리 





### JCF 계층 구조

- `java.util` 패키지에 아래와 같은 클래스와 인터페이스들이 포함되어 있다.



![](https://static.javatpoint.com/images/java-collection-hierarchy.png)





![](https://www.ictdemy.com/images/1/java/collections/java_collection_api_diagram.svg)



![](https://helpezee.files.wordpress.com/2019/08/capture1.jpg?w=1024&h=627)





### Collection 인터페이스

| No.  | Method                                                | Description                                                  |
| :--- | :---------------------------------------------------- | :----------------------------------------------------------- |
| 1    | public boolean add(E e)                               | 컬렉션에 추가하는 메서드                                     |
| 2    | public boolean addAll(Collection<? extends E> c)      | 파라미터에 넘겨준 컬렉션 안의 모든 elements들을 추가하는 메서드 |
| 3    | public boolean remove(Object element)                 | 파라미터에 넘겨준 Object를 컬렉션 내에서 제거하는 메서드     |
| 4    | public boolean removeAll(Collection<?> c)             | 파라미터에 넘겨준 컬렉션 안의 모든 elements들을 제거하는 메서드 |
| 5    | default boolean removeIf(Predicate<? super E> filter) | 파라미터로 넘겨준 predicate를 만족하는 elements들을 모두 제거하는 메서드 |
| 6    | public boolean retainAll(Collection<?> c)             | 파라미터로 넘겨준 컬렉션 안의 elements들만 남기고 나머지는 전부 제거하는 메서드 |
| 7    | public int size()                                     | 컬렉션 안의 elements 개수를 리턴하는 메서드                  |
| 8    | public void clear()                                   | 컬렉션 안의 elements 들을 모두 제거하는 메서드               |
| 9    | public boolean contains(Object element)               | 파라미터로 넘겨준 element가 컬렉션 안에 존재하면 true 반환, 아니면 false 반환하는 메서드 |
| 10   | public boolean containsAll(Collection<?> c)           | 파라미터로 넘겨준 Collection 내의 모든 elements 들을 포함하면 true, 아니면 false 반환하는 메서드 |
| 11   | public Iterator iterator()                            | iterator를 반환하는 메서드                                   |
| 12   | public Object[] toArray()                             | 해당 컬렉션을 배열 형태로 반환하는 메서드                    |
| 13   | public \<T> T[] toArray(T[] a)                        | 해당 컬렉션을 배열 형태로 반환하는 메서드(runtime에 지정한 타입으로 반환됨) |
| 14   | public boolean isEmpty()                              | 컬렉션이 비어있으면 true, 아니면 false 반환                  |
| 15   | default Stream\<E> parallelStream()                   | ParrallelStream 반환                                         |
| 16   | default Stream\<E> stream()                           | Stream 반환                                                  |
| 17   | default Spliterator\<E> spliterator()                 | Spliterator 반환                                             |
| 18   | public boolean equals(Object element)                 | Collection 끼리 비교                                         |
| 19   | public int hashCode()                                 | Hash Code 반환                                               |





### Iterator 인터페이스

`Iterator` 인터페이스는 오직 순방향으로만 iterating 할 수 있는 기능을 제공한다.



| No.  | Method                   | Description                                            |
| :--- | :----------------------- | :----------------------------------------------------- |
| 1    | public boolean hasNext() | 다음 element가 존재하면 true, 아니면 false 반환        |
| 2    | public Object next()     | 다음 element를 반환하면서 cursor를 다음으로 옮긴다.    |
| 3    | public void remove()     | iterator에 의해 반환된 가장 마지막 element를 삭제한다. |





### Iterable 인터페이스

- 모든 collection 클래스들의 root 인터페이스



이 인터페이스는 아래와 같은 오직 하나의 **abstract method**를 포함한다.

```java
Iterator<T> iterator()
```





### Collection 인터페이스

- Collection framework 안의 모든 클래스들이 구현하고 있는 인터페이스

- 모든 collection들이 가져야 하는 공통 메서드들을 선언하고 있음

  예를들면, `Boolean add(Object obj)`,`Boolean addAll(Collection c)`,`void clear()` 등등





### List Interface

- Object들을 **순서를 매겨서** 저장
- **중복 값** 저장 가능
- List 인터페이스의 구현체로 `ArrayList`, `LinkedList`, `Vector`, `Stack`이 있음





### ArrayList

- 동적 배열 사용

- 삽입된 순서 유지
- non-synchronized
- random access 가능 (O(1))





### LinkedList

- 이중 연결 리스트 사용
- 삽입된 순서 유지
- non-synchronized





### Vector

- 동적 배열 사용
- 삽입된 순서 유지
- **synchronized**
- Collection framework가 가지고 있지 않은 많은 메서드들을 포함하고 있음



### Stack

- Vector의 subclass
- LIFO 구조





### Queue 인터페이스

- FIFO 구조
- Queue 인터페이스를 implement 하고 있는 것 들은 `PriorityQueue`, `Deque`, `ArrayDeque` 등이 있다.





### PriorityQueue

- null 값 허용 X





### Deque 인터페이스

- **d**ouble **e**nded **que**ue라서 양쪽에서 넣고 뺄 수 있다.
- Queue 인터페이스를 implement 하고 있다.





### ArrayDeque

- Deque 인터페이스의 구현체
- ArrayDeque는 capacity 제한이 없기 때문에 ArrayList와 Stack보다 빠르다.





### Set 인터페이스

- 순서 안 따짐 (unordered)
- 중복 값 저장 X
- 최대 한 개의 null 값만 저장 가능
- 구현체로는 `HashSet`, `LinkedHashSet`, `TreeSet` 등이 있다.





### HashSet

- Hash 테이블을 이용해서 저장함



### LinkedHashSet

- LinkedList로 구현한 Set
- HashSet을 상속 받음 (Set 인터페이스도 implement 함)
- 삽입 순서 유지
- null 값 여러 개 허용



### SortedSet 인터페이스

- Set은 Set인데 element 들이 오름차순으로 정렬되어 저장됨



### TreeSet 

- Tree 구조를 이용해 저장함