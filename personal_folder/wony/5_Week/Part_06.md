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

#### 21.2.1 MultipartFile 타입

 - web.xml 의 \<servlet> 내부에 \<multipart-config> 선언해야한다
 - Java 사용 예
 ```java
@PostMapping("/uploadFormAction")
public void uploadFormPost(MultipartFile[] uploadFile, Model model) {
    
    for(MultipartFile multipartFile : uploadFile) {
        
        log.info("-----------------------------");
        log.info("Upload File Name = " + multipartFile.getOriginalFilename());
        log.info("Upload File Size = " + multipartFile.getSize());
        
    }
}
 ```

  - 메서드 종류

  |메서드|설명
  |--|--
  |String getName()|파라미터의 이름 \<input> 태그의 이름
  |String getOriginalFileNmae()|업로드되는 파일의 이름
  |boolean isEmpty()|파일이 존재하지 않는 경우 true
  |long getSize()|업로드되는 파일의 크기
  |byte[] getBytes()|byte[]로 파일 데이터 변환
  |iuputStream getInputStream()|파일데이터와 연결된 inputStream을 반환
  |transferTo(File file)|파일의 저장

  - 파일 저장
    - 업로드 되는 파일을 저장하는 방법은 간단히 **transferTo()** 를 이용한다
```java
@PostMapping("/uploadFormAction")
	public void uploadFormPost(MultipartFile[] uploadFile, Model model) {
		
		String uploadFolder = "/Users/yun-wonhui/Desktop/upload";
		
		for(MultipartFile multipartFile : uploadFile) {
			
			log.info("-----------------------------");
			log.info("Upload File Name = " + multipartFile.getOriginalFilename());
			log.info("Upload File Size = " + multipartFile.getSize());
			
			
			File saveFile = new File(uploadFolder, multipartFile.getOriginalFilename());
			
			try {
				multipartFile.transferTo(saveFile);
			} catch (Exception e) {
				// TODO: handle exception
				log.error(e.getMessage());
			}
		}
	}
```

### 21.3 Ajax를 이용하는 파일 업로드

 - Ajax를 이용하는 첨부파일 처리는 FormData라는 객체를 이용하는데 IE의 경우 10 이후의 버전부터 지원되므로 브라우저의 제약이 있을 수 있다.

 ```javascript
<script type="text/javascript">
$(document).ready(function(){
    $('#uploadBtn').on('click', function(e){

        var formData = new FormData();
        
        var inputFile = $('input[name="uploadFile"]');
        
        var files = inputFile[0].files;
        
        console.log(files);
        
    })
})
</script>
 ```
  - 결과(File 4개 등록)

 <img src='../img/FileUploadAjax.png'></br>
  - JQuery를 이용하는 경우 파일업로드는 **FormData** 라는 객체를 이용한다(브라우저 제약있음)
  - FormData는 쉽게 말해 가상의 \<form> 태그로 생각하면 된다.

```javascript
  <script type="text/javascript">
  	$(document).ready(function(){
  		$('#uploadBtn').on('click', function(e){

  			var formData = new FormData();
  			
  			var inputFile = $('input[name="uploadFile"]');
  			
  			var files = inputFile[0].files;
  			
  			console.log(files);
  			
  			//add filedate to formdata
  			for(var i = 0; i < files.length; i++){
  				
  				formData.append("uploadFile",files[i]);

  			}
  			
  			$.ajax({
  				url:'/uploadAjaxAction',
  				processData : false,
  				contentType : false,
  				data : formData,
  				type : 'POST',
  				success : function(result){
  					alert("Uploaded");
  				}
  			});
  			
  		});
  	});
  </script>
```
 - 첨부파일 데이터는 formData에 추가한뒤 Ajax를 통해 formData 자체를 전송한다.
    - 이 때 processData, contentType은 반드시 'false'로 지정해야하만 전송이 된다.
        > - processData : 일반적으로 서버에 전달되는 데이터는 query string('url?i_title=제목')으로 전달된다. data 파라미터로 전달된 데이터를 jQuery 내부적으로 query string으로 만드는데, 파일 전송의 경우 이를 하지 않아야 하기에 false로 설정한다.
        > - contentType : 기본 값이 'application/x-www-form-urlencoded;charset=UTF-8'인데 파일의 경우 'multipart/form-data'로 전송이 되어야 하므로 false로 설정한다.

 출처 : https://repacat.tistory.com/38

