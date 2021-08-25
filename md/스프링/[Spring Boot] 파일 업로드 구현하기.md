# [Spring Boot] 파일 업로드 구현하기



## 1. `MultipartResolver` 설정

`Multipart` 지원 기능을 이용하려면 먼저 `MultipartResolver`를 스프링 빈으로 등록해야 한다.

`MultipartResolver`가 하는 일은 `Multipart` 형식으로 데이터가 전송된 경우, 해당 데이터를 스프링 MVC에서 사용할 수 있도록 변환해주는 일을 한다.



`MultipartResolver`는 인터페이스로, 스프링이 제공하는 기본 구현체는 `CommonsMultipartResolver`이다.

`CommonsMultipartResolver`는 CommonsFileUpload API를 이용해 `Multipart`를 처리해준다.





### `MultipartResolver` 빈 등록

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Bean
    public MultipartResolver multipartResolver() {
        CommonsMultipartResolver multipartResolver = new CommonsMultipartResolver();

        multipartResolver.setDefaultEncoding("UTF-8");
        multipartResolver.setMaxUploadSizePerFile(5 * 1024 * 1024);

        return multipartResolver;
    }
}
```



※ CommonsMultipartResolver 클래스의 프로퍼티

| property         | type   |                                                              |
| ---------------- | ------ | ------------------------------------------------------------ |
| maxUploadSize    | long   | 최대 업로드 가능한 바이트 크기, -1은 제한이 없음을 의미한다. 기본 값은 -1이다. |
| maxInMemorysize  | int    | 디스크에 임시 파일을 생성하기 전에 메모리에 보관할 수 있는 최대 바이트 크기,기본 값은 10240 바이트이다. |
| defaultEncording | String | 요청을 파싱할 때 사용할 캐릭터 인코딩, 지정하지 않은 경우 HttpServletRequest.setEncording() 메서드로 지정한 캐릭터셋이 사용된다.아무 값도 없을 경우 ISO-8859-1을 사용한다. |





### 라이브러리 의존성 등록

아래의 두 라이브러리를 등록해서 코드를 더 간결하게 짤 수 있다.

```xml
<dependency>
    <groupId>commons-io</groupId>
    <artifactId>commons-io</artifactId>
    <version>2.6</version>
</dependency>

<dependency>
    <groupId>commons-fileupload</groupId>
    <artifactId>commons-fileupload</artifactId>
    <version>1.3.3</version>
</dependency>
```





### `application.yml`

```yaml
spring:
  servlet:
    multipart:
      location: C:/Users/user/Desktop/file/
      max-file-size: 1MB
      max-request-size: 1MB
      enabled: true
```



#### 🔧 옵션

| 옵션                  | 설명                             | 기본값 |
| --------------------- | -------------------------------- | ------ |
| **enabled**           | 멀티파트 업로드 지원여부         | true   |
| **fileSizeThreshold** | 파일이 메모리에 기록되는 임계 값 | 0B     |
| **location**          | 업로드된 파일의 임시 저장 공간   |        |
| **maxFileSize**       | 파일의 최대 사이즈               | 1MB    |
| **maxRequestSize**    | 요청한 최대 사이즈               | 10MB   |





### form 태그 작성

파일 업로드를 할 때는 `form` 태그의 `enctype` 속성이 `multipart/form-data`이고, `method` 속성이 `post`여야 한다.

그래야 `MultipartResolver`가 MultipartFile 객체를 컨트롤러에 전달 가능하다.

```html
<form action="submitReport1" method="POST" enctype="multipart/form-data">
	학번 : <input type="text" name="studentNumber" /><br/>
	리포트 파일 : <input type="file" name="report" /><br/>
	<input type="submit" />
