# RedirectAttributes 객체
- 일회성으로만 데이터를 전달
- addFlashAttribute()로 보관된 데이터는 단 한 번만 사용할 수 있게 보관됨
- 내부적으로는 HttpSession을 통해 처리
- redirect 처리할 때 사용
```java
@PostMapping("/register")
	public String register(BoardVO board, RedirectAttributes rttr) {
		log.info("register: " + board);
		
		service.register(board);
		
		rttr.addFlashAttribute("result", board.getBno());
		
		return "redirect:/board/list";
	}
```
- 예시의 경우 게시물 등록 후 목록으로 이동
```jsp
<script type="text/javascript">
$(document).ready(function(){
	var result = '<c:out value="${result}" />';
})
</script>
```
- list 페이지에서 확인 가능
- 새로운 게시물의 번호는 addFlashAttribute()로 저장되었기 때문에 한 번도 사용된 적이 없다면 result에 번호가 담김
- 사용자가 직접 /board/list를 호출하거나 새로고침을 통해 호출하는 경우엔 result에 빈값이 담김
- 이를 활용하면 경고창이나 모달창 등을 보여주는 방식으로 처리 가능<br>
(게시물 등록이 완료되었습니다 메시지 등등..)