## 04 질의 문자열



### 4.1 질의 문자열의 개요

이번 절에서는 질의 문자열이 무엇인지와 인코딩 규칙, 전달 방식의 규칙 등을 알아보겠습니다.



#### 4.1.1 질의 문자열이란?

웹 클라이언트에서 웹서버에 정보를 요청할 때 정해진 방식으로 데이터를 전달할 수 있으며, 이때 사용하는 문자열을 **질의 문자열**이라고 합니다. 



대표적인 웹 클라이언트인 웹 브라우저에서 다양한 정보를 전달할 수 있도록 입력 양식을 제시할 수 있는데, 이때 사용되는 HTML 태그가 바로 **\<form>** 입니다. \<form> 태그는 웹 브라우저 화면에 버튼, 텍스트 필드, 체크 박스, 라디오 버튼과 같은 GUI 환경의 입출력 도구들을 출력하는 기능과 서버 프로그램의 URI를 설정하는 기능 등이 있습니다.  \<form> 태그의 기능을 알고 있어야 게시판 입력이나 회원가입, 회원인증 화면 등 사용자로부터 다양한 정보를 입력하게 하는 입력 양식을 웹 브라우저 화면에 출력할 수 있습니다.



#### 4.1.2 질의 문자열 전송 규칙

질의 문자열 작업을 하려면 먼저 사용자가 데이터를 입력할 수 있는 화면을 만들어야 합니다. 또한, 입력한 자료를 서버로 전달해야 하며, 서버에서는 전달받은 문자열들을 추출하여 사용자가 요청한 서비스를 처리해야 합니다.



그런데 우선 질의 문자열 작업을 본격적으로 하기 전에 우리가 알아야 하는 특성이 있습니다. 그것은 질의 문자열들이 클라이언트에서 서버 쪽으로 전달될 때의 특성인데요. 이 특성들을 알아야 올바른 화면을 구성할 수 있고, 서버로 전달된 문자열들을 처리할 수 있습니다. 클라이언트가 입력한 데이터는 그대로 서버로 전달되는 것이 아니라, 정해진 규칙으로 인코딩(encoding)되어 전달됩니다. 이 규칙은 요청방식과 관계없이 동일하게 적용됩니다.



질의 문자열이 클라이언트 쪽에서 네트워크를 통해 서버로 전달될 때 다음과 같은 규칙이 있습니다.



**1) name=value 형식으로 전달되며, 여러 개의 name=value 쌍이 있을 때는 &를 구분자로 사용한다.**

ex) id=guest&name=Amy



**2) 영문자, 숫자, 일부 특수문자는 그대로 전달되고, 이를 제외한 나머지 문자는 % 기호와 함께 16진수로 바뀌어 전달된다.**

ex) id=guest&name=%C8%AB%B1%E6%B5%BF



전달되는 질의 문자열에 영문자, 숫자, 일부 특수문자, 즉 아스키(ASCII) 문자코드에 해당하는 문자는 그대로 서버로 전달됩니다. 그러나 이 문자를 제외한 나머지 문자는 % 기호와 함께 16진수로 변환하여 전달됩니다. 따라서 서버 쪽에서는 변환된 문자코드에 대해 적절한 복원 처리를 해줘야 문자열이 깨지지 않습니다.



**3) 공백 문자는 + 기호로 변경되어 전달된다.**

ex) id=guest&name=John+Smith



질의 문자열에 공백 문자가 있을 때는 + 기호로 변환되어 전달되는데요. URL에는 공백을 포함할 수 없는 특성이 있습니다. 만약에 공백이 있다면 공백이 나타나기 바로 전까지를 하나의 URL로 인식하기 때문에 이를 막기 위해 + 기호를 써줍니다.





---

### 4.2 HTML 입력양식



#### 4.2.1 \<form> 태그

질의 문자열을 입력하거나 선택할 수 있는 화면을 작성할 때는 HTML의 \<form> 태그 단위로 작업합니다.



다음은 \<form> 태그를 사용하는 방법입니다. 

`<form action="서버프로그램 경로" method="요청 방식">`



- **action**: \<form> 태그의 action 속성에는 클라이언트가 \<form>과 \</form> 태그 사이에 입력한 질의 문자열들을 전달받아 처리할 서버 프로그램을 지정합니다. action 속성으로 지정할 수 있는 서버 측 프로그램으로는 이 책에서 다루는 Servlet이나 JSP 등이 있는데요. 해당 프로그램이 있는 서버의 경로를 지정합니다.

  

