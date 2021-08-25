# [Design Patterns] Factory Pattern



## 1. 문제점

### **피자 가게 예시**



**피자 주문 메서드**

```java
Pizza orderPizza() {

    Pizza pizza;
    
    if(type.equals("cheese")) {
        pizza = new CheesePizza();
    } else if (type.equals("greek")) {
        pizza = new GreekPizza();
    } else if (type.equals("pepperoni")) {
        pizza = new PepperoniPizza();
    }
    
    // 만약 피자 메뉴가 추가되거나 삭제되면, 위의 if-else 문을 계속 변경해줘야 한다...
    
    pizza.prepare();
    pizza.bake();
    pizza.cut();
    pizza.box();
    
    return pizza;
}
```

- 위의 코드는 코드 변경에 대해 닫혀있지 않다. 피자 가게에서 메뉴를 변경하려면 위의 코드를 직접 수정해야 한다.
  - 바뀌는 부분은 if-else 문이다. 그 외의 부분은 변하지 않는다.
  - 바뀌는 부분을 캡슐화하자.





### 객체 생성 부분을 캡슐화

이제 객체 생성하는 부분을 `orderPizza()` 메서드에서 뽑아내야 한다는 것 자체는 결정된 상태입니다.

그런데 어떻게 하면 좋을까요??

일단 객체 생성 코드만 따로 빼서 피자 객체를 만드는 일만 전담하는 다른 객체에 위임하도록 합시다.



이 객체에는 **팩토리(Factory)**라는 이름을 붙입시다.

객체 생성을 처리하는 클래스를 팩토리라고 부릅니다.

우리의 예시에서는 `SimplePizzaFactory`가 이 역할을 하게 됩니다.

피자가 필요할 때마다 이 `SimplePizzaFactory`에게 피자 객체를 하나 만들어달라고 부탁하면 됩니다.

그러면 이제 더 이상 `orderPizza()` 메서드에서 어떤 피자를 만들어야 하는지 고민하지 않아도 됩니다.



```java
public class SimplePizzaFactory {
    public Pizza createPizza(String type) {
        Pizza pizza = null;
        
        if(type.equals("cheese")) {
            pizza = new CheesePizza();
        } else if(type.equals("pepperoni")) { 
        	pizza = new PepperoniPizza(); 
        } else if(type.equals("clam")) {
            pizza = new ClamPizza();
        } else if(type.equals("veggie")) {
            pizza = new VeggiePizza();
        }
        
        return pizza;
    }
}
```



```java
public class PizzaStore {
    SimplePizzaFactory factory;
    
    public PizzaStore(SimplePizzaFactory factory) {
        this.factory = factory;
    }
    
    
    public Pizza orderPizza(String type) {
        Pizza pizza;
        
        pizza = factory.createPizza(type);
        
        pizza.prepare();
        pizza.bake();
        pizza.cut();
        pizza.box();
        return pizza;
    }
}
```



 