- Ajax 파일 업로드
 ```java
 @PostMapping("/uploadAjaxAction")
public void puloadAjaxPost(MultipartFile[] uploadFile) {
    
    log.info("updata ajax post......");
    
    String uploadFolder = "/Users/yun-wonhui/Desktop/upload";
    
    for(MultipartFile multipartFile : uploadFile) {
        
        log.info("-----------------------------------");
        log.info("Upload File Name : " + multipartFile.getOriginalFilename());
        log.info("Upload File Size : " + multipartFile.getSize());
        
        String uploadFileName = multipartFile.getOriginalFilename();
        
        //IE has file path
        uploadFileName = uploadFileName.substring(uploadFileName.lastIndexOf("\\")+1);
        log.info("only file name: " + uploadFileName );
        
        File saveFile = new File(uploadFolder,uploadFileName);
        
        try {
            
            multipartFile.transferTo(saveFile);
            
        } catch (Exception e) {
            // TODO: handle exception
            log.error(e.getMessage());
        }
        
    }
    
}
 ```
 - IE의 경우에는 전체 파일의 경로가 전송되므로, 마지막 '\'를 기준으로 잘라낸 문자열이 실제 파일 이름이 된다.

 - 고려해야할 점
    - 동일한 이름으로 파일이 업로드 되었을 경우 기존 파일이 사라지는 문제
    - 이미지 파일의 경우에는 원본 파일의 용량이 큰 경우 섬네일 이미지를 생성해야 하는 문제
    - 이미지 파일과 일반 파일을 구분해서 다운로드 혹은 페이지에서 조회하도록 처리하는 문제
    - 첨부파일 공격에 대비하기 위한 업로드 파일의 확장자 제한

## **Chapter 22** 파일 업로드 상세 처리

### 22.1 중복된 이름의 첨부파일 처리

 - 일반적으로 '년/월/일' 단위의 폴더를 생성해서 파일을 저장한다.
    - java.io.File에 존재하는 mkdirs()를 이용하면 필요한 상위 폴더 까지 한번에 생성할 수 있다.
    ```java
    private String getFolder() {
		SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
		
		Date date = new Date();
		
		String str = sdf.format(date);
		
		return str.replace("-", File.separator);
	}
    ```
- java.util.UUID의 값을 이용해서 처리
```java
UUID uuid = UUID.randomUUID();
			
uploadFileName = uuid.toString() + "_" + uploadFileName;
```
 - randomUUID()를 이용해서 임의의 값을 생성한다.

### 22.2 섬네일 이미지 생성

 - 생성 방법
    - JDK 1.4 부터는 ImageIO를 제공하기에 이를 이용가능
    - ImgScalr와 같은 별도의 라이브러리 사용가능
    - 보통의 JDK에 포함된 API를 이용하는 방식보다 별도의 라이브러리를 사용하는 경우가 많은데. 이는 축소했을 떄의 크기나 해상도를 직접 조절하는 작업을 줄이기 위해서다.
    - 해당 예제는 Thumbnailator 라이브러리를 이용한다
```xml
<!-- https://mvnrepository.com/artifact/net.coobird/thumbnailator -->
<dependency>
<groupId>net.coobird</groupId>
<artifactId>thumbnailator</artifactId>
<version>0.4.8</version>
</dependency>
```

