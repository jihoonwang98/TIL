## 07 고급 기능



### 7.1 필터(Filter)

필터는 클라이언트로부터 서블릿이 요청되어 수행될 때 필터링 기능을 제공하기 위한 기술입니다. **필터가 수행되는 시점**은 <u>요청된 서블릿이 수행되기 전과 후</u>이며, 필터 기능을 사용하여 서블릿의 처리와 유지 보수를 더욱 효과적으로 할 수 있습니다. 이 절에서는 필터의 기능과 구현방법 그리고 처리 메커니즘을 학습합니다.



#### 7.1.1 필터 개요

**필터**란 서블릿 2.3부터 추가된 기능으로 **서블릿이 수행되기 전이나 후에 추가 기능을 수행**할 수 있습니다. 필터 기능을 구현하려면 서블릿과는 별도의 객체로 분리해야 하며, 이때는 `javax.servlet.Filter`를 상속받아야 합니다. Filter를 상속받은 객체에 클라이언트로부터 요청받은 페이지 실행 전이나 후에 필터링할 내용을 구현합니다. 그리고 구현된 필터가 어떤 페이지를 필터링하는지 웹서버에 설정합니다. 그래서 클라이언트는 필터가 존재하는지 알지 못합니다.



다음은 필터로 구현할 수 있는 기능에 대한 목록입니다.

- 서블릿이 호출되기 전에 서블릿 요청을 가로채는 기능
- 서블릿이 호출되기 전에 요청 내용을 점검하는 기능
- 요청 헤더의 수정과 조정 기능
- 서블릿이 호출된 후에 서블릿 응답을 가로채는 기능
- 응답 헤더의 수정과 조정 기능



필터는 클라이언트로부터 요청된 페이지가 실행될 때마다 자동으로 함께 실행됩니다.



필터 기능을 활용하여 처리하는 기능 중에 대표적인 것이 '로그 기록'과 '한글 처리'입니다. 어떤 페이지가 실행될 때마다 그 페이지 실행에 관한 내용, 즉 어떤 클라이언트가 언제, 얼마 동안이나 실행했는지를 나타내는 로그 기록은 각 서블릿의 내용과는 관련이 없으므로 분리하여 구현하는 것이 유지보수 면에서 효율적입니다.



한글 처리는 여러 페이지에서 수행해야 하는 작업입니다. 한글 처리를 필터를 활용해서 수행하면 한글을 처리하는 필터를 하나 만들고, 여러 개의 서블릿에 대하여 필터링하도록 합니다. 그러면 한글 처리를 각 서블릿마다 일일이 작업해야 했던 것을 한 번의 작업으로 처리할 수 있습니다.



필터로 구현할 수 있는 기능들을 살펴보면 **인증, 로그, 오류 처리, 데이터 압축, 변환** 그리고 **응답 내용 추가** 기능 등이 있습니다.





#### 7.1.2 필터 구현: Filter 인터페이스

필터를 구현할 때는 `javax.servlet.Filter` 인터페이스를 상속받고, `Filter` 객체에 있는 모든 메소드를 재정의합니다. `Filter` 객체가 선언하고 있는 메소드는 다음과 같습니다. 이 메소드들은 모두 콜백(callback) 메소드로서 특정한 상황이 발생하면 서버가 자동으로 호출합니다.



- `init(FilterConfig)`

  **필터 객체가 생성될 때 호출**되는 메소드입니다. 필터 객체는 웹 애플리케이션 서비스가 올라가면서, 즉 웹서버가 시작될 때 한 번만 생성된 다음, 계속 사용되므로 init() 메소드는 서버가 시작될 때 **한 번만** 호출됩니다. 따라서 init() 메소드에는 주로 초기화 기능을 구현합니다.



- `destroy()`

  **필터 객체가 삭제될 때 호출**되는 메소드입니다. 따라서 destroy() 메소드에는 주로 자원 해제 기능을 구현합니다.



- `doFilter(ServletRequest, ServletResponse, FilterChain)`

  필터를 구현하는 목적은 어떤 서블릿을 필터링하기 위해서입니다. doFilter() 메소드는 **필터링 설정한 서블릿을 실행할 때마다 호출**되는 메소드로서 실제 **필터링 기능을 구현**하는 메소드입니다. init() 메소드는 필터 객체 생성 시 한 번, destory() 메소드는 필터 객체 삭제 시 한 번만 호출되지만, doFilter() 메소드는 필터링 설정한 **서블릿이 실행될 때마다** 호출됩니다.





