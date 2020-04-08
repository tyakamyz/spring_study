># REST의 @Annotation 

<details markdown="1">
<summary>전송방식에 따른 Annotation</summary>

- ## 전송방식에 따른 어노테이션

|작업|HTTP Method|URI 예제|어노테이션|Operation Performed|
|---|---|---|---|---|
|등록(Create)|POST|/members/new|@PostMapping|리소스를 가져옴|
|조회(Read)|GET|/members/{id}|@GetMapping|정의된 의미가 없으면 자원을 생성|
|수정(Update)|PUT|/members/{id}+body(json데이터 등)|@RequestMapping|자원을 생성하거나 업데이트|
|삭제(Delete)|DELETE|/members/{id}|@DeleteMapping|자원을 삭제|
> 예제
```java
package org.zerock.controller;

import java.util.List;

import org.springframework.http.HttpStatus;
import org.springframework.http.MediaType;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.DeleteMapping;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;
import org.zerock.domain.Criteria;
import org.zerock.domain.ReplyPageDTO;
import org.zerock.domain.ReplyVO;
import org.zerock.service.ReplyService;

import lombok.AllArgsConstructor;
import lombok.extern.log4j.Log4j;

@RequestMapping("/replies/")
@RestController
@Log4j
@AllArgsConstructor
public class ReplyController {
	
	private ReplyService service;
	
	//댓글 등록
	@PostMapping(value="/new", consumes="application/json", produces= {MediaType.TEXT_PLAIN_VALUE})
	public ResponseEntity<String> create(@RequestBody ReplyVO vo){
		log.info("ReplyVO: " + vo);
		
		int insertCount = service.register(vo);
		
		log.info("Reply INSERT COUNT: " + insertCount);
		
		return insertCount == 1
				? new ResponseEntity<>("success", HttpStatus.OK)
				: new ResponseEntity<>(HttpStatus.INTERNAL_SERVER_ERROR);
				//삼항 연산자 처리
	}
	
	// 댓글 목록
	@GetMapping(value="/pages/{bno}/{page}", produces= {MediaType.APPLICATION_ATOM_XML_VALUE, MediaType.APPLICATION_JSON_UTF8_VALUE})
	public ResponseEntity<ReplyPageDTO> getList(@PathVariable("page") int page, @PathVariable("bno") Long bno){
		Criteria cri = new Criteria(page, 10);
		
		log.info("get Reply List bno: " + bno);
		
		log.info("cri : " + cri);
		
		return new ResponseEntity<>(service.getListPage(cri, bno), HttpStatus.OK);
	}
	
	// 댓글 조회
	@GetMapping(value="/{rno}", produces= {MediaType.APPLICATION_ATOM_XML_VALUE, MediaType.APPLICATION_JSON_UTF8_VALUE})
	public ResponseEntity<ReplyVO> get(@PathVariable("rno") Long rno){
		log.info("get : " + rno);
		
		return new ResponseEntity<>(service.get(rno), HttpStatus.OK);
	}
	
	// 댓글 삭제
	@DeleteMapping(value="/{rno}", produces= {MediaType.TEXT_PLAIN_VALUE})
	public ResponseEntity<String> remove(@PathVariable("rno") Long rno){
		log.info("remove : " + rno);
		
		return service.remove(rno) == 1
				? new ResponseEntity<>("success", HttpStatus.OK)
				: new ResponseEntity<>(HttpStatus.INTERNAL_SERVER_ERROR);
	}
	
	// 댓글 수정
	@RequestMapping(method= {RequestMethod.PUT, RequestMethod.PATCH}, value="/{rno}", consumes="application/json", produces= {MediaType.TEXT_PLAIN_VALUE})
	public ResponseEntity<String> modify(@RequestBody ReplyVO vo, @PathVariable("rno") Long rno){
		vo.setRno(rno);
		
		log.info("rno : " + rno);
		log.info("modify : " + vo);
		
		return service.modify(vo) == 1
				? new ResponseEntity<>("success", HttpStatus.OK)
				: new ResponseEntity<>(HttpStatus.INTERNAL_SERVER_ERROR);
	}
}
```
</details>

-------

<details markdown="1">
<summary>@PathVariable</summary>

- ## @PathVariable
    - '?' 뒤에 파라미터를 추가하는 형식의 '쿼리 스트링' 방식 대신 사용
    - URL 상에 경로의 일부를 파라미터로 사용
    ```java
    // http://localhost:8080/controller/sample/product/bags/234
    // http://localhost:8080/controller/sample/product/bags/234.json
    @GetMapping(value="/product/{cat}/{pid}")
    public String[] getPath(@PathVariable("cat") String cat, @PathVariable("pid") int pid) {
        return new String[] {"category: " + cat, "productid: " + pid};
    }
    ```
</details>

-------

<details markdown="1">
<summary>@RequestBody</summary>

