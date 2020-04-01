# 4장. REST 방식과 Ajax를 이용하는 댓글 처리
> ## ch16. REST 방식으로 전환
- REST 데이터 처리를 위한 어노테이션
    |어노테이션|기능|
    |----|---|
    |@RestController|Controller가 REST 방식을 처리하기 위한 것임을 명시 함|
    |@ResponseBody|일반적인 JSP와 같은 뷰로 전달되는 게 아니라 데이터 자체를 전달하기 위한 용도|
    |@PathVariable|URL 경로에 있는 값을 파라미터로 추출하려고 할때 사용|
    |@CrossOrigin|Ajax의 크로스 도메인 문제를 해결해주는 어노테이션|
    |@RequestBody|JSON 데이터를 원하는 타입으로 바인딩 처리|
-------
- @RestController
    - JSP와 달리 순수한 데이터를 반환하는 형태이므로 다양한 포맷의 데이터 전송 가능
    - 주로 많이 사용 형태 : 일반문자열, JSON, XML 등
    - produces는 MIME TYPE을 의미
    >일반 문자열 반환
    - @Controller에서는 문자열 return값이 jsp 파일의 이름으로 처리되지만, @RestController에서는 순수한 문자열로 처리 됨
    - SampleController.java
    ```java
    package org.zerock.controller;

    import org.springframework.http.MediaType;
    import org.springframework.web.bind.annotation.GetMapping;
    import org.springframework.web.bind.annotation.RequestMapping;
    import org.springframework.web.bind.annotation.RestController;

    import lombok.extern.log4j.Log4j;

    @RestController
    @RequestMapping("/sample")
    @Log4j
    public class SampleController {
        
        @GetMapping(value = "/getText", produces = "text/plain; charset=UTF-8")
        public String getText() {
            log.info("MIME TYPE : " + MediaType.TEXT_PLAIN_VALUE);
            
            return "안녕하세요";
        }
        
    }
    ```
    >객체 반환
    - SampleVO.java
    ```java
    package org.zerock.domain;

    import lombok.AllArgsConstructor;
    import lombok.Data;
    import lombok.NoArgsConstructor;

    @Data
    @AllArgsConstructor
    @NoArgsConstructor
    public class SampleVO {
        private Integer mno;
        private String firstName;
        private String lastName;
    }
    ```
    - SampleController.java
        - /sample/getSample 로 호출하는 경우 json 타입의 데이터가 전달되는 것을 확인 할수 있음
        - /sample/getSample.json 으로 호출하는 경우 json 타입의 데이터가 전달되는 것을 확인 할수 있음
    ```java
    @GetMapping(value="/getSample", produces={MediaType.APPLICATION_JSON_UTF8_VALUE, MediaType.APPLICATION_XML_VALUE})
	public SampleVO getSample() {
		return new SampleVO(112, "스타", "로드");
	}
    ```
    - SampleController.java
        - produces 생략가능 (결과는 위와 같음)
    ```java
    @GetMapping(value="/getSample2")
	public SampleVO getSample2() {
		return new SampleVO(112, "스타2", "로드2");
	}
    ```
    >컬렉션 타입의 객체 반환
    - SampleContoller.java
    ```java
    @GetMapping(value="/getList")
	public List<SampleVO> getList(){
		return IntStream.range(1,10).mapToObj(i -> new SampleVO(i, i+"First", i+ " Last")).collect(Collectors.toList());
	}
	
	@GetMapping(value="/getMap")
	public Map<String, SampleVO> getMap(){
		Map<String,SampleVO> map = new HashMap<>();
		map.put("First", new SampleVO(111,"그루트", "주니어"));
		
		return map;
	}
    ```
    > ResponseEntity 타입
    - SampleController.java
        - REST 호출 방식의 경우 데이터 자체를 전송하는 방식으로 처리되기 때문에 데이터를 요청한 쪽에서 정상적인 데이터인지 비정산적인 데이터인지 구분할 수 있는 방법이 필요
        - 예제의 경우 height값이 150 미만인 값을 파라미터로 받게되면 502 에러를 같이 보냄(개발자도구 - 네트워크에서 확인 가능)
    ```java
    @GetMapping(value="/check", params= {"height", "weight"})
	public ResponseEntity<SampleVO> check(Double height, Double weight){
		SampleVO vo = new SampleVO(0, "" + height, "" + weight);
		
		ResponseEntity<SampleVO> result = null;
		
		if(height < 150) {
			result = ResponseEntity.status(HttpStatus.BAD_GATEWAY).body(vo);
		}else {
			result = ResponseEntity.status(HttpStatus.OK).body(vo);
		}
		
		return result;
	}
    ```