- **method**: method 속성값으로 GET이나 POST 등을 지정할 수 있는데요. 다른 값도 더 있지만, 일반적으로 GET이나 POST를 사용합니다. 





#### 4.2.2 입력 태그들

HTML에서 제공하는 몇 가지 입력 태그들의 예를 살펴보겠습니다.



**(1) 텍스트 입력상자(한 줄)**

`<input type="text" name="변수이름" maxlength="숫자" size="숫자" value="문자열">`



**(2) 체크박스**

`<input type="checkbox" name="변수이름" value="문자열">`



**(3) 라디오 버튼**

`<input type="radio" name="변수이름" value="문자열">`



**(4) 펼침목록**

```html
<select name="변수이름">
	<option value="문자열">항목이름
	<option value="문자열">항목이름
</select>
```



**(5) 텍스트 입력 상자(여러 줄)**

`<textarea cols="숫자" rows="숫자" name="변수 이름"> </textarea>`



**(6) 전송 버튼**

`<input type="submit" value="문자열"/>`



**(7) 초기화 버튼**

`<input type="reset" value="문자열" />`





#### 4.2.4 입력양식 작성



WebContent에 member.html의 이름으로 새로운 html 파일을 생성합니다.



```html
<!DOCTYPE html>
<html>
<head>
<meta charset="EUC-KR">
<title>Form Test</title>
</head>
<body>
	<h3>회원 정보</h3>
	<form action="queryTest" method="GET">
		ID : <input type="text" name="id"/> <br/>
		비밀번호 : <input type="password" name="pwd"/> <br/>
		이름 : <input type="text" name="name"/> <br/>
		취미 :
			<input type="checkbox" name="hobby" value="climbing" /> 등산
			<input type="checkbox" name="hobby" value="sports" /> 운동
			<input type="checkbox" name="hobby" value="reading" /> 독서
			<input type="checkbox" name="hobby" value="traveling" /> 여행 <br/>
		성별 : 
			<input type="radio" name="gender" value="male" /> 남자
			<input type="radio" name="gender" value="female" /> 여자 <br/>
		종교 :
			<select name="religion">
				<option value="Christianity"> 기독교
				<option value="Buddhism"> 불교
				<option value="Catholicism"> 천주교
				<option value="atheism"> 무교			
			</select> <br/>
		자기소개 : <br/>
			<textarea cols="30" rows="10" name="introduction"></textarea><br/>
		
		
		<input type="submit" value="전송"/>
		<input type="reset" value="지우기" />
	</form>

</body>
</html>
```





---

### 4.3 요청방식에 따른 처리



#### 4.3.1 GET 방식으로 처리

먼저 GET 방식으로 서비스를 요청하는 예제를 작성해보겠습니다. 앞에서 작성한 member.html 파일에서 \<form> 태그의 method 속성값을 "GET"으로 지정합니다.



`<form action="queryTest" method="GET">`



브라우저에서 member.html을 실행한 후 데이터들을 입력 및 선택한 후 <전송> 버튼을 클릭합니다. <전송> 버튼을 클릭하면 \<form> 태그의 action 속성에 지정한 프로그램이 실행됩니다. 우리는 아직 서버 프로그램을 만들지 않았으므로 "404 (Not found)" 에러를 응답받습니다. 



![image-20210311115605062](C:\Users\user\AppData\Roaming\Typora\typora-user-images\image-20210311115605062.png)



그런데 브라우저의 주소 줄을 확인해 보면 member.html의 폼에서 입력한 데이터들이 name=value&name=value 형태로 표시된 것을 알 수 있습니다. 이것이 GET 방식으로 질의 문자열을 전송했을 때 가장 대표적인 특징입니다.



GET 방식으로 질의 문자열을 보낼 때는 **데이터의 크기에 제한**이 있습니다. 왜냐하면, 서버에서 인식할 수 있는 URI 길이는 제한적이기 때문입니다. GET 방식은 데이터의 길이가 255 바이트 미만이고, 외부에 노출되어도 상관없는 데이터를 전달할 때 적합한 요청방식입니다. 



또한, GET 방식은 질의 문자열이 URI에 포함되어 전달되므로 질의 문자열들을 인코딩/디코딩하는 추가 작업이 없어 **처리속도 면에서 빠른** 장점이 있습니다.





