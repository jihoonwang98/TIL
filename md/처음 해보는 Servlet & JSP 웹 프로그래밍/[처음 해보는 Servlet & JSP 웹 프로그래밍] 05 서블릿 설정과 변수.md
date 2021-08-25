# 05 서블릿 설정과 변수



### 5.1 서블릿 환경설정

서블릿은 웹에서 클라이언트로부터 요청받아서 실행되는 자바 프로그램으로서, 주로 서비스 처리를 위한 데이터 준비 작업과 메소드 호출 역할을 합니다. 이러한 작업을 하려면 서블릿 페이지 내에서가 아니라 서버에서 설정해야 하는 부분이 있는데요. 이번 절에서는 **서블릿이 작업하는 데 필요한 내용을 설정하는 방법**과 **서블릿에서 설정한 내용을 추출하여 사용하는 방법**에 대해 살펴보겠습니다.



#### 5.1.1 web.xml

서버에서 **서블릿 실행에 관한 정보**를 설정할 때는 web.xml에 \<servlet> 태그로 설정합니다. web.xml 파일은 서버가 시작할 때 웹서버가 사용하는 환경설정 파일입니다. 웹 애플리케이션 서비스 실행에 관한 전반적인 내용을 정의하는 환경설정 파일입니다. 서블릿 또한 웹 애플리케이션 서비스를 실행하기 위해 존재하는 파일이므로 web.xml에 정의합니다.



web.xml 파일에 서블릿을 정의하려면 다음과 같이 \<servlet> 태그를 추가합니다.

```xml
~생략~
<servlet>
	<servlet-name>initParam</servlet-name>
 	<servlet-class>com.edu.test.InitParamServlet</servlet-class>
 	<init-param>
 		<param-name>id</param-name>
 		<param-value>guest</param-value>
 	</init-param>
 	
 	<init-param>
  		<param-name>password</param-name>
  		<param-value>1004</param-value>
  	</init-param>
  	<load-on-startup>1</load-on-startup>
</servlet>
  
<servlet-mapping>
	<servlet-name>initParam</servlet-name>
  	<url-pattern>/initParamTest</url-pattern>
</servlet-mapping>
~생략~
```



**(1) \<servlet> 태그**

**\<servlet>** 태그는 설정하려는 <u>서블릿을 등록</u>합니다. \<servlet> 태그를 사용하면 반드시 하위 태그로 \<servlet-name>과 \<servlet-class>가 나와야 합니다. 그렇지 않으면 web.xml 오류가 발생하고 해당 웹 애플리케이션은 웹서버에 서비스 준비가 완료되지 않아서 클라이언트에 서비스되지 않습니다.



`<servlet-name>initParam</servlet-name>`



**\<servlet-name>**은 \<servlet> 태그를 사용할 때 반드시 설정해야 하는 태그로서 <u>서블릿의 이름</u>을 지정합니다. 여기에 지정한 이름은 이후 해당 서블릿을 참조할 때 사용합니다.



`<servlet-class>com.edu.test.InitParamServlet</servlet-class>`



**\<servlet-class>** 태그에는 <u>서블릿의 클래스 이름</u>을 지정합니다. \<servlet-class> 또한 \<servlet> 태그 사용 시 반드시 설정해야 하며, 클래스 이름을 패키지명과 함께 대소문자를 구분하여 정확하게 입력해야 합니다.



여기까지 보면 위 예의 경우 com.edu.test.InitParamServlet 클래스를 "initParam"이라는 이름의 서블릿으로 등록합니다.





**(2) \<init-param> 태그**

**\<init-param>** 태그는 <u>서블릿에 변수를 전달할 때</u> 사용합니다. \<servlet>의 하위 태그로서 필요할 때 선택해서 사용할 수 있습니다. 서블릿을 실행하면서 필요한 값을 외부에서 전달받아 실행할 수 있는데요. 서블릿 소스에서 직접 값을 지정해서 사용해도 되지만, 실행환경에 맞게 동적으로 값을 할당하고자 할 때 외부에서 값을 전달할 수 있습니다. <u>\<init-param> 태그는 반드시 \<param-name>과 \<param-value> 태그로 구성해야 합니다</u>.



`<param-name>password</param-name>`



\<param-name>은 \<init-param> 태그 사용 시 반드시 설정해야 하는 태그로서 변수의 이름을 지정합니다. 서블릿 측에서는 \<param-name>에 지정한 값을 이용하여 변수의 값을 추출합니다. 이때 변수의 이름은 대소문자, 철자를 구분하여 \<param-name>에서 정확히 일치하는 변수의 이름을 찾아서 값을 추출합니다.



`<param-value>1004</param-value>`



\<param-value>는 매핑되는 \<param-name>에 저장되는 변수의 값을 지정하는 태그입니다.





**(3) \<load-on-startup> 태그**