다음과 같은 내용으로 2개의 필터 객체를 작성하겠습니다.



```java
//FlowFilterOne.java
package com.edu.test;

import java.io.IOException;

import javax.servlet.*;

public class FlowFilterOne implements Filter{
	
	@Override
	public void init(FilterConfig filterConfig) throws ServletException {
		System.out.println("init() 호출...one");
	}

	@Override
	public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
			throws IOException, ServletException {
		System.out.println("doFilter() 호출 전...one");
		chain.doFilter(request, response);
		System.out.println("doFilter() 호출 후...one");
	}

	
	@Override
	public void destroy() {
		System.out.println("destroy() 호출...one");
	}
}
```



```java
//FlowFilterTwo.java
package com.edu.test;

import java.io.IOException;

import javax.servlet.*;

public class FlowFilterTwo implements Filter{
	
	@Override
	public void init(FilterConfig filterConfig) throws ServletException {
		System.out.println("init() 호출...two");
	}

	@Override
	public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
			throws IOException, ServletException {
		System.out.println("doFilter() 호출 전...two");
		chain.doFilter(request, response);
		System.out.println("doFilter() 호출 후...two");
	}

	
	@Override
	public void destroy() {
		System.out.println("destroy() 호출...two");
	}
}
```





#### 7.1.3 필터 등록: \<filter> 태그

Filter 인터페이스를 상속받아 필터 객체를 구현한 다음에는 구현된 필터를 서버에 등록해야만 동작합니다. 서버에 필터를 등록하는 방법은 web.xml 파일에 다음과 같은 태그를 사용합니다.



**[필터 등록 태그 구조]**

```xml
<filter>
	<filter-name> </filter-name>
	<filter-class> </filter-class
	<init-param>
		<param-name> </param-name>
		<param-value> </param-value>
	</init-param>
</filter>
```



- **\<filter>**: `javax.servlet.Filter` 인터페이스를 상속받고 있는 객체를 등록하는 태그. 웹 애플리케이션 서비스가 준비되면서 웹서버가 web.xml 파일에서 \<filter> 태그를 읽으면 해당하는 객체를 찾아가 객체를 생성합니다. 그래서 필터 객체가 생성되는 시점은 웹 애플리케이션 서비스가 시작될 때입니다.



- **\<filter-name>**: 등록하는 필터의 논리적인 이름을 정합니다. 이름은 개발자가 임의로 정하는 이름으로서 서버에 필터를 등록하는 이름입니다. \<filter> 태그의 필수 하위 태그입니다.



- **\<filter-class>**: 등록하는 필터의 클래스 이름을 지정합니다. 웹서버는 \<filter-class>에 등록된 클래스를 찾아가 객체를 생성하기 때문에 실제 클래스 이름을 패키지 이름과 함께 정확하게 지정해야 객체가 올바르게 생성됩니다. \<filter> 태그의 필수 하위 태그입니다.



- **\<init-param>**: web.xml에서 필터 객체에 변수를 전달할 때 사용하는 태그입니다. \<filter> 태그의 선택 하위 태그입니다.



- **\<param-name>**: 필터 객체에 전달하고자하는 변수의 이름을 지정합니다. \<init-param> 태그의 필수 하위 태그입니다.



- **\<param-value>**: 필터 객체에 전달하려는 변숫값을 지정합니다. \<param-value>는 \<init-param> 태그의 필수 하위 태그입니다.





**[참고 web.xml 작성 시 태그 순서]**

**?**가 표시된 태그는 사용되지 않을 수도 있고 사용된다면 **한 번만** 나올 수 있는 태그란 의미이며, *****가 표시된 태그는 사용되지 않을 수도 있고 또는 **여러 번** 사용될 수 있는 태그란 의미입니다. 



```xml
<display-name?>
<description?>
<distributable?>
<context-param*>
<filter*>
<filter-mapping*>
<listener*>
<servlet*>
<servlet-mapping*>
<welcome-file-list?>
<error-page*>
```



앞에서 작성한 FlowFilterOne.java와 FlowFilterTwo.java 필터 객체를 각각 'flow1'과 'flow2'라는 이름으로 web.xml에 등록하겠습니다. web.xml에 다음 태그를 추가합니다.



