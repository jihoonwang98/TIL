# ArrayList 정리



### ArrayList

- size 제한이 없는 동적 배열
- 중복 값 저장 O
- non-synchronized
- 삽입 순서 유지





### ArrayList 상속 구조



![](https://static.javatpoint.com/images/arraylist.png)





### ArrayList class declaration

```java
public class ArrayList<E> extends AbstractList<E> implements List<E>, RandomAccess, Cloneable, Serializable
```





### ArrayList 생성자

| Constructor                          | Description                                                  |
| :----------------------------------- | :----------------------------------------------------------- |
| ArrayList()                          | It is used to build an empty array list.                     |
| ArrayList(Collection<? extends E> c) | It is used to build an array list that is initialized with the elements of the collection c. |
| ArrayList(int capacity)              | It is used to build an array list that has the specified initial capacity. |





### ArrayList 메서드

| Method                                                       | Description                                                  |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| void [add](https://www.javatpoint.com/java-arraylist-add-method)(int index, E element) | It is used to insert the specified element at the specified position in a list. |
| boolean [add](https://www.javatpoint.com/java-arraylist-add-method)(E e) | It is used to append the specified element at the end of a list. |
| boolean [addAll](https://www.javatpoint.com/java-arraylist-addall-method)(Collection<? extends E> c) | It is used to append all of the elements in the specified collection to the end of this list, in the order that they are returned by the specified collection's iterator. |
| boolean [addAll](https://www.javatpoint.com/java-arraylist-addall-method)(int index, Collection<? extends E> c) | It is used to append all the elements in the specified collection, starting at the specified position of the list. |
| void [clear](https://www.javatpoint.com/java-arraylist-clear-method)() | It is used to remove all of the elements from this list.     |
| void ensureCapacity(int requiredCapacity)                    | It is used to enhance the capacity of an ArrayList instance. |
| E get(int index)                                             | It is used to fetch the element from the particular position of the list. |
| boolean isEmpty()                                            | It returns true if the list is empty, otherwise false.       |
| [Iterator()](https://www.javatpoint.com/java-arraylist-iterator-method) |                                                              |
| [listIterator()](https://www.javatpoint.com/java-arraylist-listiterator-method) |                                                              |
| int lastIndexOf(Object o)                                    | It is used to return the index in this list of the last occurrence of the specified element, or -1 if the list does not contain this element. |
| Object[] toArray()                                           | It is used to return an array containing all of the elements in this list in the correct order. |
| <T> T[] toArray(T[] a)                                       | It is used to return an array containing all of the elements in this list in the correct order. |
| Object clone()                                               | It is used to return a shallow copy of an ArrayList.         |
| boolean contains(Object o)                                   | It returns true if the list contains the specified element   |
| int indexOf(Object o)                                        | It is used to return the index in this list of the first occurrence of the specified element, or -1 if the List does not contain this element. |
| E remove(int index)                                          | It is used to remove the element present at the specified position in the list. |
| boolean [remove](https://www.javatpoint.com/java-arraylist-remove-method)(Object o) | It is used to remove the first occurrence of the specified element. |
| boolean [removeAll](https://www.javatpoint.com/java-arraylist-removeall-method)(Collection<?> c) | It is used to remove all the elements from the list.         |
| boolean removeIf(Predicate<? super E> filter)                | It is used to remove all the elements from the list that satisfies the given predicate. |
| protected void [removeRange](https://www.javatpoint.com/java-arraylist-removerange-method)(int fromIndex, int toIndex) | It is used to remove all the elements lies within the given range. |
| void replaceAll(UnaryOperator<E> operator)                   | It is used to replace all the elements from the list with the specified element. |
| void [retainAll](https://www.javatpoint.com/java-arraylist-retainall-method)(Collection<?> c) | It is used to retain all the elements in the list that are present in the specified collection. |
| E set(int index, E element)                                  | It is used to replace the specified element in the list, present at the specified position. |
| void sort(Comparator<? super E> c)                           | It is used to sort the elements of the list on the basis of specified comparator. |
| Spliterator<E> spliterator()                                 | It is used to create spliterator over the elements in a list. |
| List<E> subList(int fromIndex, int toIndex)                  | It is used to fetch all the elements lies within the given range. |
| int size()                                                   | It is used to return the number of elements present in the list. |
| void trimToSize()                                            | It is used to trim the capacity of this ArrayList instance to be the list's current size. |



size, isEmpty, get, set, iterator, listIterator -> O(1)

add -> amortized O(1) 

다른 모든 operations -> O(n)



size <= capacity



not synchronized





**ArrayList**

- List 인터페이스의 Resizable-array 구현체

- Operation 시간 복잡도 

  - size, isEmpty, get, set, iterator, listIterator -> O(1)
  - add -> amortized O(1)
  - 다른 모든 operations -> O(n)
  - constant factor가 LinkedList보다 낮다.

- size와 capacity의 차이

  - size: 리스트가 현재 가지고 있는 **원소 개수**
  - capacity: 리스트에서 원소들을 저장하기 위해 쓰고 있는 **array의 size**

  - size <= capacity
  - capacity는 애플리케이션에서 현재 값을 확인 불가
    - ensureCapacity() 메서드를 통해 최소 capacity 지정 가능 (capacity 늘리는 거임)
    - 이 메서드를 통해 reallocation을 어느 정도 줄일 수 있다.

- Non-synchronized 

  - 만약 다수의 thread들이 동시에 ArrayList 인스턴스에 접근해서

    해당 인스턴스에 structural modification을 하고 싶으면,

    반드시 ArrayList를 사용하기 전에 synchronize를 시켜야 한다.

    > **structural modification**이란, 
    >
    > array의 structure에 변형을 가하는 operation을 말한다. 
    >
    > 예를 들면, add나 delete 또는 array resize. 단순히 element의 값을 setting 하는 것은 structural modification이 아니다.)

  - Collections.synchronizedList 메서드를 이용하면 synchronized된 ArrayList를 반환받을 수 있다.

    - `List list = Collections.synchronizedList(new ArrayList(...));`

- iterator와 listIterator 사용 시 주의점

  - iterator 자신의 remove나 add 메서드를 사용하지 않고 

    list가 structurally modified 되면, ConcurrentModificationException이 발생한다.





**ArrayList의 필드와 메서드**



**[필드]**

- **transient Object[] elementData**

```java
/* 
The array buffer into which the elements of the ArrayList are stored. 
The capacity of the ArrayList is the length of this array buffer. 
Any empty ArrayList with elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA will be expanded to DEFAULT_CAPACITY when the first element is added. 

ArrayList의 element들이 실제 저장되는 array.
ArrayList의 capacity는 이 array의 length를 말하는 것이다.

elementData가 DEFAULTCAPACITY_EMPTY_ELEMENTDATA와 똑같은 비어 있는 ArrayList는
첫 번째 element가 추가되면 DEFAULT_CAPACITY로 늘어날 것이다.

참고로 private static final int DEFAULT_CAPACITY = 10;로
정의되어 있음
*/

transient Object[] elementData; // non-private to simplify nested class access
```



ArrayList에 저장한 원소들이 실제 내부적으로 저장되는 배열이라고 보면 된다.









- **private static final int DEFAULT_CAPACITY**

```java
/*
Default initial capacity.
*/

private static final int DEFAULT_CAPACITY = 10;
```





```java
/**
 * Shared empty array instance used for default sized empty instances. We
 * distinguish this from EMPTY_ELEMENTDATA to know how much to inflate when
 * first element is added.
 */

/*
 default sized empty instance들에 사용되는 공유 empty array instance.
 우리는 첫번째 element가 추가되었을 때 얼마나 크기를 늘릴지 구분하기 위해
 DEFAULTCAPACITY_EMPTY_ELEMENTDATA와 EMPTY_ELEMENTDATA를 구별해야 한다.
*/

private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
```





**[메서드]**

- **NoArgsConstructor**

```java
/**
* Constructs an empty list with an initial capacity of ten.
*/

/* initial capacity가 10인 빈 list를 만든다. */

public ArrayList() {
	this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
 // this.elementData = {}
}
```



- 파라미터로 **int initialCapacity**를 받는 **Constructor**

```java
/**
 * Constructs an empty list with the specified initial capacity.
 * 
 * @param  initialCapacity  the initial capacity of the list
 * @throws IllegalArgumentException if the specified initial capacity
 *         is negative
*/

/* initial capacity가 지정된 비어 있는 list를 만든다.*/


public ArrayList(int initialCapacity) {
    if (initialCapacity > 0) {
        this.elementData = new Object[initialCapacity];
    } else if (initialCapacity == 0) {
        this.elementData = EMPTY_ELEMENTDATA;
    } else {
        throw new IllegalArgumentException("Illegal Capacity: "+
                                           initialCapacity);
    }
}
```





- **private Object[] grow()**
- **private Object[] grow(int minCapacity)**

```java
/* 그냥 대강 설명하자면
   처음에 ArrayList의 size(elementData 길이)가 0이면 10 반환하고
   아니면 ArrayList의 size * 1.5 된 값이 반환된다.
*/

private int newCapacity(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    if (newCapacity - minCapacity <= 0) {
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA)
            return Math.max(DEFAULT_CAPACITY, minCapacity);
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        return minCapacity;
    }
    return (newCapacity - MAX_ARRAY_SIZE <= 0)
        ? newCapacity
        : hugeCapacity(minCapacity);
}


/**
 * Increases the capacity to ensure that it can hold at least the
 * number of elements specified by the minimum capacity argument.
 * 
 * minCapacity만큼의 elements들을 저장할 수 있는 새로운 배열 객체를 만들어서
 * 기존 배열의 elements들을 복사하고 남는 elements들에는 null 또는 0을 채워서 리턴한다.
 *
 * @param minCapacity the desired minimum capacity
 * @throws OutOfMemoryError if minCapacity is less than zero
*/
   
private Object[] grow(int minCapacity) {
    return elementData = Arrays.copyOf(elementData,
                                       newCapacity(minCapacity));
}

private Object[] grow() {
    return grow(size + 1);
}
```



> Arrays.copyOf(원본 배열, 복사할 길이);
>
> elementData의 length보다 newCapacity(minCapacity)가 큰 값이 넘어오면
>
> 배열이 0이나 null로 채워짐.



- **public boolean add(E e)**
- **private void add(E e, Object[] elementData, int s)**

```java

/**
 * Appends the specified element to the end of this list.
 * list 뒤에다가 파라미터로 넘어온 element를 갖다 붙인다.
 * @param e element to be appended to this list
 * @return {@code true} (as specified by {@link Collection#add})
*/


public boolean add(E e) {
    modCount++;
    add(e, elementData, size);
    return true;
}


private void add(E e, Object[] elementData, int s) {
  	if (s == elementData.length)
        elementData = grow();
    elementData[s] = e;
    size = s + 1;
}
```







아래의 MyArrayList 클래스는 

java.util.ArrayList의 private 필드들을 public으로 고쳐서 만든 클래스다.

```java
public class MyArrayListEx {
    public static void main(String[] args) {
        MyArrayList<Integer> list = new MyArrayList<>();

        System.out.println("Before Insert");
        printCurrentStatus(list);


        list.add(1);
        System.out.println("After Add 1");
        printCurrentStatus(list);


        for (int i = 2; i <= 10; i++) {
            list.add(i);
        }
        System.out.println("After Add 2 ~ 10");
        printCurrentStatus(list);


        list.add(11);
        System.out.println("After Add 11");
        printCurrentStatus(list);
    }

    private static void printCurrentStatus(MyArrayList<Integer> list) {
        System.out.println("elementData = " + Arrays.toString(list.elementData));
        System.out.println("capacity = " + list.elementData.length);
        System.out.println("size = " + list.size);
        System.out.println();
    }
}

// 출력 결과
Before Insert
elementData = []
capacity = 0
size = 0

After Add 1
elementData = [1, null, null, null, null, null, null, null, null, null]
capacity = 10
size = 1

After Add 2 ~ 10
elementData = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
capacity = 10
size = 10

After Add 11
elementData = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, null, null, null, null]
capacity = 15
size = 11
```





