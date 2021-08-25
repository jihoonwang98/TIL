## 08 JSP 프로그래밍



### 8.1 JSP란?

JSP(JavaServer Pages)는 HTML, XML과 같은 동적 웹 콘텐츠를 생성하는 애플리케이션을 만들기 위한 J2EE 플랫폼에 속하는 자바 기술입니다. 



#### 8.1.1 JSP 개념

동적인 콘텐츠를 만들 때는 어떠한 형태로든 "콘텐츠를 어떻게 생성할지"를 지시하는 프로그래밍이 필요합니다. 그런데 서블릿처럼 프로그램 소스 안에 HTML 태그를 처리하면, 변경이 일어날 때마다 매번 컴파일해줘야 해서 동적인 콘텐츠를 만들기 어렵습니다. 

JSP 기술은 동적으로 콘텐츠를 생성하기 위해 프로그래밍 코드가 담긴 **스크립트**를 포함할 수 있게 하고, HTML과 유사한 태그를 통해 어려운 자바 코딩 없이도 자바 객체를 사용할 수 있게 합니다.





JSP 기술은 다음과 같은 개념을 기반으로 만들어졌습니다.



**템플릿 데이터**

대부분 동적 웹 콘텐츠를 이루는 많은 부분은 고정되어 있거나 템플릿(Template) 데이터 입니다. 템플릿 데이터는 전형적으로 텍스트나 XML 조각, HTML 태그들일 수 있습니다. JSP 기술은 이러한 템플릿 데이터들이 변형되지 않도록 처리하는 방법을 지원합니다.



```html
<html>
<body>
	<h1>hello world!</h1>
</body>
</html>
```



JSP에서 이러한 템플릿 데이터 부분은 해석하지 않고 그대로 출력해줍니다. 서블릿은 HTML 코드 부분을 `out.print("<html>")`과 같이 출력해줘야 하지만, JSP는 프로그램적인 명령문들만 컨테이너가 해석해서 처리하고, HTML 태그 부분은 그대로 HTML로 처리되므로 별도의 명령문들로 처리할 필요가 없습니다.





**동적인 데이터의 추가**

JSP 기술은 템플릿 데이터에 동적인 데이터를 끼워 넣을 수 있는 간단하지만 강력한 방법을 제공합니다.



```jsp
<html>
<body>
	<h1>hello world!</h1>
    <%= request.getParameter("name") %>
</body>
</html>
```



위 코드는 name이라는 질의 문자열을 추출하여 출력하는 코드입니다. JSP에서 제공하는 내장 객체 request를 이용해서 쉽게 동적인 데이터를 추가할 수 있습니다. 이처럼 JSP에서는 템플릿 데이터와 함께 출력을 쉽게 할 수 있는 '표현식(expression)'이라는 스크립팅 요소를 제공하고 있으며, JSP 2.0은 좀 더 확장된 기능으로 EL(Expression Language)이라는 것도 제공하고 있어서 동적인 콘텐츠를 쉽게 작업할 수 있습니다.





**기능의 추상화**

추상화는 객체지향 프로그램에서 사용하는 용어로서 세부 구현은 숨기고 기능을 사용할 수 있도록 구현하는 것을 의미하며, 재사용성을 높이는 기술입니다. JSP 기술은 이런 기능의 추상화를 위해 다음 두 가지 메커니즘을 제공합니다.



- **자바빈즈(JavaBeans) 컴포넌트 아키텍처**: 컴포넌트라는 개념은 규격에 맞게 조각들을 만들어 놓고 조각들을 이용해 완성된 하나의 제품을 만들자는 것인데요. 자바에서도 규격에 맞는 조각이 있는데 이것을 자바빈즈(JavaBeans)라고 하며, JSP에서 사용하는 컴포넌트는 JSP 자바빈즈 라고 합니다.



- **태그 라이브러리**: 자주 사용하는 기능을 매번 구현하는 것이 아니라, JSP 태그로 만들어 사용한다면 한 번의 작성으로 여러 곳에서 사용할 수 있어 재사용성을 높일 수 있습니다.





#### 8.1.2 JSP 장점

JSP 기술은 다음과 같은 장점을 제공합니다.



- **Write Once, Run Anywhere properties**

- **역할 분리**
- **컴포넌트와 태그 라이브러리의 재사용**
- **정적 콘텐츠와 동적 콘텐츠의 분리**
- **액션, 표현식, 스크립팅 제공**
- **N-tier 엔터프라이즈 애플리케이션을 위한 웹 접근 레이어**





### 8.2 JSP 동작 원리

JSP는 응답정보를 만들기 위해 요청을 어떻게 처리할 것인가를 명세한 태그 기반의 문서입니다. JSP 내에는 템플릿 데이터와 동적인 기능을 담당하는 액션들이 혼합되어 있으며, JAVA2 기반에서 동작합니다.



JSP 기술은 다양한 방법으로 동적인 콘텐츠를 작성할 수 있도록 지원하는데요. 다음은 JSP 기술이 지원하는 주요 항목으로 JSP를 작성할 때 사용합니다.



- **표준 지시자(Standard Directives)**
- **표준 액션(Standard Actions)**
- **스크립팅 요소(Scripting Element)**
- **태그 확장 메커니즘(Tag Extension Mechanism)**
- **템플릿 콘텐츠(Template Content)**





**웹 애플리케이션**

