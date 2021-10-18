# .DS_Store 파일을 git에 포함시키지 않기

> https://philographer.github.io/development/gitignore-ds-store/

### 해결방법

- 이미 `.DS_Store`가 커밋되었다면 다음 명령어를 사용한다.

  `.DS_Store`를 찾아서 0번째 인자로 받고 이를 `$ git rm`하는 명령이다.

  ```shell
  find . -name .DS_Store -print0 | xargs -0 git rm --ignore-unmatch
  ```

- 모든 `repository`에서 제거하고 싶을 때는 다음의 명령어를 사용한다.

  DS_Store가 상위 폴더에 있을 때나 이름이 '..DS_Store'일 때도 포함하지 않을 수 있다.

  ```shell
  $ echo ".DS_Store" >> ~/.gitignore_global
  $ echo "._.DS_Store" >> ~/.gitignore_global
  $ echo "**/.DS_Store" >> ~/.gitignore_global
  $ echo "**/._.DS_Store" >> ~/.gitignore_global
  $ git config --global core.excludesfile ~/.gitignore_global
  ```

