# [Design Patterns] 옵저버 패턴



## 옵저버 패턴이란?



![](https://docs.google.com/drawings/d/snLdAzERapBIMf_eiqYl3_w/image?parent=e/2PACX-1vT4FUrp2Lwg9UQwUiiA8j87QUfsgpP83NqlYrGMbP97kLrYyPMr1oUVKr7sfY-9kRXoX5ZanX5zA3WM&rev=407&drawingRevisionAccessToken=eNsC_RJ_zP5y5A&h=524&w=601&ac=1)



![](https://johngrib.github.io/post-img/observer-pattern/structure.jpg)



- **Subject**(주제) 인터페이스

  - 신문 구독 서비스로 예를 들면, 신문사가 여기에 해당한다. 데이터의 주인.

  - registerObserver(): 옵저버로 등록
  - removeObserver(): 옵저버 목록에서 탈퇴시킴
  - notifyObservers(): 상태가 바뀔 때마다 모든 옵저버들에게 연락을 하기 위한 메서드

  

- **ConcreteSubject** 구현체

  - getState() / setState(): Subject(주제) 클래스의 상태를 설정하거나 알아내기 위한 메서드

  

- **Observer** 인터페이스
  - 신문 구독 서비스로 예를 들면, 신문 구독자가 여기에 해당한다. 
  - 옵저버는 Subject의 상태가 변경되는 경우 변경된 데이터를 받기를 기대한다.
  - update(): Subject의 상태가 바뀌었을 때 호출되는 메서드





### 일대다 관계

- 옵저버 패턴에서 상태를 저장하고 지배하는 것은 **Subject** 객체다.
- 이러한 **Subject** 객체에 의존하는 **Observer**는 여러 개가 있을 수 있다.
- **Subject : Observer** == 일대다 관계





### 느슨한 결합(Loose Coupling)

- 느슨한 결합이란?
  - 두 객체가 느슨하게 결합되어 있다는 것은, 그 둘이 상호작용을 하긴 하지만 서로에 대해 서로 잘 모른다는 것을 의미한다.
  - 서로 상호작용을 하는 객체 사이에는 가능하면 느슨하게 결합해야 프로그램이 유연해진다. (변화에 대응하기 쉽다)

- 옵저버 패턴에서의 느슨한 결합

  - 옵저버 패턴에서는 Subject와 Observer가 서로 느슨하게 결합되어 있는 디자인을 제공한다.

  - Subject가 Observer에 대해서 아는 것은 특정 인터페이스(Observer 인터페이스)를 구현한다는 것 뿐이다.

  - 느슨하게 결합되어 있기 때문에 새로운 Observer를 추가하는 것이 쉽다.

    - Subject는 Observer 인터페이스를 구현하는 객체의 목록에만 의존하기 때문에

      Observer 인터페이스만 구현한다면 Observer로 추가할 수 있다.

  - Subject와 Observer를 서로 독립적으로 재사용할 수 있다.

  - Subject와 Observer가 바뀌더라도 서로에게 영향을 미치지 않는다.

    - 둘이 서로 느슨하게 결합되어 있기 때문에 Subject 혹은 Observer 인터페이스를 구현한다는 조건만 지키면 어떻게 변경해도 문제가 생기지 않는다.

