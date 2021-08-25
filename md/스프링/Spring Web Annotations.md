# Spring Web Annotations



### 1. Overview

이번 포스팅에서는, `org.springframework.web.bind.annotation` 패키지의 **Spring Web Annotation**들에 대해 알아보자.





### 2. @RequestMapping

**@RequestMapping**의 **속성**

- **path(name, value)**: Class 또는 Method가 mapping 되는 path를 지정 (name과 value는 aliases)
- **method**: HTTP Method를 지정
- **params**: 필터링할 **request parameter** 조건을 지정
- **headers**: 필터링할 **HTTP header** 조건을 지정





**(1)** **@Controller** 클래스 안에 있는 **request handler method**에 붙일 때

```java
@Controller
class VehicleController {
    @RequestMapping(value = "/vehicles/home", method = RequestMethod.GET)
    String home() {
        return "home";
    }
}
```





**(2)** **@Controller** 어노테이션과 함께 **클래스 레벨**에 붙일 때

```java
@Controller
@RequestMapping(value = "/vehicles", method = RequestMethod.GET)
class VehicleController {

    @RequestMapping("/home")
    String home() {
        return "home";
    }
}
```



위의 두 가지 예시 **(1), (2)**의 **home()** 메서드는 같은 URL을 처리한다.



또한, 스프링 4.3 버전 이후 **@GetMapping, @PostMapping, @PutMapping, @DeleteMapping, @PatchMapping** 등을 통해  **@RequestMapping** 어노테이션에 해당 **method가 지정된 효과를 낼 수 있다.







### 3. @RequestBody

**@RequestBody**를 객체 앞에 붙여주면 **HTTP request의 body**를 해당 객체에 mapping시켜 주는 역할을 한다.



```java
@PostMapping("/save")
void saveVehicle(@RequestBody Vehicle vehicle) {
    // ...
}
```







### 4. @PathVariable

**@PathVariable** 어노테이션은 **method parameter** 앞에 붙여서 해당 parameter가 **URI template variable(경로 변수)**와 mapping 됨을 알려준다.



**@PathVariable**은 **@RequestMapping**과 함께 쓰인다.

**@RequestMapping**에서 먼저 **경로 변수**가 포함된 **URI template**을 지정한 후에 **method parameter**앞에 **@PathVariable("경로변수명")**을 붙여서 해당 **경로 변수**와 bind 시킨다.



**@RequestVariable**의 **속성**은 **name(value)**으로 **@RequestMapping**에서 지정한 **경로변수명**을 써주면 해당 **경로변수**와 bind된다.

```java
@RequestMapping("/{id}")
Vehicle getVehicle(@PathVariable("id") long id) {
    // ...
}
```





만약 **경로변수명**과 **method parameter 변수명**이 일치하는 경우, **@PathVariable**의 속성을 생략할 수 있다.

```java
@RequestMapping("/{id}")
Vehicle getVehicle(@PathVariable long id) {
    // ...
}
```





게다가, **@PathVariable**의 **required** 속성을 통해 **경로 변수 매핑**을 **optional**하게 만들 수도 있다.

```java
@RequestMapping("/{id}")
Vehicle getVehicle(@PathVariable(required = false) long id) {
    // ...
}
```







### 5. @RequestParam

**@RequestParam** 어노테이션을 통해 **HTTP request parameter**들을 처리할 수 있다.



```java
@RequestMapping
Vehicle getVehicleByParam(@RequestParam("id") long id) {
    // ...
}
```



위의 메서드를 예로 들면 `경로?id=42`로 요청이 오면 **요청 파라미터**로 **이름**은 **id**, **값**은 **42**가 넘어오기 때문에 이를 매핑해주기 위해 **method parameter** 앞에 **@RequestParam("요청 파라미터명")**을 써주면 **method parameter**와 binding된다.





하지만, 위처럼 사용할 경우 **요청 파라미터** 중에 **@RequestParam**으로 지정한 **요청 파라미터**가 넘어오지 않으면 **BadRequest**로 **HTTP 4\*\* **에러가 발생한다.



이런 경우 **defaultValue** 속성을 이용하자.

**defaultValue** 속성을 이용하면, Spring이 해당 **요청 파라미터**를 찾지 못하거나 해당 값이 empty value일 때 **defaultValue** 속성에 명시된 값을 **method parameter**에 bind 시켜준다.



```java
@RequestMapping("/buy")
Car buyCar(@RequestParam(defaultValue = "5") int seatCount) {
    // ...
}
```







### 6. Response Handling Annotations



#### 6.1 @ResponseBody

**(1)** **request handler method**에 **@ResponseBody**를 붙이는 경우

- 해당 **method**의 **return 값**을 **response** 그 자체로 간주한다.



**(2)** **@Controller** 어노테이션과 함께 클래스 레벨에 **@ResponseBody**를 붙이는 경우

- 해당 클래스 안에 있는 모든 **request handler method**들의 **return 값**을 **response** 그 자체로 간주한다.



```java
@ResponseBody
@RequestMapping("/hello")
String hello() {
    return "Hello World!";
}
```





#### 6.2 @ExceptionHandler

이 어노테이션을 **request handler method** 위에 붙여 주면, 해당 **handler method**를 **custom error handler method**로 선언할 수 있다.

**Spring**은 **request handler method**가 명시된 예외를 throw할 때 이 method를 부른다.



```java
@ExceptionHandler(IllegalArgumentException.class)
void onIllegalArgumentException(IllegalArgumentException exception) {
    // ...
}
```



발생한 예외는 메서드의 parameter에 argument로 전달된다.





#### 6.3 @ResponseStatus

메서드에 **@ResponseStatus** 어노테이션을 부여하고 **value**에 **상태 코드**를 지정하면, 그 **응답의 상태 코드**를 지정할 수 있다. 아무것도 지정하지 않으면 **200 OK**가 반환된다.

**reason** 속성을 이용하면 사용자에게 **원인**을 리턴해 줄 수 있다.



```java
@ExceptionHandler(IllegalArgumentException.class)
@ResponseStatus(HttpStatus.BAD_REQUEST)
void onIllegalArgumentException(IllegalArgumentException exception) {
    // ...
}
```







### 7. Other Web Annotations



#### 7.1 @Controller

클래스에 **@Controller** 어노테이션을 붙여 **Spring MVC controller**로 정의할 수 있다.



#### 7.2 @RestController

**@RestController = @Controller + @ResponseBody**



#### 7.3 @ModelAttribute

**(1)** **method parameter** 앞에 붙이는 경우

**(2)** **method** 위에 붙이는 경우





**@ModelAttribute**을 사용하면 **model**에 존재하는 객체들에 접근할 수 있다.



```java
@PostMapping("/assemble")
void assembleVehicle(@ModelAttribute("vehicle") Vehicle vehicleInModel) {
    // ...
}
```



**@PathVariable**이나 **@RequestParam**처럼, **model key**와 **parmeter명**이 일치하면 **model key**를 명시하지 않아도 된다.

```java
@PostMapping("/assemble")
void assembleVehicle(@ModelAttribute Vehicle vehicle) {
    // ...
}
```



만약, **method 위**에 **@ModelAttribute**를 붙이는 경우, **Spring**은 해당 **method**의 **return 값**을 **model**에 넣는다.

```java
@ModelAttribute("vehicle")
Vehicle getVehicle() {
    // ...
}
```



**value** 속성에 아무것도 지정하지 않으면, **Spring**은 **method명**을 **model key**로 사용한다.

```java
@ModelAttribute
Vehicle vehicle() {
    // ...
}
```





> 참고: https://www.baeldung.com/spring-mvc-annotations