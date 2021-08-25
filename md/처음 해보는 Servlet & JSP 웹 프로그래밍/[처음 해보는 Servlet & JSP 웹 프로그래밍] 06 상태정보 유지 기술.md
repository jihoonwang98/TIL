## 06 상태정보 유지 기술



### 6.1 상태정보 유지

웹에서 사용하는 HTTP 프로토콜의 통신 방식은 클라이언트와 서버 간의 연결을 **클라이언트로부터 요청이 있을 때마다 매번 새롭게 연결하는 방식**입니다. 요청이 있을 때마다 연결 작업이 새롭게 이루어지고, 서버가 클라이언트에게 응답을 보내는 즉시 끊어집니다. 이처럼 클라이언트와 서버 간에 연결상태가 유지되지 않는 통신 방식을 '**무상태(Stateless)**'라고 합니다.



무상태 통신 방식의 특징은 연결이 유지되지 않기 때문에 서비스를 요청한 **클라이언트에 대한 정보가 유지되지 않습니다**. 동일한 클라이언트의 요청이더라도 요청 단위로 연결이 맺어져서 이전의 작업은 지금의 연결 작업과는 아무런 관계가 없습니다. 그래서 클라이언트가 이전 요청에서의 처리결과를 계속해서 다른 요청에서도 사용하고 싶다면 **서버 측이든, 클라이언트 측이든 어딘가에 저장해서 정보를 유지해야 합니다**.



이처럼 클라이언트나 서버에 계속된 요청에서 사용할 수 있도록 저장한 정보들을 '**상태정보(State Information)**'라고 합니다.



다음은 상태정보 유지 기술들을 저장 위치와 저장 기간에 따라 분류한 것입니다.



#### 6.1.1 저장 위치 분류

정보를 저장하는 위치에 따라 상태정보 유지 기술을 분류하면 다음과 같습니다.



- **클라이언트 측에 저장 기술**

  웹에서 클라이언트는 웹 브라우저를 의미합니다. 그래서 클라이언트 측에 저장한다는 것은 웹 브라우저에 저장하는 것을 의미합니다. 웹 브라우저가 종료되기 전까지 정보를 유지할 수도 있고, 또는 웹 브라우저가 종료된 이후에도 계속 유지하게 할 수도 있습니다. 이처럼 클라이언트 측에 정보를 유지하는 기술에는 **쿠키(Cookie)**가 있습니다. 다음은 서블릿에서 쿠키 기능이 있는 객체입니다.

  

  - `javax.servlet.http.Cookie`





- **서버 측에 저장 기술**

  서버 측에 저장한다는 것은 서버의 **힙 메모리 영역**에 만들어진 객체에 상태정보를 저장(등록)하는 것을 의미합니다. 상태정보가 저장된 객체가 힙 메모리 영역에 존재하는 한 등록된 상태정보는 계속해서 사용할 수 있습니다. 다음은 서블릿에서 서버 측에 상태정보를 저장할 수 있는 객체입니다.

  

  - `javax.servlet.ServletContext`
  - `javax.servlet.http.HttpSession`
  - `javax.servlet.http.HttpServletRequest`





#### 6.1.2 유지 기간 분류

클라이언트나 서버 측에 저장된 정보들은 무한정 유지되는 것이 아니라 기간이 한정되어 있습니다. 정보가 저장되어 유지되는 기간을 기준으로 상태정보 유지 기술을 분류하면 다음과 같습니다.



- **웹 애플리케이션 단위 유지**

  웹 애플리케이션 단위로 유지한다는 것은 웹 애플리케이션이 서비스되고 있는 동안 유지하는 것을 의미합니다. 생명주기가 웹 애플리케이션과 같은 객체는 **ServletContext**입니다. ServletContext 객체는 웹 애플리케이션 서비스가 시작될 때 생성되고 종료될 때 소멸합니다. 그래서 ServletContext 객체에 상태정보를 저장하면 웹 애플리케이션이 서비스되고 있는 동안은 계속해서 사용할 수 있습니다.

  

  - `javax.servlet.ServletContext`





- **클라이언트 단위 유지**

  클라이언트 단위로 유지한다는 것은 클라이언트별로 구분해서 상태정보를 유지한다는 의미입니다. 예를 들어, A 클라이언트가 계속해서 사용하고자 하는 상태정보가 있는데, 이 상태정보를 다른 클라이언트는 사용할 수 없어야 할 때 클라이언트 단위로 유지해야 합니다. 대표적인 예가 로그인 작업입니다.

  

  **쿠키**는 클라이언트 측에 상태정보를 유지하는 기술이어서 당연히 클라이언트 단위로 상태정보가 유지되어 사용되며, 서버 측에서는 클라이언트별로 생성되는 **HttpSession** 객체를 통해 클라이언트 단위로 상태정보를 유지할 수 있습니다.

  

  - `javax.servlet.http.Cookie`

  - `javax.servlet.http.HttpSession`

    



- **요청 단위 유지**

  요청 단위로 유지한다는 것은 클라이언트의 서비스 요청 단위로 유지한다는 것입니다. 웹에서는 클라이언트로부터 요청이 있을 때마다 새로운 연결 작업이 이루어지며, 클라이언트로 응답이 이루어지면 연결은 해제됩니다. 이렇게 하나의 요청에서만 상태정보를 유지하고자 할 때 **HttpServletRequest** 객체를 통해 할 수 있습니다.

  

  - `javax.servlet.http.HttpServletRequest`





---

### 6.2 ServletContext

웹 애플리케이션 단위로 정보를 서버 쪽에 유지할 수 있는 방법은 **ServletContext** 객체를 사용하는 것입니다. 이번 절에서는 ServletContext의 기본 개념과 주요 메소드에 대해 알아보겠습니다.



#### 6.2.1 ServletContext 생성

ServletContext는 서블릿 컨테이너와 통신하기 위해서 사용되는 메소드를 지원하는 인터페이스입니다. 다음 그림과 같이 서블릿 컨테이너가 시작될 때 웹서버에 등록된 웹 애플리케이션 단위로 하나의 ServletContext 객체가 자동으로 생성됩니다. 그리고 웹 애플리케이션 서비스가 중지될 때 소멸합니다. 즉, ServletContext 객체는 웹 애플리케이션과 생명주기(life cycle)가 같습니다. ServletContext 객체를 간단하게 '웹 컨텍스트' 또는 '컨텍스트'라고 합니다.