\<load-on-startup> 태그를 사용하면 웹 서비스가 시작될 때 서블릿 객체를 생성할 수 있습니다. 서블릿 객체가 메모리에 생성되는 시점은 클라이언트로부터 최초의 요청이 있을 때입니다. 서버에 서블릿 클래스 파일이 존재하더라도 클라이언트로부터 실행 요청이 없으면 객체가 메모리에 생성되지 않습니다.



그런데 어떤 서블릿은 요청부터 응답까지 서비스 처리에 전반적으로 관여하기도 합니다. 또는 클라이언트로부터 요청이 들어오기 전에 어떤 기능을 미리 준비하는 서블릿도 있습니다. 이처럼 **미리 준비되어 있다가 서비스 처리에 관여하는 서블릿은 클라이언트의 요청과 상관없이 웹 서비스가 시작될 때 객체를 생성하여 대기하고 있어야 합니다**. 이럴 때 사용하는 태그가 **\<load-on-startup>**입니다.



\<load-on-startup> 태그의 값으로 숫자를 지정하는데요. 이 숫자는 객체가 생성되는 우선순위를 의미합니다. **서버가 시작될 때 생성해야 하는 서블릿 객체가 여러 개일 때 \<load-on-startup> 태그의 값으로 우선순위를 정합니다**. 숫자값이 낮을수록 우선순위가 높습니다.





#### 5.1.2 ServletConfig

web.xml 파일에 \<servlet> 태그로 서블릿에 관한 환경설정을 하였습니다. \<servlet> 태그 안에 서블릿의 이름, 초기 파라미터, 객체 생성 여부 등의 서블릿에 관한 속성들을 설정할 수 있었습니다. 만일 서블릿 속성 중에서 \<init-param> 속성이 지정되었다면, web.xml에서 서블릿으로 파라미터를 전달한 것이므로 서블릿 페이지 내에서 \<init-param>의 변수를 추출해서 사용해야 합니다.



이처럼 web.xml의 **\<servlet> 태그에 설정한 정보를 서블릿 페이지 내에서 추출**할 때는 <u>ServletConfig 객체에서 제공하는 메소드</u>를 사용합니다. ServletConfig 객체는 서블릿이 실행될 때 자동으로 생성됩니다. 



다음은 서블릿 실행 시 ServletConfig 객체가 생성되는 시점을 그림으로 표현했습니다.



