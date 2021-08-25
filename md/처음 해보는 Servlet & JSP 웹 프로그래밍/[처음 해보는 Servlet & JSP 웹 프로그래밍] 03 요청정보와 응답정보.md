## 03 요청정보와 응답정보



### 3.1 객체 생성 및 삭제

**HttpServletRequest**는 클라이언트가 서버에 보내는 **요청**정보를 처리하는 객체이고, **HttpServletResponse**는 서버가 클라이언트로 보내는 **응답**정보를 처리하는 객체입니다. 



클라이언트의 요청으로부터 시작되는 **웹서버의 처리 순서**를 알아보면 다음과 같습니다.

**1)** 클라이언트가 웹 브라우저에서 서비스를 요청합니다. 이때 HTTP 프로토콜 기반으로 요청정보가 만들어져 웹서버에 전달됩니다.

**2)** 웹서버는 클라이언트로부터 전달받은 요청정보의 URI를 살펴보고, 서블릿이라면 서블릿 컨테이너에 처리를 넘깁니다.

**3)** 서블릿 컨테이너는 요청받은 서블릿 클래스 파일을 찾아서 실행합니다.

**4)** 실행할 때 첫 순서는 최초의 요청인지를 파악합니다. 최초의 요청이라면 메모리에 로딩 후 객체를 생성하고 init() 메소드를 호출합니다.

**5)** init() 메소드 실행이 끝난 다음에는 최초의 요청이든지 그렇지 않든지 서블릿 실행 요청이 들어올 때마다 실행되는 작업으로, 서블릿 컨테이너는 HttpServletRequest와 HttpServletResponse 객체를 생성합니다. HttpServletRequest 객체는 클라이언트로부터 요청받은 정보를 처리할 목적으로 생성하고, HttpServletResponse 객체는 클라이언트에게 보내는 응답정보를 처리할 목적으로 생성합니다.

**6)** service() 메소드를 호출합니다. 이때, 앞에서 생성한 HttpServletRequest와 HttpServletResponse 객체의 주소를 인자로 넘깁니다. service() 메소드에서는 인자로 받은 두 객체를 사용하여 프로그램을 구현합니다.

**7)** service() 메소드가 완료되면 클라이언트에게 응답을 보내고 서버에서 실행되는 프로그램은 완료됩니다. 이때, HttpServletRequest와 HttpServletResponse 객체는 소멸합니다.



이러한 처리 과정에서 꼭 기억해야 할 점은 HttpServletRequest와 HttpServletResponse 객체의 생존 기간입니다. 한번 더 강조하자면, **HttpServletRequest와 HttpServletResponse 객체는 service() 메소드가 실행되기 전에 생성되었다가 끝나면 소멸**합니다. 따라서 service() 메소드가 실행되는 동안에만 메모리에 상주하고 있어서 그동안에만 사용할 수 있습니다.





### 3.2 응답정보 처리 - HttpServletResponse

서비스를 요청한 클라이언트에게 응답하기 위한 기능을 처리할 때 javax.servlet 패키지의 ServletResponse 인터페이스를 사용합니다. 이 인터페이스를 이용하여 클라이언트의 요청에 응답하기 위한 출력스트림을 추출하거나 버퍼의 크기를 설정하고, 응답할 내용의 타입과 문자셋을 설정하는 등의 작업을 수행할 수 있습니다.



또한, HttpServletResponse 인터페이스는 우리가 웹 애플리케이션을 개발하면서 응답 관련 작업을 수행할 때 사용하는 ServletResponse 인터페이스를 상속합니다. 따라서 일반적인 네트워크 통신에서 응답과 관련된 메소드들을 포함하고 있으며, 여기에 HTTP 프로토콜 통신 기반의 응답 관련 메소드들도 확장하여 포함하고 있습니다.



- ServletResponse: 일반적인 네트워크 통신에서의 응답 관련 메소드 제공
- HttpServletResponse: HTTP 통신 기반의 응답 관련 메소드 확장 제공





**ServletResponse 인터페이스의 주요 메소드**

