# jQuery Ajax 통신





## 1. $.get()

- HTTP GET 요청을 사용하여 서버에서 데이터를 로드한다.



```js
$.get(url, parameters, callback)
```

매개변수로 명시된 URL을 사용하여 서버에 대한 GET 요청을 전송한다.

매개변수는 쿼리 문자열로 전달한다.





### 매개변수

| 변수명     | 타입             | 설명                                                         |
| ---------- | ---------------- | ------------------------------------------------------------ |
| url        | String           | GET 메서드로 연결하는 서버 측 자원의 URL                     |
| parameters | Object \| String | URL에 덧붙이는 쿼리 문자열을 구성하려고 이름과 값의 쌍으로 프로퍼티를 지닌 개체, 미리 구성 및 인코딩된 쿼리 문자열 |
| callback   | Function         | 요청이 완료되면 호출되는 함수. 응답 본문은 이 콜백 함수의 첫 번째 매개변수로 전달되며, 상태 값은 두 번째 매개변수로 전달된다. |



### 반환값

XHR 인스턴스







## 2. $.getJSON()

- HTTP GET 요청을 사용하여 서버에서 JSON 형식으로 인코딩된 데이터를 로드한다.



```js
$.getJSON(url, parameters, callback)
```

매개변수로 명시된 URL을 사용하여 서버에 대한 GET 요청을 전송한다.

매개변수는 쿼리 문자열로 전달한다.

응답은 JSON 문자열로 해석되며, 결과로 만들어진 데이터는 콜백 함수에 전달된다.



### 매개변수

| 변수명     | 타입             | 설명                                                         |
| ---------- | ---------------- | ------------------------------------------------------------ |
| url        | String           | GET 메서드로 연결하는 서버 측 자원의 URL                     |
| parameters | Object \| String | URL에 덧붙이는 쿼리 문자열을 구성하려고 이름과 값의 쌍으로 프로퍼티를 지닌 개체, 미리 구성 및 인코딩된 쿼리 문자열 |
| callback   | Function         | 요청이 완료되면 호출되는 함수. 응답 본문은 이 콜백 함수의 첫 번째 매개변수로 전달되며, 상태 값은 두 번째 매개변수로 전달된다. |







## 3. $.post()

- HTTP POST 요청을 사용하여 서버에서 데이터를 로드한다.



```js
$.post(url, parameters, callback)
```

매개변수로 명시된 URL을 사용하여 서버에 대한 POST 요청을 전송한다.

parameters는 요청의 본문으로 전달한다.



### 매개변수

| 변수명     | 타입             | 설명                                                         |
| ---------- | ---------------- | ------------------------------------------------------------ |
| url        | String           | POST 메서드로 연결하는 서버 측 자원의 URL                    |
| parameters | Object \| String | URL에 덧붙이는 쿼리 문자열을 구성하려고 이름과 값의 쌍으로 프로퍼티를 지닌 개체, 미리 구성 및 인코딩된 쿼리 문자열 |
| callback   | Function         | 요청이 완료되면 호출되는 함수. 응답에서 JSON 문자열로 만들어진 데이터 값은 이 콜백 함수의 첫 번째 매개변수로 전달되며, 상태 값은 두 번째 매개변수로 전달된다. |



### 반환값

XHR 인스턴스







## 4. $.ajax()

- 비동기 HTTP (Ajax) 요청을 수행한다.



```js
$.ajax(options)
```

요청의 생성 방법과 통보 받을 콜백을 제어하고자

전달된 options를 사용하여 Ajax 요청을 전송한다.



### 매개변수

| 변수명  | 타입   | 설명                                                         |
| ------- | ------ | ------------------------------------------------------------ |
| options | Object | 요청에 대한 매개변수를 정의하는 프로퍼티를 소유한 객체 인스턴스 |



### 반환값

XHR 인스턴스



### 함수의 옵션

