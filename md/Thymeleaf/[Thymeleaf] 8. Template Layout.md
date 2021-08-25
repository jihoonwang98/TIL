# [Thymeleaf] 8. Template Layout

**목차**

- 8.1 Including template fragments
- 8.2 Parameterizable fragment signatures
- 8.3 Flexible layouts: beyond mere fragment insertion
- 8.4 Removing template fragments
- 8.5 Layout Inheritance





## 8.1 Including template fragments



### Fragment 정의하고 불러오는 방법

템플릿 내에서 다른 footers, headers, menus.. 들 같은 part들을 include하고 싶다고 하자.

타임리프는 이러한 part(부분, 일부)들을 "fragments(파편, 조각)"라고 부른다.

**<u>이러한 fragment들을 정의하기 위해서는 `th:fragment` 속성을 사용한다.</u>**





**[예제]**

모든 페이지의 하단부에 footer를 추가하고 싶다고 하자.

이를 위해, 우리는 `/templates/footer.html` 파일을 아래와 같이 작성했다.

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
  <body>
      
    <!-- copy라는 이름의 fragment를 정의 -->
    <div th:fragment="copy">
      &copy; 2011 The Good Thymes Virtual Grocery
    </div>
  </body>
</html>
```



위의 코드에서 `th:fragment` 속성을 이용해 *copy*라는 이름의 fragment를 정의했다.

이제 우리는 우리의 홈페이지 html에서

`th:insert`나 `th:replace` 속성을 사용해서 이 fragment를 불러올 수 있다.

> `th:include`도 쓸 수 있지만, Thymeleaf 3.0 버전 이후로 권장되는 방법은 아니다.







아래와 같이 `/templates/index.html` 파일을 작성해보자.

```html
<body>
  <div th:insert="~{footer :: copy}"></div>
</body>
```



`th:insert` 속성은 속성값으로 *fragment expression*(**~{...}**)을 받는다.

위의 예제의 fragment expression은 non-complex fragment expression이므로,

중괄호(~{ , })를 써주는 것은 선택사항이다.

따라서 아래와 같이 쓸 수도 있다.

```html
<body>
  <div th:insert="footer :: copy"></div>
</body>
```





### Fragment expression 문법

fragment expression은 세 가지 형식이 있다.

- `~{templatename::selector}`

  **templatename**이라는 이름의 template에 가서 

  **selector**로 지정된 Markup Selector를 적용해서 fragment를 가져온다.

  

  이때, **selector**에 간단히 fragment 이름을 써줘도 괜찮다.

  따라서 `~{templatename::fragmentname}`과 같은 형식으로 써줘도 잘 동작한다.

  위의 예제에서의 `~{footer :: copy}`도 이 문법을 이용한 것이다.



- `~{templatename}`

  **templatename**이라는 이름의 template 전체를 가져온다.



- `~{::selector}` 또는 `~{this::selector}`

  같은 템플릿 내에 있는 **selector**와 매칭되는 fragment를 삽입한다.





### `th:fragment` 속성 없이 fragment 불러오기

`th:fragment` 속성으로 fragment로 등록하지 않아도 

html 요소를 불러오는 방법이 있다.



```html
/templates/footer.html 파일

...
<div id="copy-section">
  &copy; 2011 The Good Thymes Virtual Grocery
</div>
...
```



위와 같은 footer 파일의 **copy-section** div를 뽑아오고 싶다면, 아래와 같이 작성하면 된다.

```html
<body>
  ...
  <div th:insert="~{footer :: #copy-section}"></div>
</body>
```

**selector** 부분에 CSS selector와 비슷한 포맷으로 써주면,

**id** 속성을 통해 해당 div를 참조할 수 있다.



> Markup Selector에 대한 자세한 내용은 [링크](https://www.thymeleaf.org/doc/tutorials/3.0/usingthymeleaf.html#appendix-c-markup-selector-syntax) 참조.









### `th:insert`와 `th:replace`의 차이 (`th:include`)

- `th:insert`
  - 불러온 fragment를 host 태그(`th:insert` 속성을 가지고 있는 태그)의 body에 끼워 넣는다.
- `th:replace`
  - 불러온 fragment를 host 태그와 바꿔치기 한다.
- `th:include`
  - `th:insert`와 비슷하지만, fragment를 삽입하는 대신 fragment의 contents를 삽입하기만 한다.
  - 쓰지말자.



**[예제]**



```html
/templates/footer.html
<footer th:fragment="copy">
  &copy; 2011 The Good Thymes Virtual Grocery
