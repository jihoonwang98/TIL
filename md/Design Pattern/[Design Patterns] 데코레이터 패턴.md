# [Design Patterns] 데코레이터 패턴



## 해결하고자 하는 문제점



### 스타버즈 커피숍 주문 시스템



![](https://docs.google.com/drawings/d/s5o4GEsmqTb6M9vhF7XibgQ/image?parent=e/2PACX-1vT4FUrp2Lwg9UQwUiiA8j87QUfsgpP83NqlYrGMbP97kLrYyPMr1oUVKr7sfY-9kRXoX5ZanX5zA3WM&rev=600&drawingRevisionAccessToken=eTV1VOjMk5vPIA&h=418&w=601&ac=1)

- 스타버즈 커피숍은 모든 음료들의 정보를 위와 같은 설계로 저장하고 있습니다.

- 커피숍에 가보신 분들은 아시겠지만, 커피를 주문할 때는 스팀 우유나 두유, 모카 등을 추가할 수도 있습니다.

  물론 공짜는 아니기 때문에 토핑을 추가할 때마다 가격이 올라갑니다.

  이런 기능을 위 그림의 주문 시스템에 도입할 경우 굉장한 **<u>문제</u>**가 생기게 됩니다.. (아래의 그림 참고)







![](https://docs.google.com/drawings/d/siV7b9cBAqazTKRw-DoqXNA/image?parent=e/2PACX-1vT4FUrp2Lwg9UQwUiiA8j87QUfsgpP83NqlYrGMbP97kLrYyPMr1oUVKr7sfY-9kRXoX5ZanX5zA3WM&rev=551&drawingRevisionAccessToken=PZWawmGu4ZlYmw&h=362&w=601&ac=1)



- 토핑을 추가하는 경우 가격이 달라지기 때문에 토핑을 추가할 수 있는 모든 경우의 수마다 클래스를 새로 정의하여 `cost()` 메서드를 재정의해주어야 합니다.
- 위와 같이 설계된 시스템의 경우 음료와 토핑의 개수가 늘어날 수록 구현해야 하는 클래스의 개수 역시 폭발적으로 늘어나게 됩니다...







## 해결책 #1

**Idea: 추가된 토핑 계산을 `Beverage` 클래스에서 처리한다.**

 

![](https://docs.google.com/drawings/d/sRKqK2lCHVfMn4aZ-07vraQ/image?parent=e/2PACX-1vT4FUrp2Lwg9UQwUiiA8j87QUfsgpP83NqlYrGMbP97kLrYyPMr1oUVKr7sfY-9kRXoX5ZanX5zA3WM&rev=762&drawingRevisionAccessToken=m7H5pvzVQ2FrMw&h=453&w=601&ac=1)





### 문제점

- 토핑이 추가될때마다 `Beverage`에 `set토핑()`, `has토핑()` 메서드를 추가해야하고, `cost()` 메서드도 고쳐야 합니다.
- 어떤 음료는 특정 토핑을 추가하면 안 되는 경우도 있을 수 있습니다.
  - 아이스티에는 휘핑 크림을 추가하지 않습니다...

- 손님이 더블 모카를 주문하면 큰일납니다..







## 해결책 #2 - Decorator Pattern 

데코레이터 패턴을 사용하면 토핑을 추가한 음료를 다음과 같은 식으로 해결할 수 있습니다.



**[예제] Mocha와 Whip 토핑을 추가한 Dark Roast 커피 주문하기** 

1. **DarkRoast 객체 가져오기**

   ![](https://docs.google.com/drawings/d/sS9Q0BO6bYZfnhLrJqG-sWA/image?parent=e/2PACX-1vT4FUrp2Lwg9UQwUiiA8j87QUfsgpP83NqlYrGMbP97kLrYyPMr1oUVKr7sfY-9kRXoX5ZanX5zA3WM&rev=45&drawingRevisionAccessToken=czUMLd9m-AV_9Q&h=110&w=142&ac=1)

   

2. **Mocha 객체로 장식하기**

   ![](https://docs.google.com/drawings/d/sQwrFjAiUjgre84smzcDmvQ/image?parent=e/2PACX-1vT4FUrp2Lwg9UQwUiiA8j87QUfsgpP83NqlYrGMbP97kLrYyPMr1oUVKr7sfY-9kRXoX5ZanX5zA3WM&rev=32&drawingRevisionAccessToken=-EV2R0Plt7x_VA&h=229&w=288&ac=1)

   

3. **Whip 객체로 장식하기**

   

   ![](https://docs.google.com/drawings/d/sA2IYdUs3LAAjvMOIjEgmZg/image?parent=e/2PACX-1vT4FUrp2Lwg9UQwUiiA8j87QUfsgpP83NqlYrGMbP97kLrYyPMr1oUVKr7sfY-9kRXoX5ZanX5zA3WM&rev=37&drawingRevisionAccessToken=yPHFMaAZycGAwA&h=303&w=353&ac=1)

   

4. **cost() 메서드 호출. 이때 토핑의 가격을 계산하는 일은 해당 객체들에게 위임된다.**

   

   ![](https://docs.google.com/drawings/d/sLOkoLZVFRHewhUEXkxqYVg/image?parent=e/2PACX-1vT4FUrp2Lwg9UQwUiiA8j87QUfsgpP83NqlYrGMbP97kLrYyPMr1oUVKr7sfY-9kRXoX5ZanX5zA3WM&rev=445&drawingRevisionAccessToken=dProQ3tbh6qZFQ&h=352&w=684&ac=1)







### 데코레이터 패턴 특징

- 데코레이터의 부모 클래스는 자신이 장식하고 있는 객체의 부모 클래스와 같습니다.

- 한 객체를 여러 개의 데코레이터로 감쌀 수 있습니다.

- 데코레이터는 자신이 감싸고 있는 객체와 같은 부모 클래스를 가지고 있기 때문에

  원래 객체(싸여져 있는 객체)가 들어갈 자리에 데코레이터 객체를 집어넣어도 상관 없습니다.

- 데코레이터는 자신이 장식하고 있는 객체에게 어떤 행동을 위임하는 것 외에 원하는 추가적인 작업을 수행할 수 있습니다.

- 객체는 언제든지 감쌀 수 있기 때문에 실행중인 필요한 데코레이터를 마음대로 적용할 수 있습니다.







### 데코레이터 패턴 정의

>**Decorator Pattern**이란?
>
>데코레이터 패턴에서는 객체에 추가적인 요건을 동적으로 첨가한다.
>
>데코레이터는 서브클래스를 만드는 것을 통해서 기능을 유연하게 확장할 수 있는 방법을 제공한다.



![](https://docs.google.com/drawings/d/e/2PACX-1vQRfFz1E20oqUozOaneoxq7T8BSTbt0W9wUK_Fx-04Z1wscjpEywiMp7BP-A7cwlY-hqfSg5HHBbp1y/pub?w=1161&h=863)



### 데코레이터 패턴 커피숍에 적용하기



![](https://docs.google.com/drawings/d/e/2PACX-1vTdYB3PC-yP5qveIw-oNbF6z22WBd7GWmeNT036Ga7oVMw4GrV0_F5f_iueFCXyMv7ns1MVBfNbE6Hi/pub?w=1524&h=965)