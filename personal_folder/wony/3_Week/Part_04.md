# **Part 04** REST 방식과 Ajax를 이용하는 댓글 처리

## **Chapter 16** REST방식으로 전환

 - 흔히 URL(Uniform Resource Location)과 URL(Uniform Resource Identifier)를 같은 의미로 사용하는 경우가 많다. 엄밀하게는 URL은 URI의 하위 개념이기 때문에 혼용해도 무방하다.
 - URI 는 '자원의 식별자' 라는 의미로 사용된다.
 - URL은 '이곳에 가면 당신이 원하는 것을 찾을 수 있습니다.'와 같은 상징적인 의미가 좀더 강하다면 URI는 '당신이 원하는 곳의 주소는 여기입니다.'와 같이 좀더 현실적이고 구체적인 의미가 있다.
 - URI의 'I'는 마치 데이터베이스의 PK와 같은 의미로 사용된다.
 
 - REST는 'Representational State Transfer'의 약어로 하나의 URI는 하나의 고유한 리소스(Resource)를 대표하도록 설계된다는 개념에 전송방식을 결합해 원하는 작업을 지정한다.
    - 예로 '/board/123'은 게시물 중에서 123번이라는 고유한 의미를 가지도록 설계하고, 이에 대한 처리는 GET, POST 방식과 같이 추가적인 정보를 통해서 결정한다.
    - 따라서 REST 방식은 '**URI + GET/POST/PUT/DELETE/...**'로 구성된다.
 - REST 관련 어노테이션

 |어노테이션|기능|
 |-------|---|
 |@RestController|Controller가 REST방식을 처리하기 위한 것임을 명시한다|
 |@ResponseBody|일반적인 JSP와 같은 뷰로 전달되는게 아니라 데이터 자체를 전달하기 위한 용도|
 |@PathVariable|URL 경로에 있는 값을 파라미터로 추출하려고 할 때 사용|
 |@CrossOrigin| Ajax의 크로스 도메인 문제를 해결해주는 어노테이션|
 |@RequestBody|JSON 데이터를 원하는 타입으로 바인딩 처리|

 ### 16.1 @RestController
  - REST 방식에서 가장 먼저 기억해야 하는 점은 서버에서 전송하는 것이 순수한 데이터라는 점이다.
  - 스프링 4부터는 @Controller 외 @RestController라는 어노테이션을 추가해서 해당 Controller의 모든 메서드의 리턴타입을 기존과 다르게 처리한다는 것을 명시한다.
  - @RestController 이전에는 @Controller와 메서드 선언부에 @ResponseBody를 이용해서 동일한 결과를 만들 수 있다.
  - @RestController는 메서드의 리턴타입으로 사용자가 정의한 클래스 타입으르 사용할 수 있고, 이를 JSON이나 XML로 자동으로 처리할 수 있다.

  ```java
  @RestController
@RequestMapping("/sample")
@Log4j
public class SampleController {
	
	@GetMapping(value = "/getText", produces = "text/plain; charset=UTF-8")
	public String getText() {
		
		log.info("MIME TYPE: " + MediaType.TEXT_PLAIN_VALUE);
		
		return "안녕하세요";
	}

}
  ```
  - 기존의 @Controller는 문자열을 반환하는 경우에는 JSP 파일의 이름으로 처리하지만, @RestController의 경우에는 순수한 데이터가 된다.
  - @GetMapping에 사용된 **produces**속성은 해당 메서드가 생산하는 MIME 타입을 의미한다. 예제와 같이 문자열로 직접 지정할 수 있고, 메서드 내의 MediaType이라는 클래스를 이용할 수도 있다.
  - 별도의 페이지 생성 필요없이 순수한 텍스트 '안녕하세요'가 페이지에 나오는 것을 볼 수 있다.

  #### 16.1.1 객체의 반환
  - 객체를 반환하는 작업은 JSON이나 XML을 이용한다.

  ```java
  @Data
@AllArgsConstructor
@NoArgsConstructor
public class SampleVO {
	
	private Integer mno;
	private String firstName;
	private String lastName;

}
```
  
 - @AllArgsConstructor 모든속성을 사용하는 생성자 어노테이션
 - @NoArgsConstructor 비어있는 생성자를 만드는 어노테이션