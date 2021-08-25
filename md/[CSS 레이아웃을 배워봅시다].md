# [CSS 레이아웃을 배워봅시다] 





## 1. `display` 속성

모든 element는 딱 한 개의 `display` 속성값을 가지고 있다.

이 `display` 속성은 CSS에서 레이아웃을 제어할 때 가장 중요한 속성이다.

`display` 속성은 element를 화면에 어떻게 드러나게 할지를 결정한다.





### 종류

- `none`
- `block`
- `inline`
- `inline-block`
- `flex`
- `list-item`





### `block` 속성

- 일반적으로 설정하지 않아도 `div`가 갖게되는 기본값
- width가 자신의 컨테이너의 100%가 되게 함 (가로 한 줄 다 차지함)
- 새로운 줄에서 시작해서 좌우로 최대한 늘어난다.
- `block` 요소는 `inline` 요소, `block` 요소 둘 다 포함 가능
- `block` 요소는 `width`, `height`, `padding`, `margin`으로 레이아웃 수정 가능
- `block` 요소는 `vertical-align` 적용 안됨
- `block` 요소는 끝나는 지점에서 자동으로 줄바꿈됨
- `block` 요소 종류
  - `ul`, `li`, `ol`, `dl`, `div`, `p`, `h1` ~ `h6`, `form`, `header`, `footer`, `section`, `table`, `pre`, `blockquote`, `article`, `nav`등등





### `inline` 속성

- 일반적으로 설정하지 않아도 `span`이 갖게되는 기본값
- 컨텐츠가 끝나는 지점까지 넓이를 가진다. (넓이를 필요한 만큼만 차지한다)
- `inline` 요소는 `inline` 요소 포함 가능, `block` 요소 포함 불가능
- `inline` 요소는 `width`, `height`으로 레이아웃 수정 불가능
  - `padding`, `margin`은 좌우만 수정 가능
- `inline` 요소는 `line-height` 혹은 `text-align` 가능
- `inline` 요소는 끝나는 지점에서 줄바꿈 없음
- `inline` 요소 종류
  - `a`, `img`, `span`, `input`, `textarea`, `select`, `i`, `b`, `button`





### `none` 속성

- 화면에서 사라짐. 공간도 차지 안함.







## 2. `margin: auto`

```css
#main {
  width: 600px;
  margin: 0 auto; 
}
```



블록 요소의 `width`를 설정하면 컨테이너의 좌우 가장자리로 늘어나지 않게 할 수 있다.

그런 다음 `margin`을 `auto`로 설정해 해당 요소를 컨테이너 안에서 가로 중앙에 오게 할 수 있다.

요소는 지정한 `width`를 차지하게 되고, 나머지 공간은 두 `margin`에 균등하게 나눠질 것이다.



브라우저 창이 요소 너비보다 좁을 때 유일하게 문제가 발생한다.

브라우저에서는 페이지에 가로 스크롤바를 만들어 이 문제를 해결한다.

그럼 이 같은 상황을 개선해 보자.





> **[참고]**
>
> ```css
> <!-- 위 아래 왼쪽 오른쪽 margin을 주지 않는다 -->
> margin: 0;
> 
> <!-- 100 같은 숫자는 단위가 없어서 적용 X / %나 px을 써줘야함 -->
> margin: 100;
> margin: 100px;
> 
> <!-- 위아래 margin 100px, 양옆 margin auto -> 균등 분배 -> 요소가 정중앙 배치 -->
> margin: 100px auto;
> ```



> **[참고] `margin: 0 auto` 안될때**
>
> - `width` 속성 조작 불가능할 때: 폭의 연산이 불가능하면 가운데 정렬이 안된다.
> - `inline` 속성의 태그





## 3. `max-width`

```css
#main {
  max-width: 600px;
  margin: 0 auto; 
}
```



`max-width`를 사용하면 스크롤바가 사라진다.





## 4. 박스 모델

`width`에 관해 이야기하는 김에 `width`와 관련해서 반드시 알아둬야 할 **박스 모델**에 대해 이야기하겠습니다.

요소의 `width`를 설정할 경우 해당 요소는 실제로 여러분이 설정한 것보다 크게 나타날 수 있습니다.

이것은 요소의 `border`와 `padding`이 지정된 `width` 이상으로 요소를 늘리기 때문입니다.

다음 예제를 보시면 너비 값을 동일하게 설정해도 결과적으로 크기가 다릅니다.



```css
.simple {
  width: 500px;
  margin: 20px auto;
}

.fancy {
  width: 500px;
  margin: 20px auto;
  padding: 50px;
  border: solid blue 10px;
}
```



오랫동안 이 문제를 해결하는 방법은 `width` 값을 조절하는 것이었습니다.

하지만 더 좋은 방법이 있습니다.





## 5. `box-sizing`

오랜 시간에 걸쳐 사람들은 `width` 값을 조절하는 해법이 그다지 만족스럽지 않다는 사실을 깨닫고 `box-sizing`이라고 하는 새로운 CSS 속성를 만들어냈습니다. 

요소에 `box-sizing`을 지정하면 해당 요소의 `padding`과 `border`가 더는 `width`를 늘리지 않습니다.