![](https://t1.daumcdn.net/cfile/tistory/99ADAD4A5C733EB615)

> 사진 출처: https://tmxhsk99.tistory.com/135



WAS에 등록된 웹 애플리케이션 단위로 컨텍스트가 생성되는 이유는 서블릿 컨테이너가 웹 애플리케이션 단위로 모든 자원을 관리할 수 있게 하기 위해서입니다. 즉, 웹 애플리케이션 내에 있는 모든 서블릿 그리고 JSP 간에 정보를 공유할 수 있고, 서블릿 컨테이너에 대한 정보를 추출할 수 있게 하는 기술이 바로 ServletContext입니다.



웹 애플리케이션 서비스가 시작될 때 생성된 ServletContext 객체의 추출 방법은 메소드를 이용합니다. 메소드를 이용해 추출되는 ServletContext 객체는 웹 애플리케이션 단위로 사용하기 때문에 동일한 웹 애플리케이션에 존재하는 서블릿들은 동일한 ServletContext 객체를 사용하게 됩니다. ServletContext를 추출하는 메소드는 ServletConfig의 **getServletContext()**입니다.



서블릿을 실행할 때 최초의 요청이면 ServletConfig 객체가 생성되며 init() 메소드의 인자값으로 전달됩니다. ServletConfig 객체에서 현재 웹 어플리케이션에 할당된 ServletContext 객체의 주솟값을 추출할 수 있는 getServletContext() 메소드를 제공합니다.





**(1) init() 메소드를 재정의하여 추출하는 방법**

다음은 init() 메소드를 재정의하여 ServletConfig 객체를 인자로 받아 ServletContext 객체의 주솟값을 추출하는 예제입니다. ServletContextTest1Servlet이라는 이름의 새로운 서블릿을 작성합니다.



```java
package com.edu.test;

import java.io.*;
import javax.servlet.*;
import javax.servlet.http.*;
import javax.servlet.annotation.*;


@WebServlet("/context1")
public class ServletContextTest1Servlet extends HttpServlet{
	
	ServletContext sc;
	@Override
	public void init(ServletConfig config) throws ServletException {
		sc = config.getServletContext();
	}
	
	@Override
	protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
		resp.setContentType("text/html;charset=UTF-8");
		PrintWriter out = resp.getWriter();
		out.print("Context: " + sc);
		out.close();
	}

}

```





**(2) HttpServlet을 통해 추출하는 방법**

ServletContext 객체의 주솟값을 추출할 수 있는 두 번째 방법은 HttpServlet 객체를 이용하는 방법입니다. 서블릿을 구현할 때 반드시 상속해야 하는 HttpServlet 객체의 상위 클래스인 GenericServlet에서 ServletConfig를 상속받아 메소드를 재정의하고 있기 때문에 HttpServlet을 통해서도 getServletContext() 메소드를 사용할 수 있습니다.



다음은 HttpServlet을 통해서 ServletContext의 주솟값을 추출하는 예제입니다. ServletContextTest1Servlet 소스를 다음과 같이 수정합니다.



```java
~생략~
@Override
protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
	resp.setContentType("text/html;charset=UTF-8");
	PrintWriter out = resp.getWriter();
	ServletContext sc = this.getServletContext();
	out.print("Context : " + sc);
}
~생략
```







#### 6.2.2 ServletContext 변수

서버가 시작될 때 생성되는 ServletContext 객체는 웹 애플리케이션 단위로 생성되며, 동일한 웹 애플리케이션에 있는 모든 페이지는 동일한 ServletContext를 사용합니다. 그래서 ServletContext 객체가 가지고 있는 변수는 동일한 웹 애플리케이션에 속한 모든 페이지에서 사용할 수 있는 글로벌한 변수입니다. 웹 애플리케이션 단위로 사용할 수 있는 변수를 선언하고 활용하려면 web.xml에 변수를 선언한 다음, 서블릿에서 ServletContext 객체로 추출해서 사용합니다.



**(1) ServletContext 변수 설정**

ServletContext 객체에 변수를 설정하려면 web.xml 파일에서 \<servlet> 태그 위에 다음과 같은 코드를 추가합니다.



```xml
~생략~
<display-name>edu</display-name>
<context-param>
	<param-name>contextConfig</param-name>
  	<param-value>/WEB-INF/context.xml</param-value>
</context-param>

<servlet>
~생략~
```



- `<context-param>`: ServletContext 객체에 변수를 설정하고자 할 때 사용하는 태그

- `<param-name>`: 변수의 이름 설정
- `<param-value>`: 변수의 값 설정



**(2) ServletContext 변수 추출**

web.xml에 \<context-param>으로 설정한 변수를 추출할 때는 ServletContext 객체에서 제공하는 getInitParameter() 메소드를 활용합니다. getInitParameter() 메소드의 인자로 \<param-name> 값을 지정하면 \<param-value>에 지정한 값을 String 타입으로 반환합니다. ServletContextTest1Servlet을 다음과 같이 수정합니다.



```java
~생략~
@Override
protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
	resp.setContentType("text/html;charset=UTF-8");
	PrintWriter out = resp.getWriter();
	ServletContext sc = this.getServletContext();
	String location = sc.getInitParameter("contextConfig");
	out.print("location : " + location);
	out.close();
}
~생략~
```



**[참고] 개발 시 \<context-param> 태그의 실제 사용 용도**

우리가 하나의 서비스를 처리하기 위한 로직을 구현할때 기능별로 파일들을 분리하여 작업합니다. 이때 각 페이지 수준으로 환경설정 파일이 만들어집니다. 

웹서버가 사용하는 환경설정 파일은 web.xml이지만, 서비스 처리를 위해서는 각 페이지 수준의 환경설정 파일들도 읽어 들이도록 설정해야 하는데요. 환경설정 파일들은 서버의 웹 애플리케이션 서비스 시작과 동시에 읽어 들여야 합니다. 따라서 웹 애플리케이션 서비스 시작과 동시에 생성되는 ServletContext 객체에 환경설정 파일들에 대한 정보를 변수로 전달하고, 실제 환경설정을 하는 페이지에서는 ServletContext 객체를 통해 전달받은 변수를 사용해 환경설정 파일을 찾아가 설정 작업을 진행합니다.

예를 들면, 다음과 같습니다.

```xml
~생략~
<context-param>
	<param-name>contextConfigLocation</param-name>
	<param-value>
		/WEB-INF/presentation_layer.xml,
		/WEB-INF/service_layer.xml,
		/WEB-INF/persistent.xml,
	</param-value>
</context-param>
~생략~
```







#### 6.2.3 서버 정보 추출

웹서버는 웹 애플리케이션 단위로 정보를 나누어 관리하며 서비스합니다. 웹서버에는 웹 애플리케이션당 하나씩 ServletContext 객체가 생성되어 있습니다. 웹 애플리케이션 단위로 만들어진 ServletContext 객체를 통해 웹 애플리케이션에 관한 정보를 추출할 수 있습니다.



다음은 ServletContext 객체에서 웹 애플리케이션에 관한 정보를 추출하는 메소드를 활용한 예제입니다. ServletContextTest2Servlet이라는 이름의 새로운 서블릿을 작성합니다.



```java
package com.edu.test;

import java.io.*;
import javax.servlet.*;
import javax.servlet.http.*;
import javax.servlet.annotation.*;

@WebServlet("/context2")
public class ServletContextTest2Servlet extends HttpServlet{
	@Override
	protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
		resp.setContentType("text/html;charset=UTF-8");
		PrintWriter out = resp.getWriter();
		
		ServletContext sc = this.getServletContext();
		
		out.print("서블릿 버전 : " + sc.getMajorVersion() + "." + sc.getMinorVersion());
		out.print("<br>서버 정보 : " + sc.getServerInfo());
		out.print("<br>웹 애플리케이션 경로 : " + sc.getContextPath());
		out.print("<br>웹 애플리케이션 이름 : " + sc.getServletContextName());
		out.print("<br>파일 실제 경로 : " + sc.getRealPath("/name.html"));
		sc.log("로그 기록!!");
		
		out.close();
		
	}
	
}
```













#### 6.2.4 웹 애플리케이션 단위 정보 공유

ServletContext 객체는 웹 애플리케이션 단위로 사용되는 객체입니다. 즉, 동일한 웹 애플리케이션 안에 있는 모든 페이지에서 동일한 ServletContext 객체를 사용합니다. 그래서 ServletContext 객체를 이용하여 웹 애플리케이션 단위로 정보를 유지함으로써 공유할 수 있는 것입니다.



이처럼 여러 페이지 간에 데이터를 공유하기 위해 사용하는 메소드들은 다음과 같습니다.



- `void setAttribute(String name, Object value)`

  웹 애플리케이션 범위에서 공유할 데이터를 ServletContext 객체에 **등록**하는 메소드입니다. 



- `Object getAttribute(String name)`

  ServletContext 객체에 등록한 데이터를 추출하는 메소드입니다. 인자값으로는 찾으려는 데이터의 등록된 이름을 지정합니다. 동일한 이름으로 등록된 데이터를 찾아서 값을 반환해줍니다. getAttribute() 메소드로 데이터를 추출한 다음에는 항상 원래 데이터 타입으로 캐스팅해서 사용합니다.



- `void removeAttribute(String name)`

  ServletContext 객체에 setAttribute() 메소드로 등록한 데이터를 삭제합니다. 인자값으로 삭제할 데이터의 등록된 이름을 지정합니다.



다음은 ServletContext 객체를 활용하여 데이터를 공유하는 예제입니다. **ShareObject**라는 이름으로 새 클래스를 생성한 다음, 웹 애플리케이션 범위 내에서 공유할 객체를 생성합니다.



```java
// ShareObject.java
package com.edu.test;

public class ShareObject {
	private int count;
	private String str;
	
	public int getCount() {
		return count;
	}

	public void setCount(int count) {
		this.count = count;
	}
	
	public String getStr() {
		return str;
	}
	
	public void setStr(String str) {
		this.str = str;
	}
}

```



다음으로 ServletContext 객체에 데이터를 등록하는 서블릿을 작성합니다.

```java
//ServletContextTest3Servlet.java
package com.edu.test;

import java.io.*;
import javax.servlet.*;
import javax.servlet.annotation.*;
import javax.servlet.http.*;

@WebServlet("/context3")
public class ServletContextTest3Servlet extends HttpServlet{
	@Override
	protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
		resp.setContentType("text/html;charset=UTF-8");
		PrintWriter out = resp.getWriter();
		
		ServletContext sc = this.getServletContext();
		
		ShareObject obj1 = new ShareObject();
		obj1.setCount(100);
		obj1.setStr("객체 공유 테스트 - 1");
		sc.setAttribute("data1", obj1);
		
		ShareObject obj2 = new ShareObject();
		obj2.setCount(200);
		obj2.setStr("객체 공유 테스트 - 2");
		sc.setAttribute("data2", obj2);
		
		out.print("ServletContext 객체에 데이터 등록을 하였습니다.");
		out.close();
	}
}

```



이어서 ServletContextTest4Servlet이라는 이름으로 ServletContext 객체에 등록된 데이터를 추출하는 서블릿을 작성합니다.



```java
//ServletContextTest4Servlet.java

package com.edu.test;

import java.io.*;
import javax.servlet.*;
import javax.servlet.annotation.*;
import javax.servlet.http.*;

@WebServlet("/context4")
public class ServletContextTest4Servlet extends HttpServlet{

	@Override
	protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
		resp.setContentType("text/html;charset=UTF-8");
		PrintWriter out = resp.getWriter();
		
		ServletContext sc = this.getServletContext();
		
		ShareObject obj1 = (ShareObject) sc.getAttribute("data1");
		ShareObject obj2 = (ShareObject) sc.getAttribute("data2");
		
		out.print("DATA 1: " + obj1.getCount() + " , " + obj1.getStr() + "<br>");
		out.print("DATA 2 : " + obj2.getCount() + " , " + obj2.getStr() + "<br>");
		out.close();
		
	}
}
```





---

### 6.3 쿠키(Cookie)

웹 서비스 중에는 클라이언트 단위로 상태정보를 유지해야 하는 상황이 많습니다. 클라이언트 단위로 상태정보를 유지하게 하려면 **쿠키와 세션**을 이용합니다. 



쿠키와 세션은 다음과 같은 기준에 따라 선택하여 사용합니다.

- 상태정보의 유지 기간이 브라우저가 종료될 때까지인가의 여부
- 유지하려는 정보의 저장 위치(서버, 클라이언트)
- 유지하려는 정보가 공개되어도 되는지의 여부





**쿠키와 세션의 차이점**

| 구분             | 쿠키       | 세션                         |
| ---------------- | ---------- | ---------------------------- |
| 저장 위치        | 클라이언트 | 서버                         |
| 저장 데이터 타입 | 텍스트     | 객체                         |
| 저장 데이터 크기 | 제한 있음  | 서버에서 수용할 수 있는 만큼 |





#### 6.3.1 쿠키 속성

**쿠키**란, <u>서버가 클라이언트에 저장하는 정보</u>로서 클라이언트 쪽에 필요한 정보를 저장해놓고 필요할 때 추출하는 것을 지원하는 기술입니다. 클라이언트와의 연결이 끊어져도 클라이언트마다 개별적으로 상태정보를 유지하고자 할 때 쿠키 기술을 활용할 수 있습니다.



클라이언트에 저장된 쿠키 정보는 이후 다시 서버에 방문할 때 자동으로 요청정보의 헤더 안에 포함되어 전달됩니다. 쿠키는 name과 value로 구성된 정보로서 사용 목적에 따라 적절한 name과 value를 지정하고, 필요에 따라서는 쿠키의 유지 시간, 유효 디렉터리, 유효 도메인 등의 속성을 함께 지정할 수 있습니다.



쿠키는 URL 이름과 만료 날짜, 유효 경로 그리고 보안 필드와 같은 여러 항목으로 나누어집니다. 이 정보는 실제로 요청정보의 헤더를 통해 전달되는데 쿠키 설정에 관한 내용을 정리하면 다음과 같습니다.

- 쿠키를 설정할 때는 name=value 형식으로 구성되며, 쿠키 정보 외에도 expire, path, domain 그리고 secure 등 여러 속성을 선택적으로 추가하여 지정할 수 있다.
- 쿠키의 name은 ASCII 문자만을 사용해야 하며 한 번 설정된 쿠키의 name은 수정할 수 없다.
- "expire=날짜" 속성은 쿠키의 유지 시간이다. 유지 시간이 없는 쿠키는 쿠키를 설정받은 브라우저가 실행되는 동안만 유효하다.
- "path=경로" 속성은 클라이언트에 저장된 쿠키가 전달되는 서버의 유효 디렉토리를 지정하는 속성이다. "path=/"은 서버의 모든 디렉토리라는 의미로 해석되어 적용된다. 이 속성이 생략되면 쿠키를 설정하는 파일이 존재하는 디렉토리가 유효 디렉토리가 된다.
- "domain=서버정보" 속성은 클라이언트에 저장된 쿠키가 전달되는 유효 서버를 지정하는 속성이다. 생략되면 쿠키를 설정하는 서버가 유효 서버가 된다.
- secure 속성이 설정되면 클라이언트에서 HTTPS나 SSL과 같은 보안 프로토콜로 요청할 때만 서버에 전송된다.





#### 6.3.2 쿠키 생성

서블릿에서는 쿠키를 설정하고 전송하며 전송된 쿠키를 추출하는 기능을 API에서 지원하고 있어서 간단하게 처리할 수 있습니다. 



- **쿠키 생성: Cookie(String name, String value)**

  쿠키를 생성하려면 **javax.servlet.http.Cookie** 객체를 생성합니다.



- **쿠키 유효 시간 설정: setMaxAge(int expiry)**

  클라이언트로 전송되는 **쿠키의 유효 시간을 설정**할 때는 Cookie 객체의 setMaxAge() 메소드를 사용합니다. 인자값으로 정수를 지정하며, 이 값은 Cookie의 유효 시간의 초(second)를 의미합니다.

  만약 정숫값을 0으로 지정하면 쿠키 삭제를 의미하고, 음수값을 지정하면 쿠키가 클라이언트로 전송된 후 브라우저가 종료되면 쿠키도 자동으로 삭제됩니다. **개발 시 setMaxAge() 메소드로 유효 시간을 지정하지 않은 쿠키도 음수값이 적용**되어 브라우저가 전송받은 후 브라우저가 종료되면 쿠키도 함께 삭제됩니다.



- **쿠키 경로 설정: setPath(String uri)**

  현재 접속 중인 서버에서 이전에 클라이언트에게 전송한 쿠키가 있으면 기본적으로 요청정보 헤더 안에 쿠키가 포함되어 서버 쪽으로 전송됩니다. 서버의 모든 요청에 대하여 쿠키가 서버 쪽으로 전송되는 것이 아니라, **특정 경로의 요청에서만 쿠키를 전송하고자 할 때** setPath() 메소드를 사용하여 경로를 지정할 수 있습니다. setPath() 메소드의 인자값으로 경로를 지정하면, <u>지정된 경로와 그것의 하위 경로의 요청</u>에 대해서만 클라이언트로부터 쿠키가 전송됩니다.



- **쿠키 도메인 설정: setDomain(String domain)**

  쿠키는 기본적으로 전송된 서버에서만 읽어 들여 사용할 수 있습니다. 그런데 어떤 웹 서비스는 하나의 서버에서만 전체 서비스를 하는 것이 아니라, 여러 대의 서버가 연결되어 서비스를 처리합니다. 이러한 웹 서비스에서는 **쿠키의 도메인 설정을 통해 하나의 서버에서 클라이언트로 전송된 쿠키를 다른 서버에서 읽어 들여 사용**할 수 있습니다. 이때 사용하는 메소드가 setDomain()입니다.

  setDomain() 메소드의 인자 값으로 서버의 도메인을 지정하는데 "www.edu.com"처럼 지정하면 정확히 일치하는 도메인에서만 쿠키를 읽어 들일 수 있고, ".edu.com"처럼 지정하면 it.edu.com이나 math.edu.com처럼 ".edu.com"이 포함된 모든 도메인 서버에서 쿠키를 읽어 들일 수 있습니다.



- **쿠키 전송: addCookie(Cookie cookie)**

  **생성된 쿠키를 클라이언트로 보낼 때**는 HttpServletResponse 객체의 addCookie() 메소드를 이용합니다. addCookie() 메소드의 인자값으로 전송할 Cookie 객체를 설정하면 쿠키에 설정된 내용으로 클라이언트 쪽으로 쿠키가 전송됩니다.





이제 앞에서 살펴본 메소드들을 사용해서 쿠키를 생성하여 클라이언트로 전송하는 예제를 작성해 보겠습니다. CookieTest1Servlet이라는 이름으로 새로운 서블릿 파일을 작성합니다.



```java
//CookieTest1Servlet.java
package com.edu.test;

import java.io.*;
import javax.servlet.*;
import javax.servlet.annotation.*;
import javax.servlet.http.*;

@WebServlet("/cookie1")
public class CookieTest1Servlet extends HttpServlet{

	@Override
	protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
		resp.setContentType("text/html;charset=UTF-8");
		PrintWriter out = resp.getWriter();
		
		Cookie c1 = new Cookie("id", "guest");
		c1.setPath("/");
		resp.addCookie(c1);
		
		Cookie c2 = new Cookie("code", "0001");
		c2.setMaxAge(60*60*3);  // 3 hours
		c2.setPath("/"); 
		resp.addCookie(c2);
		
		Cookie c3 = new Cookie("subject", "java");
		c3.setMaxAge(60*60*24*10);  // 10 days
		c3.setPath("/");
		resp.addCookie(c3);
		
		out.print("쿠키 전송 완료");
		out.close();
		
	}
}
```





#### 6.3.3 쿠키 추출



- **쿠키 추출 : Cookie[] getCookies()**

  클라이언트로 전송된 쿠키를 서버 쪽에서 읽어 들이려면 HttpServletRequest 객체의 getCookies() 메소드를 이용합니다.



- **쿠키 검색: String getName()**

  HttpServletRequest 객체의 getCookies() 메소드는 서버가 전송한 쿠키를 한꺼번에 읽어 들여 반환하므로 반환된 쿠키 중에서 원하는 쿠키를 찾는 작업을 해야 합니다. 쿠키를 검색할 때는 쿠키의 이름을 가지고 검색하며, 쿠키의 이름만을 추출할 때는 Cookie 객체의 getName() 메소드를 사용합니다.



- **쿠키 값 추출: String getValue()**

  읽어 들인 쿠키 중에서 원하는 쿠키를 이름으로 검색하여 찾은 다음에는 쿠키의 값을 추출하여 사용해야 합니다. 쿠키의 값을 추출할 때는 Cookie 객체의 getValue() 메소드를 사용합니다.



이제 앞에서 살펴본 메소드들을 이용하여 앞에서 클라이언트에 보낸 쿠키들을 읽어 들인 후 쿠키 이름과 쿠키 값을 추출하여 출력하는 예제를 작성해보겠습니다. CookieTest2Servlet이라는 이름의 새로운 서블릿을 작성합니다.



```java
//CookieTest2Servlet.java

package com.edu.test;

import java.io.*;
import javax.servlet.*;
import javax.servlet.annotation.*;
import javax.servlet.http.*;

@WebServlet("/cookie2")
public class CookieTest2Servlet extends HttpServlet{

	@Override
	protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
		resp.setContentType("text/html;charset=UTF-8");
		PrintWriter out = resp.getWriter();
		
		Cookie[] list = req.getCookies();
		for(int i = 0; list != null && i < list.length; i++) {
			out.println(list[i].getName() + ": " + list[i].getValue());
		}
		out.close();
	}
}
```



앞에서 3개의 쿠키를 전송할 때 id=guest 쿠키는 유효 시간을 설정하지 않고 전송하였습니다. 유효 시간을 설정하지 않으면 브라우저가 가지고 있다가 브라우저가 종료될 때 삭제됩니다. 따라서 브라우저를 종료한 다음, cookie2를 다시 실행하면 id=guest 쿠키가 삭제되었음을 확인할 수 있습니다.





#### 6.3.4 클라이언트 단위 정보 공유

쿠키를 이용하여 클라이언트가 현재 사이트에 몇 번째 방문인지, 즉 방문 횟수를 기록 및 출력하는 예제를 작성해 보겠습니다. CookieTest3Servlet이라는 이름의 새로운 서블릿을 생성합니다.



```java
//CookieTest3Servlet.java

package com.edu.test;

import java.io.*;
import javax.servlet.*;
import javax.servlet.annotation.*;
import javax.servlet.http.*;

@WebServlet("/cookie3")
public class CookieTest3Servlet extends HttpServlet {

	@Override
	protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
		int cnt = 0;
		Cookie[] list = req.getCookies();
		for(int i = 0; list != null && i < list.length; i++) {
			if(list[i].getName().equals("count")) {
				cnt = Integer.parseInt(list[i].getValue());
				break;
			}
		}
		
		cnt++;
		Cookie c = new Cookie("count", cnt + "");
		c.setMaxAge(60 * 60 * 24 * 10);
		resp.addCookie(c);
		
		resp.setContentType("text/html;charset=UTF-8");
		PrintWriter out = resp.getWriter();
		out.print("<h1>방문 횟수: " +cnt);
		out.close();
	}
}
```







---

### 6.4 세션(Session)

HTTP 기반으로 동작하는 클라이언트가 서버에 정보를 요청할 때 생성되는 "상태정보"를 **세션**이라고 합니다. 세션은 HttpSession이라는 인터페이스 객체로 표현되며, HttpSession 객체는 HttpServletRequest의 getSession()이나 getSession(true) 메소드를 이용하여 생성할 수 있습니다. 



HttpSession 객체가 생성될 때는 요청을 보내온 클라이언트 정보, 요청 시간 정보 등을 조합한 세션 ID가 부여되며, 이 세션 ID는 클라이언트 측에 쿠키 기술로 저장됩니다. **HttpSession 객체는 서버에 생성되며, 클라이언트에는 세션 ID가 쿠키 기술로 저장되어 각 클라이언트에 대해 생성되는 HttpSession 객체를 클라이언트마다 개별적으로 유지 및 관리**합니다.



클라이언트마다 개별적으로 생성되어 유지되는 HttpSession 객체는 요청을 보내온 클라이언트와 서버간에 일정 시간(최대 시간은 브라우저가 살아 있는 동안) 동안 각 클라이언트의 상태정보를 서버에 저장하여 유지하고자 하는 목적으로 사용되는 객체입니다. 그리고 클라이언트마다 상태정보를 일정 시간 동안 개별적으로 유지하여 사용하는 기술을 '세션 트래킹(session tracking)'이라고 합니다.



**세션 트래킹 기능의 구현 순서**는 다음과 같습니다.

**1) **클라이언트를 위한 세션을 준비한다. 이전에 이 클라이언트를 위해 생성된 세션이 이미 존재하면 존재하는 세션을 추출하고, 그렇지 않으면 새로 생성한다. 세션이 새로 생성될 때는 고유한 ID가 하나 부여되며, 이 ID는 클라이언트에 쿠키 기술로 저장된다.

