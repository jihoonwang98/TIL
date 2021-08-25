# MIME-Type, Content-Type



### MIME이란?

MIME은 Multipurpose Internet Mail Extensions의 약자로 쉽게 말하면 파일 변환을 뜻한다.

MIME은 이메일과 함께 동봉할 파일을 텍스트 문자로 전환해서 이메일 시스템을 통해 전달하기 위해 개발되었기 때문에 Internet Mail Extensions다.

그렇지만 현재는 웹을 통해 여러 형태의 파일을 전달하는데 쓰이고 있다.



### MIME 사용 이유

MIME을 사용하기 전까지는 UUEncode 방식을 사용했으며 이 방식을 보완하기 위한 새로운 인코딩 방식으로 등장한 것이 바로 MIME이다.

예전에는 텍스트 파일을 주고받는데 ASCII로 공통된 표준에 따르기만 하면 문제가 없었다.

하지만 네트워크를 통해 ASCII 파일이 아닌 바이너리 파일을 보내는 경우가 생기게 되었다.

이러한 바이너리 파일은 음악 파일, 동영상 파일, 워드 파일 등등을 말한다.

이러한 바이너리 파일들은 ASCII만으로 전송이 불가능하여 바이너리 파일을 텍스트 파일로 변환 후 전송해야 했다.



**바이너리 파일 -> 텍스트 파일**로의 변환을 **인코딩**이라 하고,

**텍스트 파일 -> 바이너리 파일**로의 변환을 **디코딩**이라 한다.



MIME으로 인코딩을 한 파일은 Content-type 정보를 담게 되며, Content-type은 여러가지 타입이 있다.





### Content-Type이란?

담고 있는 데이터가 어떤 종류의 데이터인지 알려주는 놈



**문법**

```
Content-Type: text/html; charset=UTF-8
Content-Type: multipart/form-data; boundary=something
```



**예제**

**1. Content-Type in HTML Forms**

```html
<form action="/" method="post" enctype="multipart/form-data">
  <input type="text" name="description" value="some text">
  <input type="file" name="myFile">
  <button type="submit">Submit</button>
</form>
```

HTML 폼 전송으로 일어나는 POST 요청 내에서, 

요청의 `Content-Type`은 `<form>` 태그의 `enctype` 속성으로 지정할 수 있다.



요청은 다음과 같이 날라간다.

```http
POST /foo HTTP/1.1
Content-Length: 68137
Content-Type: multipart/form-data; boundary=---------------------------974767299852498929531610575
Content-Disposition: form-data; name="description"
---------------------------974767299852498929531610575

some text

---------------------------974767299852498929531610575
Content-Disposition: form-data; name="myFile"; filename="foo.txt"
Content-Type: text/plain

(content of the uploaded file foo.txt)

---------------------------974767299852498929531610575
```



- multipart 개체에 대해 `boundary` 디렉티브는 필수인데, 이메일 게이트를 통해 매우 탄탄해졌다고 알려진 charset의 1~70개의 문자들로 구성되며, 빈 공백으로 끝나지 않습니다. 이는 메시지의 멀티 파트 경계선을 캡슐화하기 위해 사용됩니다.







html form 태그 사용 POST 방식 요청 default -> 'application/x-www-form-urlencoded'

jQuery의 ajax 요청 default -> 'application/x-www-form-urlencoded'





- Ajax로 데이터 보낼때 Content-Type 설정안하면 

  default 값은 `application/x-www-form-urlencoded; charset=UTF-8`

- Ajax로 JSON 형식으로 보내고 싶으면 Content-Type을 `application/json`으로 지정해주어야 한다.







**Content-Type: application/x-www-form-urlencoded**

```http
POST / HTTP/1.1
Host: localhost
Content-type: application/x-www-form-urlencoded

name=kim&age=10
```

- x-www-form-urlencoded의 경우 key1=value1&key2=value2...의 형식으로 요청이 간다.



**Content-Type: application/json**

```http
POST / HTTP/1.1
Host: localhost
Content-type: application/json

{
	"name":"kim",
	"age":"10"
}
```