</form>
```



### `MultipartFile` 인터페이스

※ MulripartFile 인터페이스의 주요 메서드

| 메소드                                            | 설명                                                         |
| ------------------------------------------------- | ------------------------------------------------------------ |
| String getName()                                  | 파라미터 이름을 구한다.                                      |
| **String getOriginalFilename()**                  | **업로드한 파일의 이름을 구한다.**                           |
| String isEmpty()                                  | 업로드한 파일이 존재하지 않는 경우 true를 리턴한다.          |
| long getSize()                                    | 업로드한 파일의 크기를 구한다.                               |
| **byte[ ] getBytes() throws IOExcetion**          | **업로드한 파일 데이터를 구한다.**                           |
| InputStream getInputStream() throws IOException   | 업로드한 파일 데이터를 읽어오는 InputStream을 구한다. InputStream의 사용이 끝나면 알맞게 종료해주어야 한다. |
| **void transferTo(File dest) throws IOException** | **업로드한 파일 데이터를 지정한 파일에 저장한다**.           |



업로드 한 파일 데이터를 구하는 가장 단순한 방법은 MultipartFile.getByte() 메서드를 이용하는 것이다. 바이트 배열을 구한 뒤에 파일이나 DB등에 저장하면된다.

```java
if(mutipartFile.isEmpty()){
    byte[ ] fileData = multipartFile.getBytes();
    //byte 배열을 파일/DB/네트워크 등으로 전송
    ...
}
```



업로드한 파일데이터를 특정 파일로 저장하고 싶다면 MultipartFile.**transferTo()** 메서드를 사용하는 것이 편리하다.

```java
if(mutipartFile.isEmpty()){
    File file = new File(fileName);
    multipartFile.transferTo(file);
    ...
}
```





```java
private FileOutputStream fos = null;
        
public String uploadResult(@RequestParam("selFile") MultipartFile mFile) throws Exception, IOException {
    String fileExtension = mFile.getOriginalFilename().substring(mFile.getOriginalFilename().lastIndexOf("."));
    String storedFileName = UUID.randomUUID().toString().replaceAll("-",  "") + fileExtension;
    String filePath = "C:\\eclipse\\workspace\\memPid\\src\\main\\webapp\\resources\\";
    File file = new File(filePath + storedFileName);
    //첫 번째 방법, MultipatrFile클래스의 transferTo()를 사용하여 업로드하려는 파일을 특정파일로 저장
    //MultipartFile.transferTo(File file) - Byte형태의 데이터를 File객체에 설정한 파일 경로에 전송한다.
    mFile.transferTo(file);
    
    //두 번째 방법, MultipatrFile클래스의 getBytes()로 multipartFile의 데이터를 바이트배열로 추출한 후, FileOutputStream클래스의 write()로 파일을 저장
    try {
        //MultipatrFile클래스의 getBytes()를 사용해서 multipartFile의 데이터를 바이트배열로 추출
        byte fileData[] = mFile.getBytes();
        fos = new FileOutputStream(filePath + mFile.getOriginalFilename());
        //FileOutputStream클래스의 write()로 파일을 filePath에 저장
        fos.write(fileData);
    } catch (Exception e) {
        e.printStackTrace();
    }finally {
        if(fos != null) {
            fos.close();
        }
    }
    return "upload_result";
}
```







### 컨트롤러

```java
@Value("${spring.servlet.multipart.location}")
private String filePath;

@PostMapping("/submitReport1")
public String submit(@RequestParam String studentNumber, @RequestParam MultipartFile report) throws IOException {
    String originalFileName = report.getOriginalFilename();
    String extension = FilenameUtils.getExtension(originalFileName);


    String storedFileName = UUID.randomUUID().toString().replaceAll("-", "") + "." + extension;

    File file = new File(filePath + storedFileName);
    report.transferTo(file);

    System.out.println("학번: " + studentNumber));
    System.out.println("업로드한 파일 원래 이름: " + originalFileName);
    System.out.println("저장된 파일 이름: " + storedFileName);
    System.out.println("파일 사이즈: " + report.getSize());
    return "제출 완료";
}
```









