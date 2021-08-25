ObjectMapper가 객체를 어떻게 변환하는지 실험해봄

```java
public class MyBean {
    public int id;
    private String name;

    public MyBean(int id, String name) {
        this.id = id;
        this.name = name;
    }

    public String getTheName() {
        return name;
    }

    public int getYesIAmId() {
        return id;
    }

    public int getYEAHId() {
        return id;
    }
}
```



위처럼 클래스를 작성한 후 

```java
MyBean bean = new MyBean(1, "MyBean");
String result = new ObjectMapper().writeValueAsString(bean);
System.out.println(result);
```

위 코드를 실행해보니깐,



```java
{"id":1,"theName":"MyBean","yeahid":1,"yesIAmId":1}
```

이렇게 나온다.





---



```java
public class MyBean {
    public int id;
    public char c;
    private String name;

    public MyBean(int id, String name, char c) {
        this.id = id;
        this.name = name;
        this.c = c;
    }
}
```

이렇게 써놓고(getter 하나도 없이)



```java
MyBean bean = new MyBean(1, "MyBean", 'c');
String result = new ObjectMapper().writeValueAsString(bean);
System.out.println(result);
```

출력하면,



```json
{"id":1,"c":"c"}
```

이렇게 나온다.







### 결론

**기본형 변수 값**은 자동으로 **<u>"변수명": 값</u>** 형태로 변환되고,

**레퍼런스형 변수 값**은 default로 변환되지 않는다. **getter**를 정의해서 변환하라고 해야 한다.



정의된 **getter**(get머시기 꼴)는 모두 **<u>"머시기": 값</u>** 형태로 변환된다.

 