------------
- @RestController에서 사용하는 어노테이션
    - @RestController는 기존 @Controller에서 사용하던 일반적인 타입 또는 사용자가 정의한 타입(클래스)을 사용
    - 추가로 사용하는 어노테이션이 존재함
        - @PathVariable
            - 일반 컨트롤러에서도 사용이 가능하지만 REST 방식에서 자주 사용됨
            - URL 경로의 일부를 파라미터로 사용할 때 이용
        - @RequestBody
            - JSON 데이터를 원하는 타입의 객체로 변환해야 하는 경우에 주로 사용
    - @PathVariable
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
    - @RequestBody
        - 전달된 요청(request)의 내용(body)을 이용해서 해당 파라미터의 타입으로 변환을 요구함
        - 내부적으로는 HttpMessageConverter 타입의 객체들을 이용해서 다양한 포맷의 입력 데이터를 변환할 수 있음
        - 대부분의 경우에는 JSON 데이터를 서버에 보내서 원하는 타입의 객체로 변환하는 용도로 사용되지만, 경우에 따라서는 원하는 포맷의 데이터를 보내고, 이를 해석해서 원하는 타입으로 사용하기도 함
        - Ticket.java
        ```java
        package org.zerock.domain;

        import lombok.Data;

        @Data
        public class Ticket {
            private int tno;
            private String owner;
            private String grade;
        }
        ```
        - SampleController.java
        ```java
        @PostMapping("/ticket")
        public Ticket convert(@RequestBody Ticket ticket) {
            log.info("convert....ticket" + ticket);
            
            return ticket;
        }
        ```
---------
- REST 방식의 테스트
    - GET 방식이 아니고, POST 등의 방식으로 지정되어있으며, JSON 형태의 데이터를 처리하는 것을 브라우저에서 개발하려면 많은 시간과 노력이 들어감
    - 때문에 Junit과 spring-test를 활용하는 것을 권장
    ```java

    package org.zerock.controller;

    import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.post;
    import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

    import org.junit.Before;
    import org.junit.Test;
    import org.junit.runner.RunWith;
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.http.MediaType;
    import org.springframework.test.context.ContextConfiguration;
    import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
    import org.springframework.test.context.web.WebAppConfiguration;
    import org.springframework.test.web.servlet.MockMvc;
    import org.springframework.test.web.servlet.setup.MockMvcBuilders;
    import org.springframework.web.context.WebApplicationContext;
    import org.zerock.domain.Ticket;

    import com.google.gson.Gson;

    import lombok.Setter;
    import lombok.extern.log4j.Log4j;

    @RunWith(SpringJUnit4ClassRunner.class)
    //Test for Controller
    @WebAppConfiguration
    @ContextConfiguration({"file:src/main/webapp/WEB-INF/spring/appServlet/servlet-context.xml",
        "file:src/main/webapp/WEB-INF/spring/root-context.xml"}) // XML Version
    //@ContextConfiguration(classes = {RootConfig.class, ServletConfig.class}) // Java Version
    @Log4j
    public class SampleControllerTests {
        @Setter(onMethod_ = @Autowired)
        private WebApplicationContext ctx;
        
        private MockMvc mockMvc;
        
        @Before
        public void setup() {
            this.mockMvc = MockMvcBuilders.webAppContextSetup(ctx).build();
        }
        
        @Test
        public void testConvert() throws Exception{
            // TestConvert()는 SampleContropller에 작성해 둔 convert() 메서드를 테스트하기 위해서 작성
            // SmpleConteroller의 convert()는 JSON으로 전달되는 데이터를 받아서 Ticket 타입으로 변환함
            // 이를 위해 해당 데이터가 JSON이라는 것을 명시 해야함
            Ticket ticket = new Ticket();
            ticket.setTno(123);
            ticket.setOwner("Admin");
            ticket.setGrade("AAA");
            
            String jsonStr = new Gson().toJson(ticket);
            
            log.info(jsonStr);
            
            // mockMvc는 contentType()을 이용해서 전달하는 데이터가 무엇인지를 알려줄 수 있음
            // 코드 내의 Gson 라이브러리는 Java의 객체를 JSON 문자열로 변환하기 위해서 사용
            mockMvc.perform(post("/sample/ticket")
                    .contentType(MediaType.APPLICATION_JSON)
                    .content(jsonStr))
                    .andExpect(status().is(200));
        }
    }
    ```