| 함수                                      | 기능                                                         |
| ----------------------------------------- | ------------------------------------------------------------ |
| PrintWriter getWriter()                   | 서비스를 요청한 클라이언트와 서버 간에 연결된 PrintWriter 객체를 생성하여 반환한다. |
| void setBufferSize(int size)              | 출력스트림의 버퍼 크기를 설정한다.                           |
| void setCharacterEncoding(String charset) | 응답정보 인코딩에 사용할 문자를 설정한다.                    |
| void setContentLength(int len)            | 응답정보의 데이터 길이를 설정한다.                           |
| void setContentType(String type)          | 응답정보의 데이터 형식(MIME 타입)을 설정한다.                |
| void setLocale(Locale loc)                | 클라이언트가 사용하는 언어, 국가코드 등 클라이언트의 환경을 설정한다. |



ServletResponse 인터페이스에는 네트워크 환경에서 필요한 응답 관련 메소드들을 정의하고 있지만, HTTP 프로토콜 기반의 응답과 관련된 메소드들은 ServletResponse를 확장한 HttpServletResponse 인터페이스에서 정의하고 있습니다. HttpServletResponse는 쿠키 설정, HTTP 응답 헤더 설정 등 HTTP 프로토콜과 직접적인 관계가 있는 메소드들을 포함하고 있습니다.









**HttpServletResponse 인터페이스 주요 메소드**

| 함수                                              | 기능                                                         |
| ------------------------------------------------- | ------------------------------------------------------------ |
| void addCookie(Cookie cookie)                     | 인자값으로 주어진 쿠키를 응답정보의 헤더에 추가한다. 쿠키는 응답정보의 Set-Cookie 헤더의 값으로 추가되어 클라이언트로 전송된다. |
| String encodeRedirectURL(String url)              | 클라이언트와 서버 간 세션이 유지되는 상태에서 브라우저 쿠키를 지원하지 않을 때 주어진 URL 뒤에 세션 아이디를 추가하고 인코딩하여 요청을 재전송한다. |
| String encodeURL(String url)                      | 주어진 URL에 세션 아이디를 추가하여 인코딩해서 반환한다.     |
| void sendRedirect(String location)                | 응답을 클라이언트가 요청한 URL이 아니라 sendRedirect()에 주어진 URL로 재전송한다. 매개변수 location은 절대 URL이나 상대 URL로 지정한다. 이 메소드는 서버의 특정 자원이 다른 URL로 이동할 때 사용할 수 있는 메소드이다. |
| public void setDateHeader(String name, long date) | 날짜를 밀리 초로 변환하여 주어진 이름과 날짜를 응답정보 헤더에 설정한다. |
| public void setHeader(String name, String value)  | 응답정보의 헤더에 주어진 이름과 값을 설정한다.               |
| public void setIntHeader(String name, int value)  | 주어진 이름과 정수값을 갖도록 응답정보 헤더에 추가한다.      |
| public void setStatus(int sc)                     | 응답으로 전송될 HTTP 응답에 대한 상태코드를 설정한다.        |





#### 3.2.1 출력 응답

HttpServletResponse 객체를 활용하여 클라이언트 쪽으로 문자열을 전송한 다음, 웹 브라우저에 문자열을 출력하는 예제를 작성해보겠습니다.



<이클립스 디렉토리 구조>

![image-20210311100108656](C:\Users\user\AppData\Roaming\Typora\typora-user-images\image-20210311100108656.png)



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
		out.close();
	}
}
```



위의 서블릿을 실행하면,

![image-20210311100154424](C:\Users\user\AppData\Roaming\Typora\typora-user-images\image-20210311100154424.png)



위와 같이 잘 접속되는 것을 확인할 수 있습니다.





#### 3.2.2 한글 응답

웹 프로그램을 개발할 때 자주 발생하는 문제 중 하나가 한글이 올바르게 표시되지 않고 깨져서 나오는 것입니다. 이렇게 깨진 한글을 그냥 놔둘 수는 없으므로 복원해야 하는데요. 이번 절에서는 서버에서 웹 브라우저로 보낸 한글을 웹 브라우저로 출력했을 때 응답정보 헤더에 한글을 처리하기 위해 설정하는 내용에 대해 알아보겠습니다.



```java
package com.edu.test;

import java.io.*;

import javax.servlet.*;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.*;


@WebServlet("/third")
public class ThirdServlet extends HttpServlet {

