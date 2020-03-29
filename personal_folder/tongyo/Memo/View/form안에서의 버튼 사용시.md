> ## form 안에서의 버튼 사용 시
- form 안에서의 모든 버튼은 submit으로 처리 됨
- 버튼 2개 이상 사용하여 다른 기능을 구현해야 할 경우 처리 방법
- 해당 예제는 수정, 삭제, 목록 버튼을 함께 구현해야 할 경우
- 주의할 점! 버그인지 모르겠으나 'function(){}' 를 '() => {}' 약식으로 사용할 경우 '$(this).data("oper");' 값을 찾지 못함 
```jsp
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
			formObj.empty();
		}
		formObj.submit();
	});
});
</script>
```
```jsp
<form role="form" action="/board/modify" method="post">
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
```