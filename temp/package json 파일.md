## package.json

> [모두 알지만 모두 모르는 package.json](https://programmingsummaries.tistory.com/385)
>
> [구름](https://edu.goorm.io/learn/lecture/557/%ED%95%9C-%EB%88%88%EC%97%90-%EB%81%9D%EB%82%B4%EB%8A%94-node-js/lesson/174371/package-json)

- package.json은 프로젝트 정보와 의존성(dependencies)을 관리하는 문서입니다.

- package.json 파일을 작성할 때는 JS 객체 리터럴이 아니라 올바른 **<u>JSON 포맷</u>**이어야 한다.

- 직접 작성할 수도 있고 `npm init` 명령을 통해서 자동으로 생성할 수도 있다.

- package.json에서 가장 중요한 항목은 "name"과 "version"이다.

  - 필수로 입력되어야 하며, 이 항목들이 누락되면 당신의 패키지는 설치할 수 없다.
  - name과 version을 통해 각 패키지의 고유성을 판별하게 된다.
  - 따라서 패키지의 내용을 변경하려면 version을 변경해야만 한다.

  

```json
{
	"name" : "test",
  "version" : "1.1.6",
	"description" : "javascript's test programming.",
	"keywords" : ["util", "f", "server", "client", "browser"],
	"author" : "Goorm",
	"contributors" : [],
	"dependencies" : [],
	"repository" : {"type": "git", "url" : "git://gitbub.com/documentcloud/test.git" },
	"main" : "test.js"
}
```



| Key             | Value                                                        |
| --------------- | ------------------------------------------------------------ |
| name            | **프로젝트 이름**으로, 가장 중요합니다. <br />중앙 저장소에 배포할 때 version과 함께 <u>필수 항목</u>입니다. <br />url로 사용되고, 설치할 때 디렉토리 이름이 되기 때문에 url이나 디렉터리에서 쓸 수 없는 이름을 사용하면 안 됩니다. <br />또한, 이름에 node나 js가 들어가면 안 됩니다. <br />name은 214자보다 짧아야 하며, 점(.)이나 밑줄(_)로 시작할 수 없습니다. <br />대문자를 포함해서는 안 되며, require() 함수의 인수로 사용되며 짧고 알기 쉬운 것으로 짓는 것이 좋습니다. |
| version         | 프로젝트 버전을 정의합니다. 3단계 버전을 사용하며, - 로 태그 이름을 적을 수 있습니다. <u>필수 항목</u>이다. |
| description     | 프로젝트 설명으로, 문자열로 기술합니다. npm search로 검색된 리스트에 표시되기 때문에 사람들이 패키지를 찾아내고 이해하는 데 도움이 됩니다. |
| keywords        | 프로젝트를 검색할 때 참조되는 키워드입니다. description과 마찬가지로 npm search로 검색된 리스트에 표시됩니다. |
| homepage        | 프로젝트 홈페이지 주소입니다. url 항목과는 다르며, url을 설정하면 예상치 못한 움직임을 하게 되므로 주의합니다. |
| author          | 프로젝트 작성자 정보로, 한 사람만을 지정합니다. JSON 형식으로 name, email, url 옵션을 포함합니다. |
| contributors    | 프로젝트에 참여한 공헌자 정보로, 여러 사람을 배열로 지정할 수 있습니다. |
| repository      | 프로젝트의 소스 코드를 저장한 저장소의 정보입니다.소스 코드에 참여하고자 하는 사람들에게 도움이 될 수 있습니다. 프로젝트의 홈페이지 url을 명시해서는 안 됩니다. |
| scripts         | 프로젝트에서 자주 실행해야 하는 명령어를 scripts로 작성해두면 npm 명령어로 실행 가능합니다. |
| config          | 소스 코드에서 config 필드에 있는 값을 환경 변수처럼 사용할 수 있습니다. |
| private         | 이 값을 true로 작성하면 중앙 저장소로 저장하지 않습니다.     |
| dependencies    | 프로젝트 의존성 관리를 위한 부분입니다. 이 프로젝트가 어떤 확장 모듈을 요구하는지 정리할 수 있습니다.일반적으로 package.json에서 가장 많은 정보가 입력되는 곳입니다.애플리케이션을 설치할 때 이 내용을 참조하여 필요한 확장 모듈을 자동으로 설치합니다.따라서 개발한 애플리케이션이 특정한 확장 모듈을 사용한다면 여기에 꼭 명시를 해주어야 합니다.또한, npm install 명령은 여기에 포함된 모든 확장 모듈들을 설치하게 되어 있습니다. |
| devDependencies | 개발할 때만 의존하는 확장 모듈을 관리합니다.                 |
| engine          | 실행 가능한 노드 버전의 범위를 결정합니다.                   |