지금까지 살펴본 **GET 요청방식의 특성**을 요약하면 다음과 같습니다.

- 전달되는 질의 문자열이 요청정보 헤더의 URI에 추가되어 전달된다.
- 전달되는 질의 문자열의 내용이 외부에 노출된다.
- 전달되는 질의 문자열의 길이가 제한적이다.
- 전달되는 질의 문자열의 인코딩/디코딩 작업이 필요없어서 처리속도가 빠르다.
- 전달되는 질의 문자열을 직접 URI에 추가할 수 있다.







그렇다면 **GET 방식으로 요청되는 상황**에 대해 알아봅시다.

- \<a> 태그를 클릭하여 요청하는 경우
- 브라우저 주소 줄에 URL을 입력하여 요청하는 경우
- \<form> 태그에서 method 속성을 생략하여 요청하는 경우









#### 4.3.2 POST 방식으로 처리

이번에는 요청방식을 POST 방식으로 변경하여 실행해봅시다. 그래서 GET 방식으로 요청했을 때와 어떤 차이가 있으며, POST 방식의 특징은 무엇인지 알아보겠습니다. 



member.html 파일에서 \<form> 태그의 action 속성값을 "POST"로 지정합니다.

`<form action="queryTest" method="POST">`



브라우저에서 member.html을 실행한 후 데이터들을 입력하고 전송을 클릭합니다.



![image-20210311120350194](C:\Users\user\AppData\Roaming\Typora\typora-user-images\image-20210311120350194.png)



GET 방식과 마찬가지로 아직 서버 프로그램이 없어서 에러를 응답하지만, 여기서는 웹 브라우저의 주소 줄을 확인합시다.



GET 방식과는 달리 질의 문자열은 안보이고, 클라이언트가 서버에 요청한 서버 프로그램의 경로만 보입니다. 이것이 GET과 POST 방식의 눈에 띄는 가장 큰 차이입니다.



POST 방식은 <u>질의 문자열이 요청정보의 몸체에 포함</u>됩니다. 따라서 외부에 노출되지 않고 서버에 전달되며, 질의 문자열의 길이에 제한도 없습니다. 이러한 특징에 따라 클라이언트에서 보내는 데이터가 외부로 노출되면 안 되거나, 데이터 크기가 클 때는 POST 방식을 사용합니다. 그리고 POST 방식은 클라이언트 측에서 요청정보의 몸체에 전달하는 질의 문자열들을 인코딩해서 보내고, 이를 전달받은 서버 측에서는 다시 디코딩하는 추가 작업이 필요합니다.



**POST 요청 방식**은 다음과 같은 특징이 있습니다.

- 전달되는 질의 문자열이 요청정보의 몸체에 포함되어 전달됨.
- 전달되는 질의 문자열이 외부에 노출되지 않음.
- 전달되는 질의 문자열의 길이에 제한이 없음.
- \<form> 태그를 사용해야만 요청할 수 있다.





---

### 4.4 서블릿 작성

웹에서는 클라이언트가 전달하는 모든 정보들은 HTTP의 요청정보에 포함되어 서버로 전달되어 처리됩니다. 그래서 서버 프로그램을 구현하면서 서비스를 요청한 클라이언트에 관한 정보가 필요하다면 HTTP의 요청정보에서 추출하여 사용합니다. 



이처럼 클라이언트가 보낸 정보들은 HTTP의 요청정보에 담겨서 전달되고, 서버에서 요청정보를 처리할 때 사용하는 객체는 HttpServletRequest입니다. 이번 절에서는 HttpServletRequest 객체를 사용해 클라이언트로부터 전달된 질의 문자열을 추출하는 방법을 알아보겠습니다.



#### 4.4.1 메소드 구현



QueryTestServlet.java 서블릿 소스 파일을 작성합니다.

```java
package com.edu.test;

import java.io.IOException;
import java.io.PrintWriter;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.*;

@WebServlet("/queryTest")
public class QueryTestServlet extends HttpServlet {

	@Override
	protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
		resp.setContentType("text/html;charset=UTF-8");
		PrintWriter out = resp.getWriter();
		out.print("<html><head><title>Query 문자열 테스트</title></head>");
		out.print("<body>");
		out.print("<h1>GET 방식으로 요청되었습니다.</h1>");
		out.print("</body></html>");
		out.close();
	}

	@Override
	protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
		resp.setContentType("text/html;charset=UTF-8");
		PrintWriter out = resp.getWriter();
		out.print("<html><head><title>Query 문자열 테스트</title></head>");
		out.print("<body>");
		out.print("<h1>POST 방식으로 요청되었습니다.</h1>");
		out.print("</body></html>");
		out.close();
	}
}

```





