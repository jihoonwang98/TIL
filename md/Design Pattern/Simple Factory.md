# Simple Factory



## 1. Simple Factory



### 피자 가게 예시



![asdf](https://lh5.googleusercontent.com/EtxJLJ9CxS_4Ombn5Qri_46OtOlLYwpf647rt8rsb3LEtwqY3KZvHDfG4uYgQfe7BsLUBsUUMELJpScMsfv5ZhfEafb4LuYXup7faV8VRCmnnZNTAEmxfdY_-KhHGBrS4PVonZqX)





### `Pizza` 인터페이스

```java
public interface Pizza {

    void prepare();
    void bake();
    void cut();
    void box();

    int getPrice();
    String getDescription();
}
```





### Pizza 클래스들 

**CheesePizza**

```java
public class CheesePizza implements Pizza {

    private static final int PRICE = 12_000;

    @Override
    public void prepare() {
        System.out.println("치즈피자 준비중!!");
    }

    @Override
    public void bake() {
        System.out.println("치즈피자 굽는중!!");
    }

    @Override
    public void cut() {
        System.out.println("치즈피자 자르는중!!");
    }

    @Override
    public void box() {
        System.out.println("치즈피자 포장하는중!!");
    }

    @Override
    public int getPrice() {
        return PRICE;
    }

    @Override
    public String getDescription() {
        return "고소하고 맛있는 치즈 피자에요";
    }
}
```



**ClamPizza**

```java
public class ClamPizza implements Pizza {

    private static final int PRICE = 14_000;

    @Override
    public void prepare() {
        System.out.println("Clam 피자 준비중!!");
    }

    @Override
    public void bake() {
        System.out.println("Clam 피자 굽는중!!");
    }

    @Override
    public void cut() {
        System.out.println("Clam 피자 자르는중!!");
    }

    @Override
    public void box() {
        System.out.println("Clam 피자 포장하는중!!");
    }

    @Override
    public int getPrice() {
        return PRICE;
    }

    @Override
    public String getDescription() {
        return "해산물이 들어간 CLAM 피자에요";
    }
}
```



**PepperoniPizza**

```java
public class PepperoniPizza implements Pizza {

    private static final int PRICE = 9_000;

    @Override
    public void prepare() {
        System.out.println("페페로니 피자 준비중!!");
    }

    @Override
    public void bake() {
        System.out.println("페페로니 피자 굽는중!!");
    }

    @Override
    public void cut() {
        System.out.println("페페로니 피자 자르는중!!");
    }

    @Override
    public void box() {
        System.out.println("페페로니 피자 포장하는중!!");
    }

    @Override
    public int getPrice() {
        return PRICE;
    }

    @Override
    public String getDescription() {
        return "짭잘한 페페로니 피자에요";
    }
}
```



**VeggiePizza**

```java
public class VeggiePizza implements Pizza {

    private static final int PRICE = 10_000;

    @Override
    public void prepare() {
        System.out.println("야채 피자 준비중!!");
    }

    @Override
    public void bake() {
        System.out.println("야채 피자 굽는중!!");
    }

    @Override
    public void cut() {
        System.out.println("야채 피자 자르는중!!");
    }

    @Override
    public void box() {
        System.out.println("야채 피자 포장하는중!!");
    }

    @Override
    public int getPrice() {
        return PRICE;
    }

    @Override
    public String getDescription() {
        return "건강에 좋은 야채 피자에요";
    }

}
```









**아래에 나오는 클래스들은 위에서 만든 Pizza 클래스들을 생성해서 이용하는 클래스들이다.**



### `PizzaOrder` 클래스

```java
public class PizzaOrder {

    public Pizza orderPizza(String type) {

        switch (type) {
            case "cheese":
                return new CheesePizza();

            case "clam":
                return new ClamPizza();

            case "pepperoni":
                return new PepperoniPizza();

            case "veggie":
                return new VeggiePizza();
        }

        return null;
    }
}
```



### `PizzaRecommendation` 클래스

```java
public class PizzaRecommendation {

    private List<Pizza> pizzas;

    public PizzaRecommendation() {
        pizzas = new ArrayList<>();
        pizzas.add(new CheesePizza());
        pizzas.add(new ClamPizza());
        pizzas.add(new PepperoniPizza());
        pizzas.add(new VeggiePizza());
    }

    public Pizza recommend() {
        Random random = new Random(System.currentTimeMillis());
        return pizzas.get(random.nextInt(4));
    }
}
```





### `PizzaShopMenu` 클래스

```java
public class PizzaShopMenu {

    private List<Pizza> pizzas;

    public PizzaShopMenu() {
        pizzas = new ArrayList<>();
        pizzas.add(new CheesePizza());
        pizzas.add(new ClamPizza());
        pizzas.add(new PepperoniPizza());
        pizzas.add(new VeggiePizza());
    }

    public void explainMenus() {
        pizzas.forEach(p -> System.out.println(p.getDescription()));
    }
}
```





### 의존 관계

![pizza2](https://lh6.googleusercontent.com/3VjE6_s91JHbCvzlOkILuLYxLmPJu1sTADIA80_GSgU9shHnUz_Ry18OWTdQuBYs8UTjICnroWWxVB6REygG4ECAdWXe6w8AgiGj_6GzDK9P-rcww9EA1t7SN13kSHmx5ovqBlwV)

- `PizzaOrder`, `PizzaRecommendation`, `PizzaShopMenu` 클래스가

  인터페이스인 `Pizza`에만 의존하는 것이 아니라 

  구상 클래스인 `CheesePizza`, `ClamPizza`, `PepperoniPizza`, `VeggiePizza`에 모두 의존한다.

  - 구상 클래스에 직접적으로 의존할 경우 구상 클래스가 하나 추가&middot;삭제 될 때마다 수정해야 하는 코드가 늘어난다.



### 문제점

-  `PizzaOrder`, `PizzaRecommendation`, `PizzaShopMenu` 클래스들은 Pizza 구상 클래스를 직접 생성하고 있다.
  - `new` 키워드를 사용하면 인터페이스가 아닌 특정 구상 클래스에 의존하게 된다.
  - 이렇게 구상 클래스에 의존하게 되면 피자가 추가되거나 삭제될 때마다 코드를 수정해야 한다.







#### 새로운 피자 추가될 때?

```java
public class PizzaOrder {

    public Pizza orderPizza(String type) {

        switch (type) {
            case "cheese":
                return new CheesePizza();

            case "clam":
                return new ClamPizza();

            case "pepperoni":
                return new PepperoniPizza();

            case "veggie":
                return new VeggiePizza();
                
            /* 추가 */
            case "new":
                return new NewPizza();                
            /* 추가 */
        }

        return null;
    }
}
```



```java
public class PizzaRecommendation {

    private List<Pizza> pizzas;

    public PizzaRecommendation() {
        pizzas = new ArrayList<>();
        pizzas.add(new CheesePizza());
        pizzas.add(new ClamPizza());
        pizzas.add(new PepperoniPizza());
        pizzas.add(new VeggiePizza());
        
        
        /* 추가 */
        pizzas.add(new NewPizza());              
        /* 추가 */
    }

    ...
}
```



```java
public class PizzaShopMenu {

    private List<Pizza> pizzas;

    public PizzaShopMenu() {
        pizzas = new ArrayList<>();
        pizzas.add(new CheesePizza());
        pizzas.add(new ClamPizza());
        pizzas.add(new PepperoniPizza());
        pizzas.add(new VeggiePizza());
        
        
        /* 추가 */
        pizzas.add(new NewPizza());              
        /* 추가 */
    }
    
    ...
}
```

- 구상 클래스를 `new`로 직접 생성하고 있는 클래스들은 모두 수정되어야 한다.
- 삭제될 때도 마찬가지로 다 수정해줘야 한다.







## 해결책

구상 클래스를 생성해주는 팩토리 클래스를 따로 만든다.

구상 클래스를 생성해야 할 때는 팩토리 클래스에게 요청한다.



### `SimplePizzaFactory` 클래스

```java
public class SimplePizzaFactory {

    public Pizza createPizza(String type) {
        switch (type) {
            case "cheese":
                return new CheesePizza();

            case "clam":
                return new ClamPizza();

            case "pepperoni":
                return new PepperoniPizza();

            case "veggie":
                return new VeggiePizza();
        }

        return null;
    }


    public List<Pizza> getAllPizzas() {
        List<Pizza> pizzas = new ArrayList<>();
        pizzas.add(new CheesePizza());
        pizzas.add(new ClamPizza());
        pizzas.add(new PepperoniPizza());
        pizzas.add(new VeggiePizza());

        return pizzas;
    }
}
```





### 변경된 `PizzaOrder` 클래스

```java
public class PizzaOrder {

    private SimplePizzaFactory factory;

    public PizzaOrder(SimplePizzaFactory factory) {
        this.factory = factory;
    }

    public Pizza orderPizza(String type) {
        // 구상 클래스를 생성하는 책임을 factory에게 위임한다.
        return factory.createPizza(type);
    }
}
```



### 변경된 `PizzaRecommendation` 클래스

```java
public class PizzaRecommendation {

    private SimplePizzaFactory factory;

    public PizzaRecommendation(SimplePizzaFactory factory) {
        this.factory = factory;
    }

    public Pizza recommend() {
        // 구상 클래스를 생성하는 책임을 factory에게 위임한다.
        List<Pizza> pizzas = factory.getAllPizzas();
        
        Random random = new Random(System.currentTimeMillis());
        return pizzas.get(random.nextInt(4));
    }
}
```





### 변경된 `PizzaShopMenu` 클래스

```java
public class PizzaShopMenu {
    private SimplePizzaFactory factory;

    public PizzaShopMenu(SimplePizzaFactory factory) {
        this.factory = factory;
    }

    public void explainMenus() {
        // 구상 클래스를 생성하는 책임을 factory에게 위임한다.
        List<Pizza> pizzas = factory.getAllPizzas();
        
        pizzas.forEach(p -> System.out.println(p.getDescription()));
    }
}
```





### 변경된 의존 관계

![pizza3](https://lh5.googleusercontent.com/EnqZz1HdzkQT5c2CgcBD_uJh4Mb-r6DGEjpp6O4qlhLmaBxWZXvAFGPtN_6N7tt-xjwrvqoP9FaLBA3MZHIVSpB_5wkJIKkjgTSBl1HTk_T40IUwhha5OPQm-YzeeWP9mFEJ-e5Z)







### 새로운 피자가 추가될 때?



**NewPizza 클래스**

```java
public class NewPizza implements Pizza {
    @Override
    public void prepare() {
        System.out.println("New 피자 준비하는중");
    }

    @Override
    public void bake() {
        System.out.println("New 피자 굽는중");
    }

    @Override
    public void cut() {
        System.out.println("New 피자 자르는중");
    }

    @Override
    public void box() {
        System.out.println("New 피자 포장하는중");
    }

    @Override
    public int getPrice() {
        return 33_000;
    }

    @Override
    public String getDescription() {
        return "따끈따끈한 신상 피자입니다";
    }
}
```



**SimplePizzaFactory 클래스**

```java
public class SimplePizzaFactory {

    public Pizza createPizza(String type) {
        switch (type) {
            case "cheese":
                return new CheesePizza();

            case "clam":
                return new ClamPizza();

            case "pepperoni":
                return new PepperoniPizza();

            case "veggie":
                return new VeggiePizza();

            /* 추가 */
            case "new":
                return new NewPizza();
            /* 추가 */
        }

        return null;
    }


    public List<Pizza> getAllPizzas() {
        List<Pizza> pizzas = new ArrayList<>();
        pizzas.add(new CheesePizza());
        pizzas.add(new ClamPizza());
        pizzas.add(new PepperoniPizza());
        pizzas.add(new VeggiePizza());
        
        /* 추가 */
        pizzas.add(new NewPizza());
        /* 추가 */

        return pizzas;
    }
}
```

- 새로운 피자 클래스 `NewPizza`를 만들어준 후 `SimplePizzaFactory` 클래스만 수정해주면 된다.
- 어떤 클래스의 코드를 수정해야할 지 명확해진다.