**2)** 유지하고자 하는 정보를 저장할 목적의 객체를 생성하여 세션에 등록한다.

**3)** 클라이언트가 요청을 전달할 때마다 세션에 등록된 정보를 담고 있는 객체를 추출하여 원하는 기능에 사용한다.

**4)** 세션이 더 이상 필요없는 시점에서 세션을 삭제한다.





#### 6.4.1 HttpSession 생성

HttpSession 객체를 사용하고자 할 때는 개발자가 수동으로 생성하는 것이 아니라, 메소드를 이용하여 생성하거나 기존의 HttpSession 객체의 주솟값을 추출하여 사용합니다. HttpSession 객체를 얻으려면 HttpServletRequest 객체의 다음 메소드를 사용합니다.



- **HttpServletRequest의 getSession()**

  클라이언트가 가지고 있는 세션 ID와 동일한 세션 객체를 찾아서 주솟값을 반환합니다. 만일 세션이 존재하지 않으면 새로운 HttpSession 객체를 생성하여 반환합니다.



- **HttpServletRequest의 getSession(boolean create)**

  클라이언트가 가지고 있는 세션 ID와 동일한 세션 객체를 찾아서 주솟값을 반환합니다. 만일 세션이 존재하지 않으면, 매개변수 create의 값이 true인지 false인지에 따라서 다르게 동작합니다. true이면 getSession() 메소드와 마찬가지로 새로운 HttpSession 객체를 생성하여 반환합니다. 그러나 false이면 새로운 HttpSession 객체를 생성하지 않고 null을 반환합니다.