#### 4.4.2 서블릿 연결

QueryTestServlet.java 소스에서 doGet()과 doPost() 메소드를 재정의하여 서블릿 소스를 완성하였습니다. 이제 member.html과 QueryTestServlet을 연결해서 실행해보겠습니다. member.html에서 폼의 데이터를 전달해서 처리할 서버 프로그램으로 QueryTestServlet을 실행하려면 다음과 같이 member.html 소스에서 \<form> 태그의 action 속성에 QueryTestServlet과 매핑되는 정보를 지정합니다.



```html
~생략~
<body>
	<h3>회원 정보</h3>
	<form action="queryTest" method="get">
~생략~
```



action 속성값으로 "queryTest"를 지정했는데요. 이는 상대 경로로 주소(URI)를 지정한 것입니다. 현재 member.html 파일이 속한 웹 어플리케이션의 경로는 /edu이고, 파일은 /(루트) 디렉터리에 있으므로 실제 요청되는 URI는 /edu/queryTest입니다. action 속성값을 "queryTest"로 지정한 이유는 QueryTestServlet.java에서 서블릿 매핑정보를 @WebServlet("/queryTest")로 지정하였기 때문입니다.



QueryTestServlet이 속한 웹 애플리케이션의 경로는 /edu이고, @WebServlet 값을 "/queryTest"로 지정했으므로 실제 지정되는 URI는 /edu/queryTest입니다. 이는 member.html에서 \<form> 태그의 action 속성값과 일치합니다. 따라서 member.html에서 전송 버튼을 클릭하면 QueryTestServlet이 실행됩니다.



그럼, queryTest를 GET 방식과 POST 방식으로 모두 테스트해보겠습니다.



**GET 방식**

member.html에서 \<form> 태그의 method 속성값을 "get"으로 지정합니다.

`<form action="queryTest" method="get">`



member.html을 실행한 후 전송을 클릭합니다. QueryTestServlet의 doGet() 메소드가 호출됩니다.



![image-20210311123457513](C:\Users\user\AppData\Roaming\Typora\typora-user-images\image-20210311123457513.png)





**POST 방식**

member.html에서 \<form> 태그의 method 속성값을 "post"으로 지정합니다.

`<form action="queryTest" method="post">`



member.html을 실행한 후 전송을 클릭합니다. QueryTestServlet의 doPost() 메소드가 호출됩니다.



![image-20210311123857673](C:\Users\user\AppData\Roaming\Typora\typora-user-images\image-20210311123857673.png)





#### 4.4.3 질의 문자열 추출

질의 문자열은 어떠한 방식으로 요청하였든지 HTTP의 요청 정보에 포함되어 전달됩니다. HTTP의 요청 정보에 대한 값을 추출하는 기능을 가지고 있는 객체는 HttpServletRequest입니다. 질의 문자열 추출 또한 HttpServletRequest에서 제공하는 메소드를 이용합니다.





- **String getParameter(String name)**

  이 메소드는 질의 문자열로 넘어온 값을 하나씩 추출할 때 사용합니다. **질의 문자열에서 name이 중복되지 않고 유일하게 하나만 넘어올 때 사용합니다.**

  

  ex) id=guest&pwd=amy&age=23 -> name이 중복되지 않으므로 getParameter() 메소드를 씀.

  color=red&color=blue&color=yellow -> name이 중복되므로 getParameter() 메소드를 쓰지 않음.



- **String[] getParameterValues**

  이 메소드는 **같은 이름으로 여러 개의 변수가 전달되었을 때 한번에 모든 값을 추출**하여 String 타입의 배열로 받고 싶을 때 사용합니다.



- **String getQueryString()**

  이 메소드는 클라이언트가 전달한 **질의 문자열 전체를 하나의 String으로 추출**해줍니다. 그런데 이 메소드는 **GET 방식** 요청에서만 사용할 수 있습니다. 왜냐하면, HTTP 요청정보의 URI 정보에서 ? 다음에 나오는 문자열들을 추출해주기 때문입니다.



