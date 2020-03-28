## - 뒤로가기, 앞으로가기의 중복 등록, 다운로드 등의 처리 막기

  #### - 위의 뒤로가기 문제가 발생하는 이유로는 '뒤로가기', '앞으로가기'를 하면 서버를 다시 호출하는게 아니라 과거에 자신이 가진 모든 데이터를 활용하기 때문이다. (일회성 데이터 사용방식인 RedirectAttributes 또한 예외없이 사용되는 것을 확인 할 수 있다.)

  >  - 해결방안  
  >  스택 구조로 동학하는 '**window 의 history**' 객체 사용

  - 기존 코드
  ```js
  $(document).ready(function(){
		var result = '<c:out value="${result}"/>';
	
	checkModal(result);
	
	function checkModal(result){
		if(result === ''){
			return;
		}
		
		if(parseInt(result) > 0){
			$(".modal-body").html("게시글 " + parseInt(result) + " 번이 등록되었습니다.");
		}
		
		$("#myModal").modal("show");
	}
  )};
```

- history 적용
  ```js
  $(document).ready(function(){
		var result = '<c:out value="${result}"/>';
	
	checkModal(result);
	
	history.replaceState({}, null, null);
	
	function checkModal(result){
		if(result === '' || history.state){
			return;
		}
		
		if(parseInt(result) > 0){
			$(".modal-body").html("게시글 " + parseInt(result) + " 번이 등록되었습니다.");
		}
		
		$("#myModal").modal("show");
	}
  )};
```
 - 최초 checkModal함수 작동시에 모달창을 생성 한 후 'history.replaceState' 를 통해 history에 한번 작동 했다는 표시를 둔다
 - 다음번의 checkModal 함수 작동시 history.state를 체크함으로 써 해당 함수가 이전에 사용된지를 확인 하여 Modal창 생성을 막는다.