#### 6.4.2 HttpSession 메소드

HttpServletRequest의 getSession() 또는 getSession(boolean) 메소드를 사용하여 HttpSession 객체를 얻어낸 후 다음과 같은 HttpSessio 객체에서 가지고 있는 메소드를 이용하여 클라이언트 단위로 정보를 유지하는 작업을 합니다. 즉 세션 트래킹 기술을 활용합니다.



| 접근자 & 반환형    | 메소드                                  | 기능                                                         |
| ------------------ | --------------------------------------- | ------------------------------------------------------------ |
| public Object      | getAttribute(String name)               | HttpSession 객체에 등록된 정보 중 getAttribute() 메소드의 인자값으로 지정된 데이터의 값을 반환한다. |
| public Enumeration | getAttributeNames()                     | HttpSession 객체에 등록되어 있는 모든 정보의 이름만을 반환한다. |
| public String      | getId()                                 | HttpSession 객체에 지정된 세션 ID를 반환한다.                |
| public long        | getCreationTime()                       | HttpSession 객체가 생성된 시간을 밀리초 단위로 반환한다.     |
| public long        | getLastAccessedTime()                   | 클라이언트 요청이 마지막으로 시도된 시간을 밀리초 단위로 반환한다. |
| public int         | getMaxInactiveInterval()                | 클라이언트의 요청이 없을 때 서버가 현재의 세션을 언제까지 유지할지를 초 단위로 반환한다. 기본 유효 시간은 30분으로 지정되어 있다. |
| public void        | invalidate()                            | 현재의 세션을 삭제한다.                                      |
| public boolean     | isNew()                                 | 서버 측에서 새로운 HttpSession 객체를 생성한 경우에는 true를 반환하고, 기존 세션이 유지되고 있는 경우라면 false를 반환한다. |
| public void        | setAttribute(String name, Object value) | HttpSession 객체에 name으로 지정된 이름으로 value 값을 등록한다. |
| public void        | removeAttribute(String name)            | HttpSession 객체에서 name으로 지정된 객체를 삭제한다.        |
| public void        | setMaxInactiveInterval(int second)      | HttpSession 객체의 유지 시간을 설정한다. 지정된 시간이 지나면 HttpSession 객체는 자동 삭제된다. |



