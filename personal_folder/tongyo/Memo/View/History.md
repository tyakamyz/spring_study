# History 객체 조작을 활용한 뒤로가기 버그 수정 방법
- 브라우저에서 뒤로가기/앞으로가기 사용 시 서버에서 다시 호출하는 것이 아니라 과거에 자신이 가진 데이터를 활용함
- 모달창, 다운로드, 메시지 등의 이벤트 진행 후 다음 페이지로 이동하고 다시 뒤로가기를 통해 해당페이지로 돌아올 경우 같은 이벤트가 또 발생하는 것을 차단하는 방법
- 해당 예제는 뒤로가기 시에 모달창이 한번 더 띄우는 것을 방지하는 소스
```jsp
<script type="text/javascript">
$(document).ready(function(){
	var result = '<c:out value="${result}" />';
	
	checkModal(result);
	
    /* history에서 다시 보여줄 필요 없다는 표식을 남김 */
	history.replaceState({},null,null);
	
	function checkModal(result){
        /* history에 체크했던 내용이 있다면 다시 호출하지 않음 */
		if(result === '' || history.state){
			return;
		}
		
		if(parseInt(result) > 0){
			$(".modal-body").html("게시글 " + parseInt(result) + "번이 등록되었습니다.");
		}
		
		$("#myModal").modal("show");
	}
	
	$("#regBtn").on("click", ()=>{
		self.location = "/board/register";
	});
});
</script>
```