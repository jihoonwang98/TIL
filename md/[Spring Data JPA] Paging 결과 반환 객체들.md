# [Spring Data Jpa] Paging Result Return 객체들



> 참고 : https://www.programmersought.com/article/8559117013/



이번 포스팅은 Paging 결과를 저장하는 객체들에 대해 살펴본다.



![](https://www.programmersought.com/images/370/aa4421c407831243ad01c8d6113e4752.png)





## 인터페이스 `Page`

`Page` 인터페이스는 pagination result set의 Page를 모델링한 인터페이스다.



| 인터페이스                                                   | 설명                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| int getTotalPages()                                          | 총 Page 수를 리턴                                            |
| int getTotalElements()                                       | 총 Element 개수를 리턴                                       |
| \<S> Page\<S> map(Converter<? super T, ? extends S> converter) | `Page` 안의 Content(실제 데이터 List) 속에 들어있는 element들을 converter로 들어온 함수로 변환해서 새로운 `Page` 객체로 mapping 해주는 함수. (Entity -> Dto 변환 시 유용하다.) |



`Page` 인터페이스는 `Slice` 인터페이스를 상속받는다.

그래서 위에 나온 메서드들 이외에도 `Slice` 인터페이스에 정의된 다양한 메서드들을 가지고 있다.







## 인터페이스 `Slice` (a piece of data)

`Slice`는 어떤 record set 내의 몇 개의 연속적인 record들(a piece of data, 데이터 뭉치)을 모델링하기 위해 만들어진 인터페이스다. 

그리고 앞 뒤로 데이터 뭉치가 더 있다면 이것들을 반환하는 메서드들도 정의되어 있다.



`Page` 역시 결과 집합에서 몇 개의 연속적인 record들을 뽑아서 Page 정보와 함께 제공해야 하므로, `Slice` 타입이라고도 볼 수 있다.



| 인터페이스                                                   | 설명                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| int getNumber()                                              | 전체 data set을 한 개당 크기가 `size`인 `Slice`(데이터 묶음)로 나눴을 때, 현재 `Slice`의 index를 반환한다. `Page` 관점에서 보면 그냥 페이지 번호다. |
| int getSize()                                                | `Slice` 한 개에 들어갈 수 있는 data record 개수를 `size`라고 한다. 이때 getSize() 메서드는 이 `size`를 리턴한다. |
| int getNumberOfElements()                                    | 현재 `Slice`에 들어있는 element 개수를 리턴한다.             |
| List \<T> getContent()                                       | 현재 `Slice`에 들어있는 data record collection(실제 데이터)을 리턴한다. |
| boolean hasContent()                                         | 현재 `Slice`에 data record가 포함되어 있는지 없는지(Content가 있는지 없는지)를 리턴한다. |
| Sort getSort()                                               | 현재 `Slice`의 `Sort` 옵션을 리턴한다.                       |
| boolean isFirst()                                            | 전체 data set에서 현재 `Slice`가 첫 번째 `Slice`인지 아닌지를 리턴한다. **주의: 첫 번째 `Slice` <u>인덱스는 0부터 시작</u>** |
| boolean isLast()                                             | 전체 data set에서 현재 `Slice`가 마지막 `Slice`인지 아닌지를 리턴한다. |
| boolean hasPrevious()                                        | 이전 `Slice`가 존재하는지 안하는지를 리턴한다.               |
| boolean hasNext()                                            | 다음 `Slice`가 존재하는지 안하는지를 리턴한다.               |
| Pageable nextPageable()                                      | 다음 `Slice`에 대한 paged query object(`Pageable`)가 존재하면 반환하고, 없으면 `null`을 반환한다. |
| Pageable previousPageable()                                  | 이전 `Slice`에 대한 paged query object(`Pageable`)가 존재하면 반환하고, 없으면 `null`을 반환한다. |
| \<S> Slice\<S> map(Converter<? super T, ? extends S> converter) | `Slice` 내의 데이터들을 converter로 들어온 함수로 변환해서 새로운 `Slice`로 mapping 해주는 메서드. |





## 추상 클래스 `Chunk`

추상 클래스 `Chunk`는 `Slice` 인터페이스를 implement하고 있다.

`Chunk`는 전체 data set의 총 record 개수에 대한 정보를 포함하고 있지 않은, 말 그대로 데이터 뭉치(블록)다.

그래서 `Slice` 인터페이스의 메서드 중 `hasNext()` 메서드는 구현하고 있지 않다.



`Chunk`는 추상 클래스이므로, 페이징 된 data를 표현하는 객체를 만들고 싶을 때는

`Chunk`가 아닌 `Chunk`의 구현 클래스, `PageImpl`을 사용해야 한다.







## `PageImpl` 클래스

페이징 결과를 저장하는 `Chunk` 추상 클래스와 `Page` 인터페이스의 구현 클래스.



`PageImpl` 객체를 만들 때는 아래의 세 가지 기본 정보를 기반으로 생성한다.

- Paging data record 컬렉션(List\<T> content)
- 페이징 결과에 대응하는 `Pageable` 객체 (paging request 정보를 담은 객체)
- 전체 data collection의 총 record 개수



생성자가 아래와 같은 메서드 시그니처를 가진다.

```java
public PageImpl(List<T> content, Pageable pageable, long total) {
    ...
}
```