앞에서 살펴본 HttpSession 객체에 관한 메소드를 테스트하는 예제를 작성해보겠습니다. SessionTestServlet이라는 이름으로 새로운 서블릿 파일을 작성합니다.



```java
// SessionTestServlet.java

package com.edu.test;

import java.io.*;
import javax.servlet.*;
import javax.servlet.annotation.*;
import javax.servlet.http.*;

@WebServlet("/sessionTest")
public class SessionTestServlet extends HttpServlet{

	@Override
	protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
		resp.setContentType("text/html;charset=UTF-8");
		PrintWriter out = resp.getWriter();
		String param = req.getParameter("p");
		String msg = null;
		HttpSession session = null;
		
		if(param.equals("create")) {
			session = req.getSession();
			if(session.isNew()) {
				msg = "새로운 세션 객체가 생성됨";
			} else {
				msg = "기존의 세션 객체가 리턴됨";
			}
		} else if(param.equals("delete")) {
			session = req.getSession(false);
			if(session != null) {
				session.invalidate();
				msg = "세션 객체 삭제 작업 완료";
			} else {
				msg = "삭제할 세션 존재하지 않음";
			}
		} else if(param.equals("add")) {
			session = req.getSession(true);
			session.setAttribute("msg", "메시지입니다");
			msg = "세션 객체에 데이터 등록 완료";
		} else if(param.equals("get")) {
			session = req.getSession(false);
			if(session != null) {
				String str = (String) session.getAttribute("msg");
				msg = str;
			} else {
				msg = "데이터를 추출할 세션 객체 존재하지 않음";
			}
		} else if (param.equals("remove")) {
			session = req.getSession(false);
			if(session != null) {
				session.removeAttribute("msg");
				msg = "세션 객체 데이터 삭제 완료";
			} else {
				msg = "데이터를 삭제할 세션 객체 존재하지 않음";
			}
		} else if (param.equals("replace")) {
			session = req.getSession();
			session.setAttribute("msg", "새로운 메시지입니다");
			msg = "세션 객체에 데이터 등록 완료";
		}
		
		out.print("처리 결과: " + msg);
		out.close();
	}
}
```