> #### 22.2.1 이미지 파일의 판단
 - 화면에서 약간의 검사를 통해 업로드되는 파일의 확장자를 검사하지만, Ajax를 사용하는 호출은 반드시 브라우저만을 통해 들어오는 것이 아니므로 확인할 필요가 있다.
 ```java
 private boolean checkImageType(File file) {
		
		try {
			
			String contentType = Files.probeContentType(file.toPath());
			
			return contentType.startsWith("image");
			
		} catch (Exception e) {
			// TODO: handle exception
			e.printStackTrace();
		}
		
		return false;
	}
 ```
 > - 이미지 파일 판단 체크
 ```java
private boolean checkImageType(File file) {
    
try {
    
        Path path = file.toPath();
//			String contentType = URLConnection.guessContentTypeFromName(path.toString());

    String contentType = new MimetypesFileTypeMap().getContentType(path.toString());
//			String contentType = Files.probeContentType(path);
    
} catch (Exception e) {
    // TODO: handle exception
    e.printStackTrace();
}

return false;
}
 ```
 > - 예제의 Files.probeContentType(path)를 사용하려 했지만 해당 함수에서 null값이 반환되 제대로된 마인타입을 체크할 수 없었다.
 > - 대신으로 URLConnection.guessContentTypeFormName(); 함수로 마인타입을 체크했다.
 > - 이후 추가로 파일 작업하는 도중 위의 URLConnection 인터페이스도 파일 확인을 하지 못하고 null 값을 받아와서 MimetypesFileTypeMap() 클래스를 사용했다. 
 > - 위의 문제점으로는 현재 예제의 java versiondl 1.8 openjdk로 예상된다. MimetypesFileTypeMap 클래스는 java 1.6부터 사용 가능하기에 openjdk에 사용에도 문제가 없는 걸로 판단된다.
 - 섬네일 생성

 ```java
File saveFile = new File(uploadPath,uploadFileName);
multipartFile.transferTo(saveFile);
// check image typoe file
if(checkImageType(saveFile)) {
    FileOutputStream thumbnail = new FileOutputStream(new File(uploadPath, "s_" + uploadFileName));
    
    Thumbnailator.createThumbnail(multipartFile.getInputStream(), thumbnail, 100,100);
    
    thumbnail.close();

 ```

 ### 22.3 업로드된 파일의 데이터 반환

 - 브라우저로 전송해야 하는 데이터는 다음과 같은 정보를 포함하도록 설계하여야 한다.
    - 업로드된 파일의 이름과 원본 파일의 이름
    - 파일이 저장된 경로
    - 업로드된 파일이 이미지인지 아닌지에 대한 정보
 - 위에 대한 처리 방식
    1. 업로드된 경로가 포함된 파일이름 반환
    1. 별도의 객체를 생성해서 처리하는 방법
    - 1의 경우 브라우저에서 처리해야할 일이 많기에 예제는 2의 방식으로 구현
```java
@PostMapping(value = "/uploadAjaxAction", produces = MediaType.APPLICATION_JSON_UTF8_VALUE)
	@ResponseBody
	public ResponseEntity<List<AttachFileDTO>> puloadAjaxPost(MultipartFile[] uploadFile) {
		
		List<AttachFileDTO> list = new ArrayList<AttachFileDTO>();
		
		log.info("updata ajax post......");
		
		String uploadFolder = "/Users/yun-wonhui/Desktop/upload";
		
		
		// make folder -----
		String uploadFolderPath = getFolder();
		File uploadPath = new File(uploadFolder, uploadFolderPath);
		log.info("upload path : " + uploadPath);
		
		if(uploadPath.canExecute() == false) {
			uploadPath.mkdirs();
		}
		// make yyyy/MM/dd
		
		for(MultipartFile multipartFile : uploadFile) {
			
			AttachFileDTO attchDto = new AttachFileDTO();
			
			log.info("-----------------------------------");
			log.info("Upload File Name : " + multipartFile.getOriginalFilename());
			log.info("Upload File Size : " + multipartFile.getSize());
			
			String uploadFileName = multipartFile.getOriginalFilename();
			
			//IE has file path
			uploadFileName = uploadFileName.substring(uploadFileName.lastIndexOf("\\")+1);
			log.info("only file name: " + uploadFileName );
			
			attchDto.setFileName(uploadFileName);
			
			UUID uuid = UUID.randomUUID();
			
			uploadFileName = uuid.toString() + "_" + uploadFileName;
			
			// File saveFile = new File(uploadFolder,uploadFileName);
			
			try {
				File saveFile = new File(uploadPath,uploadFileName);
				multipartFile.transferTo(saveFile);
				
				attchDto.setUuid(uuid.toString());
				attchDto.setUploadPath(uploadFolderPath);
				
				// check image typoe file
				if(checkImageType(saveFile)) {
					FileOutputStream thumbnail = new FileOutputStream(new File(uploadPath, "s_" + uploadFileName));
					
					Thumbnailator.createThumbnail(multipartFile.getInputStream(), thumbnail, 100,100);
					
					thumbnail.close();
				}
				
				list.add(attchDto);
				
			} catch (Exception e) {
				// TODO: handle exception
				log.error(e.getMessage());
			}
			
		}
		
		return new ResponseEntity<List<AttachFileDTO>>(list,HttpStatus.OK);
	}
```

## **Chapter 23** 브라우저에서 섬네일 처리

- 브라우저에서의 첨부파일 업로드 결과로 JSON객체를 반환했을시 남은 작업
    - 업로드 후에 업로드 부분을 초기화
    - 결과 데이터를 이용해 화면에서 섬네일이나 파일 이미지를 보여주는 작업

### 23.1 \<input type='file'> 초기화