	protected void doGet(HttpServletRequest req, HttpServletResponse rsp) throws ServletException, IOException {
		PrintWriter out = rsp.getWriter();
		out.print("<h1>좋은 하루!</h1>");
		out.close();
	}
}
```



위의 소스는 클라이언트와 서버 간의 out 출력스트림을 만들어 이를 이용해 out.print("\<h1>좋은 하루!\</h1>"); 문자열을 전송하는 소스입니다.



그런데 서버 프로그램에서 클라이언트로 데이터를 전송하면서 명시하지 않은 정보가 있습니다. 그것은 **보내는 데이터의 타입과 문자셋 정보**입니다. 응답정보의 헤더에 데이터 타입과 문자셋을 설정해서 보내야지만 클라이언트는 받은 데이터를 타입에 맞게 처리할 수 있고, 인코딩 작업도 할 수 있습니다.



그런데 서버가 이 정보를 명시하지 않으면 기본값으로 처리합니다. 즉, 문서타입은 text/html, 문자셋은 8859_1로 처리합니다. 그래서 "\<h1>좋은 하루!\</h1>"란 문자열을 보내면 HTML의 \<H1> 태그로 처리하고, 인코딩 문자셋은 8859_1로 처리하는데요. 아쉽게도 8859_1 문자셋은 한글을 지원하지 않습니다. 따라서 다음처럼 한글이 깨져서 나옵니다.



![image-20210311101039559](C:\Users\user\AppData\Roaming\Typora\typora-user-images\image-20210311101039559.png)



이처럼 한글이 깨지는 문제를 방지하려면 서버가 클라이언트로 보내는 데이터의 문서타입과 한글을 지원하는 문자셋을 응답정보 헤더에 설정해서 보내야 합니다. 문서타입과 문자셋을 설정하려면 HttpServletResponse의 setContentType() 메소드를 사용합니다. 소스를 다음처럼 수정합니다.



```java
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
		out.print("<h1>좋은 하루!</h1>");
		out.close();
	}
}

