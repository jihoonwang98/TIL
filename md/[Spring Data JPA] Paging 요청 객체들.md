# [Spring Data Jpa] Paging 요청 객체들



> 참고: https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/domain/Pageable.html



이번 포스팅은 Paging Request Object들에 대해 살펴본다.



![](https://media.vlpt.us/post-images/conatuseus/6ff84670-c01c-11e9-b44b-c13d9af8c699/image.png)





## `Pageable` 인터페이스

페이징 요청 정보를 위한 인터페이스다.



| 메서드                                   | 설명                                                         |
| :--------------------------------------- | :----------------------------------------------------------- |
| Pageable first()                         | first page를 요청하는 Pageable 객체를 리턴한다.              |
| long getOffset()                         | page 번호와 page size를 고려해서 offset를 리턴한다.          |
| int getPageNumber()                      | page 번호를 리턴한다.                                        |
| int getPageSize()                        | page size를 리턴한다.                                        |
| Sort getSort()                           | Sort 객체를 리턴한다.                                        |
| default Sort getSortOr(Sort sort)        | 현재 Sort를 리턴하거나 현재 Sort가 없으면 파라미터로 받은 Sort를 반환한다. |
| boolean hasPrevious()                    | 현재 Pageable 객체의 이전 Pageable 객체를 접근할 수 있는지 없는지를 리턴한다. |
| default boolean isPaged()                | 현재 Pageable 객체가 페이징 정보를 갖고 있는지 없는지를 리턴한다. |
| default boolean isUnpaged()              | 현재 Pageable 객체가 페이징 정보를 갖고 있지 않으면 true 리턴. |
| Pageable next()                          | 다음 Page를 요청하는 Pageable 객체를 리턴한다.               |
| static Pageable ofSize(int pageSize)     | size가 pageSize인 첫 번째 Page를 요청하는 Pageable 객체를 리턴한다. |
| Pageable previousOrFirst()               | 이전 Pageable 객체가 존재하면 그대로 반환하고, 만약 존재하지 않으면(현재가 첫 번째 페이지 요청 Pageable 객체라면) 첫 번째 페이지 요청 Pageable 객체를 리턴한다. |
| default Optional\<Pageable> toOptional() | Pageable을 Optional\<Pageable>로 변환해서 반환해준다.        |
| static Pageable unpaged()                | 어떠한 페이징 정보도 들고 있지 않은 Pageable 객체를 반환해준다. |
| Pageable withPage(int pageNumber)        | pageNumber 정보가 적용된 Pageable 객체를 반환해준다.         |





## `PageRequest` 구현 클래스



| 메서드                                                       | 설명                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| static PageRequest of(int page, int size)                    | 정렬되지 않는 PageRequest 객체를 생성한다.                   |
| static PageRequest of(int page, int size, Sort.Direction direction, String... properties) | Sort direction과 properties가 적용된 PageRequest 객체를 생성한다. 여기서 properties는 정렬 기준 필드를 말한다. |
| static PageRequest of(int page, int size, Sort sort)         | Sort 파라미터가 적용된 PageRequest 객체를 생성한다.          |
| static PageRequest ofSize(int pageSize)                      | first page(페이지 번호 0)를 요청하는 PageRequest 객체를 생성한다. (페이지 크기는 pageSize) |
| PageRequest first()                                          | 첫번째 page를 요청하는 PageRequest 객체를 반환한다.          |
| Sort getSort()                                               | Sort 객체를 리턴한다.                                        |
| PageRequest next()                                           | 다음 page를 요청하는 PageRequest 객체를 반환한다.            |
| PageRequest previous()                                       | 이전 page를 요청하는 PageRequest 객체를 반환한다.            |





















