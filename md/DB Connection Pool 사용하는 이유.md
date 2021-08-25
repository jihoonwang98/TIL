## DB Connection Pool 사용하는 이유

**DB 커넥션 풀**이란? 

- **DB Connection **객체를 여러 개 생성하여 **풀(Pool)**에 담아 놓고 필요할 때 꺼내 쓰는 방식
- 즉, **Connection** 객체를 미리 만들어 두고, 필요할 때마다 빌리고, 사용한 다음 반납하는 방식



**DB 커넥션 풀을 사용하는 <u>이유</u>???**

먼저, **DB 커넥션** 객체를 <u>하나만</u> 만들어 사용했을 때의 문제점을 살펴보자.





![](https://docs.google.com/drawings/d/sTheDfH0NTZCU4Z34dvHU4A/image?parent=e/2PACX-1vSgWs9-SkKDVDplyC96i2vj5TuZvbmLHvACG_NMQ3eEa1hFtJwCGbFZVRaHriS3HOeS8O0XIQ6gqqII&rev=428&h=321&w=601&ac=1)



위의 그림에서 **DAO 1, 2, 3**이 사용하는 **Statement**는 같은 **Connection**에서 생성한 객체다. **SQL**문을 실행하다 보면 **DAO 3**처럼 예외가 발생했을 때 이전 상태로 되돌려야 하는 경우가 있다. 이렇게 작업하기 전 상태로 되돌리는 것을 '**롤백(rollback)**'이라고 하는데, 안타깝게도 <u>**Statement** 객체에는 롤백 기능이 없다</u>. <u>롤백 기능은 **Connection** 객체를 통해서만 수행 가능하다</u>.



문제는 위의 그림에서처럼 **DAO 3**가 **Connection** 객체의 **rollback** 기능을 호출할 경우, 그 **Connection**을 통해 이루어지는 모든 작업도 **rollback**된다는 것이다. 즉, **DAO 1, 2**가 작업한 것까지 이전 상태로 되돌려진다. 



따라서 웹 브라우저의 요청을 처리할 때, <u>각 요청에 대해 **개별 DB Connection**을 사용해야 한다.</u>





**[위의 문제점의 해결책]**

**(1)** 그렇다면 이를 해결하기 위해 **SQL 작업을 할 때마다 DB Connection 객체를 생성**하는 것은 어떤가?

**Connection**을 맺을 때마다 **DB 서버**는 사용자 인증과 권한 검사를 수행하고 요청 처리를 위한 준비 작업을 해야 한다. 따라서 요청이 들어올 때마다 **Connection**을 생성한다면 속도가 느려져서 성능에 문제가 생길 수 있다..



**(2) **<u>**DB Connection Pool** 이용하기</u>

**DB Connection Pool**을 이용하면, 각 요청에 대해 **별도의 Connection 객체**를 사용하기 때문에 다른 작업에 영향을 주지 않는다. 또한, 사용한 **DB Connection** 객체는 버리지 않고 **Pool**에 보관해 두었다가 다시 사용하기 때문에, 가비지가 생성되지 않고 속도도 빨라진다.