```



위처럼 "text/html;charset=UTF-8"을 매개변수로 setContentType() 메소드를 호출하면, 응답 정보 헤더에 문서의 타입과 문자셋을 지정할 수 있습니다.

응답정보의 Content-Type 헤더에는 두 가지 정보를 설정해야 하는데요. 세미콜론(;)을 기준으로 앞에는 문서의 타입을 지정하고, 뒤에는 charset= 다음에 문자셋을 지정합니다.



![image-20210311101351368](C:\Users\user\AppData\Roaming\Typora\typora-user-images\image-20210311101351368.png)



다시 실행해보면, 위와 같이 한글이 잘 나오는 것을 확인할 수 있습니다.













### 3.3 요청정보 처리 - HttpServletRequest

이번 절에서는 요청정보가 무엇인지, 그리고 요청정보 추출과 관련된 주요 API들의 종류와 기능에 대해 알아보겠습니다.



브라우저에 적절한 URL 문자열을 이용하여 웹서버에 서블릿 수행을 요청할 때 일정한 형식의 다양한 정보를 서버로 전달하는데요. 이때 클라이언트가 서버로 전달하는 요청정보들은 다음과 같습니다.



- 클라이언트의 IP 주소, 포트 번호
- 클라이언트가 전송한 요청 헤더 정보(클라이언트에서 처리 가능한 문서 타입의 종류, 클라이언트 프로그램 정보, 처리 가능한 문자셋 정보, 쿠키 정보)
- 요청 방식, 요청 프로토콜의 종류와 버전, 요청하는 파일의 URI, 요청받은 서버의 정보
- 서버의 호스트 이름, 포트 번호
- 사용자가 서블릿 요청 시 추가로 전달한 정보
- 질의(Query) 문자열(웹서버에 서비스를 요청하면서 추가로 전달한 name=value 형태의 데이터)





위와 같은 다양한 요청정보는 HttpServletRequest 인터페이스의 getXXX() 메소드를 통해 추출할 수 있고, HttpServletRequest는 service()나 doGet() 또는 doPost() 메소드의 첫 번째 인자로 전달됩니다. 클라이언트가 보낸 요청정보를 처리할 때 사용하는 HttpServletRequest 인터페이스는 ServletRequest를 상속받습니다. 







**ServletRequest의 주요 메소드**

| 함수                                     | 기능                                                         |
| ---------------------------------------- | ------------------------------------------------------------ |
| Object getAttribute(String name)         | ServletRequest 객체 안에 등록된 데이터를 추출하여 반환한다.  |
| Enumeration getAttributeNames()          | ServletRequest 객체 안에 등록된 데이터들의 이름 전부를 하나의 Enumeration 객체에 담아서 반환한다. |
| String getCharacterEncoding()            | 클라이언트가 서버에 서비스를 요청할 때 사용한 문자들의 인코딩 문자셋을 반환한다. |
| int getContentLength()                   | 서비스 요청 시 보낸 요청정보 몸체에 포함된 데이터의 길이를 반환한다. 만약 길이를 알 수 없을 때는 -1를 반환한다. |
| ServletInputStream getInputStream()      | 요청정보 몸체로부터 바이너리 데이터를 읽어들이기 위해 한 번에 한 줄씩 읽을 수 있는 ServletInputStream 객체를 반환한다. |
| String getParameter(String name)         | 클라이언트가 보낸 질의 문자열 중에서 인자로 지정된 name과 일치하는 것을 찾아 name의 value를 반환한다. |
| Enumeration< String> getParameterNames() | 클라이언트가 서버로 보낸 질의 문자열들의 이름을 하나의 Enumeration 객체에 담아서 반환한다. |
| String[] getParameterValues(String name) | 클라이언트가 서버로 보낸 질의 문자열 중에서 인자로 지정된 name과 일치하는 모든 값을 찾아 하나의 String 타입의 배열에 담아 반환한다. |
| String getProtocol()                     | 클라이언트가 서버에 서비스를 요청하면서 사용한 프로토콜 정보를 반환한다. |
| BufferedReader getReader()               | 요청정보 몸체로부터 문자 인코딩에 따라 텍스트를 읽어들이기 위한 BufferedReader 객체를 반환한다. |
| String getRemoteAddr()                   | 서버에 서비스를 요청한 클라이언트의 IP 주소를 반환한다.      |
| String getScheme()                       | 서비스 요청 시 사용한 http, https, 또는 ftp 등과 같은 프로토콜 이름을 반환한다. |
| String getServerName()                   | 서비스를 요청받은 서버의 이름을 반환한다.                    |
| int getServerPort()                      | 클라이언트의 서비스를 요청받은 서버 포트 번호를 반환한다.    |
| ServletContext getServletContext()       | 서버가 시작될 때 웹 애플리케이션 단위로 생성된 ServletContext 객체의 주소를 추출하여 반환한다. |
| void removeAttribute(String name)        | ServletRequest 객체에 setAttribute(name) 메소드를 이용하여 등록된 데이터를 삭제한다. |
| void setAttribute(String name, Object o) | 클라이언트의 또 다른 서비스 요청에서도 계속해서 사용하고 싶은 데이터는 서버에 저장해야 하는데 ServletRequest 객체 안에 저장(등록)해준다. |
| void setCharacterEncoding(String env)    | 클라이언트가 요청정보 몸체에 포함해서 보내는 문자열들을 지정된 문자셋을 이용해 인코딩해준다. |



ServletRequest 객체는 네트워크 기반에서 사용되는 기본적인 메소드들로 구성되어 있으며, HTTP 프로토콜에 기반하는 세션이나 쿠키와 같은 정보를 추출하는 메소드는 지원하지 않습니다. HTTP 프로토콜 기반의 기능을 지원하는 메소드는 HttpServletRequest 객체에서 확장 지원하고 있습니다.



다음은 HttpServletRequest에서 추가된 메소드입니다.



**HttpServletRequest의 메소드**

| 함수                                      | 기능                                                         |
| ----------------------------------------- | ------------------------------------------------------------ |
| String getHeader(String headerName)       | HTTP 요청 헤더에 지정된 headerName의 값을 문자열로 반환한다. 만일 HTTP 요청 헤더에 headerName의 값이 없으면 null을 반환한다. |
| Enumeration getHeaderNames()              | HTTP 요청 헤더에 포함된 모든 헤더의 이름을 Enumeration으로 반환한다. |
| Enumeration getHeaders(String headerName) | HTTP 요청 헤더에 포함된 headerName의 모든 값을 Enumeration으로 반환한다. |
| int getIntHeader(String headerName)       | HTTP 요청 헤더에 포함된 headerName의 값을 int로 반환한다. 지정된 headerName의 값을 int로 변환할 수 없을 때 NumberFormat Exception이 발생하고, headerName 헤더가 HTTP 요청 헤더에 없을 때 -1을 반환한다. |
| long getDateHeader(String headerName)     | HTTP 요청 헤더에 포함된 headerName의 값을 밀리초로 변환하여 long으로 반환한다. 지정된 header의 값을 int로 변환할 수 없을 때 IllegalArgumentException이 발생하고, headerName 헤더가 HTTP 요청 헤더에 없을 때 -1을 반환한다. |
| String getPathInfo()                      | 클라이언트가 서비스 요청 시 보낸 요청 URL의 뒷부분에 있는 path 정보를 반환한다. |
| HttpSession getSession()                  | 서비스를 요청한 클라이언트가 사용하는 HttpSession 객체를 반환한다. 만일 반환할 HttpSession 객체가 없으면 새로 생성하여 반환한다. |
| HttpSession getSession(boolean create)    | 서비스를 요청한 클라이언트가 사용하는 HttpSession 객체를 반환한다. 만일 반환한 HttpSession 객체가 없으면 getSession(true)이면 새로 생성하여 반환한다. 그러나 getSession(false)이면 HttpSession 객체를 새로 생성하지 않고 null을 반환한다. |
| String getRequestedSessionId()            | 서비스를 요청한 클라이언트가 사용하는 HttpSession의 ID를 반환한다. |
| boolean isRequestedSessionIdValid()       | 서비스를 요청한 클라이언트가 사용하는 HttpSession 객체가 유효한지를 판단한다. |
| boolean isRequestedSessionIdFromCookie()  | 서비스를 요청한 클라이언트가 사용하는 HttpSession의 ID가 쿠키로 전달되면 true, 그렇지 않다면 false를 반환한다. |
| boolean isRequestedSessionIdFromURL()     | 서비스를 요청한 클라이언트가 사용하는 HttpSession의 ID가 URL에 포함되면 true, 그렇지 않다면 false를 반환한다. |
| Cookie[] getCookies()                     | 서비스를 요청받은 서버가 서비스를 요청한 클라이언트에게 이전에 보낸 모든 쿠키를 추출한다. |
| String getRequestURI()                    | 클라이언트가 서비스 요청 시 보낸 URL에서 URI 부분만 반환한다. |
| String getQueryString()                   | 클라이언트가 GET 방식으로 서버에 보낸 질의 문자열들을 모두 추출하여 반환한다. |
| String getMethod()                        | 클라이언트가 서비스를 요청할 때 요청한 방식의 이름을 반환한다. |
| String getPathTranslated()                | 클라이언트가 서비스 요청 시 보낸 URL의 경로 정보를 절대경로(path)로 변경하여 반환한다. |



#### 3.3.1 네트워크 정보

HttpServletRequest 인터페이스에서 제공하는 메소드 중에서 서버와 클라이언트의 네트워크 정보를 추출하는 메소드들을 실행한 후 결괏값을 확인하는 예제를 작성해보겠습니다. NetInfoServlet.java라는 이름으로 새로운 클래스 파일을 만들고 다음과 같은 코드를 작성합니다.



```java
package com.edu.test;

