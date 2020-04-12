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
<summary>MIME 타입 전체목록</summary>

- ## MIME 타입 전체목록
> MIME Type
<table class="standard-table">
 <thead>
  <tr>
   <th scope="col">확장자</th>
   <th scope="col">문서 종류</th>
   <th scope="col">MIME 타입</th>
  </tr>
 </thead>
 <tbody>
  <tr>
   <td><code>.aac</code></td>
   <td>AAC 오디오 파일</td>
   <td><code>audio/aac</code></td>
  </tr>
  <tr>
   <td><code>.abw</code></td>
   <td><a class="external" href="https://en.wikipedia.org/wiki/AbiWord" rel="noopener">AbiWord</a> 문서</td>
   <td><code>application/x-abiword</code></td>
  </tr>
  <tr>
   <td><code>.arc</code></td>
   <td>아카이브 문서 (인코딩된 다중 파일)</td>
   <td><code>application/octet-stream</code></td>
  </tr>
  <tr>
   <td><code>.avi</code></td>
   <td>AVI: Audio Video Interleave</td>
   <td><code>video/x-msvideo</code></td>
  </tr>
  <tr>
   <td><code>.azw</code></td>
   <td>아마존 킨들 전자책 포맷</td>
   <td><code>application/vnd.amazon.ebook</code></td>
  </tr>
  <tr>
   <td><code>.bin</code></td>
   <td>모든 종류의 이진 데이터</td>
   <td><code>application/octet-stream</code></td>
  </tr>
  <tr>
   <td><code>.bz</code></td>
   <td>BZip 아카이브</td>
   <td><code>application/x-bzip</code></td>
  </tr>
  <tr>
   <td><code>.bz2</code></td>
   <td>BZip2 아카이브</td>
   <td><code>application/x-bzip2</code></td>
  </tr>
  <tr>
   <td><code>.csh</code></td>
   <td>C-Shell 스크립트</td>
   <td><code>application/x-csh</code></td>
  </tr>
  <tr>
   <td><code>.css</code></td>
   <td>Cascading Style Sheets (CSS)</td>
   <td><code>text/css</code></td>
  </tr>
  <tr>
   <td><code>.csv</code></td>
   <td>Comma-separated values (CSV)</td>
   <td><code>text/csv</code></td>
  </tr>
  <tr>
   <td><code>.doc</code></td>
   <td>Microsoft Word</td>
   <td><code>application/msword</code></td>
  </tr>
  <tr>
   <td><code>.epub</code></td>
   <td>Electronic publication (EPUB)</td>
   <td><code>application/epub+zip</code></td>
  </tr>
  <tr>
   <td><code>.gif</code></td>
   <td>Graphics Interchange Format (GIF)</td>
   <td><code>image/gif</code></td>
  </tr>
  <tr>
   <td><code>.htm<br>
    .html</code></td>
   <td>HyperText Markup Language (HTML)</td>
   <td><code>text/html</code></td>
  </tr>
  <tr>
   <td><code>.ico</code></td>
   <td>Icon 포맷</td>
   <td><code>image/x-icon</code></td>
  </tr>
  <tr>
   <td><code>.ics</code></td>
   <td>iCalendar 포맷</td>
   <td><code>text/calendar</code></td>
  </tr>
  <tr>
   <td><code>.jar</code></td>
   <td>Java 아카이브 (JAR)</td>
   <td><code>application/java-archive</code></td>
  </tr>
  <tr>
   <td><code>.jpeg</code><br>
    <code>.jpg</code></td>
   <td>JPEG 이미지</td>
   <td><code>image/jpeg</code></td>
  </tr>
  <tr>
   <td><code>.js</code></td>
   <td>JavaScript (ECMAScript)</td>
   <td><code>application/js</code></td>
  </tr>
  <tr>
   <td><code>.json</code></td>
   <td>JSON 포맷</td>
   <td><code>application/json</code></td>
  </tr>
  <tr>
   <td><code>.mid</code><br>
    <code>.midi</code></td>
   <td>Musical Instrument Digital Interface (MIDI)</td>
   <td><code>audio/midi</code></td>
  </tr>
  <tr>
   <td><code>.mpeg</code></td>
   <td>MPEG 비디오</td>
   <td><code>video/mpeg</code></td>
  </tr>
  <tr>
   <td><code>.mpkg</code></td>
   <td>Apple Installer Package</td>
   <td><code>application/vnd.apple.installer+xml</code></td>
  </tr>
  <tr>
   <td><code>.odp</code></td>
   <td>OpenDocuemnt 프리젠테이션 문서</td>
   <td><code>application/vnd.oasis.opendocument.presentation</code></td>
  </tr>
  <tr>
   <td><code>.ods</code></td>
   <td>OpenDocuemnt 스프레드시트 문서</td>
   <td><code>application/vnd.oasis.opendocument.spreadsheet</code></td>
  </tr>
  <tr>
   <td><code>.odt</code></td>
   <td>OpenDocument 텍스트 문서</td>
   <td><code>application/vnd.oasis.opendocument.text</code></td>
  </tr>
  <tr>
   <td><code>.oga</code></td>
   <td>OGG 오디오</td>
   <td><code>audio/ogg</code></td>
  </tr>
  <tr>
   <td><code>.ogv</code></td>
   <td>OGG 비디오</td>
   <td><code>video/ogg</code></td>
  </tr>
  <tr>
   <td><code>.ogx</code></td>
   <td>OGG</td>
   <td><code>application/ogg</code></td>
  </tr>
  <tr>
   <td><code>.pdf</code></td>
   <td>Adobe <a class="external" href="https://acrobat.adobe.com/us/en/why-adobe/about-adobe-pdf.html" rel="noopener">Portable Document Format</a> (PDF)</td>
   <td><code>application/pdf</code></td>
  </tr>
  <tr>
   <td><code>.ppt</code></td>
   <td>Microsoft PowerPoint</td>
   <td><code>application/vnd.ms-powerpoint</code></td>
  </tr>
  <tr>
   <td><code>.rar</code></td>
   <td>RAR 아카이브</td>
   <td><code>application/x-rar-compressed</code></td>
  </tr>
  <tr>
   <td><code>.rtf</code></td>
   <td>Rich Text Format (RTF)</td>
   <td><code>application/rtf</code></td>
  </tr>
  <tr>
   <td><code>.sh</code></td>
   <td>Bourne 쉘 스크립트</td>
   <td><code>application/x-sh</code></td>
  </tr>
  <tr>
   <td><code>.svg</code></td>
   <td>Scalable Vector Graphics (SVG)</td>
   <td><code>image/svg+xml</code></td>
  </tr>
  <tr>
   <td><code>.swf</code></td>
   <td><a class="external" href="https://en.wikipedia.org/wiki/SWF" rel="noopener">Small web format</a> (SWF) 혹은 Adobe Flash document</td>
   <td><code>application/x-shockwave-flash</code></td>
  </tr>
  <tr>
   <td><code>.tar</code></td>
   <td>Tape Archive (TAR)</td>
   <td><code>application/x-tar</code></td>
  </tr>
  <tr>
   <td><code>.tif<br>
    .tiff</code></td>
   <td>Tagged Image File Format (TIFF)</td>
   <td><code>image/tiff</code></td>
  </tr>
  <tr>
   <td><code>.ttf</code></td>
   <td>TrueType Font</td>
   <td><code>application/x-font-ttf</code></td>
  </tr>
  <tr>
   <td><code>.vsd</code></td>
   <td>Microsft Visio</td>
   <td><code>application/vnd.visio</code></td>
  </tr>
  <tr>
   <td><code>.wav</code></td>
   <td>Waveform Audio Format</td>
   <td><code>audio/x-wav</code></td>
  </tr>
  <tr>
   <td><code>.weba</code></td>
   <td>WEBM 오디오</td>
   <td><code>audio/webm</code></td>
  </tr>
  <tr>
   <td><code>.webm</code></td>
   <td>WEBM 비디오</td>
   <td><code>video/webm</code></td>
  </tr>
  <tr>
   <td><code>.webp</code></td>
   <td>WEBP 이미지</td>
   <td><code>image/webp</code></td>
  </tr>
  <tr>
   <td><code>.woff</code></td>
   <td>Web Open Font Format (WOFF)</td>
   <td><code>application/x-font-woff</code></td>
  </tr>
  <tr>
   <td><code>.xhtml</code></td>
   <td>XHTML</td>
   <td><code>application/xhtml+xml</code></td>
  </tr>
  <tr>
   <td><code>.xls</code></td>
   <td>Microsoft Excel</td>
   <td><code>application/vnd.ms-excel</code></td>
  </tr>
  <tr>
   <td><code>.xml</code></td>
   <td><code>XML</code></td>
   <td><code>application/xml</code></td>
  </tr>
  <tr>
   <td><code>.xul</code></td>
   <td>XUL</td>
   <td><code>application/vnd.mozilla.xul+xml</code></td>
  </tr>
  <tr>
   <td><code>.zip</code></td>
   <td>ZIP archive</td>
   <td><code>application/zip</code></td>
  </tr>
  <tr>
   <td><code>.3gp</code></td>
   <td><a class="external" href="https://en.wikipedia.org/wiki/3GP_and_3G2" rel="noopener">3GPP</a> 오디오/비디오 컨테이너</td>
   <td><code>video/3gpp</code><br>
    <code>audio/3gpp</code> if it doesn't contain video</td>
  </tr>
  <tr>
   <td><code>.3g2</code></td>
   <td><a class="external" href="https://en.wikipedia.org/wiki/3GP_and_3G2" rel="noopener">3GPP2</a> 오디오/비디오 컨테이너</td>
   <td><code>video/3gpp2</code><br>
    <code>audio/3gpp2</code> if it doesn't contain video</td>
  </tr>
  <tr>
   <td><code>.7z</code></td>
   <td><a class="external" href="https://en.wikipedia.org/wiki/7-Zip" rel="noopener">7-zip</a> 아카이브</td>
   <td><code>application/x-7z-compressed</code></td>
  </tr>
 </tbody>
</table>

> 출처
- https://developer.mozilla.org/ko/docs/Web/HTTP/Basics_of_HTTP/MIME_types/Common_types
</details>