- \<input type='file'>은 다른 DOM 요소와 다르게 readonly라 별도의 방법으로 초기화 시켜야 한다.
```javascript
var cloneObj = $(".uploadDiv").clone();
$(".uploadDiv").html(cloneObj.html());
```
 - 아무내용 없는 객체를 복사한 다음 첨부파일 업로드 후 복사한 객체를 덮어 씌운다.

 ### 23.2 업로드된 이미지 처리

  - 섬네일은 서버를 통해 특정 URI를 호출하면 보여줄 수 있도록 처리한다.
  - 서버에서 섬네일은 GET방식을 통해서 가져올 수 있도록 처리한다.
  - 특정한 URI 뒤에 파일 이름을 추가하면 이미지 파일 데이터를 가져와서 \<img> 태그를 장성하는 과정을 통해 처리한다.
  - 주의 점으로 경로나 파일 이름에 한글, 공빽 등의 문자가 들어가면 문제가 발생하므로 JavaScript의 **encodeURIComponent()** 함수를 이용해서 URI에 문제가 없는 문자열을 생성해서 처리한다.
  
```java
@GetMapping("/display")
@ResponseBody
public ResponseEntity<byte[]> getFile(String fileName){
    
    log.info("fileName : " + fileName);
    
    File file = new File("/Users/yun-wonhui/Desktop/upload/" + fileName);
    
    log.info("file : " + file);
    
    ResponseEntity<byte[]> result = null;
    
    try {
        HttpHeaders header = new HttpHeaders();
        
        header.add("Content-Type",  URLConnection.guessContentTypeFromName(file.toPath().toString()));
        
        result = new ResponseEntity<byte[]>(FileCopyUtils.copyToByteArray(file),header,HttpStatus.OK);
    } catch (Exception e) {
        // TODO: handle exception
        e.printStackTrace();
    }
    return result;
}
```
- byte[] 로 이미지 파일의 데이터를 전송할 때는 브라우저에 보내주는 MIME 타입이 파일의 종류에 따라 달라지는점을 주의 해서 적절한 MIME 타입 데이터를 Http의 헤더 메시지에 포함될 수 있도록 처리한다.

## **Chapter 24** 첨부파일의 다운로드 혹은 원본 보여주기

 - 첨부파일이 이미지인 경우 섬네일 이미지를 클릭했을 때 화면에 크게 원본 파일을 보여주는 형태로 처리되어야 한다.
- 이 경우 브라우저에서 새로운 \<div> 등을 생성 해서 처리하는 방식을 이용하는데 흔히 'light-box'라고 한다. jQuery를 이용하는 많은 플러그인들이 있으므로, 이를 이용하거나 직접 구현이 가능하다.

### 24.1 첨부파일의 다운로드

 - 서버에서 MIME 타입을 다운로드 타입으로 지정하고, 적절한 헤더 메시지를 통해서 다운로드 이름을 지정하게 처리한다.
 - 이미지와 달리 다운로드는 MIME 타입이 고정된다.
 ```java
@GetMapping(value = "/download", produces = MediaType.APPLICATION_OCTET_STREAM_VALUE)
@ResponseBody
public ResponseEntity<Resource> downloadFile(String fileName){
    
    log.info("fownload file : " + fileName);
    
    Resource resource = new FileSystemResource("/Users/yun-wonhui/Desktop/upload/" + fileName);
    
    log.info("resource : " + resource);

    return null;
}
 ```
  - ResponseEntity<>의 타입은 byte[] 등을 사용할 수 있으나, 위의 예제는 Resource 타입을 이용해 좀 더 간단히 처리 하였다.

```java
@GetMapping(value = "/download", produces = MediaType.APPLICATION_OCTET_STREAM_VALUE)
@ResponseBody
public ResponseEntity<Resource> downloadFile(String fileName){
    
    log.info("fownload file : " + fileName);
    
    Resource resource = new FileSystemResource("/Users/yun-wonhui/Desktop/upload/" + fileName);
    
    log.info("resource : " + resource);
    
    String resourceName = resource.getFilename();
    
    HttpHeaders headers = new HttpHeaders();
    
    try {
        
        headers.add("Content-Disposition", "attachment; filename=" + new String(resourceName.getBytes("UTF-8"), "ISO-8859-1"));
        
    } catch (Exception e) {
        // TODO: handle exception'
        e.printStackTrace();
    }

    return new ResponseEntity<Resource>(resource, headers, HttpStatus.OK);
}
```
- MIME 타입은 다운로드를 할 수 있는 'application/octet-stream'으로 지정
- 다운로드 시 저장되는 이름은 'Cotent-Disposition'을 이용해서 저장한다.
    - 파일 이름에 대한 문자열 처리는 파일 이름이 한글인 경우 저장할 때 깨지는 문제를 막기 위해서 이다.