```xml
~생략~
  <filter>
  	<filter-name>flow1</filter-name>
  	<filter-class>com.edu.test.FlowFilterOne</filter-class>
  </filter>
  
  <filter>
  	<filter-name>flow2</filter-name>
  	<filter-class>com.edu.test.FlowFilterTwo</filter-class>
  </filter>
~생략~
```



**[실행 결과]**



서버를 다시 재시작하면,



![image-20210312170625055](C:\Users\user\AppData\Roaming\Typora\typora-user-images\image-20210312170625055.png)



서버를 중지하면,



![image-20210312170642066](C:\Users\user\AppData\Roaming\Typora\typora-user-images\image-20210312170642066.png)



위와 같은 결과가 출력됩니다.





#### 7.1.4 필터 매핑: \<filter-mapping> 태그

Filter 인터페이스를 상속하여 필터 객체를 구현한 후 web.xml에 \<filter> 태그로 등록하면 필터 객체는 웹 애플리케이션 서비스가 시작될 때 객체가 생성되어 실행 준비를 완료합니다. 필터 객체는 클라이언트가 필터링 설정한 서블릿을 요청할 때만 실행됩니다. 이제 필터 작업의 마지막 단계로 **web.xml에 등록한 필터들이 어떤 서블릿을 필터링할 것인지 그 대상을 설정**하는 일만 남았습니다. 필터링 설정은 web.xml에 다음 태그를 사용하여 설정합니다.



**[필터 매핑 태그 구조]**

```xml
<filter-mapping>
	<filter-name> </filter-name>
	<url-pattern> </url-pattern>
</filter-mapping>

<filter-mapping>
	<filter-name> </filter-name>
	<servlet-name> </servlet-name>
</filter-mapping>
```



- **\<filter-mapping>**: 앞에서 \<filter> 태그로 등록된 필터가 어떤 서블릿을 필터링할 것인지 설정하는 태그



- **\<filter-name>**: 실행할 필터 이름을 지정합니다.



- **\<url-pattern>**: 필터링할 서블릿을 지정합니다. 서블릿을 지정할 때는 클라이언트가 요청하는 URL을 지정합니다. URL을 지정할 때는 전체 주소를 지정하는 것이 아니라, 웹 애플리케이션 이름까지는 생략한 후 다음부터 지정합니다.

  예를 들어, `http://localhost:8080/edu/second` 요청에 대하여 실행되는 페이지를 필터링하고자 한다면, 앞에는 생략하고 /second만 지정하면 됩니다. 어차피 현재 web.xml은 /edu 웹 애플리케이션에 대한 설정이므로 별도로 지정하지 않아도 고정되어 있기 때문입니다. 그리고 url로 필터링할 페이지를 지정하기 때문에 꼭 서블릿만 필터링할 수 있는 것은 아닙니다.



- **\<servlet-name>**:  필터링할 서블릿을 지정할 때 \<url-pattern> 태그로 클라이언트가 실행을 요청하는 url로 지정할 수 있지만, \<serlvet-name> 태그를 사용할 수도 있습니다. \<servlet-name>의 값은 \<servlet>의 \<servlet-name> 태그에서 지정된 값만을 사용할 수 있습니다. 







필터 매핑 예제를 작성해봅시다. web.xml에 다음과 같은 태그들을 추가합니다.

```xml
~생략~
  <filter-mapping>
  	<filter-name>flow1</filter-name>
  	<url-pattern>/second</url-pattern>
  </filter-mapping>
    
  <filter-mapping>
  	<filter-name>flow2</filter-name>
  	<url-pattern>/*</url-pattern>
  </filter-mapping>
~생략~
```





이제 필터 기능을 실습할 SecondServlet.java 파일을 작성합니다.

```java
package com.edu.test;

import java.io.*;

import javax.servlet.*;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.*;


@WebServlet("/second")
public class SecondServlet extends HttpServlet {

	protected void doGet(HttpServletRequest req, HttpServletResponse res) throws ServletException, IOException {
		System.out.println("SecondServlet!!");
		PrintWriter out = res.getWriter();
		out.print("<html><head><title>Test</title></head>");
		out.print("<body><h1>have a nice day!!</h1></body></html>");
		out.print("</html>");
		out.close();
	}
}
```



**[실행 결과]**



페이지에는 have a nice day!!가 출력되고, 콘솔창에는 아래와 같이 출력된다.



![image-20210312172057503](C:\Users\user\AppData\Roaming\Typora\typora-user-images\image-20210312172057503.png)





위와 같은 결과가 나온 이유을 알고자 실행 순서를 알아봅시다.



