# :yum:  JSP 문법 정리  :yum:  





## JSP 페이지의 구성 요소



### [JSP 페이지의 구성 요소]

- **디렉티브(Directive)**
- **스크립트: 스크립틀릿(Scriptlet), 표현식(Expression), 선언부(Declaration)**
- **표현 언어(Expression Language)**
- **기본 객체(Implicit Object)**
- **정적인 데이터**
- **표준 액션 태그(Action Tag)**
- **커스텀 태그(Custom Tag)와 표준 태그 라이브러리(JSTL)**





### [디렉티브(Directive)]

**디렉티브(Directive)**: JSP 페이지에 대한 **설정 정보**를 지정

```jsp
[구문]
<%@ 디렉티브명 속성1="값1" 속성2="값2" ... %>

[EX]
<%@ page contentType="text/html;charset=UTF-8" %>
```





- **JSP가 제공하는 디렉티브**

| 디렉티브 (Directive) | 설명                                                         |
| -------------------- | ------------------------------------------------------------ |
| **page**             | JSP 페이지에 대한 정보를 지정한다. JSP가 생성하는 문서의 타입, 출력 버퍼의 크기, 에러 페이지 등 JSP 페이지에서 필요로 하는 정보를 설정한다. |
| **taglib**           | JSP 페이지에서 사용할 태그 라이브러리를 지정한다.            |
| **include**          | JSP 페이지의 특정 영역에 다른 문서를 포함시킨다.             |







### [스크립트 요소 3가지]

**스크립트 요소**: JSP에서 문서의 내용을 **동적으로 생성**하기 위해 사용되는 것.

- **스크립틀릿(Scriptlet)**: 자바 코드를 실행한다.
- **표현식(Expression)**: 값을 출력한다.
- **선언부(Declaration)**: 자바 메서드(함수)를 만든다.





#### 스크립틀릿(Scriptlet): 자바 코드 실행

```jsp
[구문]
<%
	자바코드1;
	자바코드2;
	자바코드3;
	...
%>	
```





#### 표현식(Expression): 값 출력

```jsp
[구문]
<%= 값 %>
```





#### 선언부(Declaration): 메서드 선언

```jsp
[구문]
<%!
	public 리턴타입 메서드명(파라미터목록) {
		자바코드1;
		자바코드2;
		...
		자바코드n;
		return 값;
	}
%>
```











### [기본 객체]

**기본 객체**: 웹 애플리케이션 프로그래밍을 하는 데 필요한 기능을 제공해주는 객체들.



- **JSP가 제공하는 기본 객체**

| 기본 객체       | 실제 타입                              | 설명                                                        |
| --------------- | -------------------------------------- | ----------------------------------------------------------- |
| **request**     | javax.servlet.http.HttpServletRequest  | 클라이언트의 요청 정보를 저장                               |
| **response**    | javax.servlet.http.HttpServletResponse | 응답 정보를 저장                                            |
| **pageContext** | javax.servlet.jsp.PageContext          | JSP 페이지에 대한 정보를 저장                               |
| **session**     | javax.servlet.http.HttpSession         | HTTP 세션 정보를 저장                                       |
| **application** | javax.servlet.ServletContext           | 웹 애플리케이션에 대한 정보를 저장                          |
| **out**         | javax.servlet.jsp.JspWriter            | JSP 페이지가 생성하는 결과를 출력할 때 사용하는 출력 스트림 |
| **config**      | javax.servlet.ServletConfig            | JSP 페이지에 대한 설정 정보를 저장                          |
| **page**        | java.lang.Object                       | JSP 페이지를 구현한 자바 클래스 인스턴스                    |
| **exception**   | java.lang.Throwable                    | 익셉션 객체. 에러 페이지에서만 사용                         |









### [표현 언어]

**표현 언어:** JSP 스크립트 태그(<%= ... %>)를 대신하여 JSP 값들을 좀더 편리하게 출력하기 위해 제공되는 언어 

ex)  <%= hello %>   --->   **${hello}**







### [표준 액션 태그와 태그 라이브러리]

**액션 태그:** 스크립트 요소를 사용하지 않고(HTML 태그 형태로) 다른 페이지의 Servlet이나 자바빈의 객체에 접근할 수 있도록 태그를 이용해 구현된 기능

