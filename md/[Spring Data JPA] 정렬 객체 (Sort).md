# [Spring Data Jpa] 정렬 객체 (Sort)



> 참고: https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/domain/Sort.html





## `Sort` 클래스

Sort 옵션을 지정하기 위한 클래스.



Nested Class로는 아래와 같은 클래스들이 있다.

- `Sort.Direction`: 정렬 방향(내림차순, 오름차순) 지정

- `Sort.NullHandling`: null 처리 지정

- `Sort.Order`: Sort.Direction과 property 쌍을 구현한다.

  

| 메서드                                                       | 설명                                                |
| :----------------------------------------------------------- | :-------------------------------------------------- |
| static Sort by(List<Sort.Order> orders)                      | 주어진 orders로 새로운 Sort 객체를 만든다.          |
| static Sort by(Sort.Direction direction, String... properties) | 주어진 direction과 properties로 Sort 객체를 만든다. |
| static Sort by(Sort.Order... orders)                         | 주어진 orders로 새로운 Sort 객체를 만든다.          |
| static Sort by(String... properties)                         | 주어진 properties로 새로운 Sort 객체를 만든다.      |







## `Sort.Order` 내부 클래스

| 생성자                                                       | 설명                                                        |
| ------------------------------------------------------------ | ----------------------------------------------------------- |
| Order(Sort.Direction direction, String property)             | direction과 property 쌍으로 Order 객체 생성                 |
| Order(Sort.Direction direction, String property, Sort.NullHandling nullHandlingHint) | direction과 property 쌍, nullHandlingHint로 Order 객체 생성 |









## `Sort.Direction` Enum

| Enum 상수 |
| :-------- |
| `ASC`     |
| `DESC`    |









## `Sort.NullHandling` Enum

| Enum 상수들                                                  |
| :----------------------------------------------------------- |
| `NATIVE`:  DB가 null 처리하는 방식대로 처리하게 냅둔다.      |
| `NULLS_FIRST`:  null인 값이 non null인 값들보다 먼저 오게 처리한다. |
| `NULLS_LAST`:  null인 값이 non null인 값들보다 뒤에 오게 처리한다. |







