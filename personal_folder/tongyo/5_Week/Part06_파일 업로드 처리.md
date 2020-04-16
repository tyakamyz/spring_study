# 6장. 파일 업로드 처리
> ## ch21. 파일 업로드 방식
- 파일 업로드 방식
    - \<form> 태그를 이용하는 방식
        - 브라우저의 제한이 없어야 하는 경우에 사용
        - 일반적으로 페이지 이동과 동시에 첨부파일을 업로드하는 방식
        - \<iframe>을 이용해서 화면의 이동 없이 첨부파일을 처리하는 방식
    - Ajax를 이용한 방식
        - 첨부파일을 별도로 처리하는 방식
        - \<input type="file">을 이용하고 Ajax로 처리하는 방식
        - HTML5의 Drag And Drop 기능이나 jQuery 라이브러리를 이용해서 처리하는 방식
-------
- \<form> 방식의 파일 업로드
    > uploadForm.jsp
    ```jsp
    <%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
    <!DOCTYPE html>
    <html>
    <head>
    <meta charset="UTF-8">
    <title>Insert title here</title>
    </head>
    <body>
        <from action="uploadFormAction" method="post" enctype="mulipart/form-data">
        <input type="file" name="uploadFile" multiple>
        <button>Submit</button>
        </from>
    </body>
    </html>
    ```
    > UploadController.java
    ```java
    package org.zerock.controller;

    import java.io.File;

    import org.springframework.stereotype.Controller;
    import org.springframework.ui.Model;
    import org.springframework.web.bind.annotation.GetMapping;
    import org.springframework.web.bind.annotation.PostMapping;
    import org.springframework.web.multipart.MultipartFile;

    import lombok.extern.log4j.Log4j;

    @Controller
    @Log4j
    public class UploadController {
        @GetMapping("/uploadForm")
        public void uploadForm() {
            
            log.info("upload form");
        }
        
        @PostMapping("/uploadFormAction")
        public void uploadFormPost(MultipartFile[] uploadFile, Model model) {
            
            String uploadFolder = "/Users/tongbook/Desktop/study/upload";
            
            for(MultipartFile multipartFile : uploadFile) {
                log.info("----------------------");
                log.info("Upload File Name: " + multipartFile.getOriginalFilename());
                log.info("Upload File Size: " + multipartFile.getSize());
                
                File saveFile = new File(uploadFolder, multipartFile.getOriginalFilename());
                
                try {
                    multipartFile.transferTo(saveFile);
                }catch(Exception e) {
                    log.error(e.getMessage());
                }
            }
        }
    }
    ```
-------
- Ajax를 이용하는 파일 업로드
    - Ajax는 FormData 타입의 객체에 파일을 담음 (가상의 form태그 와 같다고 생각하면 쉬움)
    > uploadAjax.jsp
    ```jsp
    <%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
    <!DOCTYPE html>
    <html>
    <head>
    <meta charset="UTF-8">
    <title>Insert title here</title>
    </head>
    <body>
        <h1>Upload with Ajax</h1>
        
        <div class="uploadDiv">
            <input type="file" name="uploadFile" multiple>
        </div>
        
        <button id="uploadBtn">Upload</button>
        
        <script src="https://code.jquery.com/jquery-3.5.0.js"
        integrity="sha256-r/AaFHrszJtwpe+tHyNi/XCfMxYpbsRg2Uqn0x3s2zc="
        crossorigin="anonymous"></script>
        
        <script>
            $(document).ready(function(){
                $("#uploadBtn").on("click", function(e){
                    var formData = new FormData();
                    
                    var inputFile = $("input[name='uploadFile']");
                    
                    var files = inputFile[0].files;
                    
                    console.log(files);
                    
                    //add filedata to formData
                    for(var i=0; i<files.length; i++){
                        formData.append("uploadFile", files[i]);
                    }
                    
                    $.ajax({
                        url : "/uploadAjaxAction",
                        processData : false,	// ajax로 파일 전송 시 반드시 false
                        contentType : false,	// ajax로 파일 전송 시 반드시 false
                        data : formData,
                        type : "POST",
                        success : function(result){
                            alert("uploaded");
                        }
                    });
                });
            });
        </script>
    </body>
    </html>
    ```
    > uploadController.java
    ```java
    @GetMapping("/uploadAjax")
	public void uploadAjax() {
		log.info("upload ajax");
	}
	
	@PostMapping("/uploadAjaxAction")
	public void uploadAjaxPost(MultipartFile[] uploadFile) {
		log.info("update ajax post");
		
		String uploadFolder = "/Users/tongbook/Desktop/study/upload";
		
		for(MultipartFile multipartFile : uploadFile) {
			log.info("----------------");
			log.info("Upload File Name : " + multipartFile.getOriginalFilename());
			log.info("Upload File Size : " + multipartFile.getSize());
			
			String uploadFileName = multipartFile.getOriginalFilename();
			
			// IE has file path
			uploadFileName = uploadFileName.substring(uploadFileName.lastIndexOf("\\") + 1);
			log.info("only file name : " + uploadFileName);
			
			File saveFile = new File(uploadFolder, uploadFileName);
			
			try {
				multipartFile.transferTo(saveFile);
			}catch(Exception e) {
				log.error(e.getMessage());
			}
		}
	}
    ```
---------
> ## ch22. 파일 업로드 상세 처리
- 고려해야 하는점
    - 동일한 이름으로 파일이 업로드 되었을 때 기존 파일이 사라지는 문제
    - 이미지 파일의 경우에는 원본 파일의 용량이 큰 경우 섬네일 이미지를 생성해야 하는 문제
    - 이미지 파일과 일반 파일을 구분해서 다운로드 혹은 페이지에서 조회하도록 처리하는 문제
    - 첨부파일 공격에 대비하기 위한 업로드 파일의 확장자 제한
    -----
- 파일의 확장자나 크기의 사전 처리
    > uploadAjax.jsp
    ```jsp
    <!-- 파일의 확장자, 크기 사전 처리 -->
    var regex = new RegExp("(.*?)\.(exe|sh|zip|alz)$");
    var maxSize = 5242880;	// 5MB
    
    function checkExtension(fileName, fileSize){
        if(fileSize >= MaxSize){
            alert("파일 사이즈 초과");
            return false;
        }
        
        if(regex.test(fileName)){
            alert("홰당 종류의 파일은 업로드할 수 없습니다.");
            return false;
        }
        return true;
    }