![](https://t1.daumcdn.net/cfile/tistory/99EABA505C6E51E712?download)

> 사진 출처: https://tmxhsk99.tistory.com/134



web.xml에 설정한 서블릿에 대한 정보를 추출하고자 할 때는 init() 메소드의 인자로 넘어오는 ServletConfig를 사용합니다. 5, 6 번은 서블릿의 요청이 최초 요청이든지 두 번째 이후의 요청이든지 상관없이 매번 실행되는 부분입니다. ServletConfig 객체는 서블릿이 최초로 요청이 들어왔을 때 한 번만 생성됩니다. 따라서 서블릿 페이지당 하나씩 존재하며 서블릿에 대한 정보를 처리하는 기능을 합니다.





다음 소스는 init() 메소드를 재정의하여 ServletConfig 객체를 init() 메소드의 인자로 받아서 web.xml 파일에 \<servlet> 태그로 설정한 서블릿 정보를 추출하여 확인하는 예제입니다.

```java
//InitParamServlet.java
package com.edu.test;

import java.io.*;
import javax.servlet.*;
import javax.servlet.http.*;


public class InitParamServlet extends HttpServlet {
	
	String id, password;
	
	@Override
	public void init(ServletConfig config) throws ServletException {
		id = config.getInitParameter("id");
		password = config.getInitParameter("password");
	}

	protected void doGet(HttpServletRequest req, HttpServletResponse rsp) throws ServletException, IOException {
		rsp.setContentType("text/html;charset=UTF-8");
		PrintWriter out = rsp.getWriter();
		out.print("<h2>서블릿 초기 추출 변수</h2>");
		out.print("<h3>ID : " + id + "</h3>");
		out.print("<h3>PASSWORD : " + password + " </h3>");
		out.close();
	}
}

```



getInitParameter()는 ServletConfig가 가지고 있는 메소드로서 web.xml 파일의 \<servlet> 속성 중에서 \<init-param>으로 지정한 변수의 값을 추출할 때 사용합니다. getInitParameter()의 인자값으로 \<param-name> 태그에 지정한 값을 넣어주면 그에 대응되는 \<param-value>의 값을 String 형태로 반환합니다.





**[실행 결과]**



![image-20210311165438191](C:\Users\user\AppData\Roaming\Typora\typora-user-images\image-20210311165438191.png)





**ServletConfig 또 다른 사용방법**

ServletConfig 객체를 사용하는 또 다른 방법은 HttpServlet 객체를 이용하는 방법입니다. HttpServlet의 부모 객체인 GenericServlet 객체가 상속하고 있는 인터페이스를 보면 ServletConfig가 있습니다. 따라서 우리가 만드는 서블릿 페이지에서 init() 메소드를 재정의하지 않고도 ServletConfig 객체의 메소드를 바로 사용할 수 있습니다.



```java
package com.edu.test;

import java.io.*;
import javax.servlet.http.*;
import javax.servlet.*;


public class ServletConfigTestServlet extends HttpServlet{
	@Override
	protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
		resp.setContentType("text/html;charset=UTF-8");
		PrintWriter out = resp.getWriter();
		String env = this.getInitParameter("charset");
		req.setCharacterEncoding(env);
		out.print("<h3> 이름: " +req.getParameter("name"));
		out.close();
	}
}
```







---

### 5.2 서블릿 변수



#### 5.2.1 서블릿 동시 요청

웹 프로그램을 개발하는 방식은 두 가지가 있습니다. **웹서버의 직접적인 호출로 실행하는 CGI 방식**과 **애플리케이션 서버가 실행하는 방식**입니다. 지금 학습하고 있는 서블릿은 서블릿 컨테이너라고 하는 웹 애플리케이션 서버가 실행하는 방식입니다. CGI와 서블릿 실행 방식은 서로 다른데요. 두 가지 방식의 차이점에 대해 알아보겠습니다.



**[CGI 프로그램 실행 방식]**

![CGI 프로그램 실행 방식](https://t1.daumcdn.net/cfile/tistory/99420C395C6E76B60D)

> 사진 출처: https://tmxhsk99.tistory.com/134



CGI 프로그램은 클라이언트로부터 **요청이 들어올 때마다 독립적인 프로세스가 만들어지며**, 메모리에는 프로세스를 실행하기 위한 데이터가 로딩됩니다. 



이러한, CGI 실행 방식은 여러 사용자 요청이 빈번한 성격의 웹 서비스로는 적합하지 않습니다. 왜냐하면 **클라이언트의 요청 수에 비례하여 프로세스가 만들어지고 그만큼 메모리 사용량도 함께 증가하기 때문입니다**. 따라서 대량의 트래픽이 발생할 수 있는 서비스에는 서버에 부하가 가중될 수 있습니다.





**[서블릿 실행 방식]**

![서블릿 실행 방식](https://t1.daumcdn.net/cfile/tistory/994FEA3F5C6E795027)

> 사진 출처: https://tmxhsk99.tistory.com/134





서블릿은 **최초 요청**일 때 <u>프로세스를 만들고</u> **서블릿 요청이 있을 때마다** <u>그 안에 스레드를 만들어서</u> service() 메소드를 실행하며, 두 번째 이후의 요청부터는 이미 만들어진 프로세스 안에 service() 메소드를 실행하기 위한 스레드만 새로 생성한 다음, 이 스레드 안에서 service() 메소드를 실행합니다.



이것이 CGI 프로그램의 실행 방식과의 차이점입니다. **CGI 프로그램**은 <u>요청할 때마다 독립적으로 프로세스를 만들어서 실행</u>하지만, **서블릿**은 <u>최초 요청 시 하나의 프로세스를 생성하고 이후부터는 이 프로세스 안에 스레드만 새로 생성</u>하여 실행합니다. 그래서 CGI 프로그램 실행 방식보다 서블릿 실행 방식이 서버 부하나 메모리 사용 면에서 훨씬 효율적입니다.





#### 5.2.2 서블릿 변수 특징

서블릿을 개발하면서 변수를 사용할 때 **멤버변수인지 지역변수인지 구분**하여 사용할 수 있어야 합니다. 하나의 서블릿에 여러 클라이언트가 공유해서 사용해야 하는 데이터는 **멤버변수**로 선언하며, 각각의 클라이언트가 독립적으로 사용해야 하는 데이터는 **지역변수**로 선언해야 합니다. 



서블릿은 하나의 프로세스를 생성한 다음, 동일한 서블릿을 요청하는 클라이언트에 대하여 공통적인 프로세스를 사용하며 service() 메소드를 실행하기 위한 스레드만 클라이언트별로 독립적으로 생성하여 실행합니다.



다음 그림은 앞에서 살펴본 서블릿 실행 방식의 그림에서 힙 메모리와 스택 메모리를 표시한 그림입니다.



![](https://t1.daumcdn.net/cfile/tistory/99228F4F5C6E7C5621)



**멤버변수**는 객체 생성 시 **힙** 메모리에 생성되며 서블릿을 실행하는 클라이언트들이 **공통**으로 사용합니다. 그러나 service() 메소드가 사용하는 **지역변수**는 **스택** 메모리에 생성되며, 클라이언트마다 **독립**적으로 사용하는데요. 이는 서블릿의 service() 메소드를 실행하는 스레드마다 스택 메모리가 독립적이기 때문입니다. 따라서 지역변수는 클라이언트마다 별개로 사용됩니다.



#### 5.2.3 서블릿 지역 변수

서블릿의 지역 변수는 여러 클라이언트가 동시에 요청했을 때, 요청마다 개별적으로 할당됩니다. 그러므로 각 클라이언트의 요청 수만큼 메모리 영역을 개별적으로 할당하여 지역적으로 처리할 때만 지역변수를 선언하여 활용합니다.



다음은 서블릿의 지역변수 특성을 확인하는 예제입니다. 새로운 서블릿 **LocalTestServlet**을 작성합니다.



```java
package com.edu.test;

import java.io.*;
import javax.servlet.http.*;
import javax.servlet.*;
import javax.servlet.annotation.WebServlet;


@WebServlet("/local")
public class LocalTestServlet extends HttpServlet{
	
	@Override
	protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
		int number = 0;
		String str = req.getParameter("msg");
		resp.setContentType("text/html;charset=UTF-8");
		PrintWriter out = resp.getWriter();
		out.println("<html><head><title>MultiThread Test</title></head>");
		out.println("<body><h2>처리 결과(지역 변수)</h2>");
		while(number++ < 10) {
			out.print(str + " : " + number + "<br>");
			out.flush();
			System.out.println(str + " : " + number);
			try {
				Thread.sleep(1000);
			} catch (Exception e) {
				System.out.println(e);
			}
		}
		
		out.println("<h2>Done " + str + " !!</h2>");
		out.println("</body></html>");
		out.close();
	}
}
```



- http://localhost:8080/edu/local?msg=one
- http://localhost:8080/edu/local?msg=two



위의 URL들에 동시에 접속하면 아래와 같은 결과가 나온다.



**[실행 결과]**

![image-20210311175209312](C:\Users\user\AppData\Roaming\Typora\typora-user-images\image-20210311175209312.png)



![image-20210311175218177](C:\Users\user\AppData\Roaming\Typora\typora-user-images\image-20210311175218177.png)



클라이언트가 보낸 질의 문자열 mst의 값을 추출하여 저장하는 변수 str을 **지역변수**로 선언했습니다. 그래서 각 스레드의 스택 영역에 별도의 str 변수가 **클라이언트별**로 만들어져 사용되므로 한 클라이언트가 str 변숫값을 수정해도 다른 클라이언트에게는 영향을 미치지 않습니다. 





#### 5.2.4 서블릿 멤버 변수

동일한 서블릿을 여러 클라이언트가 동시에 요청했을 때 서블릿 객체는 하나만 생성되어 멀티 스레드로 동작하므로 서블릿의 멤버변수는 여러 클라이언트가 공유하게 됩니다. 그러므로 각 클라이언트들의 동시 요청 수와 관계없이 하나의 메모리 공간을 할당하여 전역적으로 처리할 때만 멤버변수를 선언하여 활용합니다.



다음은 서블릿의 멤버변수 특성을 점검하는 예제입니다.



새로운 서블릿 **MemberTestServlet**을 작성합니다.

```java
package com.edu.test;

import java.io.*;
import javax.servlet.http.*;
import javax.servlet.*;
import javax.servlet.annotation.WebServlet;


@WebServlet("/member")
public class MemberTestServlet extends HttpServlet{
	String str; 
	
	@Override
	protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
		int number = 0;
		str = req.getParameter("msg");
		resp.setContentType("text/html;charset=UTF-8");
		PrintWriter out = resp.getWriter();
		out.println("<html><head><title>MultiThread Test</title></head>");
		out.println("<body><h2>처리 결과(지역 변수)</h2>");
		while(number++ < 10) {
			out.print(str + " : " + number + "<br>");
			out.flush();
			System.out.println(str + " : " + number);
			try {
				Thread.sleep(1000);
			} catch (Exception e) {
				System.out.println(e);
			}
		}
		
		out.println("<h2>Done " + str + " !!</h2>");
		out.println("</body></html>");
		out.close();
	}
}
```





- http://localhost:8080/edu/member?msg=one
- http://localhost:8080/edu/member?msg=two



위의 URL들에 동시에 접속하면 아래와 같은 결과가 나온다.



**[실행 결과]**



![image-20210311175909990](C:\Users\user\AppData\Roaming\Typora\typora-user-images\image-20210311175909990.png)



![image-20210311175918727](C:\Users\user\AppData\Roaming\Typora\typora-user-images\image-20210311175918727.png)



예제의 처리 결과를 보면 이전의 지역변수 예제와 다릅니다. str 변수는 멤버변수로 선언하였기 때문에 객체 생성 시 힙 영역에 만들어진 후 모든 클라이언트가 공유합니다.