**1)** 클라이언트로부터 `http://localhost:8080/edu/second` 요청이 들어왔습니다.

**2)** web.xml의 \<url-pattern>/second\</url-pattern> 조건에 만족합니다. 그래서 flow1 필터의 doFilter() 메소드가 실행됩니다.

**3)** web.xml의 \<url-pattern>/*\</url-pattern> 조건에도 만족합니다. flow2 필터의 doFilter() 메소드가 실행됩니다.

**4)** web.xml에 더 이상 조건에 만족하는 \<filter-mapping>이 없습니다. 그러면 클라이언트가 요청한 서블릿이 실행됩니다.



이와 같은 실행 순서를 그림으로 표현하면 다음과 같습니다.



![image-20210312174425193](C:\Users\user\AppData\Roaming\Typora\typora-user-images\image-20210312174425193.png)





만약 flow1 -> flow2 순서가 아니라 flow2 -> flow1 순서로 실행하고 싶다면, web.xml의 \<filter-mapping> 태그의 순서를 바꾸면 됩니다. web.xml에 작성된 순서대로 조건을 검사하여 필터를 실행하기 때문입니다.



다음 그림은 클라이언트가 /second 요청을 했을 때 서버 측에서 실행되는 메소드들의 실행 순서를 표현한 것입니다.



![](https://docs.google.com/drawings/d/sIA5LOKazNIV6c1d1BkwIxw/image?parent=e/2PACX-1vT0HEa6CTBtK09CzJ_UFu4u7vIqrkXKcR8OhhUxAKBEH5I5mxqjqu38cY38DaGNifCsUty637jpQ4i5&rev=471&h=391&w=601&ac=1)

필터 객체를 구현하는 기본적인 구조는 다음과 같습니다.

```java
public class SampleFilter implemenets Filter {
	public void init(FilterConfig config) {
        // -> 필터 객체 생성 시 한 번 수행되는 초기화 코드
    }
    
    public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain) throws java.io.IOException, ServletException { 
    	// -> 서블릿 수행 전에 수행할 코드
        chain.doFilter(req, res);
        // -> 서블릿 수행 후에 수행할 코드
    }
    
    public void destroy() {
        // -> 필터 객체 해제 시 한 번 수행되는 해제 코드
    }
}
```





- **FilterChain**

  [메소드 사양] `void doFilter(ServletRequest request, ServletResponse response)`

  

  FilterChain은 필터가 실행될 때 doFilter() 메소드의 세 번째 인자로 전달되는 객체로서, **web.xml 파일에 설정한 모든 \<filter-mapping> 정보를 가지고 있습니다**. 즉, 클라이언트 요청에 대하여 필터들의 실행 순서를 알고 있는 객체이지요. 하나의 요청에 필터들이 실행되고, 서블릿이 실행될 때 처리 흐름을 제어할 수 있는 객체가 FilterChain입니다.





#### 7.1.5 한글 처리 필터

필터 기능으로 처리하는 대표적인 기능이 한글 처리입니다. 한글은 전달할 때와 전달받을 때 동일한 문자코드를 사용해야 한글이 깨지지 않습니다. 그래서 전달받은 페이지에서 한글 처리를 해야 하는데 매페이지마다 한글 처리를 하지 않고, 한글 처리 기능이 있는 필터를 구현한 다음, 한글 처리가 필요한 페이지들에서 필터를 사용하면 일괄적으로 처리할 수 있습니다.



필터를 이용해서 한글을 처리하는 예제를 작성해보겠습니다. 먼저 여러 페이지에서 적용되는 것을 확인하기 위해 입력 페이지를 3개 만들겠습니다.



```html
//input1.html
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>입력 페이지</title>
</head>
<body>
<form action="output1" method="post">
	이름: <input type="text" name="name">
	<input type="submit" value="확인">
</form>
</body>
</html>

//input2.html
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>입력 페이지</title>
</head>
<body>
<form action="output2" method="post">
	과목: <input type="text" name="subject">
	<input type="submit" value="확인">
</form>
</body>
</html>

//input3.html
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>입력 페이지</title>
</head>
<body>
<form action="output3" method="post">
	학과: <input type="text" name="dept">
	<input type="submit" value="확인">
</form>
</body>
</html>
```



입력 페이지에서 전달된 한글을 추출하여 출력하는 서블릿 페이지를 3개 작성하겠습니다. 

```java
//Output1Servlet.java
package com.edu.test;

import java.io.*;
import javax.servlet.*;
import javax.servlet.annotation.*;
import javax.servlet.http.*;

@WebServlet("/output1")
public class Output1Servlet extends HttpServlet{
	@Override
	protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
		resp.setContentType("text/html;charset=UTF-8");
		PrintWriter out = resp.getWriter();
		out.print(req.getParameter("name"));
		out.close();
	}

}

//Output2Servlet.java
package com.edu.test;

import java.io.*;
import javax.servlet.*;
import javax.servlet.annotation.*;
import javax.servlet.http.*;

@WebServlet("/output2")
public class Output2Servlet extends HttpServlet{
	@Override
	protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
		resp.setContentType("text/html;charset=UTF-8");
		PrintWriter out = resp.getWriter();
		out.print(req.getParameter("subject"));
		out.close();
	}

}

//Output3Servlet.java
package com.edu.test;

import java.io.*;
import javax.servlet.*;
import javax.servlet.annotation.*;
import javax.servlet.http.*;

@WebServlet("/output3")
public class Output3Servlet extends HttpServlet{
	@Override
	protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
		resp.setContentType("text/html;charset=UTF-8");
		PrintWriter out = resp.getWriter();
		out.print(req.getParameter("dept"));
		out.close();
	}

}
```



클라이언트로부터 입력받은 한글을 처리하는 필터를 작성하겠습니다. 이전의 실습에서 flow2라는 이름으로 등록된 필터가 모든 페이지를 필터링하는 것으로 설정하였습니다. 모든 요청에 대하여 한글 처리를 하기 위해 flow2 필터, 즉 FlowFilterTwo.java 소스에 한글을 변환하는 기능을 추가하겠습니다.



```java
//FlowFilterTwo.java
~생략~
	@Override
	public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
			throws IOException, ServletException {
		request.setCharacterEncoding("UTF-8");  // <- 한 줄 추가!!
		System.out.println("doFilter() 호출 전...two");
		chain.doFilter(request, response);
		System.out.println("doFilter() 호출 후...two");
	}
~생략~
```



실행시켜보면 한글 처리가 잘 되는것을 확인할 수 있습니다.





- **FilterConfig**

  FilterConfig는 필터 객체의 init() 메소드의 인자값으로 전달되는 객체로서, 필터에 대한 정보값을 추출하는 메소드를 가지고 있습니다. 필터 객체의 정보는 web.xml의 \<filter> 태그로 지정하므로 \<filter> 태그에 대한 정보값을 추출할 목적으로 사용합니다. 



FilterConfig 객체를 활용하는 예제를 작성해보겠습니다. 앞에서 작성한 소스에서 문제점을 지적한다면 한글 처리할 때 소스에서 직접 문자코드를 지정하는 부분입니다. 설정값을 소스에 직접 코딩하지 않고, 외부에 값을 설정하고 읽어 들여 설정하는 방법으로 리팩터링 해봅시다. 



먼저, web.xml 파일에서 flow2 필터 등록 태그를 다음과 같이 수정합니다.

```xml
~생략~
  <filter>
  	<filter-name>flow2</filter-name>
  	<filter-class>com.edu.test.FlowFilterTwo</filter-class>
  	<init-param>
  		<param-name>en</param-name>
  		<param-value>UTF-8</param-value>
  	</init-param>
  </filter>
~생략~
```



FlowFilterTwo.java 소스를 다음과 같이 수정합니다.

```java
~생략~
public class FlowFilterTwo implements Filter{
	
	String charset;
	
	@Override
	public void init(FilterConfig filterConfig) throws ServletException {
		System.out.println("init() 호출...two");
		charset = filterConfig.getInitParameter("en");
	}

	@Override
	public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
			throws IOException, ServletException {
		request.setCharacterEncoding(charset);
~생략~
```





#### 7.1.6 @WebFilter 어노테이션

앞 절에서는 필터 기능을 사용할 때 다음과 같은 순서로 작업했습니다.

**1)** Filter 인터페이스를 상속받아 필터 객체를 구현한다.

**2)** \<filter> 태그로 web.xml에 필터 객체를 등록한다.

**3)** \<filter-mapping> 태그로 web.xml에 필터 매핑을 설정한다.



이처럼 web.xml을 이용해서 필터를 설정했는데요. 그런데 같은 작업을 어노테이션을 이용하여 설정할 수도 있습니다. 어노테이션을 이용할 때는 다음과 같은 순서로 작업합니다.



**1)** Filter 인터페이스를 상속받아 필터 객체를 구현한다.

**2)** 필터 객체에 @WebFilter() 어노테이션을 선언한다.

**3)** @WebFilter()에 filterName 속성을 추가한다.

**4)** @WebFilter()에 urlPatterns 속성을 추가한다.





**(1) @WebFilter 속성**

- **urlPatterns, value**: 필터링할 페이지의 실행요청 URL을 지정합니다. 반드시 지정해야 할 속성으로서, 둘 중 어느 것을 사용해도 상관없습니다.
- **servletNames**: 필터링할 서블릿의 이름을 지정합니다.
- **filterName**: 등록하는 필터의 이름을 지정합니다.
- **initParams**: 필터에 전달하는 초기 파라미터값을 지정합니다.





**(2) 필터 매핑 설정 방법**

@WebFilter 어노테이션에서 필터링할 페이지를 설정하는 방법은 여러 가지가 있습니다.



- **@WebFilter("/login")**: 클라이언트가 요청하는 페이지의 실행 요청 URL을 설정하는 방법
- **@WebFilter("/*")**: 와일드 카드를 사용하여 설정하는 방법
- **@WebFilter(value="/hello")**: value 속성으로 지정하는 방법
- **@WebFilter(urlPatterns="/hello")**: urlPatterns 속성으로 지정하는 방법
- **@WebFilter(servletNames="MyServlet")**: 서블릿 이름으로 지정하는 방법
- **@WebFilter(servletNames={"FirstServlet", "SecondServlet"})**: 값이 여러 개일때 배열처럼 지정하는 방법



**(3) 필터 초기 파라미터 설정 방법**

@WebFilter 어노테이션에서 초기 파라미터를 설정하기 위해 사용하는 속성은 initParams입니다. 이 속성은 옵션이며 값을 지정할 때는 변수의 이름과 값을 지정해야 하는데요. @WebInitParam 어노테이션을 사용하여 지정합니다.



```java
@WebFilter(
	urlPatterns = "/*",
	initParams = @WebInitParam(name = "en", value = "UTF-8") 
)
```



**(4) @WebFilter 예제**

@WebFilter 어노테이션을 활용하여 필터 예제를 작성해 봅시다.



```java
//FlowFilterThree.java
package com.edu.test;

import java.io.IOException;

import javax.servlet.*;
import javax.servlet.annotation.WebFilter;

@WebFilter(filterName = "timer", urlPatterns="/third")
public class FlowFilterThree implements Filter{
	@Override
	public void init(FilterConfig filterConfig) throws ServletException {
	}
	
	@Override
	public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
			throws IOException, ServletException {
		long startTime = System.currentTimeMillis();
		chain.doFilter(request, response);
		long endTime = System.currentTimeMillis();
		
		long executeTime = endTime - startTime;
		System.out.println("수행 시간: " + executeTime + " ms");
	}
	
	public void destroy() {}
}
```



```java
//ThirdServlet.java
package com.edu.test;

import java.io.*;

import javax.servlet.*;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.*;


@WebServlet("/third")
public class ThirdServlet extends HttpServlet {

	protected void doGet(HttpServletRequest req, HttpServletResponse rsp) throws ServletException, IOException {
		rsp.setContentType("text/html;charset=UTF-8");
		PrintWriter out = rsp.getWriter();

		int i = 1;
		while(i <= 10) {
			out.print("<br>number: " + i);
			i++;
			
			try {
				Thread.sleep(100);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
		
		out.print("<br>실행완료!");
		out.close();
	}
}
```





---

### 7.2 리스너(Listener)

**리스너**란 어떠한 일(event)이 발생하기를 기다리다가 실제 그 일이 발생하였을 때 수행되는 메소드를 가지고 있는 자바 객체입니다. 이런 객체를 **이벤트 핸들러**라고 합니다. 서블릿에서는 이러한 **리스너**를 활용할 수가 있어서 정해진 **이벤트**가 발생할 때 **자동으로 수행**되는 처리 내용을 구현할 수 있습니다.



예를 들어, 클라이언트로부터 요청이 전달되었을 때, 요청이 끝났을 때, 세션 객체가 생성되거나 삭제 되었을 때, 그리고 서블릿 컨테이너가 시작되었을 때, 종료될 때 등 각 시점에 수행될 처리 내용을 구현하여 등록할 수 있으며 이때 활용되는 기술이 바로 리스너입니다.



서블릿에서는 **HttpServletRequest, HttpSession** 그리고 **ServletContext** 객체와 관련하여 발생하는 여러 이벤트들에 대한 이벤트 핸들러(리스너)를 구현할 수 있습니다. 이번 절에서는 서블릿의 리스너를 구현하는 방법과 실행되도록 등록하는 방법에 대하여 학습합니다.



#### 7.2.1 리스너 개요

**리스너**란 <u>정해진 이벤트가 발생했을 때 수행되는 메소드들을 제공하는 객체</u>입니다. 즉, 흔히 말하는 **이벤트 핸들러**라고 할 수 있습니다.



**(1) 리스너의 종류**

웹에서는 이벤트 소스(source)가 정해져 있습니다. **ServletContext와 HttpSession, 그리고 HttpServletRequest** 등 세 객체가 <u>이벤트 소스</u>입니다. 서블릿 API에서는 이 세 개의 객체에서 이벤트가 발생하면 자동으로 호출될 메소드들을 가지고 있는 리스너를 제공합니다. 각 객체의 생성과 해제 그리고 속성의 등록, 삭제, 대체 이벤트가 발생할 때마다 각 리스너에 있는 정해진 메소드가 호출됩니다. 그러면 각 이벤트와 관련된 리스너와 해당 메소드에 대해 살펴봅시다.



**객체 생성과 삭제 이벤트**

우선, 앞에서 언급한 세 가지 객체의 생성과 삭제 시점을 정리하면 다음과 같습니다.

| 구분                   | 생성 시점                                          | 삭제 시점                                        |
| ---------------------- | -------------------------------------------------- | ------------------------------------------------ |
| **ServletContext**     | 웹 애플리케이션의 서비스가 준비될 때(서버 시작 시) | 웹 애플리케이션 서비스가 중지될 때(서버 종료 시) |
| **HttpSession**        | 클라이언트 접속 시                                 | 클라이언트 접속 종료 시                          |
| **HttpServletRequest** | 클라이언트 서비스 요청 시                          | 클라이언트 서비스 응답 시                        |



**HttpSession** 객체는 클라이언트가 서버에 접속한 다음 처음으로 getSession()이나 getSession(true) 메소드를 실행할 때 생성되며, 웹 브라우저를 종료하거나 invalidate() 메소드가 실행되거나 세션의 유지 시간이 지나면 삭제됩니다. 

그리고 **HttpServletRequest** 객체는 클라이언트가 서버에 서비스를 요청할 때마다 생성되며 서버에서 클라이언트에게 응답을 보내면 삭제됩니다.



각 객체가 생성되거나 삭제되는 시점에 자동으로 실행하고자 하는 기능이 있다면, 해당하는 이벤트 소스의 리스너 객체를 상속받아 구현하면 됩니다. 다음은 각 객체의 생성과 삭제 이벤트가 발생할 때 실행되는 메소드가 있는 리스너 객체입니다.



| 객체명                 | 관련 API 목록                               |
| ---------------------- | ------------------------------------------- |
| **HttpServletRequest** | ServletRequestEvent, ServletRequestListener |
| **HttpSession**        | HttpSessionEvent, HttpSessionListener       |
| **ServletContext**     | ServletContextEvent, ServletContextListener |



**ServletContextListener**는 **ServletContext** 객체에서 **이벤트**가 발생했을 때 실행할 메서드를 가지고 있는 **이벤트 핸들러**입니다. 다음은 **ServletContextListener**가 가지고 있는 메서드입니다. 메서드의 이름을 보면 어떤 이벤트가 발생했을 때 실행되는 메서드인지 짐작할 수 있습니다.

- **void contextDestroyed(ServletContextEvent sce)**
- **void contextInitialized(ServletContextEvent sce)**



**contextInitialized()**는 **ServletContext** 객체가 생성될 때 실행되는 메서드이며, **contextDestroyed()**는 **ServletContext** 객체가 삭제될 때 실행되는 메서드입니다. 나머지 **HttpSession**과 **HttpServletRequest** 객체의 생성과 삭제에 관한 이벤트 핸들러에도 **~Destroyed()**와 **~Initialized()** 메서드가 있으며 동작하는 시점과 방식은 같습니다.



실제로 프로젝트를 진행하면서 <u>가장 많이 사용되는</u> **이벤트 핸들러 **객체는 **ServletContextListener**입니다. 웹 애플리케이션은 서비스를 하기 위해서 초기환경을 구축해야 하는데, 이는 서버가 시작되면서 자동으로 이루어져야 합니다. 이럴 때 **ServletContextListener** 객체를 상속받아 **contextInitialized()** 메서드에 초기환경 처리 내용을 구현합니다.









**속성의 추가, 삭제, 대체 이벤트**

**ServletContext, HttpSession, HttpServletRequest** 객체는 공통점이 있습니다. 모두 웹 상에서 여러 페이지 간의 정보 공유를 위해 **상태정보 유지** 목적으로 사용하는 객체라는 점입니다. 그래서 세 개의 객체 모두 공통적으로 속성(정보)을 등록하는 **setAttribute()**, 추출하는 **getAttribute()**, 삭제하는 **removeAttribute()** 메서드를 가지고 있습니다.



이와 같은 메서드들을 이용해 객체의 속성을 **등록, 수정, 삭제**하는 이벤트가 발생했을 때는 속성에 관한 이벤트 핸들러에 포함된 메서드가 실행됩니다. 



- **객체의 속성 관련 Listener**

| 객체명                 | 관련 API 목록                                                |
| ---------------------- | ------------------------------------------------------------ |
| **HttpServletRequest** | ServletRequestAttributeEvent<br />ServletRequestAttributeListener |
| **HttpSession**        | HttpSessionAttributeListener                                 |
| **ServletContext**     | ServletContextAttributeEvent<br />ServletContextAttributeListener |



속성에 관한 이벤트 핸들러들은 속성의 추가, 삭제, 대체에 관한 이벤트가 발생했을 때 실행되는 메서드를 가지고 있습니다. 다음은 **HttpSessionAttributeListener** 객체가 가지고 있는 메서드입니다.

- **void attributeAdded(HttpSessionBindingEvent event)**
- **void attributeRemoved(HttpSessionBindingEvent event)**
- **void attributeReplaced(HttpSessionBindingEvent event)**



**ServletContext**와 **HttpServletRequest** 객체의 속성에 관한 이벤트 핸들러들도 이와 같은 메서드를 가지고 있으며 동작 시점과 방식은 같습니다.







**(2) 리스너의 등록**

리스너 기능을 활용하려면 특정 이벤트 소스에서 이벤트가 발생했을 때 실행되기 원하는 내용을 해당 이벤트 **핸들러 객체를 상속**받아 구현한 다음, 개발된 리스너를 컨테이너가 인식하도록 **web.xml** 파일에 등록해주어야 합니다.



**web.xml** 파일에 리스너를 등록하는 태그는 다음과 같습니다.

```xml
[리스너 등록 태그 구조]
<listener>
	<listener-class> </listener-class>
</listener>
```

- **\<listener>**: 이벤트 핸들러를 상속받아 메서드를 재정의한 객체. 즉, 리스너 객체를 등록할 때 사용하는 태그입니다.
- **\<listener-class>**: **\<listener>** 태그를 사용할 때 반드시 지정해야 하는 태그로서, 실제 리스너 객체의 클래스 이름을 패키지 이름과 함께 정확하게 지정합니다.



리스너 객체는 웹서버가 시작될 때 **web.xml**에서 **\<listener>** 태그를 읽어 들이면서 **\<listener-class>**에 지정한 클래스를 찾아서 생성합니다. 즉, 서버 시작 시 리스너 객체가 생성되어 준비 작업이 완료되는 것입니다. 그리고 서버가 중지될 때 삭제됩니다. 웹 애플리케이션이 서비스되고 있는 동안 메모리에 상주하고 있으므로 이벤트가 발생하면 자동으로 메서드가 실행되는 것입니다.





#### 7.2.2 리스너 객체 구현

**(1) HttpServletRequest 객체**

**HttpServletRequest** 객체는 클라이언트에게 서블릿을 요청받았을 때 생성되었다가 요청이 끝나고 응답할 때 삭제됩니다. **HttpServletRequest** 객체가 초기화되거나 해제되는 시점에 발생하는 이벤트는 **ServletRequestEvent**이고, 객체에 속성(Attribute)을 등록, 추출 그리고 대체하는 시점에 발생하는 이벤트는 **ServletRequestAttributeEvent**입니다. 각 이벤트가 발생하면 해당 이벤트 객체를 전달하면서 리스너 객체를 상속한 객체의 정해진 메서드를 호출합니다.