| 옵션명      | 타입     | 설명                                                         |
| ----------- | -------- | ------------------------------------------------------------ |
| url         | String   | 요청 URL                                                     |
| type        | String   | 사용할 HTTP 메서드. 생략되면 기본값으로 GET을 사용           |
| data        | Object   | 요청에 전달되는 프로퍼티를 가진 객체. GET 요청이면 데이터는 쿼리 문자열로 제공된다. POST 요청이면 데이터는 요청의 본문으로 제공된다. |
| dataType    | String   | 응답의 결과로 반환되는 데이터의 종류를 식별하는 키워드. 유효한 값은 xml, html, json, jsonp, script, text이다. |
| timeout     | Number   | Ajax 요청의 제한 시간을 밀리초 단위로 설정한다.              |
| global      | Boolean  | true나 false에 따라 전역 함수(global function)를 활성화하거나 비활성화한다. |
| contentType | String   | 요청에 명시되는 컨텐츠 타입. 생략하면 'application/x-www-form-urlencoded'가 기본값으로 설정된다. |
| headers     | Object   | 요청 헤더. XMLHttpRequest 전송을 사용하여 요청과 함께 추가로 전송할 헤더 키 / 값 쌍의 객체 |
| success     | Function | 응답이 성공 상태 코드를 반환하면 호출되는 함수. 응답 본문은 이 함수의 첫 번째 매개변수로 전달되며, dataType 프로퍼티에 명시한 형태로 구성된다. 두 번째 매개변수는 상태값을 나타내는 문자열이며, 이번 경우에는 항상 'success'다. |
| error       | Function | 응답이 에러 상태 코드를 반환하면 호출되는 함수. 매개변수가 세 개 전달되는데, 각각 XHR 인스턴스, 상태 값이 항상 'error'인 메시지 문자열, 선택사항으로 XHR 인스턴스가 반환하는 예외 객체다. |
| complete    | Function | 요청이 완료되면 호출되는 함수. 매개변수 두 개가 전달되는데, 각각 XHR 인스턴스와 'success' 혹은 'error'를 나타내는 상태 메시지 문자열이다. success나 error 콜백을 명시했다면, 이 함수는 해당 콜백이 호출된 후에 실행된다. |
| beforeSend  | Function | 요청이 전송되기 앞서 먼저 호출되는 함수. 이 함수는 XHR 인스턴스를 전달받으며, 사용자 정의 헤더를 설정하거나 요청 전에 필요한 연산을 수행하는데 사용할 수 있다. |
| async       | Boolean  | false이면 요청이 동기 호출로 전송된다. 기본은 비동기 요청이다. |
| processData | Boolean  | false로 설정되면, URL 인코딩된 형태로 처리되어 전달된 데이터를 금지한다. 기본값은 데이터가 'application/x-www-form-urlencoded' 타입의 요청에 사용하는 형태의 URL로 인코딩된다. |
| ifModified  | Boolean  | true일 때 Last-Modified 헤더를 확인하며 마지막 요청 이후에 응답 컨텐츠가 변경되지 않았다면 요청이 성공한다. 만일 생략하면 헤더를 확인하지 않는다. |







```java
$.ajax({
    url:'/study/tmp/test.php', //request 보낼 서버의 경로
    type:'post', // 메소드(get, post, put 등)
    data:{'id':'admin'}, //보낼 데이터
    success: function(data) {
        //서버로부터 정상적으로 응답이 왔을 때 실행
    },
    error: function(err) {
        //서버로부터 응답이 정상적으로 처리되지 못햇을 때 실행
    }
});

$(document).ready(function(){
	$.ajax({
		type: //데이터 전송 타입,
		url : //데이터를 주고받을 파일 주소 입력,
		data: //보내는 데이터,
		dataType://문자형식으로 받기 , 
		success: function(result){
			//작업이 성공적으로 발생했을 경우
		},
		error:function(){  
            //에러가 났을 경우 실행시킬 코드
		}
	})
       
});
[출처] [AJax] 데이터 전송하기(GET,POST,JSON)|작성자 Ohsanrim
    
    
    
$(document).ready(function(){
	$('#testButton').click(function(){
		$.ajax({
			type: 'get',   //get 방식으로 받기
			url: 'jQueryAjax05_jsonData.txt',   //데이터를 주고받을 파일 주소
			dataType: 'json',  //json파일 형식으로 값 받기
			success: function(data){
				alert(JSON.stringify(data));   //data를 alert으로 찍어줌  
			},
			error: function(err){
				console.log(err);    //에러가 발생하면 콘솔 로그를 찍어준다. 
			}
		});
	});
[출처] [AJax] 데이터 전송하기(GET,POST,JSON)|작성자 Ohsanrim

    
    
function reqAjax2() {
    var name = $("#username").val();
    var phone = $("#phone").val();
    if(name.trim() =='' || phone.trim()==''){
        alert('데이터를 입력해주세요')
        return;}
    //서버로 보낼 데이터 준비 : 파라미터로 만들기 . json 으로 만들기
    var sendData = {"name":name,"phone":phone}
    $.ajax({
        url:'reqAjax2'
        , method : 'POST'
        , data: JSON.stringify(sendData)
        ,contentType : 'application/json; charset=UTF-8'
        ,dataType : 'json'
        , success :function(resp){
            alert( JSON.stringify(sendData))
        }
    })	
}


출처: https://dororongju.tistory.com/96 [웹 개발 메모장]
```



