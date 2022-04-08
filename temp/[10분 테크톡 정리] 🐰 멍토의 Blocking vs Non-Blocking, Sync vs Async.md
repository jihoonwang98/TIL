# [10분 테크톡 정리] 🐰 멍토의 Blocking vs Non-Blocking, Sync vs Async

> https://youtu.be/oEIoqGd-Sns



목차

- Blocking vs Non-Blocking
- Synchronous vs Asynchronous
- 조합 4가지 경우
- 정리





## 1. Blocking vs Non-Blocking

### 정의

- Blocking: 자신의 작업을 진행하다가 다른 주체의 작업이 시작되면 다른 작업이 끝날 때까지 <u>기다렸다가</u> 자신의 작업을 시작하는 것

  - 예시: 내가 회사에서 Blocking이라는 상사에게 문서를 전달해야 한다.

    이때, Blocking 상사는 자신이 문서를 다 읽을때까지 나에게 대기하라고 명령한다.

    Blocking 상사가 문서를 다 읽고나서야 나는 내 업무를 할 수 있다.

- Non-Blocking: 다른 주체의 작업에 <u>관련없이</u> 자신의 작업을 하는 것

  - 예시: 이번에는 Non-Blocking 상사에게 문서를 전달해야 한다.

    이때, Non-Blocking 상사는 "읽어볼테니 업무 보세요"라고 한다.



### 차이점

- Blocking vs Non-Blocking 차이점
  - 다른 주체가 작업할 때 **<u>자신의 제어권이 있는지 없는지</u>**로 볼 수 있다.





## 2. Synchronous vs Asynchronous

### 정의

- Synchronous (동기): 작업을 동시에 수행하거나, 동시에 끝나거나, **<u>끝나는 동시에 시작함을 의미</u>**

  - 예시: 이번에는 Synchronous 상사에게 문서를 전달해야 한다.

    이때, Synchronous 상사는 내가 기다리던지 내 업무를 하고 있던지 상관하지 않는다.

    하지만, Synchronous 상사에게 받은 업무는 받은 즉시 업무를 시작해야 한다. 

- Asynchronous (비동기): 시작, 종료가 일치하지 않으며, **<u>끝나는 동시에 시작을 하지 않음</u>**을 의미

  - 예시: 이번에는 Asynchronous 상사에게 문서를 전달해야 한다.

    이때, Asynchronous 상사도 내가 기다리던지 내 업무를 하고 있던지 상관하지 않는다.

    Asynchronous 상사에게 받은 업무는 바로 처리하지 않아도 괜찮다. 나중에 확인하라고 지시하는 상사임.



### 차이점

- Synchronous vs Asynchronous 차이점
  - 결과를 돌려주었을 때 **<u>순서와 결과</u>**에 관심이 있는지 없는지로 판단할 수 있다.



## 3. 조합 4가지 경우

- Blocking + Sync

  - Blocking이므로 나에게 제어권이 없는 상황. (기다려야함) 

  - Sync이므로 다른 주체가 결과를 반환하면 해당 업무를 바로 처리해야 함.

  - 예시: 내가 회사에서 BlockingSync라는 상사에게 문서를 전달해야 한다.

    이때, BlockingSync 상사는 자신이 문서를 다 읽을때까지 나에게 대기하라고 명령한다.

    BlockingSync 상사가 문서를 다 읽고나면 곧바로 가서 해당 업무를 처리해야 한다.

  - 예시2

  ![image-20220407173638272](/Users/mojo/Library/Application Support/typora-user-images/image-20220407173638272.png)

  ![image-20220407173655270](/Users/mojo/Library/Application Support/typora-user-images/image-20220407173655270.png)

  - 자바에서 입력요청을 할 때 Blocking + Sync임
  - 제어권이 넘어가서 기다리고 (Blocking)
  - 결과를 받은 즉시 다음 해당 결과를 이용해 다음 업무 수행 (Sync)

  

- Non-Blocking + Sync

  - ![image-20220407173908656](/Users/mojo/Library/Application Support/typora-user-images/image-20220407173908656.png)

  - Non-Blocking이므로 나는 내 업무 보고 있음

  - Sync이므로 결과에 관심이 많음. 받자마자 업무 시작해야 함. 그래서 중간중간에 자꾸 물어봄 

  - 예시: 내가 회사에서 Non-BlockingSync라는 상사에게 문서를 전달해야 한다.

    이때, Non-BlockingSync 상사는 자신이 문서를 다 읽을때까지 나에게 다른 일 하고 있으라고 지시한다.

    그런데, 내가 Non-BlockingSync 상사에게 너무 시달린 나머지 상사가 전달해준 일을 바로 처리해야 된다는 강박관념이 생겼다. 그래서 중간중간 물어본다.

  - 예시2: 게임에서 맵 넘어갈 때 로드율이 얼마인지 나타낼때

    ![image-20220407174331789](/Users/mojo/Library/Application Support/typora-user-images/image-20220407174331789.png) 



- Blocking + Async

- Non-Blocking + Async

  - 예시: JS에서 API 요청하고 다른 작업 하다가 콜백을 통해서 추가적인 작업 처리해야 할 때

  ![image-20220407174633361](/Users/mojo/Library/Application Support/typora-user-images/image-20220407174633361.png)





## 4. 정리

