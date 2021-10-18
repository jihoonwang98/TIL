# [npm warning] A requires a peer of B but none was installed

> [해결법(stackoverflow)](https://stackoverflow.com/questions/46053414/npm-warn-requires-a-peer-of-but-none-is-installed-you-must-install-peer)



### 문제상황

- NestJS를 설치하는데 아래와 같은 Warning이 뜸.

  ```shell
  npm WARN ajv-keywords@3.5.2 requires a peer of ajv@^6.9.1 but none is installed. You must install peer dependencies yourself.
  ```

  

![](./img/스크린샷%202021-10-18%20오전%208.31.09.png)

- 뜻: `acv-keywords@3.5.2`가 `ajv@^6.9.1`의 peer를 필요로 하지만 설치되어 있지 않다. 그래서 내가 스스로 peer dependency들을 설치해야 한다(자동으로 설치해주지 않는다).
- "A requires a peer of B but none was installed". Consider it as "A requires one of B's peers but that peer was not installed and we're not telling you which of B's peers you need."



> 근데 ajv가 뭐냐??
>
> **AJV**
>
> - [Ajv JSON schema validator](https://github.com/ajv-validator/ajv)
>
> - ajv란?
>
>   - "The fastest JSON validator for Node.js and browser."
>
>     Node.js와 브라우저를 위한 가장 빠른 JSON validator
>
>   - ajv에서 jv가 json validator인듯 하다..



### 해결책

- peer dependency를 자동으로 설치해 주는 기능은 `npm 3`에서 사라졌다.

  - [NPM Blog](http://blog.npmjs.org/post/110924823920/npm-weekly-5)
  - [Release notes of v3](https://github.com/npm/npm/releases/tag/v3.0.0)
  - 따라서 `npm 3` 이상 버전을 사용하면 peer dependency들을 자동으로 설치해 주지 않는다.

- Updated Solution

  - **<u>그래서 어떻게 해결하느냐?</u>**

  - ```shell
    npm WARN {something} requires a peer of {other thing} but none is installed. You must install peer dependencies yourself.
    ```

    위처럼 떴으면 아래처럼 입력하면 된다.

    ```shell
    npm install --save-dev "{other thing}"
    ```

  - 예제

    ```shell
    npm WARN ajv-keywords@3.5.2 requires a peer of ajv@^6.9.1 but none is installed. You must install peer dependencies yourself.
    ```

    ```shell
    npm install --save-dev "ajv@^6.9.1"
    ```

    

  