</footer>
```

위의 fragment를 아래에서 불러온다.



```html
/templates/index.html
<body>
  ...
  <div th:insert="footer :: copy"></div>
  <div th:replace="footer :: copy"></div>
  <div th:include="footer :: copy"></div>
</body>
```



위와 같은 코드는 아래와 같이 바뀐다.

```html
<body>
  ...
  <div>
    <footer>
      &copy; 2011 The Good Thymes Virtual Grocery
    </footer>
  </div>
  <footer>
    &copy; 2011 The Good Thymes Virtual Grocery
  </footer>
  <div>
    &copy; 2011 The Good Thymes Virtual Grocery
  </div>
</body>
```





## 8.2 Parameterizable fragment signatures

함수형 template fragment들을 만들기 위해,

`th:fragment`로 정의된 fragment들은 parameter들을 가질 수 있다.

```html
<div th:fragment="frag (onevar,twovar)">
    <p th:text="${onevar} + ' - ' + ${twovar}">...</p>
</div>
```



위와 같이 정의된 fragment를 불러오려면

아래와 같이 두 가지 문법 중 하나를 사용해서 불러올 수 있다.

```html
<div th:replace="::frag (${value1},${value2})">...</div>
<div th:replace="::frag (onevar=${value1},twovar=${value2})">...</div>
```

> `th:insert`를 사용하는 경우도 마찬가지다.





참고로, 두 번째 문법에서 파라미터 순서는 중요하지 않다.

```html
<div th:replace="::frag (twovar=${value2},onevar=${value1})">...</div>
```







### Fragment local variables without fragment arguments

아래와 같이 fragment가 parameter 없이 정의되어도,

```html
<div th:fragment="frag">
    ...
</div>
```



우리는 위에서 살펴본 두 번째 문법(두 번째만 됨)을 이용해서 위의 fragment를 부를 수 있다.

```html
<div th:replace="::frag (onevar=${value1},twovar=${value2})">
```



이 코드는 `th:replace` 속성과 `th:with` 속성을 이용한 아래 코드와 동일하다.

```html
<div th:replace="::frag" th:with="onevar=${value1},twovar=${value2}">
```

**Note** that this specification of local variables for a fragment – no matter whether it has an argument signature or not – does not cause the context to be emptied prior to its execution. Fragments will still be able to access every context variable being used at the calling template like they currently are.







### th:assert for in-template assertions

The `th:assert` attribute can specify a comma-separated list of expressions which should be evaluated and produce true for every evaluation, raising an exception if not.

```html
<div th:assert="${onevar},(${twovar} != 43)">...</div>
```

This comes in handy for validating parameters at a fragment signature:

```html
<header th:fragment="contentheader(title)" th:assert="${!#strings.isEmpty(title)}">...</header>
```





## 8.3 Flexible layouts: 단순한 fragment 삽입을 넘어...







## 8.5 Layout Inheritance 

*title*과 *content*를 포함하고 있는 간단한 layout의 예시를 살펴보자.



```html
/templates/layoutFile.html
<!DOCTYPE html>
<!-- 아래의 fragment는 파라미터로 
     title 태그와 content 태그를 받는다 -->
<html th:fragment="layout (title, content)" xmlns:th="http://www.thymeleaf.org">
<head>
    <title th:replace="${title}">Layout Title</title>
</head>
<body>
    <h1>Layout H1</h1>
    <div th:replace="${content}">
        <p>Layout content</p>
    </div>
    <footer>
        Layout footer
    </footer>
</body>
</html>
```



위의 layout 파일을 상속받아 html을 작성해보자.

```html
/templates/index.html
<!DOCTYPE html>
<!-- layoutFile.html 템플릿 파일안에 있는
	 layout(title, content) fragment를 불러오겠다.
	 그런데, title 파라미터에는 ~{::title}을 넘겨주고,
	 content 파라미터에는 ~{::section}을 넘겨주겠다.
     여기서 ~{::title}은 templatename이 명시되지 않았으므로 
     자기 자신(index.html 내) 안에 있는 title 태그 (<title> Page Title</title>)를 넘겨주고,
     같은 원리로, ~{::section}은 
	 <section>
        <p>Page content</p>
        <div>Included on page</div>
     </section>이 파라미터로 넘어간다.
-->

<html th:replace="~{layoutFile :: layout(~{::title}, ~{::section})}">
<head>
    <title>Page Title</title>
</head>
<body>
<section>
    <p>Page content</p>
    <div>Included on page</div>
</section>
</body>
</html>
```