import java.io.*;

import javax.servlet.*;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.*;


@WebServlet("/netInfo")
public class NetInfoServlet extends HttpServlet {

	protected void doGet(HttpServletRequest req, HttpServletResponse rsp) throws ServletException, IOException {
		rsp.setContentType("text/html;charset=EUC-KR");
		PrintWriter out = rsp.getWriter();
		out.print("<html><head><title>Request 정보 출력 Servlet</title></head>");
		out.print("<body>");
		out.print("<h3>네트워크 관련 요청 정보</h3>");
		out.print("Request Scheme: " + req.getScheme() + "<br/>");
		out.print("Server Name: " + req.getServerName() + "<br/>");
		out.print("Server Address: " + req.getLocalAddr() + "<br/>");
		out.print("Server Port: " + req.getServerPort() + "<br/>");
		out.print("Client Address: " + req.getRemoteAddr() + "<br/>");
		out.print("Client Host: " + req.getRemoteHost() + "<br/>");
		out.print("Client Port: " + req.getRemotePort() + "<br/>");
		out.print("</body></html>");
		out.close();
	}
}
```



**[실행 결과]**



![image-20210311103628045](C:\Users\user\AppData\Roaming\Typora\typora-user-images\image-20210311103628045.png)







#### 3.3.2 URL 정보

이번에는 HttpServletRequest 인터페이스에서 제공하는 메소드 중에서 URL 정보를 추출하는 메소드들을 실행한 후 결괏값을 확인하는 예제를 작성해보겠습니다. URLInfoServlet.java라는 이름으로 새로운 클래스 파일을 만들고 다음과 같은 코드를 작성합니다.



```java
package com.edu.test;