다음은 이전 페이지와 동일한 예제이지만 두 요소에 모두 `box-sizing: border-box;`를 지정했습니다.



```css
.simple {
  width: 500px;
  margin: 20px auto;
  -webkit-box-sizing: border-box;
     -moz-box-sizing: border-box;
          box-sizing: border-box;
}

.fancy {
  width: 500px;
  margin: 20px auto;
  padding: 50px;
  border: solid blue 10px;
  -webkit-box-sizing: border-box;
     -moz-box-sizing: border-box;
          box-sizing: border-box;
}
```





## 6. `position`

복잡한 레이아웃을 만들기 위해서는 `position` 속성에 관해 살펴볼 필요가 있습니다.

`position` 속성에는 다양한 값을 설정할 수 있으며, 각 값의 이름은 제대로 지어지지 않아서 기억하기가 불가능할 겁니다.

그럼 지금부터 하나씩 살펴봅시다.



### `position` 속성값의 종류

- **static**
- **relative**
- **fixed**
- **absolute**





### static

```css
.static {
	position: static;
}
```

- `position: static`은 기본값입니다.
- static으로 설정된 요소는 그다지 특별한 방식으로 위치가 지정된 것은 아닙니다.
- static(정적) 요소는 위치가 지정된 것이 아니라고 표현하며, static이 아닌 다른 값으로 지정된 요소에 대해 위치가 지정됐다고 표현합니다.





### relative

```css
.relative1 {
  position: relative;
}
.relative2 {
  position: relative;
  top: -20px;
  left: 20px;
  background-color: white;
  width: 500px;
}
```

- `relative`는 별도의 속성을 지정하지 않는 이상 `static`과 동일하게 동작합니다.
- `relative`가 지정된 요소에 `top`, `right`, `bottom`, `left`를 지정하면 기본 위치와 다르게 위치가 조정됩니다.
- 다른 콘텐츠는 해당 요소에 남긴 공백에 맞춰 들어가게끔 조정되지 **않습니다.**





### fixed

```css
.fixed {
  position: fixed;
  bottom: 0;
  right: 0;
  width: 200px;
  background-color: white;
}
```

- `fixed` 요소는 `viewport`에 상대적으로 위치가 지정되는데, 이는 페이지가 스크롤되더라도 늘 같은 곳에 위치한다는 뜻입니다.
- `relative`와 마찬가지로 `top`, `right`, `bottom`, `left` 속성이 사용됩니다.





### absolute

```css
.relative {
  position: relative;
  width: 600px;
  height: 400px;
}
.absolute {
  position: absolute;
  top: 120px;
  right: 0;
  width: 300px;
  height: 200px;
}
```

- `absolute`는 가장 다루기 까다로운 위치 지정 값입니다.

- `absolute`는 `viewport`에 상대적으로 위치가 지정되는게 아니라 

  **<u>가장 가까운 곳에 위치한 조상 요소에 상대적으로 위치가 지정된다</u>**는 점을 제외하면 `fixed`와 비슷하게 동작합니다.

- `absolute` 위치가 지정된 요소가 기준으로 삼을 조상 요소가 없으면 문서 `body`를 기준으로 삼고, 페이지 스크롤에 따라 움직입니다.





## 7. 위치 지정 예제

```css
.container {
  position: relative;
}
nav {
  position: absolute;
  left: 0px;
  width: 200px;
}
section {
  /* position is static by default */
  margin-left: 200px;
}
footer {
  position: fixed;
  bottom: 0;
  left: 0;
  height: 70px;
  background-color: white;
  width: 100%;
}
body {
  margin-bottom: 120px;
}
```





## 8. `float` 속성

`float`는 다음과 같이 이미지 주위를 텍스트로 감싸기 위해 만들어졌스빈다.

```css
img {
	float: right;
	margin: 0 0 1em 1em;
}
```







## 9. `clear` 속성

`clear` 속성은 `float`의 동작 방식을 제어하는 데 중요합니다.

아래의 두 예제를 비교해 봅시다.



```html
<div class="box">...</div>
<section>...</section>
```

```css
.box {
  float: left;
  width: 200px;
  height: 100px;
  margin: 1em;
}
```



이 경우 `section` 엘리먼트는 실제로 `div` 다음에 있습니다. 

하지만 `div`가 왼쪽으로 떠 있기 때문에 이런 결과가 나타난 것입니다. 

즉, `section` 안의 텍스트가 `div` 주위에 떠 있고 `section`이 전체를 감싸고 있습니다.

`section`을 실제로 떠 있는 엘리먼트 다음에 나타나게 하려면 어떻게 해야 할까요?



```css
.box {
  float: left;
  width: 200px;
  height: 100px;
  margin: 1em;
}
.after-box {
  clear: left;
}
```

`clear`를 이용해 이제 이 섹션을 떠 있는 `div` 아래로 옮겼습니다. 

여기서는 `left` 값을 지정해 왼쪽에 떠 있는 엘리먼트들을 비웠습니다. 

게다가 오른쪽(`right`)과 양쪽(`both`)도 비울 수 있습니다.





## 10. percent