웹 애플리케이션의 개념은 서블릿 스펙에서 상속되었습니다. 웹 애플리케이션은 다음과 같은 항목으로 구성될 수 있습니다.



- 서버상에서 동작하는 자바 런타임 환경
- 요청을 처리하고 동적 콘텐츠를 생성하는 JSP
- 요청을 처리하고 동적 콘텐츠를 생성하는 서블릿
- 서버 측 자바빈즈 컴포넌트
- HTML, DHTML, XHTML, XML 등의 페이지
- 클라이언트 측 자바 애플릿, 자바빈즈 컴포넌트, 자바 클래스 파일들
- 클라이언트 측에서 동작하는 자바 런타임 환경





**컴포넌트와 컨테이너**

컨테이너는 JSP와 서블릿 클래스를 웹 컴포넌트로 인식합니다. JSP의 수행 원리에서 배우겠지만, 요청된 JSP는 컨테이너에 전달되고 컨테이너는 해당 JSP를 해석하며, 해석된 결과물이 실제 서비스를 제공합니다. 컴포넌트와 컨테이너를 분리하는 것은 컨테이너가 제공하는 서비스를 통해 컴포넌트의 재사용을 가능하게 해주기 때문입니다.





**변환과 실행**

스크립트 언어는 실행되기 전에, 먼저 실행 가능한 코드로 번역됩니다. JSP는 컨테이너가 해석하는 텍스트 형태의 컴포넌트입니다. 실제 JSP의 실행은 변환 단계와 요청 단계로 구분합니다.



- **변환(Translation)**: 컨테이너는 JSP를 해석하여 하나의 서블릿 소스로 만든 다음에 해당 소스를 컴파일합니다. 그러면 서블릿 클래스 파일이 생성되는데요. 이 서블릿 클래스는 JSP가 실행될 수 있는 형태로 구현된 JSP 구현 클래스입니다. 이러한 변환 과정은 웹 컴포넌트가 배치되는 시점이나 해당 페이지에 대한 최초 요청이 있을 때 컨테이너가 수행합니다.



- **실행(Execution)**: 실행은 요청이 있을 때마다 발생합니다. 컨테이너는 서블릿으로 변환되어 컴파일된 구현 서블릿 클래스를 초기화하고, 이 서블릿 클래스를 통해 요청을 처리하고 응답합니다.



클라이언트가 웹 브라우저를 통해 JSP를 요청하였을 때, 구체적으로 JSP의 변환과 실행 과정이 어떻게 이루어지는지 알아보겠습니다.



![](https://t1.daumcdn.net/cfile/tistory/270B054A57C689B128)



**1)** 개발자는 HTML 태그와 JSP 태그를 사용하여 페이지를 작성한 후 확장자를 .jsp로 저장합니다. 태그를 사용하여 서블릿을 간단하게 만들 수 있는 기술이 JSP지만, 어쨋든 JSP도 서블릿이므로 서블릿으로서 동작합니다.



**2)** 클라이언트로부터 JSP 요청이 들어오면, JSP 컨테이너는 태그로 만들어진 JSP 파일을 완벽한 자바 소스로 변환하여 *.java 파일로 만듭니다.



**3)** JSP 컨테이너는 *.jsp 파일을 변환한 *.java 파일을 컴파일하여 *.class 파일을 만듭니다.



**4)** 컴파일된 자바 실행 파일은 서블릿 컨테이너에 의해 서블릿으로서 동작합니다.



**5)** 변환과 컴파일 작업은 최초의 요청이나 JSP가 변경되었을 때만 수행됩니다.





#### 8.2.2 자바 서블릿 소스

example1.jsp 파일이 변환된 자바 소스에서 주요 내용을 살펴본다.



**_jspService() 메서드**

```java
// example1.jsp
public void _jspService(final javax.servlet.http.HttpServletRequest request,
                        final javax.servlet.http.HttpServletResponse response)
    					throws java.io.IOException, javax.servlet.ServletException {
    // 생략
}
```



_jspService() 메서드는 JSP가 실행될 때마다 자동으로 호출되는 메서드입니다. 서블릿에서 서블릿이 실행될 때마다 호출되던 service() 메서드와 같다고 생각하면 됩니다. 





**공통 코드**

모든 JSP가 자바 소스로 변환될 때 _jspService() 메서드가 삽입되는데, 이때 _jspService() 메서드 내에 항상 삽입되는 코드가 있습니다.

```java
// example1.jsp
public void _jspService(final javax.servlet.http.HttpServletRequest request,
                        final javax.servlet.http.HttpServletResponse response)
    					throws java.io.IOException, javax.servlet.ServletException {
    // 생략
    
    final javax.servlet.jsp.PageContext pageContext;
    javax.servlet.http.HttpSession session = null;
    final javax.servlet.ServletContext application;
    final javax.servlet.ServletConfig config;
    javax.servlet.jsp.JspWriter out = null;
    final java.lang.Object page = this;
    javax.servlet.jsp.JspWriter _jspx_out = null;
    javax.servlet.jsp.PageContext _jspx_page_context = null;
    
    try {
        response.setContentType("text/html");
        pageContext = _jspxFactory.getPageContext(this, request, response, null, true, 8192, true);
        _jspx_page_context = pageContext;
        application = pageContext.getServletContext();
        config = pageContext.getServletConfig();
        session = pageContext.getSession();
        out = pageContext.getOut();
        _jspx_out = out;
    }
}
```

