import java.io.*;

import javax.servlet.*;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.*;


@WebServlet("/urlInfo")
public class URLInfoServlet extends HttpServlet {

	protected void doGet(HttpServletRequest req, HttpServletResponse rsp) throws ServletException, IOException {
		rsp.setContentType("text/html;charset=EUC-KR");
		PrintWriter out = rsp.getWriter();
		out.print("<html><head><title>Request 정보 출력 Servlet</title></head>");
		out.print("<body>");
		out.print("<h3>요청 방식과 프로토콜 정보</h3>");
		out.print("Request URI: " + req.getRequestURI() + "<br/>");
		out.print("Request URL: " + req.getRequestURL() + "<br/>");
		out.print("Context Path: " + req.getContextPath() + "<br/>");
		out.print("Request Protocol: " + req.getProtocol() + "<br/>");
		out.print("Servlet Path: " + req.getServletPath() + "<br/>");
		out.print("</body></html>");
		out.close();
	}
}
```



**[실행 결과]**



![image-20210311104022858](C:\Users\user\AppData\Roaming\Typora\typora-user-images\image-20210311104022858.png)









#### 3.3.3 모든 헤더 정보

HTTP 프로토콜의 요청정보는 헤더와 몸체(body)로 구성되어 있고, 헤더는 두 번째 줄 이후부터는 "name:value" 형태로 헤더 정보들이 들어있습니다. HttpServletRequest 인터페이스에서 제공하는 메소드들을 이용하여 요청정보의 헤더 정보들을 추출한 후 결괏값을 확인하는 예제를 작성해보겠습니다. HeaderInfoServlet.java라는 이름으로 새로운 클래스 파일을 만들고 다음과 같은 코드를 작성합니다.



```java
package com.edu.test;

import java.io.*;
import java.util.*;

import javax.servlet.*;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.*;


@WebServlet("/headerInfo")
public class HeaderInfoServlet extends HttpServlet {

	protected void doGet(HttpServletRequest req, HttpServletResponse rsp) throws ServletException, IOException {
		rsp.setContentType("text/html;charset=EUC-KR");
		PrintWriter out = rsp.getWriter();
		out.print("<html><head><title>Request 정보 출력 Servlet</title></head>");
		out.print("<body>");
		out.print("<h3>요청 헤더 정보</h3>");
		Enumeration<String> em = req.getHeaderNames();
		while(em.hasMoreElements()) {
			String s = em.nextElement();
			out.println(s + ": " + req.getHeader(s) + "<br/>");
		}
		out.print("</body></html>");
		out.close();
	}
}
```



**[실행 결과]**



![image-20210311104501715](C:\Users\user\AppData\Roaming\Typora\typora-user-images\image-20210311104501715.png)







#### 3.3.4 추가 정보

HttpServletRequest 인터페이스에서 제공하는 메소드 중에서 질의(Query) 문자열이나 추가 경로 정보를 추출하는 메소드들을 실행한 후 결괏값을 확인하는 예제를 작성하겠습니다. AdditionalInfoServlet.java라는 이름으로 새로운 클래스 파일을 만들고 다음과 같은 코드를 작성합니다.



```java
package com.edu.test;

import java.io.*;

