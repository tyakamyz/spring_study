> ## 페이징 계산 공식
- 시작번호를 계산하기 위한 끝번호(endPage)
    - Math.ceil 함수를 사용하면 소수점을 올림으로 처리<br>
        - 1 페이지의 경우 Math.ceil(0.1) * 10 = 10<br>
        - 10 페이지의 경우 Math.ceil(1.0) * 10 = 10<br>
        - 11 페이지의 경우 Math.ceil(1.1) * 10 = 20
    ```java
    this.endPage = (int)(Math.ceil(페이지번호 / 10.0)) * 10;

    this.endPage = (int)(Math.ceil(페이지번호 / 20.0)) * 10;

    this.endPage = (int)(Math.ceil(페이지번호 / 30.0)) * 10;
    ```
- 시작번호(startPage)
    - 화면에 페이징 번호를 10개씩 보여주는 경우
    ```java
    this.startPage = this.endPage - 9;
    ```
- 끝번호(endPage)
    ```java
    // total : 전체 데이터 수
    // amount : 한 페이지에 출력하고자 하는 레코드의 개수
    realEnd = (int)(Math.ceil((total * 1.0) / amount));

    if(realEnd < this.endPage){
        this.endPage = realEnd;
    }
    ```
- 이전(prev)
    ```java
    this.prev = this.startPage > 1;
    ```
- 다음(next)
    ```java
    this.next = this.endPage < realEnd;
    ```
--------------
> ## 페이징 디자인 소스 참조 사이트 (Bootstrap)
- https://v4-alpha.getbootstrap.com/components/pagination/
--------------
> ## 페이징 예시
- Criteria.java
```java
package org.zerock.domain;

import lombok.Getter;
import lombok.Setter;
import lombok.ToString;

@Getter
@Setter
@ToString
public class Criteria {
	private int pageNum;
	private int amount;
	
	public Criteria() {
		this(1,10);
	}
	
	public Criteria(int pageNum, int amount) {
		this.pageNum = pageNum;
		this.amount = amount;
	}
}
```
- PageDTO.java
```java
package org.zerock.domain;

import lombok.Getter;
import lombok.ToString;

@Getter
@ToString
public class PageDTO {
	private int startPage;
	private int endPage;
	private boolean prev, next;
	
	private int total;
	private Criteria cri;
	
	public PageDTO(Criteria cri, int total) {
		this.cri = cri;
		this.total = total;
		
		this.endPage = (int)(Math.ceil(cri.getPageNum() / 10.0)) * 10;
		this.startPage = this.endPage - 9;
		
		int realEnd = (int)(Math.ceil((total * 1.0) / cri.getAmount()));
		if(realEnd < this.endPage){
	        this.endPage = realEnd;
	    }
		
		this.prev = this.startPage > 1;
		this.next = this.endPage < realEnd;
	}
}
```
- BoardController.java
```java
@GetMapping("/list")
public void list(Criteria cri ,Model model) {
    log.info("list");
    
    model.addAttribute("list", service.getList(cri));
    model.addAttribute("pageMaker", new PageDTO(cri, 총데이터 수));
}
```
- list.jsp
```jsp
<script type="text/javascript">
$(document).ready(function(){
	/* 페이징 처리 */
	var actionForm = $("#actionForm");
	
	$(".paginate_button a").on("click", function(e){
		e.preventDefault();
		
		console.log("click");
		
		actionForm.find("input[name='pageNum']").val($(this).attr("href"));
		actionForm.submit();
	});
	/* 페이징 처리 종료 */
});
```
```jsp
<div class="pull-right">
    <ul class="pagination">
        <c:if test="${pageMaker.prev}">
            <li class="paginate_button previous"><a href="${pageMaker.startPage - 1}">Previous</a></li>
        </c:if>
        <c:forEach var="num" begin="${pageMaker.startPage}" end="${pageMaker.endPage}">
            <li class="paginate_button ${pageMaker.cri.pageNum==num ? 'active':''} "><a href="${num}">${num}</a></li>
        </c:forEach>
        <c:if test="${pageMaker.next}">
            <li class="paginate_button next"><a href="${pageMaker.endPage + 1}">Next</a></li>
        </c:if>
    </ul>
</div>
<form id="actionForm" action="/board/list" method="get">
    <input type="hidden" name="pageNum" value="${pageMaker.cri.pageNum}">
    <input type="hidden" name="amount" value="${pageMaker.cri.amount}">
</form>
```
--------
> ## 게시물 조회 후 페이지번호 유지하기
- list.jsp
    - jquery를 통해 form에 있는 pageNum과 amount를 파라미터로 함께 전송
