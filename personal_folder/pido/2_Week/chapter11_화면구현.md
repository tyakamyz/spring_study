
[Part3] - chapter 11
=========================

화면처리
-----------
부트스트랩 : https://startbootstrap.com/themes/   
예제 디자인 : http://samara.computer/531_adaptive/startbootstrap-master/startbootstrap-master/sb-admin-v2.php

JSP   
==> /webapp/WEB-INF/views/board/list.jsp
- servlet-context.xml 의 ViewResolver 객체로 화면 설정    
- /WEB-INF 경로는 브라우저에서 직접 접근이 불가능하므로 ViewResolver를 기본적으로 사용    
- tomcat WebSettings 의 Path 는 '/' 로 설정

CSS/js  
==> /webapp/resources/CSS 또는 js   
- servlet-context.xml 경로 참조    
 ```<resources mapping="/resources/**" location="/resources/" />```
- 부트스트랩의 link 경로는 모두 /resources/를 붙여줘야한다.

includes 
- header와 footer jsp 에 대한 처리
```java
<%@include file="../includes/header.jsp"  %>
<%@include file="../includes/footer.jsp" %>        
```

jquery   
- header 에 넣어 모든 페이지에서 사용가능하도록 처리    
```<script src="https://ajax.googleapis.com/ajax/libs/jquery/3.4.1/jquery.min.js"></script>```

jstl
- JSTL 출력과 포맷을 적용
```java
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>
<%@ taglib uri="http://java.sun.com/jsp/jstl/fmt" prefix="fmt" %>
```

<br>

화면 상 CRUD
---------------
1. 등록
- ```<form>``` 태그 이용시 ```<input>,<textarea>``` 속성은 BoardVO 클래스의 변수와 일치하여야한다. 
- 등록 시 한글 인코딩 문제   
    web.xml(XML방식)
    ```xml
    <filter>
		<filter-name>encoding</filter-name>
		<filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
		<init-param>
			<param-name>encoding</param-name>
			<param-value>UTF-8</param-value>
		</init-param>
	</filter>
	<filter-mapping>
		<filter-name>encoding</filter-name>
		<servlet-name>appServlet</servlet-name>
	</filter-mapping>
    ```
    WebConfig.java(JAVA방식)    
    ```java
    @Override
	protected Filter[] getServletFilters() {
		CharacterEncodingFilter characterEncodingFilter = new CharacterEncodingFilter();
		characterEncodingFilter.setEncoding("UTF-8");
		characterEncodingFilter.setForceEncoding(true);
		
		return new Filter[] { characterEncodingFilter };
	}
    ```
- 재전송 처리   
```RedirectAttributes객체의 addFlashAttribute()```   
일회성으로만 데이터를 전달하여 중복 전달을 방지한다. 

2. 조회
- 뒤로가기 처리   
window의 history 객체   
    * 스택구조로 동작
    *   ```javascript
        <script type="text/javascript">
            var result = '<c:out value="${result}"/>';
	        checkModal(result);

            // window의 history객체(모달 재호출 방지)
            history.replaceState({}, null, null);
            
            // 등록 확인 모달 
            function checkModal(result) {
                if(result == '' || history.state){
                    return;
                }
                if(parseInt(result) > 0) {
                    $(".modal-body").html("게시글 " + parseInt(result) + " 번이 등록되었습니다.");
                }
                $("#myModal").modal("show");
            }
        </script>
        ```
3. 수정/삭제
    ```java
    @GetMapping({"/get", "/modify"})
    public void get(@RequestParam("bno") Long bno, @ModelAttribute("cri") Criteria cri, Model model) {
        log.info("/get or modify");
        model.addAttribute("board", service.get(bno));
    }
    ```
- URL을 배열로 처리 할 수 있으므로 @GetMapping URL을 배열로 처리 
- javascript의 ```<button>```태그의 'data-oper' 속성을 이용한 동작 처리
    ```jsp
    <button type="submit" data-oper='modify' class="btn btn-default">Modify</button>
    <button type="submit" data-oper='remove' class="btn btn-danger">Remove</button>
    <button type="submit" data-oper='list' class="btn btn-info">List</button>	
    ```            
    ```javascript
    // 버튼 태그의 data-oper 속성을 통해 원하는 기능 처리 
    var operation = $(this).data("oper");    
    if(operation === 'remove') {
        formObj.attr("action", "/board/remove");
    }else if(operation === 'list') {
        // move to list
        formObj.attr("action", "/board/list").attr("method", "get");
        formObj.empty();
    }    
    ```
    * e.preventDefault() : ```<form>``` 태그의 모든 버튼은 기본적으로 submit 을 처리하는데 , 이를 방지한다. 

