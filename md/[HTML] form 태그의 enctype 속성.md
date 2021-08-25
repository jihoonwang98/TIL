# [HTML] form 태그의 enctype 속성



### 서론

`<form>` 태그의 속성인 `method`, `action`, `enctype` 등은 입력받은 데이터를 어떻게 처리할 것인지 세부적으로 설정하는데 사용된다.

`method`는 전송 방식,

`action`는 전송 목적지,

`enctype`은 전송되는 **데이터 형식**을 설정한다.





### `enctype` 속성의 종류 3가지

|              속성값               |                             설명                             |
| :-------------------------------: | :----------------------------------------------------------: |
| application/x-www-form-urlencoded | 기본값으로, 모든 문자들은 서버로 보내기 전에 URL-Encode됨을 명시함. |
|        multipart/form-data        | 모든 문자를 인코딩하지 않음을 명시함. 이 방식은 \<form> 요소가 파일이나 이미지를 서버로 전송할 때 주로 사용함. |
|            text/plain             | 공백 문자(space)는 "+" 기호로 변환하지만, 나머지 문자는 모두 인코딩되지 않음을 명시함. |

- 참고로, enctype 속성은 method 속성값이 post인 경우에만 사용가능하다.





> **URL 인코딩이란?**
>
> - 문자나 특수문자를 웹 서버와 브라우저에서 보편적으로 허용하는 형식으로 변화시키는 메커니즘
> - URL은 ASCII 문자 집합을 사용하여 인터넷을 통해서만 전송 가능
> - URL은 종종 ASCII 외부의 문자를 포함하기 때문에 URL은 유효한 ASCII 형식으로 변환되어야 한다.
> - URL 인코딩은 안전하지 않은 ASCII 문자를 "%" 다음에 두 개의 16진수로 대체한다.
> - URL은 공백을 포함할 수 없다. URL 인코딩은 일반적으로 공백을 + 또는 %20으로 바꾼다.
> - 따라서, 아스키 이외의 문자는 다 인코딩이 필요하다. (한글, 일본어, 중국어, 독일어 등등)
>
> 
>
> **예시**
>
> 입력: 헬로우 WORLD
>
> text=%ED%97%AC%EB%A1%9C%EC%9A%B0+WORLD





### `application/x-www-form-urlencoded`

```http
POST / HTTP/1.1
Host: foo.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 13

say=Hi&to=Mom
```





### `multipart/form-data`

```java
POST /test.html HTTP/1.1
Host: example.org
Content-Type: multipart/form-data;boundary="boundary"

--boundary
Content-Disposition: form-data; name="field1"

value1
--boundary
Content-Disposition: form-data; name="field2"; filename="example.txt"

value2
--boundary--
```

