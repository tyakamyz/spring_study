# Security 개발시 주의점
> 동적 표현식 사용 시 taglib 선언
```jsp
<%@ taglib uri="http://www.springframework.org/security/tags" prefix="sec" %>
```
-----
> POST 방식의 전송을 사용할 때 반드시 CSRF 토큰 사용
```jsp
<input type="hidden" name="${_csrf.parameterName}" value="${_csrf.token}" />
```
----
> Security 사용시 한글 깨짐 현상
- web.xml에 설정해줘야함 (순서 주의!)
```xml
<!-- 스프링 시큐리티 인코딩 설정. 인코딩 설정을 먼저 적용하고 스프링 시큐리티 적용! 순서 반드시 지켜야함! -->
<filter>
    <filter-name>encodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
        <param-name>encoding</param-name>
        <param-value>UTF-8</param-value>
    </init-param>
</filter>

<filter-mapping>
    <filter-name>encodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>

<!-- 스프링 시큐리티가 스프링 MVC에서 사용되기 위해서는 필터를 이용해서 스프링 동작에 관여하도록 설정해야함 -->
<filter>
    <filter-name>springSecurityFilterChain</filter-name>
    <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
</filter>

<filter-mapping>
    <filter-name>springSecurityFilterChain</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```
-----
> 수정이나 삭제 처리 시
- 어노테이션을 활용한 본인 계정 확인 방법
```java
@PreAuthorize("principal.username == #board.writer")
@PostMapping("/modify")
public String modify(BoardVO board, @ModelAttribute("cri") Criteria cri, RedirectAttributes rttr) {
    log.info("modify: " + board);
    
    if(service.modify(board)) {
        rttr.addFlashAttribute("result", "success");
    }
    
//		rttr.addFlashAttribute("pageNum", cri.getPageNum());
//		rttr.addFlashAttribute("amount", cri.getAmount());
//		rttr.addFlashAttribute("type", cri.getType());
//		rttr.addFlashAttribute("keyword", cri.getKeyword());
    
    return "redirect:/board/list" + cri.getListLink();
}

@PreAuthorize("principal.username == #writer")
@PostMapping("/remove")
public String remove(@RequestParam("bno") Long bno, @ModelAttribute("cri") Criteria cri, RedirectAttributes rttr, String writer) {
    log.info("remove....." + bno);
    
    List<BoardAttachVO> attachList = service.getAttachList(bno);
    
    if(service.remove(bno)) {
        deleteFiles(attachList);
        rttr.addFlashAttribute("result", "success");
    }
    
//		rttr.addFlashAttribute("pageNum", cri.getPageNum());
//		rttr.addFlashAttribute("amount", cri.getAmount());
//		rttr.addFlashAttribute("type", cri.getType());
//		rttr.addFlashAttribute("keyword", cri.getKeyword());
    
    return "redirect:/board/list" + cri.getListLink();
}
```
----
> 파일 업로드 시
- 스프링 시큐리티가 적용된 후에는 게시물에 파일 첨부가 정상적으로 동작하지 않는 문제점이 생김
- POST 방식으로 전송되기 때문에 발생하는 문제
- CSRF 토큰 전달이 필요
```jsp
var csrfHeaderName = "${_csrf.headerName}";
var csrfTokenValue = "${_csrf.token}";

$("input[type='file']").change(function(e) {

    var formData = new FormData();

    var inputFile = $("input[name='uploadFile']");

    var files = inputFile[0].files;

    for (var i = 0; i < files.length; i++) {

        if (!checkExtension(files[i].name, files[i].size)) {
            return false;
        }
        formData.append("uploadFile", files[i]);

    }

    $.ajax({
        url : '/uploadAjaxAction',
        processData : false,
        contentType : false,
        beforeSend : function(xhr){
            xhr.setRequestHeader(csrfHeaderName, csrfTokenValue);
        },
        data : formData,
        type : 'POST',
        dataType : 'json',
        success : function(result) {
            console.log(result);
            showUploadResult(result); //업로드 결과 처리 함수 

        }
    }); //$.ajax

});
```
-----
> 첨부파일 제거 시
- 스프링 시큐리티가 적용된 후에는 게시물에 파일 첨부가 정상적으로 동작하지 않는 문제점이 생김
- POST 방식으로 전송되기 때문에 발생하는 문제
- CSRF 토큰 전달이 필요
```jsp
$(".uploadResult").on("click", "button", function(e){
    console.log("delete file");

    var csrfHeaderName = "${_csrf.headerName}";
    var csrfTokenValue = "${_csrf.token}";

    var targetFile = $(this).data("file");
    var type = $(this).data("type");

    var targetLi = $(this).closest("li");

    $.ajax({
        url: '/deleteFile',
        data: {fileName: targetFile, type:type},
        beforeSend : function(xhr){
            xhr.setRequestHeader(csrfHeaderName, csrfTokenValue);
        },
        dataType:'text',
        type: 'POST',
        success: function(result){
            alert(result);
            
            targetLi.remove();
            }
    }); //$.ajax
});
```
-----
> CSRF 토큰 기본 설정 처리
- ajaxSend()를 이용한 코드는 모든 Ajax 전송 시 CSRF 토큰을 같이 전송하도록 셋팅하여 Ajax 사용시 매번 beforeSend를 호출해야 하는 번거로움을 줄일 수 있음
```jsp
 var csrfHeaderName = "${_csrf.headerName}";
var csrfTokenValue = "${_csrf.token}";

$(document).ajaxSend(function(e, xhr, options){
    xhr.setRequestHeader(csrfHeaderName, csrfTokenValue);
})
```
----
> 파일 첨부, 삭제 시 보안 강화
- 보안을 위해 외부에서 로그인한 사용자만이 할 수 있도록 제한
- @PreAuthorize("isAuthenticated()") 어노테이션 사용
```java
@PreAuthorize("isAuthenticated()")
@PostMapping(value = "/uploadAjaxAction", produces=MediaType.APPLICATION_JSON_UTF8_VALUE)
@ResponseBody
public ResponseEntity<List<AttachFileDTO>> uploadAjaxPost(MultipartFile[] uploadFile) {
    log.info("update ajax post");
    
    List<AttachFileDTO> list = new ArrayList();
    
    String uploadFolder = "/Users/tongbook/Desktop/study/upload";
    
    String uploadFolderPath = getFolder();
    
    // 폴더 생성
    File uploadPath = new File(uploadFolder, uploadFolderPath);
    log.info("upload path : " + uploadPath);
    
    if(uploadPath.exists() == false) {
        uploadPath.mkdir();	// yyyy/mm/dd
    }
    // 생성 종료
    
    for(MultipartFile multipartFile : uploadFile) {
        log.info("----------------");
        log.info("Upload File Name : " + multipartFile.getOriginalFilename());
        log.info("Upload File Size : " + multipartFile.getSize());
        
        AttachFileDTO attachDTO = new AttachFileDTO();
        
        String uploadFileName = multipartFile.getOriginalFilename();
        
        // IE has file path
        uploadFileName = uploadFileName.substring(uploadFileName.lastIndexOf("\\") + 1);
        log.info("only file name : " + uploadFileName);
        attachDTO.setFileName(uploadFileName);
        
        UUID uuid = UUID.randomUUID();
        
        uploadFileName = uuid.toString() + "_" + uploadFileName;
        
        try {
            File saveFile = new File(uploadPath, uploadFileName);
            multipartFile.transferTo(saveFile);
            
            attachDTO.setUuid(uuid.toString());
            attachDTO.setUploadPath(uploadFolderPath);
            
            if(checkImageType(saveFile)) {
                
                attachDTO.setImage(true);
                
                FileOutputStream thumbnail = new FileOutputStream(new File(uploadPath, "s_" + uploadFileName));
                
                Thumbnailator.createThumbnail(multipartFile.getInputStream(), thumbnail, 100, 100);
                
                thumbnail.close();
            }
            
            list.add(attachDTO);
        }catch(Exception e) {
            log.error(e.getMessage());
        }
    }
    
    return new ResponseEntity<>(list, HttpStatus.OK);
}

@PreAuthorize("isAuthenticated()")
@PostMapping("/deleteFile")
@ResponseBody
public ResponseEntity<String> deleteFile(String fileName, String type){
    log.info("deleteFile : " + fileName);
    log.info("type : " + type);
    
    File file;
    
    try {
        file = new File("/Users/tongbook/Desktop/study/upload/" + URLDecoder.decode(fileName, "UTF-8"));
        
        file.delete();
        
        if(type.equals("image")) {
            String largeFileName = file.getAbsolutePath().replace("s_", "");
            
            log.info("largeFileName : " + largeFileName);
            
            file = new File(largeFileName);
            
            file.delete();
        }
    }catch(UnsupportedEncodingException e) {
        e.printStackTrace();
        
        return new ResponseEntity<>(HttpStatus.NOT_FOUND);
    }
    
    return new ResponseEntity<String>("deleted", HttpStatus.OK);
}
```
----