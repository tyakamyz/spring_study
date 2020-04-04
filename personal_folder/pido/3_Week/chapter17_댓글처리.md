
[Part4] - chapter 17
=========================

댓글 처리
-----------------

MyBatis 는 두개 이상의 데이터를 파라미터로 전달하기 위해,   
1. 별도의 객체로 구성
2. Map 이용 
3. @Param 이용  --> 가장 간단하다.   

```java
public List<ReplyVO> getListWithPaging(@Param("cri") Criteria cri, @Param("bno") Long bno);
```
```xml
<select id="getListWithPaging" resultType="org.zerock.domain.ReplyVO">
    select rno, bno, reply, replyer, replyDate, updatedate
    from tbl_reply
    where bno = #{bno}
    order by rno asc
</select>
```
java 의 @Param("bno")와 xml의 #{bno} 가 매칭

<br>   

**ReplyController의 설계**   
@RestController 어노테이션을 이용하여 URL을 기준으로 동작할 수 있게 한다.   
- PK 기준으로 작성하는 것이 좋다.   
- 목록은 PK 사용이 불가능하므로 파라미터로 게시물번호(bno)와 페이지번호(page)들을 URL에서 표현하는 방식을 사용한다.

|작업	 |전송방식       | URL
|-----------------------|-----------------------|--------
|등록    |POST          | /replies/new
|조회	 |GET           | /replies/:rno
|수정	 |PUT or PATCH  | /replies/:rno
|삭제	 |DELETE        | /replies/:rno
|페이지  |GET           | /replies/pages/:bno/:page   

1-1. 등록   
```java
@RequestMapping("/replies/")
@RestController
@Log4j
@AllArgsConstructor
public class ReplyController {
	
	private ReplyService service;
	
	// 처리 시 데이터의 포맷(consumes)과 서버에서 보내주는 데이터 타입(produces)을 명확히 설계해야한다.
	@PostMapping(value = "/new", consumes = "application/json", produces = { MediaType.TEXT_PLAIN_VALUE })
	public ResponseEntity<String> create(@RequestBody ReplyVO vo) {	// JSON데이터를 ReplyVO타입으로 변환하도록 지정
		
		log.info("ReplyVO : " + vo);
		int insertCount = service.register(vo);
		log.info("Reply Insert Count : " + insertCount);
		
		return insertCount == 1 ? 	new ResponseEntity<>("success", HttpStatus.OK) :
									new ResponseEntity<>(HttpStatus.INTERNAL_SERVER_ERROR);
	}
	
}
```
create는 내부적으로 ReplyServiceImpl를 호출하여 register()를 호출하고,
댓글이 추가된 숫자를 확인하여 브라우저에서 200(OK) 또는 500(Internal Server Error) 을 반환한다. 

<Boomerang 크롬 확장프로그램 예시>

![image](https://user-images.githubusercontent.com/22673024/78262201-328f6500-753b-11ea-9875-592b2cb672c3.png)

<테스트 성공 시 톰캣 log와 데이터베이스 결과>
![image](https://user-images.githubusercontent.com/22673024/78262374-71251f80-753b-11ea-92dd-1fc9322b139c.png)   

![image](https://user-images.githubusercontent.com/22673024/78262471-94e86580-753b-11ea-82ee-01a11a6d9416.png)

1-2. 특정 게시물의 댓글 목록 확인   
```java
@GetMapping(value="/pages/{bno}/{page}", produces = {	MediaType.APPLICATION_XML_VALUE, 
															MediaType.APPLICATION_JSON_UTF8_VALUE })
	public ResponseEntity<List<ReplyVO>> getList(@PathVariable("page") int page, @PathVariable("bno") Long bno) {
		log.info("getList...............................");
		
		Criteria cri = new Criteria(page, 10);
		log.info(cri);
		
		return new ResponseEntity<>(service.getList(cri, bno), HttpStatus.OK);
	}
```
@PathVariable 을 이용하여 게시물 번호를 파라미터로 직접 처리한다.   
XML 타입으로 좀전에 생성한 댓글 데이터를 확인 할 수 있다.   
(URL을 .json 로 조회하면 JSON타입으로 확인 가능)
![image](https://user-images.githubusercontent.com/22673024/78263221-a1b98900-753c-11ea-9afc-90521ef80379.png)

1-3. 댓글 삭제, 조회   
```java 
// 조회
@GetMapping(value="/{rno}", produces= {	MediaType.APPLICATION_XML_VALUE, 
											MediaType.APPLICATION_JSON_UTF8_VALUE })
public ResponseEntity<ReplyVO> get(@PathVariable("rno") Long rno){
    log.info("get: " + rno);
    
    return new ResponseEntity<>(service.get(rno), HttpStatus.OK); 
}

// 삭제
@DeleteMapping(value="/{rno}", produces = { MediaType.TEXT_PLAIN_VALUE })
public ResponseEntity<String> remove(@PathVariable("rno") Long rno){
    log.info("remove : " + rno);
    
    return service.remove(rno) == 1 ? 	new ResponseEntity<>("success", HttpStatus.OK) :
                                        new ResponseEntity<>(HttpStatus.INTERNAL_SERVER_ERROR);
            
}
```
![image](https://user-images.githubusercontent.com/22673024/78265936-12ae7000-7540-11ea-9904-43c1380a8460.png)

테스트 결과 DB에서 삭제된 것을 확인 할 수 있음.

1-4. 댓글 수정
```java
@RequestMapping(method = { RequestMethod.PUT, RequestMethod.PATCH }, 
					value="/{rno}",
					consumes = "application/json",
					produces = { MediaType.TEXT_PLAIN_VALUE })
public ResponseEntity<String> modify(@RequestBody ReplyVO vo, @PathVariable("rno") Long rno) {
    
    vo.setRno(rno);
    
    log.info("rno : " + rno);
    log.info("modify : " + vo);
    
    return service.modify(vo) == 1 ? 	new ResponseEntity<>("success", HttpStatus.OK) :
                                        new ResponseEntity<>(HttpStatus.INTERNAL_SERVER_ERROR);
}
```   
* PUT 방식이나 PATCH 방식을 이용한다.   
* JSON 형태로 전달되는 데이터와 파라미터로 전달되는 댓글 번호(rno)를 처리하기 때문에  @RequestBody 와 @PathVariable 어노테이션을 사용하여 직접 처리한다. 

![image](https://user-images.githubusercontent.com/22673024/78265350-46d56100-753f-11ea-9596-cc774cd79bfb.png)

<hr />

**javascript 모듈화**      
- 모듈화 패턴의 javascript : 관련있는 함수들을 하나의 모듈처럼 묶음으로 구성하는 것.   
(클로저를 이용하는 가장 대표적인 방법)   
- Java의 클래스처럼 Javascript 를 이용해서 메서드를 가지는 객체를 구성.
- Javascript의 즉시 실행함수와 '{}'를 이용해서 객체를 구성.
- 즉시실행함수는 함수의 시행 결과가 바깥쪽에 선언된 변수에 할당된다.
- $(document).ready.. 는 여러번 나와도 상관없음! 

< reply.js 완성본>
[reply.js](https://github.com/tyakamyz/spring_study/files/4431140/reply.js.txt)
