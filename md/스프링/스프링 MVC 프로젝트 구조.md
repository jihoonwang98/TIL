# 스프링 MVC 프로젝트 구조

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=http%3A%2F%2Fcfile4.uf.tistory.com%2Fimage%2F99CD653F5C038957183EE1)



- **src/main/java**: 자바 소스파일들이 위치해 있는 디렉터리. 자바로 작성된 **Controller, Service, DAO** 파일들이 위치해 있다.
- **src/main/resources**: **자원 파일**을 위치시키는 디렉터리. **자원 파일**은 클래스 패스에 위치할 **자원 파일**을 의미하며, 스프링에서는 설정 **XML** 파일이나 **properties** 파일 등을 위치시킨다.
- **src/main/webapp**: 웹과 관련된 파일들이 위치해 있는 디렉터리. **html, css, js, jsp** 파일 등이 위치해 있다. 또한 웹 어플리케이션 구동에 필요한 XML 설정 파일들이 위치해 있다.
- **src/main/webapp/WEB-INF/spring**: 스프링 설정 파일들이 있는 디렉터리
- **src/main/webapp/WEB-INF/views:** jSP 파일의 경로
- **pom.xml:** Maven 설정 파일