#### 6.4.3 클라이언트 단위 정보 공유

세션 트래킹 기술을 활용해서 구현해야 하는 대표적인 작업이 로그인/로그아웃 기능입니다. HttpSession 객체를 이용하여 로그인과 로그아웃 처리 예제를 작성해보겠습니다.



```html
//logInOut.html
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>회원 인증</title>
</head>
<body>
	<form action="logProc" method="post">
		ID: <input type="text" name="id"><br>
		비밀번호: <input type="password" name="pwd"><br>
		<input type="submit" value="로그인">		
	</form>
	
	<a href="logProc">로그아웃</a>
</body>
</html>
```





```java
//LogInOutServlet.java
package com.edu.test;

import java.io.*;
import javax.servlet.*;
import javax.servlet.annotation.*;
import javax.servlet.http.*;

@WebServlet("/logProc")
public class LogInOutServlet extends HttpServlet{
	
	@Override
	protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
		resp.setContentType("text/html;charset=UTF-8");
		PrintWriter out = resp.getWriter();
		String id = req.getParameter("id");
		String pwd = req.getParameter("pwd");
		
		if(id.isEmpty() || pwd.isEmpty()) {
			out.print("ID 또는 비밀번호를 입력해주세요");
			return;
		}
		
		HttpSession session = req.getSession();
		if(session.isNew() || session.getAttribute("id") == null) { 
			session.setAttribute("id", id);
			out.print("로그인을 완료하였습니다.");
		} else {
			out.print("현재 로그인 상태입니다.");
		}
	}
	
	@Override
	protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
		resp.setContentType("text/html;charset=UTF-8");
		PrintWriter out = resp.getWriter();
		HttpSession session = req.getSession(false);
		
		if(session != null && session.getAttribute("id") != null) {
			session.invalidate();
			out.print("로그아웃 작업 완료");
		} else {
			out.print("현재 로그인 상태가 아님");
		}
		
		out.close();
	}
}
```





---

### 6.5 HttpServletRequest

