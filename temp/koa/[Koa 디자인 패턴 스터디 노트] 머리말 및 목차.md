# [Koa 디자인 패턴 스터디 노트] 머리말 및 목차

> https://chenshenhai.com/koajs-design-note/



### 머리말

- Koa.js는 다음 두 가지 기능만 제공하는 매우 간소화된 웹 프레임워크입니다.
- HTTP 서비스
  - HTTP 요청 처리
  - HTTP 응답 응답 처리
- 미들웨어 컨테이너
  - 미들웨어 로딩
  - 미들웨어 실행



- 다른 웹 서비스에 필요한 나머지 기능은 개발자의 요구에 따라 사용자 정의되고 개발되므로 유연성의 여지가 많이 남아 있고 웹 서비스의 개발 비용이 증가합니다. 
  - 내가 알기로는 Koa.js의 유연성으로 인한 개발 비용은 다음과 같은 두 가지 유형이 있습니다.
    - 프레임 디자인
    - 미들웨어의 선택
  - 프레임워크의 디자인은 이 요소가 더 복잡하며 나중에 이를 설명하기 위해 새 책이 열립니다. 
    - 이 책은 주로 일반적으로 사용되는 Koa.js 미들웨어를 분석하고 관련 미들웨어의 기능적 원리와 구현 방법을 추상화하고 데모를 사용하여 독자가 원리를 이해하고 공식 소스 코드에 대한 의존도를 줄이며 "사람에게 물고기를 주기"를 달성하려고 합니다.



### 목차

- Koa.js 원칙
  - [1.1 연구 준비](https://chenshenhai.com/koajs-design-note/tree/master/note/chapter01/01)
  - [1.2 사용 약속](https://chenshenhai.com/koajs-design-note/tree/master/note/chapter01/02)
  - [1.3 비동기/대기의 사용](https://chenshenhai.com/koajs-design-note/tree/master/note/chapter01/03)
  - [1.4 Node.js 네이티브 http 모듈](https://chenshenhai.com/koajs-design-note/tree/master/note/chapter01/04)
  - [1.5 미들웨어 엔진](https://chenshenhai.com/koajs-design-note/tree/master/note/chapter01/05)
  - [1.6 일반적인 미들웨어 스타일의 HTTP 서비스 구현](https://chenshenhai.com/koajs-design-note/tree/master/note/chapter01/06)
  - [1.7 가장 간단한 Koa.js 구현](https://chenshenhai.com/koajs-design-note/tree/master/note/chapter01/07)
- Koa.js의 AOP 디자인
  - [2.1 AOP 측면 지향 프로그래밍](https://chenshenhai.com/koajs-design-note/tree/master/note/chapter02/01)
  - [2.2 양파 모형 절단면](https://chenshenhai.com/koajs-design-note/tree/master/note/chapter02/02)
  - [2.3 HTTP 측면 프로세스](https://chenshenhai.com/koajs-design-note/tree/master/note/chapter02/03)
- Koa.js 미들웨어
  - [3.1 미들웨어 분류](https://chenshenhai.com/koajs-design-note/tree/master/note/chapter03/01)
  - [3.2 좁은 미들웨어](https://chenshenhai.com/koajs-design-note/tree/master/note/chapter03/02)
  - [3.3 일반화된 미들웨어](https://chenshenhai.com/koajs-design-note/tree/master/note/chapter03/03)
- 좁은 미들웨어 요청/응답 가로채기
  - [4.1 코아 로거 구현](https://chenshenhai.com/koajs-design-note/tree/master/note/chapter04/01)
  - [4.2 koa-send 구현](https://chenshenhai.com/koajs-design-note/tree/master/note/chapter04/02)
  - [4.3 koa-static 구현](https://chenshenhai.com/koajs-design-note/tree/master/note/chapter04/03)
- 좁은 미들웨어 컨텍스트 에이전트
  - [5.1 koa-view 구현](https://chenshenhai.com/koajs-design-note/tree/master/note/chapter05/01)
  - [5.2 koa-jsonp 구현](https://chenshenhai.com/koajs-design-note/tree/master/note/chapter05/02)
  - [5.3 koa-bodyparser 구현](https://chenshenhai.com/koajs-design-note/tree/master/note/chapter05/03)
- 일반화 미들웨어 간접 미들웨어 처리
  - [6.1 코아 라우터 구현](https://chenshenhai.com/koajs-design-note/tree/master/note/chapter06/01)
  - [6.2 코아 마운트 구현](https://chenshenhai.com/koajs-design-note/tree/master/note/chapter06/02)

