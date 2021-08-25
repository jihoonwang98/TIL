# Java Collection Framework Reference



![](https://static.javatpoint.com/images/java-collection-hierarchy.png)





![](https://www.ictdemy.com/images/1/java/collections/java_collection_api_diagram.svg)



![](https://helpezee.files.wordpress.com/2019/08/capture1.jpg?w=1024&h=627)







## Collection\<E> 인터페이스



| 메서드                                                       | 설명                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| boolean add(E e)<br />boolean addAll(Collection<? extends E> c) | e를 추가하거나 c에 들어 있는 요소를 추가한다. 컬렉션이 변경되면 true를 반환한다. |
| boolean remove(Object o)<br />boolean removeAll(Collection<?> c)<br />boolean retainAll(Collection\<?\> c)<br />boolean removeIf(Predicate\<? super E\> filter)<br />void clear() | 각각 o 객체, c에 들어있는 요소, c에 없는 요소, 일치하는 요소, 모든 요소를 제거한다. 처음 메서드 네 개는 컬렉션이 변경되면 true를 반환한다. |
| int size()                                                   | 컬렉션에 들어 있는 요소의 개수를 반환한다.                   |
| boolean isEmtpy()<br />boolean contains(Object o)<br />boolean containsAll(Collection\<?\> c) | 컬렉션이 비어 있거나, o를 포함하거나, c에 있는 모든 요소를 포함하면 true를 반환한다. |
| Iterator\<E\> iterator()<br />Stream\<E\> stream()<br />Stream\<E\> parallelStream()<br />Spliterator\<E\> spliterator() | 컬렉션의 요소를 방문하는 반복자, 스트림, 병렬스트림, 분할반복자를 반환한다. |
| Object[] toArray()<br />T[] toArray(T[] a)                   | 컬렉션의 요소를 담은 배열을 반환한다. 두 번째 메서드는 a의 길이가 충분하면 a 배열에 요소를 담아 반환한다. |







## List<E\> 인터페이스



| 메서드                                                       | 설명                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| boolean add(int index, E e)<br />boolean addAll(int index, Collection\<? extends E\> c)<br />boolean add(E e)<br />boolean addAll(Collection\<? extends E\> c) | index의 앞이나 리스트의 끝에 e 또는 c에 들어 있는 요소를 추가한다. 리스트가 변경되면 true를 반환한다. |
| E get(int index)<br />E set(int index, E element)<br />E remove(int index) | 지정한 인덱스에 있는 요소를 얻거나 설정하거나 제거한다. 마지막 두 메서드는 호출 이전에 해당 인덱스에 있던 요소를 반환한다. |
| int indexOf(Object o)<br />int lastIndexOf(Object o)         | o와 동일한 처음 또는 마지막 요소의 인덱스를 반환한다. 일치하는 요소가 없으면 -1을 반환한다. |
| ListIterator\<E\> listIterator()<br />ListIterator\<E\> listIterator(int index) | 모든 요소 또는 index에서 시작하는 요소에 대응하는 리스트 반복자를 돌려준다. |
| void replaceAll(UnaryOperator\<E\> operator)                 | 각 요소에 연산자를 적용한 결과로 각 요소를 교체한다.         |
| void sort(Comparator\<? super E\> c)                         | c로 지정한 순서 규칙에 따라 이 리스트를 정렬한다.            |
| static List\<E\> of(E... elements)                           | 지정한 요소들을 담은 수정 불가 리스트를 돌려준다.            |
| List\<E\> subList(int fromIndex, int toIndex)                | fromIndex에서 시작해 toIndex 앞에서 끝나는 서브리스트의 뷰를 돌려준다. |





## Collections\<T\> 클래스의 static 메서드



**static 메서드**

| 메서드 (모두 static)                                         | 설명                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| boolean disjoint(Collection\<?\> c1, Collection\<?\> c2)     | 두 컬렉션에 공통된 요소가 없으면 true를 반환한다.            |
| boolean addAll(Collection\<? super T> c, T... elements)      | 모든 요소를 c에 추가한다.                                    |
| void copy(List\<? super T\> dest, List\<? extends T\> src)   | src의 모든 요소를 dest에 같은 인덱스로 저장한다(dest의 길이는 src 이상이어야 한다). |
| boolean replaceAll(List\<T\> list, T oldVal, T newVal)       | oldVal 요소를 모두 newVal로 교체한다(둘 중 하나가 null일 수도 있다). 일치하는 요소가 하나라도 있으면 true를 반환한다. |
| void fill(List\<? super T\> list, T obj)                     | 리스트의 모든 요소를 obj로 설정한다.                         |
| List\<T\> nCopies(int n, T o)                                | o의 사본 n개를 담은 불변 리스트를 돌려준다.                  |
| int frequency(Collection\<?\> c, Object o)                   | c에 들어 있는 요소 중 o와 동일한 요소의 개수를 반환한다.     |
| int indexOfSubList(List\<?\> source, List<?\> target)<br />int lastIndexOfSubList(List\<?\> source, List\<?\> target) | source 리스트에서 target 리스트가 처음 또는 마지막으로 나타난 시작 인덱스를 반환한다. target 리스트를 찾을 수 없으면 -1을 반환한다. |
| int binarySearch(List\<? extends Comparable\<? super T\>\> list, T key)<br />int binarySearch(List\<? extends T\> list, T key, Comparator\<? super T\> c) | 리스트가 자연 순서 또는 c로 정렬되어 있다고 가정한 상태에서 키 위치를 반환한다. 키가 없으면 -i -1을 반환한다. 이때 i는 해당 키를 삽입해야 하는 위치다. |
| sort(List\<T> list)<br />sort(List\<T> list, Comparator<? super T> c) | 자연 순서 또는 c를 이용해 리스트를 정렬한다.                 |
| void swap(List\<?\> list, int i, int j)                      | 지정한 위치에 있는 요소들을 교환한다.                        |
| void rotate(List\<?\> list, int distance)                    | 인덱스 i에 있는 요소를 인덱스 (i + distance) % list.size()로 옮기는 방식으로 리스트를 순환한다. |
| void reverse(List\<?\> list)<br />void shuffle(List\<?\> list)<br />void shuffle(List\<?\> list, Random rnd) | 리스트를 뒤집거나 무작위로 뒤섞는다.                         |
| synchronized(Collection\|List\|Set\|SortedSet\|<br />NavigableSet\|Map\|SortedMap\|NavigableMap)() | 동기화 뷰를 돌려준다.                                        |
| unmodifiable(Collection\|List\|Set\|SortedSet\|<br />NavigableSet\|Map\|SortedMap\|NavigableMap)() | 수정 불가 뷰를 돌려준다.                                     |
| checked(Collection\|List\|Set\|SortedSet\|<br />NavigableSet\|Map\|SortedMap\|NavigableMap\|Queue)() | 검사 뷰를 돌려준다.                                          |







## SortedSet\<E\>의 인터페이스

| 메서드                                                       | 설명                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| E first()<br />E last()                                      | 집합의 처음과 마지막 요소를 반환한다.                        |
| SortedSet\<E\> headSet(E toElement)<br />SortedSet\<E\> subSet(E fromElement, E toElement)<br />SortedSet\<E\> tailSet(E fromElement) | fromElement에서 시작해 toElement 앞에서 끝나는 요소들의 뷰를 반환한다. |







## NavigableSet\<E\>의 인터페이스

| 메서드                                                       | 설명                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| E higher(E e)<br />E ceiling(E e)<br />E floor(E e)<br />E lower(E e) | > \| >= \| <= \| < e인 요소 중 가장 가까운 요소를 반환한다.  |
| E pollFirst()<br />E pollLast()                              | 처음 요소나 마지막 요소를 제거하고 반환한다. 집합이 비어 있으면 null을 반환한다. |
| NavigableSet\<E\> headSet(E toElement, boolean inclusive)<br />NavigableSet\<E\> subSet(E fromElement, boolean fromInclusive, E toElement, boolean toExclusive)<br />NavigableSet\<E\> tailSet(E fromElement, boolean inclusive) | fromElement에서 toElement(포함 또는 미포함) 사이에 잇는 요소들의 뷰를 반환한다. |







### Map\<K, V\>의 인터페이스

| 메서드                                                       | 설명                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| V get(Object key)<br />V getOrDefault(Object key, V defaultValue) | key가 null이 아닌 값 v와 연관되어 있으면 v를 반환한다.<br />그렇지 않으면 null 또는 defaultValue를 반환한다. |
| V put(K key, V value)                                        | key가 null이 아닌 값 v와 연관되어 있으면 key를 value와 연관시키고 v를 반환한다.<br />그렇지 않으면 엔트리를 추가하고 null을 반환한다. |
| V putIfAbsent(K key, V value)                                | key가 null이 아닌 값 v와 연관되어 있으면 value를 무시하고 v를 반환한다.<br />그렇지 않으면 엔트리를 추가하고 null을 반환한다. |
| V merge(K key, V value, BiFunction\<? super V, ? super V, ? extends V\> remappingFunction) | key가 null이 아닌 값 v와 연관되어 있으면 지정한 함수를 v와 value에 적용해서 key와 결과를 연관시키거나 결과가 null이면 키를 제거한다.<br />그렇지 않으면 key와 value를 연관시킨다. get(key)를 반환한다. |
| V compute(K key, BiFunction\<? super K, ? super V, ? extends V\> remappingFunction) | 지정한 함수를 key와 get(key)에 적용한다. key와 결과를 연관시키거나 결과가 null이면 키를 제거한다. get(key)를 반환한다. |
| V computeIfPresent(K key, BiFunction\<? super K, ? super V, ? extends V\> remappingFunction) | key가 null이 아닌 값 v와 연관되어 있으면 지정한 함수를 key와 v에 적용해서 key와 결과를 연관시키거나 결과가 null이면 해당 키를 제거한다. get(key)를 반환한다. |
| V computeIfAbsent(K key, Function\<? super K, ? extends V> mappingFunction) | key가 null이 아닌 값과 연관되어 있지 않으면 지정한 함수를 key에 적용한다. key와 결과를 연관시키거나 결과가 null이면 해당 키를 제거한다. get(key)를 반환한다. |
| void putAll(Map\<? extends K, ? extends V\> m)               | m에 들어 있는 모든 엔트리를 추가한다.                        |
| V remove(Object key)<br />V replace(K key, V newValue)       | 키, 키와 연관된 값을 제거하거나 기존 값을 교체한다. 기존 값을 반환하거나 기존 값이 없으면 null을 반환한다. |
| boolean remove(Object key, Object value)<br />boolean replace(K key, V value, V newValue) | key가 value와 연관되어 있으면 엔트리를 제거하거나 기존 값을 교체하고 true를 반환한다. 그렇지 않으면 아무것도 하지 않고 false를 반환한다. 주로 맵을 동시에 접근할 때 이 메서드들을 이용하낟. |
| int size()                                                   | 엔트리의 개수를 반환한다.                                    |
| boolean isEmpty()                                            | 맵이 비어 있는지 검사한다.                                   |
| void clear()                                                 | 모든 엔트리를 제거한다.                                      |
| void forEach(BiConsumer\<? super K, ? super V\> action)      | 지정한 액션을 모든 엔트리에 적용한다.                        |
| void replaceAll(BiFunction\<? super K, ? super V, ? extends V\< function) | 지정한 함수를 모든 엔트리에 호출한다. 키를 null이 아닌 결과와 연관시키고, null 결과와 연관된 키를 제거한다. |
| boolean containsKey(Object key)<br />boolean containsValue(Object value) | 맵이 지정한 키 또는 값을 담고 있는지 검사한다.               |
| Set\<K\> keySet()<br />Collection\<V\> values()<br />Set\<Map.Entry\<K, V\>\> entrySet() | 키, 값, 엔트리의 뷰를 반환한다.                              |
| static Map\<K, V\> of()<br />static Map\<K, V\> of(K k1, V v1)<br />static Map\<K, V\> of(K k1, V v1, K k2, V v2) ... | 키와 값을 최대 열 개 까지 담은 수정 불가 맵을 돌려준다.      |



















