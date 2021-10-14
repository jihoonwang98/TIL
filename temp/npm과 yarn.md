## npm과 yarn

> [velog npm과 yarn](https://velog.io/@kysung95/%EA%B0%9C%EB%B0%9C%EC%83%81%EC%8B%9D-npm%EA%B3%BC-yarn)

### NPM

- NPM (Node Package Manager)
  - 자바스크립트 언어를 위한 패키지 관리자로, Node.js의 기본 패키지 관리자
  - 이러한 관리 툴을 이용하여 Node.js로 만들어진 모듈을 웹에서 받아서 쉽게 설치하고 관리해주는 프로그램
  - Java에서의 maven과 비슷한 역할을 한다.
  - 또한 설치된 모듈들이 업데이트되었는지를 체크해주기도 한다.
  - command-line client인 npm과 온라인 DB인 npm registry로 이루어져 있다.
    - 일반적으로 command-line client를 npm이라고 생각하는데, 실제로 npm에는 npm registry까지 포함되어 있다.

- NPM 명령어
  - npm init : package.json 생성
  - npm install : package.json 파일 및 해당 종속성에 나열된 모든 모듈을 설치
  - npm install package_name@버전 : 특정 패키지의 특정 버전 설치
  - npm install 주소 : 특정 저장소 내 패키지 설치. 주로 github을 이와 같이 설치합니다.
  - npm install package_name -g : 옵션. 글로벌로 설치. 로컬의 다른 프로젝트도 이 패키지를 사용 가능하게 됩니다.
  - npm uninstall : 패키지 삭제 명령어입니다.
  - npm update : 설치한 패키지들을 업데이트해줍니다.
  - npm dedupe : 중복 설치된 패키지들을 정리해주는 명령어입니다.



### YARN

- yarn
  - 페이스북에서 만든 JS 패키지 매니저. npm과 같은 기능을 수행한다.
  - npm의 단점인 속도(performance), 안정성(stability), 보안성(security) 등을 보완하기 위해 만들어진 매니저 툴이다.
  - 속도(performance)
    - yarn은 다운받은 패키지 데이터를 캐시(cache)에 저장하여, 중복된 데이터는 다운로드하지않고, 캐시에 저장된 파일을 활용함으로써 이론적으로 npm에 비해 패키지 설치속도가 매우 빠릅니다. 
    - 여러개의 **패키지를 설치할 때 병렬로 처리하기 때문에 performance와 speed가 증가** 됩니다. (npm은 순차적)
  - 안정성(stability)/보안성(security)
    - npm은 패키지가 설치될 때 자동으로 코드와 의존성을 실행할 수 있도록 허용했습니다. 
      - 이 특징은 편리한 기능이지만 안정성을 위협할 수 있습니다. 
      - 특히나 보장된 정책 없이 등록한 패키지가 존재할 수 있다는 점에서 더욱 위험도가 높습니다.
    - 반면 **yarn은 yarn.lock이나 package.json으로 부터 설치만 하며, yarn.lock은 모든 디바이스에 같은 패키지를 설치하는 것을 보장하기 때문에 버전의 차이로 인해 생기는 버그를 방지해줄 수 있습니다.**

- yarn 명령어
  - yarn init : package.json 생성
  - yarn or yarn install : package.json 파일 및 해당 종속성에 나열된 모든 모듈을 설치
  - yarn add package_name@버전 : 특정 패키지의 특정 버전 설치
  - yarn add 주소 : 특정 저장소 내 패키지 설치. 주로 github을 이와 같이 설치합니다.
  - yarn global add package_name : 옵션. 글로벌로 설치. 로컬의 다른 프로젝트도 이 패키지를 사용 가능하게 됩니다.
  - yarn remove : 패키지 삭제 명령어입니다.
  - yarn upgrade : 설치한 패키지들을 업데이트해줍니다.
  - npm dedupe : 중복 설치된 패키지들을 정리해주는 명령어입니다.