#### 24.1.1 IE/Edge 브라우저 문제
- 첨부파일 다운로드 시 Chrome 브라우저와 달리 IE에서는 한글이름이 제대로 다운되지 않는다.
    - 이것은 'Content-Disposition'의 값을 처리하는 방식이 IE의 경ㅇ우 인코딩 방식이 다르기 떄문이다.
- IE를 같이 서비스 해야 한다면 HttpServletRequest에 포함된 헤더 정보들을 이용해서 요청이 발생한 브라우저가 IE계열인지 확인해서 다르게 처리하는 방식으로 처리한다.
- HTTP 헤더 메시지 중 디바이스의 정보를 알 수 있는 헤더인 **'User-Agent'** 값을이용한다.
    > - User-Agent를 통해 브라우저의 종류, 모바일/데스크톱인지 혹은 브라우저 프로그램의 종류를 구분할 수 있다.
```java
public ResponseEntity<Resource> downloadFile(@RequestHeader("User-Agent") String userAgent, String fileName){
try {
        
    String downloadName = null;
    
    if(userAgent.contains("Trident")) {
        log.info("IE browser");
        
        downloadName = URLEncoder.encode(resourceName, "UTF-8").replace("\\+", " ");
    }else if(userAgent.contains("Edge")) {
        
        log.info("Edge browser");
        
        downloadName = URLEncoder.encode(resourceName, "UTF-8");
        
        log.info("Edge name : " + downloadName);
    }else {
        log.info("Chrome browser");
        
        downloadName = new String(resourceName.getBytes("UTF-8"), "ISO-8859-1");
    }
    
    headers.add("Content-Disposition", "attachment; filename=" + downloadName);
    
} catch (Exception e) {
    // TODO: handle exception'
    e.printStackTrace();
}
```

- @RequestHeader를 이용해서 필용한 HTTP 헤더 메시지의 내용을 수집할 수 있다.
- 이를 통해 'User-Agent'의 정보를 파악하고, 값이 'MSIE' 혹은 'Trident'(IE 브라우저의 엔진이름 - IE11처리)인 경우에는 다른 방식으로 처리하도록 한다.


### 24.2 첨부파일 삭제

 - 삭제 시 고려해야 할 점
    - 이미지 파일의 경우에는 섬네일까지 같이 삭제
    - 파일을 삭제한 후 브라우저에서도 섬네일이나 파일 아이콘이 삭제되도록 처리
    - 비정상적으로 브라우저의 종료 시 업로드된 파일의 처리
 - 업로드된 첨부파일의 삭제는 업로드 시와 동일하게 \<form> 태그 , Ajax 방식 모두 사용 가능하다.
 ```java
 @PostMapping("/deleteFile")
@ResponseBody
public ResponseEntity<String> deleteFile(String fileName, String type){
    
    log.info("deleteFile : " + fileName);
    
    File file;
    
    try {
        file = new File("/Users/yun-wonhui/Desktop/upload/" + URLDecoder.decode(fileName, "UTF-8"));
        
        file.delete();
        
        if(type.equals("image")) {
            
            String largeFileName = file.getAbsolutePath().replace("s_", "");
            
            log.info("largeFileName: " + largeFileName);
            
            file = new File(largeFileName);
            
            file.delete();
        }
            
            
    } catch (Exception e) {
        // TODO: handle exception
        e.printStackTrace();
        return new ResponseEntity<String>(HttpStatus.NOT_FOUND);
    }
    
    return new ResponseEntity<String>("delete", HttpStatus.OK);
}
 ```
  - 브라우저에서 전송하는 파일 이름과 종류를 파라미터로 받아서 파일의 종류에 따라 다르게 동작한다.

  - 강제종료, 작업 관리자를 통한 종료시 첨부파일 삭제를 감지할 수 있는 방법이 없다.
    - 해결책으로는 실제로 최종적인 결과와 서버에 업로드된 파일의 목록을 비교해서 처리하는방법이다.
    - 보통 이런작업은 spring-batch나 Quartz 라이브러리를 사용한다.

## **Chpater 25** 프로젝트의 첨부파일 - 등록