- **ServletInputStream getInputStream() throws IOException**

  이 메소드는 HTTP의 **요청정보 몸체와 연결된 입력스트림을 생성하여 반환**합니다. 서블릿 프로그램에서 요청정보 몸체에 있는 데이터들을 읽어오고 싶을 때 getInputStream()에서 반환한 ServletInputStream으로 읽어올 수 있습니다. **POST 방식의 질의 문자열 전체를 한번에 추출할 때 사용할 수 있는 메소드**입니다.



이와 같은 HttpServletRequest의 메소드들을 사용하여 질의 문자열을 추출하는 예제를 작성해 보겠습니다. 먼저 GET 방식으로 처리해보겠습니다.



```sql
// QueryTestServlet.java

~ 생략 ~
public class QueryTestServlet extends HttpServlet {

	@Override
	protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
		resp.setContentType("text/html;charset=UTF-8");
		PrintWriter out = resp.getWriter();
		
		String id = req.getParameter("id");
		String password = req.getParameter("pwd");
		String name = req.getParameter("name");
		String[] hobbies = req.getParameterValues("hobby");
		String gender = req.getParameter("gender");
		String religion = req.getParameter("religion");
		String intro = req.getParameter("introduction");
		
		out.print("ID: " + id + "<br/>");
		out.print("비밀번호: "+ password + "<br/>");
		out.print("이름: " + name + "<br/>");
		out.print("취미: ");
		for(int i = 0 ; i < hobbies.length; i++) {
			out.print(hobbies[i] + " ");
		}
		out.print("<br/>");
		out.print("성별: " + gender + "<br/>");
		out.print("종교: " + religion + "<br/>");
		out.print("소개: " + intro + "<br/>");
		out.print("전체 질의 문자열: " + req.getQueryString());
		
		out.close();
	}
~ 생략 ~
```



**[실행 결과]**

![image-20210311125349550](C:\Users\user\AppData\Roaming\Typora\typora-user-images\image-20210311125349550.png)



이번에는 POST 방식으로 처리해보겠습니다.



```java
~생략~
@Override
protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException{
	resp.setContentType("text/html;charset=UTF-8");
	PrintWriter out = resp.getWriter();
	
	String id = req.getParameter("id");
	String password = req.getParameter("pwd");
	String name = req.getParameter("name");
	String[] hobbies = req.getParameterValues("hobby");
	String gender = req.getParameter("gender");
	String religion = req.getParameter("religion");
	String intro = req.getParameter("introduction");
	
	out.print("ID: " + id + "<br/>");
	out.print("비밀번호: "+ password + "<br/>");
	out.print("이름: " + name + "<br/>");
	out.print("취미: ");
	for(int i = 0 ; i < hobbies.length; i++) {
		out.print(hobbies[i] + " ");
	}
	out.print("<br/>");
	out.print("성별: " + gender + "<br/>");
	out.print("종교: " + religion + "<br/>");
	out.print("소개: " + intro + "<br/>");
	
	out.close();
}	
```



**[실행 결과]**



![image-20210311125725542](C:\Users\user\AppData\Roaming\Typora\typora-user-images\image-20210311125725542.png)



doPost() 메소드에서 질의 문자열을 추출하는 작업은 doGet() 메소드에서 사용했던 메소드를 동일하게 사용하고 있음을 알 수 있습니다. 질의 문자열을 name 단위로 추출할 때는 GET 방식과 POST 방식 모두 HttpServletRequest의 getParameter() 또는 getParameterValues() 메소드를 사용하여 추출하면 됩니다.





이제 **POST 방식에서 질의 문자열 전체를 추출하는 예제**를 작성해 보겠습니다. GET 방식에서는 getQueryString() 메소드를 사용하지만, POST 방식에서는 프로그램 내의 요청정보에서 직접 읽어와야 합니다.



```java
~생략~
@Override
protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException{
	resp.setContentType("text/html;charset=UTF-8");
	PrintWriter out = resp.getWriter();
	
	out.print("<html><head><title>Query 문자열 테스트</title></head>");
	out.print("<body>");
	out.print("<h1>POST 방식 질의 문자열 추출</h1>");
	ServletInputStream input = req.getInputStream();
	int len = req.getContentLength();
	byte[] buf = new byte[len];
	input.readLine(buf, 0, len);
	String s = new String(buf);
	out.print("전체 문자열: " + s);
	out.println("</body></html>");
	out.close();
}
~생략~
```



