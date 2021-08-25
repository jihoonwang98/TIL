# [ZeroCho - React] 정리



## 2강 첫 리액트 컴포넌트

- ```react
  const e = React.createElement;  // creatElement는 tag 만드는 거라고 생각
  ```

  

- ```react
  class LikeButton extends React.Component {
  	constructor(props) {
  		super(props);
  	}
  	
      // 화면에 어떻게 rendering할지(그릴지)
      // <button>Like</button> 같은 태그를 만들겠다. (만든다가 아님, 실제로 만들어주는애는 ReactDOM임)
  	render() {
  		return e('button', {onClick: () => this.setState({ liked: true})}, 'Like');
  	}
     
  }
  ```



- ```html
  <div id="root"></div>
  <script>
  
      const e = React.createElement;
  
      class LikeButton extends  React.Component {
          constructor(props) {
              super(props);
          }
  
          render() {
              return e('button', {onClick: () => this.setState({ liked: true})}, 'Like');
          }
      }
  
  </script>
  <script>
      ReactDOM.render(e(LikeButton), document.querySelector('#root'));
  </script>
  ```

  





## 3강 HTML 속성과 상태(state)

- 기본적으로 리액트는 `<div id="root">` 루트가 하나 필요하다.
  - 루트는 안쪽에 컴포넌트들을 렌더링하기 위해 존재한다.



- `render()` 함수의 return 문의 두번째 파라미터는 HTML 속성을 넣는 자리(객체 형식으로 표현)입니다.

  - ```react
    return e('button', {onClick: () => {console.log('clicked')}, type: 'submit'}, 'Like');
    ```

  - 위의 리턴문은 `<button onclick="() => { console.log('clicked') }" type="submit">Like</button>`으로 렌더링 된다.

  - 속성은 camel case로 작성함에 주의한다. (`onclick` -> `onClick`)



- 컴포넌트는 상태(state)를 가지고 있다.

  - 상태란 바뀔 여지가 있는 부분입니다.

  - ```java
    class LikeButton extends  React.Component {
            constructor(props) {
                super(props);
                this.state = {
                    liked: false
                };
            }
    
        	...
    }
    ```

    위처럼 constructor에 `this.state`를 설정해주고,

  - ```react
    class LikeButton extends  React.Component {
            constructor(props) {
                super(props);
                this.state = {
                    liked: false
                };
            }
    
            render() {
                return e('button', {onClick: () => { this.setState({ liked: true })}, type: 'submit'}, 'Like');
            }
    }
    ```

    위와 같이 `this.setState({ liked: true })`로 써주면 버튼 클릭시 컴포넌트의 state가 바뀐다.

    

  - 이 상태에서 컴포넌트가 상태변경이 되는 걸 화면에 표시하기 위해서 아래와 같이 코드를 작성하자.

    ```react
    class LikeButton extends  React.Component {
            constructor(props) {
                super(props);
                this.state = {
                    liked: false
                };
            }
    
            render() {
                return e('button', 
                         {onClick: () => { this.setState({ liked: true })}, type: 'submit'}, 
                         this.state.liked === true ? 'Liked' : 'Like');
            }
    }
    ```

    



## 4강 JSX와 바벨(babel)

근데 이게 문법이 잘 안와닿는다.

```react
return e('button',{onClick: () => { this.setState({ liked: true })}, type: 'submit'}, this.state.liked === true ? 'Liked' : 'Like');
```

위에처럼 써있으면 알아먹기가 힘들다.



그래서 더 편한 방법이 있다.

```react
<div id="root"></div>
<script type="text/babel">

    class LikeButton extends React.Component {
        constructor(props) {
            super(props);
            this.state = {
                liked: false
            };
        }

        render() {
            return <button
                type='submit'
                onClick={() => {this.setState({liked: true})}}
            >Like</button>
        }
    }

</script>
<script type="text/babel">
    ReactDOM.render(<LikeButton />, document.querySelector('#root'));
</script>
```

위의 코드의 `render()` 함수 부분을 보면 return 문이 html 태그 형식으로 바뀌면서 가독성이 훨씬 나아졌다.

그런데 Javascript 코드 안에 html 태그를 포함한다는게 문법적으로 말이 안되기 때문에

이 기능을 지원하기 위해 Babel(바벨)이라는 라이브러리를 스크립트로 추가해야 한다.

그리고 script의 type에 text/babel을 넣어준다.

이 html 태그 같이 생긴것을 JSX라고 한다.

JSX는 Java Script + XML이다.

사실 html 태그보다는 XML 태그에 더 가깝기 때문이다.



왜냐면 html 태그는 태그 꼭 안닫아줘도 되는데

위의 코드에서 `ReactDOM.render(<LikeButton />, ...)`의 코드는 LikeButton의 태그를 꼭 닫아줘야 한다.

좀 더 엄격하다...



아까 코드랑 완벽하게 매칭시키기 위해 컴포넌트의 state를 화면에 반영시키자.

```react
render() {
    return <button
                type='submit'
                onClick={() => {this.setState({liked: true})}}
            >{this.state.liked ? 'Liked': 'Like'}</button>;
}
```

태그 내에 {} 중괄호를 이용해서 또 JS 코드를 사용할 수 있다.

이제 돌려보면 아까랑 똑같이 잘 나온다.



참고로, 바벨이 위의 JSX를 2강에서 했던 `const e = React.createElement;`해서 했던 방식으로 바꿔주기 때문에 잘 돌아가는 것이다.