이번 절에서 HttpServletRequest 객체를 사용하여 여러 페이지 간에 정보를 공유하는 방법을 학습하겠습니다. 



HttpServletRequest 객체에 여러 페이지에서 공유할 정보를 저장한다 하더라도 service() 메소드가 종료되는 시점에 HttpServletRequest 객체도 소멸할 텐데 그곳에 정보를 저장하는 의미가 있을까요?



HttpServletRequest 객체는 하나의 서블릿 페이지가 실행되는 동안에만 메모리에 존재하는 개체이기 때문에 HttpServletRequest를 통해 정보를 유지하는 것은 무의미하다고 생각할 겁니다.



그런데 아직 학습하지 않은 내용 중에 클라이언트가 서블릿 실행 요청을 했을 때 하나의 페이지만 실행되는 것이 아니라 여러 페이지가 실행될 수 있는 상황이 발생할 수 있습니다.







![](https://t1.daumcdn.net/cfile/tistory/990E5A335C73BB5020)

> 그림 출처: https://tmxhsk99.tistory.com/135





위 그림은 클라이언트의 요청 한 번으로 두 개의 페이지가 실행된 모습입니다. 클라이언트가 A 페이지를 요청했는데 A 페이지가 실행되면서 B 페이지로 이동하여 B 페이지도 실행된 모습입니다. 즉, 한 번의 클라이언트 요청으로 여러 페이지가 실행된 것입니다. HttpServletRequest 객체를 통해 정보를 유지하는 예는 이처럼 **한 번의 요청으로 실행된 페이지 간에 데이터를 공유하고자 할 때** 사용하는 방법입니다.



위의 그림처럼 클라이언트가 A 페이지를 요청하고, A 페이지에서 B 페이지로 이동했을 때는 A와 B 페이지가 동일한 HttpServletRequest와 HttpServletResponse 객체를 사용하지만, 클라이언트가 B 페이지를 직접 실행 요청했을 때는 새로운 객체가 생성되므로 A 페이지와는 상관없는 객체를 사용하게 됩니다. 그래서 이때는 HttpServletRequest를 통해 정보를 공유할 수 없습니다.



HttpServletRequest 객체를 통한 정보 공유는 동일한 요청에서 실행된 페이지끼리만 이루어지며, 이때는 클라이언트가 요청한 페이지에서 다른 페이지로 이동해야 하는데, <u>클라이언트가 요청한 페이지가 실행되다가 다른 페이지로 이동하는 것</u>을 **'요청 재지정'**이라고 합니다. 즉, 클라이언트로부터의 요청에 대하여 서버에 존재하는 다른 자원으로 요청을 재지정하는 것을 요청 재지정이라고 하는데요. 이때 다른 자원이란, HTML, 이미지, 서블릿 그리고 JSP 등 웹 애플리케이션을 구성하는 어떤 파일이든 대상이 될 수 있습니다. <u>클라이언트에서는 서버에 보낸 요청을 다른 자원으로 재지정하는 것을 알 수 없습니다</u>. 



요청 재지정 기능을 제공하는 객체는 다음 2가지가 있습니다.

- `HttpServletResponse`
- `RequestDispatcher`







#### 6.5.1 HttpServletResponse 요청 재지정

HttpServletResponse 객체에서 제공하는 메소드를 사용하여 요청을 재지정할 때는 **요청을 재지정하는 자원이 현재 자원과 동일한 웹 애플리케이션에 속하지 않아도 상관없고, 동일한 서버에 존재하지 않아도 상관없습니다**. 



HttpServletResponse 객체에서 제공하는 요청 재지정 메소드는 다음과 같습니다.

| 접근자 & 반환형 | 메소드                        | 기능                                                         |
| --------------- | ----------------------------- | ------------------------------------------------------------ |
| public void     | sendRedirect(String location) | location에 설정된 자원으로 요청을 재지원한다.                |
| public String   | encodeRedirectURL(String url) | url에 설정된 URL 문자열에 세션 ID 정보를 추가하여 요청을 재지정한다. |



sendRedirect() 메소드를 이용하여 다른 서버의 자원으로 이동하는 예제를 작성해보겠습니다. 먼저, WebContent 아래에 site.html 파일을 작성합니다.



```html
// site.html
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>포털 사이트/title>
</head>
<body>
	<form action="portalSite">
		<input type="radio" name="site" value="naver">네이버<br>
		<input type="radio" name="site" value="daum">다음<br>
		<input type="radio" name="site" value="zum">줌<br>
		<input type="radio" name="site" value="google">구글<br>
		<input type="submit" value="이동">
	</form>
</body>
</html>
```



```java
package com.edu.test;

import java.io.*;
import javax.servlet.*;
import javax.servlet.annotation.*;
import javax.servlet.http.*;

@WebServlet("/portalSite")
public class SendRedirectTestServlet extends HttpServlet{

	@Override
	protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
		String param = req.getParameter("site");
		if(param.equals("naver")) {
			resp.sendRedirect("http://www.naver.com");
		} else if(param.equals("daum")) {
			resp.sendRedirect("http://www.daum.net");
		} else if(param.equals("zum")) {
			resp.sendRedirect("http://zum.com");
		} else if(param.equals("google")) {
			resp.sendRedirect("http://www.google.com");
		}
		
	}
}
```



이처럼 sendRedirect() 메소드는 다른 페이지로 이동할 수 있는 메소드이며, 현재 서버가 아닌 다른 서버의 자원으로도 이동할 수 있습니다.



#### 6.5.2 RequestDispatcher 요청 재지정

RequestDispatcher 객체에서 제공하는 메소드를 사용하여 요청을 재지정할 때는 **요청을 재지정하는 자원이 반드시 현재 자원과 <u>동일한</u> 웹 애플리케이션에 있어야만 합니다**.



RequestDispatcher 객체에서 제공하는 요청 재지정 메소드는 다음과 같습니다.



| 접근자 & 반환형 | 메소드                                                    | 기능                                            |
| --------------- | --------------------------------------------------------- | ----------------------------------------------- |
| public void     | forward(ServletRequest request, ServletResponse response) | 요청을 다른 자원으로 넘긴다.                    |
| public void     | include(ServletRequest request, ServletResponse response) | 다른 자원의 처리 결과를 현재 페이지에 포함한다. |





**(1) RequestDispatcher 객체 생성**

인터페이스인 RequestDispatcher 객체를 생성할 때는 다음과 같이 팩토리 메소드를 사용합니다.



- ServletContext 객체에서 제공하는 메소드
  - `RequestDispatcher getNamedDispatcher(String name)`
  - `RequestDispatcher getRequestDispatcher(String path)`



- ServletRequest 객체에서 제공하는 메소드
  - `RequestDispatcher getRequestDispatcher(String path)`



요청을 재지정할 대상에 대한 정보를 path 형식, name 등 어떤 것으로 지정하는가만 다를 뿐 대상을 지정하면서 RequestDispatcher() 메소드에서는 path를 지정할 때 절대 경로뿐만 아니라 상대 경로도 가능하지만, ServletContext 객체의 팩토리 메소드에서는 절대 경로만 지정할 수 있습니다.





**(2) forward() 메소드: forward(ServletRequest request, ServletResponse response)**

RequestDispatcher 객체의 forward() 메소드는 클라이언트의 요청으로 생성되는 HttpServletRequest와 HttpServletResponse 객체를 다른 자원에 전달하고 수행 제어를 완전히 넘겨서 다른 자원의 수행 결과를 클라이언트로 응답하도록 하는 기능의 메소드입니다. 다음 그림과 같이 클라이언트로부터의 요청을 다른 자원에 넘겨서 다른 자원으로 요청을 재지정합니다.



![](https://static.javatpoint.com/images/forward.JPG)



다음은 RequestDispatcher 객체의 forward() 메소드를 활용하는 예제입니다. DispatcherTest1Servlet이라는 이름의 새로운 서블릿을 작성합니다.



```java
//DispatcherTest1Servlet.java

package com.edu.test;

import java.io.*;
import javax.servlet.*;
import javax.servlet.annotation.*;
import javax.servlet.http.*;


@WebServlet("/dispatcher")
public class DispatcherTest1Servlet extends HttpServlet{
	
	@Override
	protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
		resp.setContentType("text/html;charset=UTF-8");
		PrintWriter out = resp.getWriter();
		out.print("<h3> Dispatcher Test1의 수행결과</h3>");
		
		ServletContext sc = this.getServletContext();
		RequestDispatcher rd = sc.getRequestDispatcher("/dispatcher2");
		rd.forward(req, resp);
		
		out.close();
	}

}
```



```java
//DispatcherTest2Servlet.java
package com.edu.test;

import java.io.*;
import javax.servlet.*;
import javax.servlet.annotation.*;
import javax.servlet.http.*;

@WebServlet("/dispatcher2")
public class DispatcherTest2Servlet extends HttpServlet{
	
	@Override
	protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
		resp.setContentType("text/html;charset=UTF-8");
		PrintWriter out = resp.getWriter();
		out.print("<h3> Dispatcher Test2의 수행 결과</h3>");
		out.close();
	}
	
}
```



**[실행 결과]**



![image-20210312151852105](C:\Users\user\AppData\Roaming\Typora\typora-user-images\image-20210312151852105.png)









**(3) include() 메소드: include(ServletRequest request, ServletResponse response)**

include()는 클라이언트의 요청으로 생성되는 HttpServletRequest와 HttpServletResponse 객체를 다른 자원에 전달하고 수행한 다음, 그 결과를 클라이언트에서 요청한 서블릿 내에 포함하여 클라이언트로 응답하는 기능의 메소드입니다.





![](https://static.javatpoint.com/images/include.JPG)

다음은 RequestDispatcher 객체의 include() 메소드를 활용하는 예제입니다. 이전에 작성된 DispatcherTest1Servlet.java 소스에서 rd.forward(req, resp); 부분만 **rd.include(req, resp);**로 변경합니다.



```java
//DispatcherTest1Servlet.java
~생략~
	ServletContext sc = this.getServletContext();
	RequestDispatcher rd = sc.getRequestDispatcher("/dispatcher2");
	rd.include(req, resp);
~생략~
```





**[실행 결과]**



![image-20210312151951297](C:\Users\user\AppData\Roaming\Typora\typora-user-images\image-20210312151951297.png)







**(4) Request 단위 정보 공유**

RequestDispatcher 객체의 요청을 재지정하는 forward()나 include() 메소드를 이용해 다른 페이지로 이동할 때는 현재 페이지가 사용하는 HttpServletRequest와 HttpServletResponse 객체를 그대로 전달하면서 이동하므로 이전 페이지나 이동한 페이지나 같은 객체를 사용합니다. 그래서 한 번의 요청으로 실행된 페이지끼리 정보를 공유하고자 할 때 HttpServletRequest를 통해 공유할 수 있습니다.



다음은 HttpServletRequest를 통해 동일한 요청 페이지 간에 정보를 공유하는 예제입니다. 먼저, bookInput.html를 작성합니다.

```html
//bookInput.html
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>책 등록</title>
</head>
<body>
<form action="bookReg" method="post">
	책제목: <input type="text" name="title"><br>
	책저자: <input type="text" name="author"><br>
	출판사: <input type="text" name="publisher"><br>
	<input type="submit" value="등록"/>
</form>
</body>
</html>
```



Book이라는 이름의 새로운 일반 자바 소스를 작성합니다. HTML 파일에서 입력한 데이터 값을 저장할 목적으로 만드는 객체입니다. 객체의 멤버변수가 bookInput.html에서 입력받는 값과 일치하는 것을 확인할 수 있습니다.

```java
//Book.java
package com.edu.test;

public class Book {
	private String title;
	private String author;
	private String publisher;
	
	public String getTitle() {
		return title;
	}
	public void setTitle(String title) {
		this.title = title;
	}
	public String getAuthor() {
		return author;
	}
	public void setAuthor(String author) {
		this.author = author;
	}
	public String getPublisher() {
		return publisher;
	}
	public void setPublisher(String publisher) {
		this.publisher = publisher;
	}
}
```





BookTest1Servlet이라는 이름의 서블릿 소스를 작성합니다.

```java
//BookTest1Servlet.java
package com.edu.test;

import java.io.*;
import javax.servlet.*;
import javax.servlet.annotation.*;
import javax.servlet.http.*;

@WebServlet("/bookReg")
public class BookTest1Servlet extends HttpServlet{
	
	@Override
	protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
		resp.setContentType("text/html;charset=UTF-8");
		PrintWriter out = resp.getWriter();
		
		req.setCharacterEncoding("UTF-8");
		
		String title = req.getParameter("title");
		String author = req.getParameter("author");
		String publisher = req.getParameter("publisher");
		
		Book book = new Book();
		book.setTitle(title);
		book.setAuthor(author);
		book.setPublisher(publisher);
		
		req.setAttribute("book", book);
		
		RequestDispatcher rd = req.getRequestDispatcher("bookOutput");
		rd.forward(req, resp);
		out.close();
	}
}
```



**[실행 결과]**



![image-20210312153433413](C:\Users\user\AppData\Roaming\Typora\typora-user-images\image-20210312153433413.png)



