**[실행 결과]**



![image-20210311130256169](C:\Users\user\AppData\Roaming\Typora\typora-user-images\image-20210311130256169.png)



---

### 4.5 한글 처리

이번 절에서는 클라이언트가 전송한 문자열에 한글이 있을 때 깨지지 않고 올바르게 표현하기 위한 방법을 알아보겠습니다. 먼저, 폼에서 한글이 입력되었을 때 어떤 결과가 나오는지 예제를 통해 살펴보겠습니다.



WebContent에 "**name.html**"이라는 이름으로 새로운 HTML 파일을 만들고, 다음과 같은 코드를 작성합니다.

```html
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>이름 입력</title>
</head>
<body>
	<form action="queryTest3" method="post">
		이름 : <input type="text" name="name">
		<input type="submit" value="전송">
	</form>
</body>
</html>
```



`<meta charset="UTF-8">`는 현재 HTML 파일이 저장될 때 사용할 인코딩 문자코드를 설정합니다. charset에 "UTF-8"로 지정했기 때문에 현재 HTML 파일은 UTF-8 문자코드를 사용해 인코딩 처리됩니다.



com.edu.test에 새로운 클래스 파일을 만들고, 이름을 "**QueryTest3Servlet.java**"으로 입력합니다. 그리고 다음과 같은 코드를 작성합니다.

```java
package com.edu.test;

import java.io.*;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.*;


@WebServlet("/queryTest3")
public class QueryTest3Servlet extends HttpServlet {

	@Override
	protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
		resp.setContentType("text/html;charset=UTF-8");
		PrintWriter out = resp.getWriter();
		out.print("<html><head><title>Query 문자열 테스트</title></head>");
		out.print("<body>");
		out.print("<h1>GET 방식으로 요청되었습니다.</h1>");
		
		String name = req.getParameter("name");
		out.print("이름: " + name + "<br/>");
		
		out.println("</body></html>");
		out.close();
	}
	
	@Override
	protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
		resp.setContentType("text/html;charset=UTF-8");
		PrintWriter out = resp.getWriter();
		out.print("<html><head><title>Query 문자열 테스트</title></head>");
		out.print("<body>");
		out.print("<h1>POST 방식으로 요청되었습니다.</h1>");
		
		String name = req.getParameter("name");
		out.print("이름: " + name + "<br/>");
		
		out.println("</body></html>");
		out.close();
	}
}
```





name.html을 실행하고 이름 입력 폼에 한글을 입력한 다음, 전송을 클릭합니다.



![image-20210311151003306](C:\Users\user\AppData\Roaming\Typora\typora-user-images\image-20210311151003306.png)





**[실행 결과]**



![image-20210311151015837](C:\Users\user\AppData\Roaming\Typora\typora-user-images\image-20210311151015837.png)



POST 방식으로 요청했기 때문에 doPost() 메소드가 실행되었습니다. 그런데 이름에 입력한 "홍길동" 문자열이 깨져서 출력됩니다.



깨진 한글을 복원하는 방법은 GET과 POST 방식이 다릅니다. GET 방식은 요청정보 헤더의 URI에, POST 방식은 요청정보 몸체에 포함되어 전달되기 때문입니다. 지금부터 **요청방식에 따라 한글로 전달된 질의 문자열을 처리하는 방법**을 알아보겠습니다.



#### 4.5.1 POST 방식 한글 처리

POST 방식은 서블릿 API에서 한글 코드 변환을 해주는 메소드를 제공하고 있어서 메소드를 사용해 간단하게 한글을 처리할 수 있습니다. POST 방식에서 한글 처리를 해주는 메소드는 HttpServletRequest의 상위 객체인 ServletRequest에서 제공하는 setCharacterEncoding() 메소드입니다.



`void setCharacterEncoding(String env) throws UnsupportedEncodingException`



setCharacterEncoding() 메소드는 인자값으로 지정된 문자코드를 사용하여 클라이언트가 전달한 요청정보 몸체에 있는 문자열들을 인코딩해주는 메소드입니다. 클라이언트로부터 질의 문자열이 전달된 후 서블릿이 실행될 때 setCharacterEncoding() 메소드로 인코딩 작업을 처리한 후 getParameter() 메소드로 추출하여 사용합니다.



