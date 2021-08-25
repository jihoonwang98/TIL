# Thymeleaf Layout 만들기



## 의존성 설정



**pom.xml에 추가**

```xml
<dependency>
	<groupId>nz.net.ultraq.thymeleaf</groupId>
	<artifactId>thymeleaf-layout-dialect</artifactId>
</dependency>
```





## 속성 정리

- **th:block** 속성:

  리액트의 fragment처럼, 안에 내용만 남고 자기 자신은 사라진다.

  여기서는 타임리프가 처리한 내용물만 남고 자기 자신의 태그는 브라우저에 렌더링되지 않는다.

  

- **th:fragment** 속성:

  여러 페이지에서 필요에 따라 포함시킬 코드 영역을 정의할 때 사용할 수 있다.

  

  사용 예시:

  - 레이아웃 영역 중 override 가능한 영역을 설정
  - 여러 페이지에서 공통적으로 사용할 코드 영역 설정

- **layout:fragment** 속성:



- **layout:decorator** 속성:

  html 태그 내에 들어간다.

  layout:decorator 속성에는 사용하고자 하는 레이아웃 파일 경로를 작성한다.

  이때, 경로 위치는 thymeleaf 템플릿 prefix로 설정한 경로를 기준으로 한다.

  (default prefix는 classpath:/templates/ )



## Layout 설정 순서

- Layout 템플릿 만들고 재정의할 컨텐츠 요소는 `layout:fragment`로 정의
- Layout 템플릿을 `layout:decorator`를 이용하여 상속받고, 컨텐츠 요소만 override
- 중복적으로 추가할 블럭이나, link, script 태그와 같은 요소를 `th:insert`로 삽입





### 1. layout.html 레이아웃 파일 만들기



