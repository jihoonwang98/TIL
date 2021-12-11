# [JS] 자바스크립트 모듈 로더 (module loader)

- 모듈 포맷

  - AMD (Asynchronous Module Definition)

    - 브라우저에서 사용되고 define 함수를 사용해서 모듈을 정의

    - require.js에 의해서 구현됨

    - 예제

      ```js
      // s1.js
      define(function() {
      	console.log("s1");
        return {
          text: 'test'
        };
      })
      ```

      ```js
      // s2.js
      require(['s1'], function(s1) {
        console.log("s2");
        document.body.appendChild(document.createTextNode(s1.text));
      });
      ```

      ```html
      // index.html
      <script data-main="s2.js" src="require.js"></script>
      ```

  - CommonJS

    - Node.js에서 사용되고 `require`와 `module.exports`를 사용해서 의존성과 모듈을 정의한다.

    - 예제

      ```js
      // double-number.js
      module.exports = function(number) {
        return number * 2;
      }
      ```

      ```js
      // index.js
      var doubleNumber = require('./double-number');
      console.log(doubleNumber(4));  // 8
      ```

  - UMD (Universal Module Definition)

    - 브라우저와 Node.js에서 둘 다 사용

    - 예제

      ```js
      // mod1.js
      (function (root, factory) {
          if (typeof module === 'object' && module.exports) {
              // Node/CommonJS
              module.exports = factory();
              console.log('mod1-1');
          } else if (typeof define === 'function' && define.amd) {
              // AMD. Register as an anonymous module.
              define(factory);
              console.log('mod1-2');
          } else {
              // Browser globals
              root.mod1 = factory();
              console.log('mod1-3');
          }
      }(this, function factory() {
          console.log('mod1-4');
          // public API
          return {
              foo: 'bar'
          };
      }));
      ```

      ```html
      // index.html
      <script src="mod1.js"></script>
      <script>
        (function (root, factory) {
          if (typeof module === 'object' && module.exports) {
            // Node/CommonJS
            factory(
              require('./mod1.js')
            );
            console.log(1);
          } else if (typeof define === 'function' && define.amd) {
            // AMD
            define([
              './mod1.js'
            ], factory);
            console.log(2);
          } else {
            // Browser globals
            factory(root.mod1);
            console.log(3, root.mod1);
          }
        }(this, function factory(mod1) {
          // Tests here
          console.log(4, mod1);
        }));
      </script>
      ```

  - ES6 (ECMAScript 2015) 모듈

    - 자바스크립트 표준 모듈

    - 2017년부터 모든 브라우저에서 지원하기 시작함

    - 예제

      ```js
      // lib.js
      export const pi = Math.PI;
      ```

      ```js
      // app.js
      import * as lib from './lib.js';
      console.log(lib.pi);  // 3.141592....
      ```

      ```html
      <!-- 머시기.html -->
      <script type="module" src="app.js"></script>
      ```

      

  

  

  

  

  

  

  