예제를 통해 메소드를 사용해보겠습니다. QueryTest3Servlet.java 소스의 doPost() 메소드를 아래와 같이 수정합니다.

```java
~생략~
@Override
protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
	resp.setContentType("text/html;charset=UTF-8");
	PrintWriter out = resp.getWriter();
	out.print("<html><head><title>Query 문자열 테스트</title></head>");
	out.print("<body>");
	out.print("<h1>POST 방식으로 요청되었습니다.</h1>");
	
	req.setCharacterEncoding("UTF-8");
	String name = req.getParameter("name");
	out.print("이름: " + name + "<br/>");
	
	out.println("</body></html>");
	out.close();
}
~생략~
```



name.html을 실행해 다시 "홍길동"을 입력하면,



**[실행 결과]**



![image-20210311151914743](C:\Users\user\AppData\Roaming\Typora\typora-user-images\image-20210311151914743.png)



한글이 깨지지 않고 정상 출력됨을 확인할 수 있습니다.





#### 4.5.2 GET 방식 한글 처리

이번에는 한글 질의 문자열을 GET 방식으로 전달했을 경우를 살펴보겠습니다. GET 방식으로 전달된 질의 문자열은 요청정보 헤더의 URI에 포함되어 전달됩니다. setCharacterEncoding() 메소드는 요청정보의 몸체에 있는 데이터들만 인코딩해주기 때문에 GET 방식은 사용할 수 없습니다. **GET 방식으로 전달된 질의 문자열들은 URI에 포함되어 전달되기 때문에 URI에 대해 인코딩 처리 작업을 해야 합니다**.



GET 방식으로 전달되는 질의 문자열들의 인코딩 작업을 할 때는 다음과 같은 질문에 답이 무엇인지 명확히 하고 작업합니다. 



- 클라이언트가 입력하는 질의 문자열의 인코딩 문자코드는 무엇인가?
- 서버에서 URI를 인코딩하는 문자코드는 무엇인가?



**1) 클라이언트가 입력하는 질의 문자열의 인코딩 문자코드는 무엇인가?**

클라이언트가 입력한 문자열들을 인코딩 처리하는 방법은 두 가지입니다. **소스에서 직접** 인코딩 문자코드를 지정해주는 방법과 **이클립스에서** 인코딩 문자를 지정하는 방법입니다.



**소스에서 직접 인코딩 문자코드를 지정해주는 방법**은

\<head>와 \</head>사이에 \<meta charset="UTF-8"> 태그처럼 사용하고자 하는 문자코드를 지정하는 것입니다.



**이클립스에서 소스의 인코딩 문자코드를 지정하는 방법**은 

이클립스에서 [Window] -> [Preferences] -> [Web] -> [HTML Files]를 선택한 다음, 'Encoding'에서 원하는 문자코드를 지정합니다. 다른 형식(CSS, JSP)의 파일도 같은 방법으로 인코딩을 지정합니다.



또는 여러 타입으로 된 문서의 인코딩 문자를 한번에 설정하고자 한다면, 이클립스에서 [Window] -> [Preferences] -> [General] -> [Content Types] -> [Text]를 선택한 후, 'Default encoding' 값을 원하는 문자코드로 입력한 다음, \<Update>와 \<OK>를 클릭합니다.





**2) 서버에서 URI를 인코딩하는 문자코드는 무엇인가?**

클라이언트로부터 전달된 URI 문자열의 인코딩 작업은 서버에서 처리해줍니다. 서버마다 기본적으로 URI의 인코딩 문자코드가 정해져 있으며 톰캣 8 버전에서는 UTF-8 문자코드가 기본값으로 적용됩니다. 만일 서버의 URI 인코딩 처리 문자코드를 지정하고 싶다면, 서버의 환경설정 파일 중 **server.xml** 파일에 인코딩 문자코드를 설정해줍니다.



server.xml 파일의 내용 중 \<Connector> 태그를 찾아 다음처럼 URIEncoding="UTF-8" 속성을 \<Connector> 태그에 추가합니다.

```xml
~생략~
<Connector connectionTimeout="머시기"
		   port="포트번호"
		   protocol="프로토콜"
		   redirectPort="포트번호"
		   URIEncoding="UTF-8" />
~생략~
```

