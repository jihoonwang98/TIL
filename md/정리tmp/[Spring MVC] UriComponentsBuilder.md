# [Spring MVC] UriComponentsBuilder



```java
@Test
public void constructUri() {
    UriComponents uriComponents = UriComponentsBuilder.newInstance()
      .scheme("http").host("www.baeldung.com").path("/junit-5").build();

    assertEquals("/junit-5", uriComponents.toUriString());
}
```

```java
@Test
public void constructUriWithQueryParameter() {
    UriComponents uriComponents = UriComponentsBuilder.newInstance()
      .scheme("http").host("www.google.com")
      .path("/").query("q={keyword}").buildAndExpand("baeldung");

     assertEquals("http://www.google.com/?q=baeldung", uriComponents.toUriString());
}
```

```java
URI uri = UriComponentsBuilder.fromUriString(legacyViewUrl)
                .path("/trend/target/dealers/sale/car")
                .queryParams(request.toMap())
                .build()
                .encode()
                .toUri();
```

```java
@Test
public void testURI2() throws Exception{
	UriComponents uriComponents = UriComponentsBuilder.newInstance()
			.path("/{a}/{b}/{c}")
			.queryParam("bno", 12)
			.queryParam("perPageNum", 20)
			.build()
			.expand("samplehome", "board", "read")
			.encode();
	logger.info("/samplehome/board/read?bno=12&perPageNum=20");  
	logger.info(uriComponents.toString()); 
    //       /samplehome/board/read?bno=12&perPageNum=20 가 생성된다.
}
```

