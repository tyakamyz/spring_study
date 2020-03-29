
[Part3] - chapter 13, 14
=========================

스프링 페이징처리
-----------------

확장성 Class 생성

```java
@Getter
@Setter
@ToString
public class Criteria {

	// paging 
	private int pageNum;    // 페이지 번호
	private int amount;     // 한 페이지당 몇개의 데이터를 보여줄 지
		
	// 페이징 생성자 기본값 지정 
	public Criteria() {
		this(1,10);
	}
	
	public Criteria(int pageNum, int amount) {
		this.pageNum = pageNum;
		this.amount = amount;
	}
}
```
Mapper 
```java
// Criteria 타입을 파라미터로 설정
public List<BoardVO> getListWithPaging(Criteria cri);
```
XML
```xml
<select id="getListWithPaging" resultType="org.zerock.domain.BoardVO">
    <![CDATA[
        select 
                bno
            ,	title
            ,	content
            ,	writer
            ,	regdate
            ,	updatedate
        from (
            select /*+ INDEX_DESC(tbl_board pk_board) */
                    rownum rn
                ,	bno
                ,	title
                ,	content
                ,	writer
                ,	regdate
                ,	updatedate
            from tbl_board
            where rownum <= 20
            )
        where rn > 10
        ]]>
</select>
```
Service 인터페이스, ServiceImpl   
Criteria 클래스를 파라미터로 처리 하도록 변경이 필요함.

Controller
```java
// Criteria 타입을 파라미터로 설정
@GetMapping("/list")
public void list(Criteria cri, Model model) {
    model.addAttribute("list", service.getList(cri));
}
```
<hr />

페이징 화면 처리
-----------------
페이징 시 필요한 정보들

* 현재 페이지 번호
* 이전과 다음으로 이동 가능한 링크 표시여부(prev, next)
* 페이지 시작, 끝 번호 

페이징 DTO 클래스로 생성자 생성
```java
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
		
		// 페이징의 끝번호
		this.endPage = (int)(Math.ceil(cri.getPageNum() / 10.0)) * 10;
		// 페이징의 시작번호
		this.startPage = this.endPage - 9;

		// 진짜 끝 페이지  
		int realEnd = (int)(Math.ceil((total * 1.0) / cri.getAmount()));
		
		if(realEnd < this.endPage) {
			this.endPage = realEnd;
		}
		
		// 이전, 다음 
		this.prev = this.startPage > 1;
		this.next = this.endPage < realEnd;
		
	}
	
}
```

Controller    
```java
// model에 담아 화면에 전달한다.
model.addAttribute("pageMaker", new PageDTO(cri, totalCount));
```

jsp
```jsp
<div class='pull-right'>
    <ul class="pagination">
        <c:if test="${pageMaker.prev }">
            <li class="paginate_button previous"><a href="${pageMarker.startPage -1 }">Previous</a></li>
        </c:if>
        
        <c:forEach var="num" begin="${pageMaker.startPage }" end="${pageMaker.endPage }">
            <li class="paginate_button ${pageMaker.cri.pageNum == num ? "active":"" } "><a href="${num }">${num }</a></li>
        </c:forEach>
        
        <c:if test="${pageMaker.next }">
            <li class="paginate_button next"><a href="${pageMaker.endPage + 1 }">Next</a></li> 
        </c:if>
    </ul>                      
    </div>
    <form id="actionForm" action="/board/list" method='get'>
        <input type='hidden' name='pageNum' value='${pageMaker.cri.pageNum }'>
        <input type='hidden' name='amount' value='${pageMaker.cri.amount }'>                    	
    </form>
```

javascript
```javascript
var actionForm = $("#actionForm");
$(".paginate_button a").on("click", function(e){
	e.preventDefault();
	console.log('click');
	
	actionForm.find("input[name='pageNum']").val($(this).attr("href"));
	actionForm.submit();
});
```
<br>

페이지 번호 유지   

GET방식 Controller 
```java
// Criteria 를 파라미터로 처리
@ModelAttribute("cri") Criteria cri 
``` 
POST방식 Controller 
```java
// Criteria 를 파라미터로 처리
@ModelAttribute("cri") Criteria cri 

// RedirectAttributes로 값을 전달
rttr.addAttribute("pageNum", cri.getPageNum());
rttr.addAttribute("amount", cri.getAmount());
```
jsp
```jsp
// form 태그 안에 hidden 값으로 데이터 전달 
<form id='operForm' action="/board/modify" method="get">
    <input type='hidden' name='pageNum' value='<c:out value="${cri.pageNum }"/> '>
    <input type='hidden' name='amount' 	value='<c:out value="${cri.amount }"/> '>
</form>    
```

javascript
```javascript
// 수정/삭제 페이지에서 목록페이지 이동시 
var formObj = $("form");
	
	$('button').on("click", function(e){
		
		// form 태그의 기본 동작인 submit으로 처리 되는것을 방지 
		e.preventDefault();
		
		// 버튼 태그의 data-oper 속성을 통해 원하는 기능 처리 
		var operation = $(this).data("oper");
		
		if(operation === 'remove') {
			formObj.attr("action", "/board/remove");
		}else if(operation === 'list') {
			// move to list
			formObj.attr("action", "/board/list").attr("method", "get");
			
			// clone: 잠시복사
			var pageNumTag 	= $("input[name='pageNum']").clone();
			var amountTag 	= $("input[name='amount']").clone();
            
            		// 데이터를 비우고 필요한 pagenum과 amount 만 다시 넣는다.
			formObj.empty();
			formObj.append(pageNumTag);
			formObj.append(amountTag);
		}
		formObj.submit();
	});
```
