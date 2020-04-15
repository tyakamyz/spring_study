# **Part 06** 파일 업로드 처리

## **Chapter 21** 파일 업로드 방식

 - 첨부파일을 서버에 전송하는 두가지 방식
    - \<form> 태그를 이용하는 방식 : 브라우저의 제한이 없어야 하는 경우 사용
        - 일반적으로 페이지 이동과 동시에 첨부파일을 업로드
        - \<iframe>을 이용해서 화면의 이동 없이 첨부파일을 처리하는 방식
    - Ajax를 이용하는 방식 : 첨부파일을 별도로 처리하는 방식
        - \<input type = 'file'>을 이용하고 Ajax로 처리하는 방식
        - HTML5의 Drag And Drop 기능이나 jQuery 라이브러리를 이용해서 처리하는 방식

 - 서버에서 첨부파일을 처리하는 방식은 응답을 HTML코드로 할지 JSON 등으로 처리할지 정도의 구분만 하면 된다.
 - 사용 API
    - cos.jar : 2002년도 이후로 개발이 종료되었으므로, 사용 권장 하지 않음
    - commons-fileupload : 가장 일반적으로 많이 활용되고, Servlet 스펙 3.0 이전에도 사용가능
    - Servlet 3.0 이상 - 3.0 이상부터는 자체적으로 파일 업로드 처리가 API상에서 지원


### 21.1 web.xml 이용하는 첨부파일 설정

 - 3.0 이상의 Servlet 설정
 - web.xml 에 \<multpart-config> 추가
 ```xml
<multipart-config>
    <location>/Users/yun-wonhui/Desktop/upload</location>
    <max-file-size>20971520</max-file-size> <!--  1MB * 20  -->
    <max-request-size>41943040</max-request-size> <!-- 40MB -->
    <file-size-threshold>20971520</file-size-threshold> <!-- 20MB -->
</multipart-config>
 ```
- 특정 사이즈의 메모리사용(file-size-threshold)
- 업로드되는 파일을 저장할 공간(location)
- 파일의 최대 크기(max-file-size)
- 한번에 올릴 수 있는 최대 크기(max-request-size)

> - web.xml의 설정은 WAS(Tomcat) 자체의 설정일 뿐이고, 스프링에서 업로드 처리는 MultipartResolver라는 타입의 객체를 빈으로 등록해야만 가능하다. 
> - Web과 관련된 설정으로 servlet-context.xml을 이용해서 설정

### 21.2 \<form> 방식의 파일 업로드

 - 파일 업로드에서 가장 신경써야 할 부분은 enctype의 속성값을 'multipart/form-data'로 지정하는 것이다
- \<input type='file'>의 경우 최근 브라우저에서 'multiple'이라는 속성을 지원하는데 이를 이용하면 하나의 \<input> 태그로 여러 개의 파일을 한번에 업로드할 수 있다.
     - 브라우저의 버전에 따라 지원여부가 달라지므로 IE의 경우 10이상에서만 사용 가능하다.
- 예제 소스
```jsp
<form action="uploadFormAction" method="post" enctype="multipart/form-data">
    <input type="file" name="uploadFile" multiple>
    <button>Submit</button>
</form>
```