**커스텀 태그: **개발자가 직접 정의할 수 있는 태그

**JSTL(JavaServerPages Standard Tag Library): **자주 사용되는 필요한 기능들을 모아놓은 커스텀 태그 라이브러리











### [JSP 주석]



- **스크립틀릿(Scriptlet)**과 **선언부(Declaration)**의 코드 블록은 자바 코드이므로 <u>자바의 주석 사용 가능</u>

```jsp
<%
	// 자바 8의 스트림을 이용한 1부터 100까지의 합 구하기
	int sum = IntStream.range(1, 101).sum();
%>
```



- **JSP 코드 자체 주석**

```jsp
<%-- 설명이 들어 온다. -- %>
```









## Tomcat 폴더 구성과 URL 매핑

**Servlet/JSP** 규약은 웹 애플리케이션이 <u>특정 폴더 구조</u>를 따르도록 제한하고 있다.



**Servlet/JSP**로 구성된 웹 애플리케이션의 기본 폴더 구조는 아래와 같다.

```
└── 톰캣 설치 경로
	└── webapps
		└── 웹애플리케이션
			├── jsp 파일들
			├── META-INF
			│   └── context.xml
			└── WEB-INF
				├── lib
				│	└── jar 파일들
				├── classes
				│	└── class 파일들
        		└── web.xml        
```



- **WEB-INF**: 웹 애플리케이션 설정 정보를 담고 있는 **web.xml** 파일이 위치
- **WEB-INF/classes**: 웹 애플리케이션에서 사용하는 **클래스(.class)** 파일이 위치
- **WEB-INF/lib**: 웹 애플리케이션에서 사용하는 **jar** 파일이 위치



WEB-INF 폴더와 그 하위 폴더를 제외한 나머지 폴더에는 웹 애플리케이션에서 사용할 JSP, HTML 이미지 등의 파일이 위치한다.





톰캣에서 웹 애플리케이션은 **[톰캣]/webapps** 폴더에 위치한다. **[톰캣]/webapps** 폴더에 있는 하위 폴더는 자동으로 웹 애플리케이션에 포함된다. 각 폴더의 이름은 웹 애플리케이션을 실행할 때 사용되는 **URL**과 관련이 있다. 



예를 들어, **webapps/app** 폴더에 위치한 애플리케이션을 실행할 때 사용하는 URL은 

`https://localhost:8080/app/example.jsp`  이런식이다.

위의 주소를 보면 URL의 경로가 **/app**으로 시작하는데, 경로 이름이 웹 애플리케이션 폴더의 이름과 같은 것을 알 수 있다. 



즉, 다음과 같은 관계가 있다.

- **[톰캣]/webapps/[웹경로]    ---->    `http://host:port[/웹경로]`**





톰캣의 **webapps** 폴더에는 **ROOT**라는 특수 폴더가 존재한다. 이 폴더는 **루트 웹 애플리케이션**을 위한 특수 폴더이다. 

```
[EX]
webapps/chap02   -->  http://localhost:8080/chap02
webapps/chap03   -->  http://localhost:8080/chap03
webapps/chap04   -->  http://localhost:8080/chap04
webapps/ROOT     -->  http://localhost:8080/
```



`http://localhost:8080/chap04`에서 웹 애플리케이션 경로에 해당하는 **/chap04**를 **컨텍스트 경로(Context Path)**라고 한다.

루트 웹 애플리케이션의 경우 **context path**는 빈 문자열("")이다.





**URL의 첫 번째 경로와 일치하는 context path를 가진 웹 애플리케이션이 존재하지 않으면**, <u>루트 웹 애플리케이션이 요청을 처리</u>한다. 

[ex] `http://localhost:8080/view/loginForm.jsp`라는 URL을 입력했을 때, **/view**를 **context path**로 갖는 웹 애플리케이션이 존재하지 않으면, 즉 **webapps/view** 폴더가 존재하지 않으면, **루트 웹 애플리케이션** 폴더인 **ROOT**를 기준으로 **/view/loginForm.jsp** 파일을 찾는다. 즉 **webapps/ROOT/view/loginForm.jsp** 파일을 찾아 실행한다.

