```jsp
<script type="text/javascript">
$(document).ready(function(){
	/* 페이징 처리 */
	var actionForm = $("#actionForm");
	
	$(".paginate_button a").on("click", function(e){
		e.preventDefault();
		
		console.log("click");
		
		actionForm.find("input[name='pageNum']").val($(this).attr("href"));
		actionForm.submit();
	});
	/* 페이징 처리 종료 */
	
	/* 조회 페이지 이동 */
	${".move"}.on("click", function(e){
		e.preventDefault();
		
		actionForm.append("<input type='hidden' name='bno' value='"+ $(this).attr("href")+ "'>");
		actionForm.attr("action", "/board/get");
		actionForm.submit();
	});
	/* 조회 페이지 이동 종료 */
	
});
</script>
```
```jsp
<table class="table table-striped table-bordered table-hover">
    <thead>
        <tr>
            <th>#번호</th>
            <th>제목</th>
            <th>작성자</th>
            <th>작성일</th>
            <th>수정일</th>
        </tr>
    </thead>
    <tbody>
        <c:forEach items="${list}" var="board">
        <tr>
            <td><c:out value="${board.bno}" /></td>
            <td><a class="move" href="<c:out value='${board.bno}' />"><c:out value="${board.title}" /></a></td>
            <td><c:out value="${board.writer}" /></td>
            <td><fmt:formatDate pattern="yyyy-MM-dd" value="${board.regdate}" /></td>
            <td><fmt:formatDate pattern="yyyy-MM-dd" value="${board.updateDate}" /></td>
        </tr>
        </c:forEach>
    </tbody>
</table>

<form id="actionForm" action="/board/list" method="get">
    <input type="hidden" name="pageNum" value="${pageMaker.cri.pageNum}">
    <input type="hidden" name="amount" value="${pageMaker.cri.amount}">
</form>
```
- BorardController.java
    - @ModelAttribute를 통해 pageNum과 amount를 전달 받음
    - <font color="red"><Strong>중요! - </Strong></font>ModelAttribute를 통해 전달 받은 데이터는 model.addAttribute로 명시해주지 않아도 파라미터가 전달됨
```java
@GetMapping({"/get", "/modify"})
public void get(@RequestParam("bno") Long bno, @ModelAttribute("cri") Criteria cri, Model model) {
    log.info("/get or /modify");
    
    model.addAttribute("board", service.get(bno));
}
```
- get.jsp
```jsp
<form id='operForm' action="/board/modify" method="get">
    <input type="hidden" id="bno" name="bno" value='<c:out value="${board.bno}"/>'>
    <input type="hidden" name="pageNum" value="${cri.pageNum}">
    <input type="hidden" name="amount" value="${cri.amount}">
</form>
```
- modify.jsp
```jsp
<form role="form" action="/board/modify" method="post">
    <input type="hidden" name="pageNum" value="${cri.pageNum}">
    <input type="hidden" name="amount" value="${cri.amount}">
    <div class="form-group">
        <label>Bno</label><input class="form-control" name="bno" value='<c:out value="${board.bno}"/>' readonly="readonly">
    </div>
</form>
```
- BoardController.java
    - 수정, 삭제 후 list로 redirect하는 부분에 페이지번호와 레코드개수를 넘겨줌