- ## @RequestBody
	- JSON 데이터를 원하는 타입의 객체로 변환해야 하는 경우에 주로 사용
	- 전달된 요청(request)의 내용(body)을 이용해서 해당 파라미터의 타입으로 변환을 요구함
    - 내부적으로는 HttpMessageConverter 타입의 객체들을 이용해서 다양한 포맷의 입력 데이터를 변환할 수 있음
    - 대부분의 경우에는 JSON 데이터를 서버에 보내서 원하는 타입의 객체로 변환하는 용도로 사용되지만, 경우에 따라서는 원하는 포맷의 데이터를 보내고, 이를 해석해서 원하는 타입으로 사용하기도 함
	> TicketVO.java
    ```java
    package org.zerock.domain;

    import lombok.Data;

    @Data
    public class TicketVO {
        private int tno;
        private String owner;
        private String grade;
    }
    ```
    > SampleController.java
    ```java
    @PostMapping("/ticket")
    public Ticket convert(@RequestBody TicketVO vo) {
        log.info("convert....ticket" + ticket);
        
        return ticket;
    }
    ```
	> REST Controller 응용
	```java
		@PostMapping(value="/new", consumes="application/json", produces= {MediaType.TEXT_PLAIN_VALUE})
		public ResponseEntity<String> create(@RequestBody ReplyVO vo){
			log.info("ReplyVO: " + vo);
			
			int insertCount = service.register(vo);
			
			log.info("Reply INSERT COUNT: " + insertCount);
			
			return insertCount == 1
					? new ResponseEntity<>("success", HttpStatus.OK)
					: new ResponseEntity<>(HttpStatus.INTERNAL_SERVER_ERROR);
					//삼항 연산자 처리
		}
	}
	```
</details>

----------

<details markdown="1">
<summary>@Produces</summary>

- ## @Produces
	- @Produces 어노테이션은 자원이 생성하여 클라이언트로 다시 보낼 수있는 MIME 미디어 유형 또는 표현을 지정하는 데 사용
	- @Produces가 클래스 수준에서 적용되면 리소스의 모든 메서드는 기본적으로 지정된 MIME 형식을 생성할 수 있음
	- 메서드 수준에서 적용되면 클래스 수준에서 적용된 모든 @Produces 어노테이션을 재정의
	- 자원의 메소드가 클라이언트 요청에서 MIME 유형을 생성 할 수없는 경우 Jersey 런타임은 HTTP "406 Not Acceptable"오류를 다시 보냄
```java
// 방법 1
@Produces({"application/xml", "application/json"})
public String doGetAsXmlOrJson() {
	...
}

// 방법 2
@GetMapping(value="/{rno}", produces= {MediaType.APPLICATION_ATOM_XML_VALUE, MediaType.APPLICATION_JSON_UTF8_VALUE})
	public ResponseEntity<ReplyVO> get(@PathVariable("rno") Long rno){
		log.info("get : " + rno);
		
		return new ResponseEntity<>(service.get(rno), HttpStatus.OK);
	}
```
</details>

---------

<details markdown="1">
<summary>@Consumes</summary>

- ## @Consumes
	- @Consumes 어노테이션은 리소스가 클라이언트에서 받아 들일 수 있거나 소비 할 수있는 표현의 MIME 미디어 유형을 지정하는 데 사용
	- @Consumes가 클래스 수준에서 적용되면 모든 응답 메서드는 기본적으로 지정된 MIME 형식을 허용함
	-  @Consumes가 메소드 레벨에서 적용되면, 클래스 레벨에서 적용된 @Consumes 어노테이션을 대체
	- 자원이 클라이언트 요청의 MIME 유형을 사용할 수 없으면 Jersey 런타임은 HTTP "415 지원되지 않는 매체 유형" 오류를 다시 보냄
```java
// 방법 1
@POST
@Consumes("text/plain")
public void postClichedMessage(String message) {
   // Store the message
}

// 방법2
@PostMapping(value="/new", consumes="application/json", produces= {MediaType.TEXT_PLAIN_VALUE})
public ResponseEntity<String> create(@RequestBody ReplyVO vo){
	log.info("ReplyVO: " + vo);
	
	int insertCount = service.register(vo);
	
	log.info("Reply INSERT COUNT: " + insertCount);
	
	return insertCount == 1
			? new ResponseEntity<>("success", HttpStatus.OK)
			: new ResponseEntity<>(HttpStatus.INTERNAL_SERVER_ERROR);
			//삼항 연산자 처리
}
```
</details>

--------

<details markdown="1">
<summary></summary>
</details>

-------------

<details markdown="1">
<summary></summary>
</details>

-------------

<details markdown="1">
<summary></summary>
</details>

-------------

<details markdown="1">
<summary></summary>
</details>

-------------

<details markdown="1">
<summary></summary>
</details>

-------------

<details markdown="1">
<summary></summary>
</details>

-------------

<details markdown="1">
<summary></summary>
</details>

-------------

<details markdown="1">
<summary></summary>
</details>

-------------