import javax.servlet.*;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.*;


@WebServlet("/addInfo")
public class AdditionalInfoServlet extends HttpServlet {

	protected void doGet(HttpServletRequest req, HttpServletResponse rsp) throws ServletException, IOException {
		rsp.setContentType("text/html;charset=EUC-KR");
		PrintWriter out = rsp.getWriter();
		out.print("<html><head><title>Request 정보 출력 Servlet</title></head>");
		out.print("<body>");
		out.print("<h3>추가적인 요청 정보</h3>");
		out.print("Request Method: " + req.getMethod() + "<br/>");
		out.print("Path Info: " + req.getPathInfo() + "<br/>");
		out.print("Path Translated: " + req.getPathTranslated() + "<br/>");
		out.print("Query String: " + req.getQueryString() + "<br/>");
		out.print("Content Length: " + req.getContentLength() + "<br/>");
		out.print("Content Type: " + req.getContentType() + "<br/>");
		out.print("</body></html>");
		out.close();
	}
}

```



**[실행 결과]**



![image-20210311104834573](C:\Users\user\AppData\Roaming\Typora\typora-user-images\image-20210311104834573.png)





https://kgvovc.tistory.com/29에서 서블릿을 요청할 수 있는 방법은 두 가지 방법이 있다고 했습니다. web.xml 파일에 설정하는 방법과 @WebServlet 어노테이션을 이용하는 방법입니다. AdditionalInfoServlet 예제는 @WebServlet("/addInfo") 어노테이션으로 설정하여 실행했습니다.



이번에는 web.xml 파일 설정으로 실행해봅시다. 먼저 AdditionalInfoServlet.java 파일에서 이전에 작성한 @WebServlet("/addInfo") 어노테이션 설정 부분을 주석 처리합니다.

```java
// @WebServlet("/addInfo")
```



이어서 web.xml 파일을 다음처럼 설정합니다.



```xml
~ 생략 ~
<servlet>
  	<servlet-name>addInfo</servlet-name>
  	<servlet-class>com.edu.test.AdditionalInfoServlet</servlet-class>
  </servlet>
  
  <servlet-mapping>
  	<servlet-name>addInfo</servlet-name>
  	<url-pattern>/addInfo/*</url-pattern>
  </servlet-mapping> 
~ 생략 ~
```



위의 코드 중 \<url-pattern> 태그 안에 `/addInfo/*` 를 입력했는데요. 여기서 *라는 문자는 일반 문자가 아니고 특수 문자로서, * 기호가 있는 자리에 어떤 경로(path)가 와도 상관이 없다는 의미입니다. 따라서, 클라이언트로부터 요청된 URL이 `http://localhost:8080/edu/addInfo`로 시작하는 조건만 만족한다면, addInfo 서블릿을 실행한다는 의미죠.



따라서 다음의 요청 URL들은 모두 addInfo 서블릿을 실행합니다.

- `http://localhost:8080/edu/addInfo/a`
- `http://localhost:8080/edu/addInfo/a/b`
- `http://localhost:8080/edu/addInfo/a/b/c`



각 요청 URL로 addInfo 서블릿을 실행한 결과는 다음과 같습니다.



**[실행 결과]**



![image-20210311105823037](C:\Users\user\AppData\Roaming\Typora\typora-user-images\image-20210311105823037.png)



![image-20210311105838475](C:\Users\user\AppData\Roaming\Typora\typora-user-images\image-20210311105838475.png)



![image-20210311105850940](C:\Users\user\AppData\Roaming\Typora\typora-user-images\image-20210311105850940.png)



![image-20210311105926308](C:\Users\user\AppData\Roaming\Typora\typora-user-images\image-20210311105926308.png)



URL에 ? 기호와 함께 name=value&name=value 형태로 정보를 추가할 수 있고, 이렇게 URL에 추가된 정보를 getQueryString() 메소드로 추출할 수 있다는 것을 확인했습니다.



또, Content Length가 -1인 이유는 GET 방식으로 전달된 요청정보의 몸체에는 어떤 데이터도 없기 때문에 -1을 반환합니다. 같은 이유로 요청정보 몸체의 문서타입을 구하는 Content Type 역시 null을 반환합니다.