```java
@PostMapping("/modify")
public String modify(BoardVO board, @ModelAttribute("cri") Criteria cri, RedirectAttributes rttr) {
    log.info("modify: " + board);
    
    if(service.modify(board)) {
        rttr.addFlashAttribute("result", "success");
    }
    
    rttr.addFlashAttribute("pageNum", cri.getPageNum());
    rttr.addFlashAttribute("amount", cri.getAmount());
    
    return "redirect:/board/list";
}

@PostMapping("/remove")
public String remove(@RequestParam("bno") Long bno, @ModelAttribute("cri") Criteria cri, RedirectAttributes rttr) {
    log.info("remove....." + bno);
    
    if(service.remove(bno)) {
        rttr.addFlashAttribute("result", "success");
    }
    
    rttr.addFlashAttribute("pageNum", cri.getPageNum());
    rttr.addFlashAttribute("amount", cri.getAmount());
    
    return "redirect:/board/list";
}
```
- modify.jsp
    - 수정페이지에서 목록으로 페이지 이동 시, 목록페이지는 pageNum과 amount만을 사용하므로<br>
    pageNum, amount을 clone()을 통해 보관해 놓고 \<form>태그의 내용을 모두 삭제한 뒤<br>
    목록페이지에 필요한 pageNum, amount만 다시 form안에 넣어줌
```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
	pageEncoding="UTF-8"%>
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>
<%@ taglib uri="http://java.sun.com/jsp/jstl/fmt" prefix="fmt" %>
	
<%@include file="../includes/header.jsp"%>

<script type="text/javascript">
$(document).ready(function(){
	var formObj = $("form");
	
	$('button').on("click", function(e){
		e.preventDefault();
		
		var operation = $(this).data("oper");
		
		console.log(operation);
		
		if(operation === 'remove'){
			formObj.attr("action", "/board/remove");
		}else if(operation === 'list'){
			formObj.attr("action", "/board/list").attr("method", "get");
			var pageNumTag = $("input[name='pageNum']").clone();
			var amountTag = $("input[name='amount']").clone();			
			
			formObj.empty();
			formObj.append(pageNumTag);
			formObj.append(amountTag);
		}
		formObj.submit();
	});
});
</script>

<div class="row">
	<div class="col-lg-12">
		<h1 class="panel-heading">Board Modify</h1>
	</div>
</div>
<!-- /.row -->
<div class="row">
	<div class="col-lg-12">
		<div class="panel panel-default">
			<div class="panel-heading">Board Modify</div>
			<!-- /.panel-heading -->
			<div class="panel-body">
				<form role="form" action="/board/modify" method="post">
					<input type="hidden" name="pageNum" value="${cri.pageNum}">
					<input type="hidden" name="amount" value="${cri.amount}">
					<div class="form-group">
						<label>Bno</label><input class="form-control" name="bno" value='<c:out value="${board.bno}"/>' readonly="readonly">
					</div>
					<div class="form-group">
						<label>Title</label><input class="form-control" name="title" value='<c:out value="${board.title}" />'>
					</div>
					<div class="form-group">
						<label>Text area</label><textarea class="form-control" rows="3" name="content"><c:out value="${board.content}" /></textarea>
					</div>
					<div class="form-group">
						<label>Writer</label><input class="form-control" name="writer" value='<c:out value="${board.writer}" />'>
					</div>
					<div class="form-group">
						<label>RegDate</label><input class="form-control" name="regdate" value='<fmt:formatDate pattern = "yyyy/MM/dd" value="${board.regdate}" />' readonly="readonly">
					</div>
					<div class="form-group">
						<label>UpdateDate</label><input class="form-control" name="updateDate" value='<fmt:formatDate pattern = "yyyy/MM/dd" value="${board.updateDate}" />' readonly="readonly">
					</div>
					<button type="submit" data-oper="modify" class="btn btn-default">Modify</button>
					<button type="submit" data-oper="remove" class="btn btn-default">Remove</button>
					<button data-oper="list" class="btn btn-default" onClick="location.href='/board/list'">List</button>
				</form>
			</div>
		</div>
	</div>
</div>
<%@include file="../includes/footer.jsp"%>
```