
[Part4] - chapter 16
=========================


REST 방식
-----------------
### REST(Representational State Transfer)   

'하나의 URI는 하나의 고유한 리소스를 대표하도록 설계된다.' + '전송방식'을 결합 → 원하는 작업을 지정     
> REST방식 : URI + GET/POST/PUT/DELETE ...   
    
- URI > URL    
- URI : '자원의 식별자'. 'I'의 의미는 마치 DB의 PK와 같은 의미. '당신이 원하는 곳의 주소는 여기입니다.'
- URL : '이곳에 가면 당신이 원하는 것을 찾을 수 있습니다.'

|어노테이션			|기능
|-------------------|-----------------------|--------
|@RestController    |Controller가 REST방식을 처리하기 위한 것임을 명시
|@ResponseBody		|뷰로 전달되는것이 아닌 데이터 자체를 전달하기 위한 용도
|@PathVariable		|URL 경로에 있는 값을 파라미터로 추출하려고 할 때 사용
|@CrossOrign		|Ajax의 크로스 도메인 문제를 해결
|@RequestBody   	|JSON 데이터를 원하는 타입으로 바인딩 처리

<hr>

1. @RestController   
* 순수한 데이터를 반환하는 형태   
* Controller에서 모든 메서드의 리턴 타입을 기존과 다르게 처리한다는 것을 명시.   
* 메서드의 리턴 타입으로 사용자가 정의한 클래스 타입 사용 가능.    
* 이를 JSON이나 XML 로 자동 처리 가능.   
<br>
    1-1. pom.xml 설정 추가   
    
    ```xml
    <!-- 브라우저에 객체를 JSON 포맷으로 변환시켜 전송 -->
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-databind</artifactId>
        <version>2.9.6</version>
    </dependency>

    <!-- Java인스턴스를 JSON타입 문자열로 변환 -->
    <dependency>
        <groupId>com.google.code.gson</groupId>
        <artifactId>gson</artifactId>
        <version>2.8.2</version>
    </dependency>
    
    <!-- XML 포맷 처리 -->
    <dependency>
        <groupId>com.fasterxml.jackson.dataformat</groupId>
        <artifactId>jackson-dataformat-xml</artifactId>
        <version>2.9.6</version>
    </dependency>
    ```
    <br>
    
    1-2. 단순 문자열 변환
    ```java
    @RestController
    @RequestMapping("/sample")
    @Log4j
    public class SampleController {

        @GetMapping(value="/getText", produces="text/plain; charset=UTF-8")
        public String getText() {
            log.info("MIME TYPE: " + MediaType.TEXT_PLAIN_VALUE);
            
            return "안녕하세요";
        }
        
    }
    ```
    - @GetMapping의 produce속성 : 해당 메서드가 생산하는 MIME 타입
    - MediaType 클래스로 MIME 타입 명시 가능
    - 문자열 return 시 기존의 @Controller처럼 JSP파일 이름이 아닌 순수한 데이터 return 

    → 결과 확인 시, 텍스트 형태의 결과를 확인 할 수 있음.
    ```
    // 크롬 개발자모드에서 실제 데이터 확인 가능
    ▼ Response Headers 
        Connection: keep-alive
        Content-Length: 15
        Content-Type: text/plain;charset=UTF-8
    ```
    <br>
    1-3. 객체 반환(XML, JSON)   

    VO
    ```java
    @Data
    @AllArgsConstructor	// 모든 속성을 사용하는 생성자를 위한 어노테이션
    @NoArgsConstructor	// 비어있는 생성자 생성
    public class SampleVO {
        
        private Integer mno;
        private String firstName;
        private String lastName;
        
    }
    ```

    Controller
    ```java
    @GetMapping(value="/getSample", produces= { MediaType.APPLICATION_JSON_UTF8_VALUE,
												MediaType.APPLICATION_XML_VALUE })
	public SampleVO getSamplet() {
		
		return new SampleVO(112,"스타","로드");
	}
    ```
    → localhost:8080/sample/getText 결과 확인 시, XML 구조의 데이터를 확인할 수 있음.

    ```
    // 크롬 개발자모드에서 실제 데이터 확인 가능
    ▼ Response Headers 
        Connection: keep-alive
        Content-Type: application/xml;charset=UTF-8
    ```

    → localhost:8080/sample/getText.json 결과 확인 시, JSON 타입의 데이터가 전달되는 것을 확인 할 수 있음. 

    *이때, @GetMapping의 produce속성은 생략 가능*

    <br>
    1-4. 객체 반환(컬렉션)   

    열, 리스트 타입의 객체 전송 
    ```java
	@GetMapping(value="/getList")
	public List<SampleVO> getList(){
		return IntStream.range(1,10).mapToObj(i -> new SampleVO(i, i + "First", i + " Last")).collect(Collectors.toList());
	}
    ```
    - 1~9 루프를 처리하면서 SampleVO 객체를 만들고 List<SampleVO>를 만들어낸다.
    - localhost:8080/sample/getList 결과 확인 시, XML 구조의 데이터를, localhost:8080/sample/getList.json 결과 확인 시, JSON 구조의 '[]'형태의 배열 0데이터를 확인할 수 있음.

    > ```
    > ★ JAVA8의 IntStream
    >
    > public void for_loop() {
    >    for (int i = 1 ; i <= 10 ; i++) {
    >        System.out.println(i);
    >    }
    >}
    >
    >↓
    >
    > public void intStream_range() {
    >    IntStream.range(1, 11).forEach(System.out::println);
    >}
    >// 1~10까지의 int 스트림 생성 


    맵 타입의 객체 전송
    ```java
    @GetMapping(value="/getMap")
	public Map<String, SampleVO> getMap(){
		
		Map<String, SampleVO> map = new HashMap<>();
		map.put("First", new SampleVO(111, "그루트", "주니어"));
		
		return map;
	}
    ```
    - '키'와 '값'을 가지는 하나의 객체 
    - '키' 속성 데이터는 태그명이 되기 때문에 문자열로 지정 

    ```
    1) xml
    <Map>
        <First>
            <mno>111</mno>
            <firstName>그루트</firstName>
            <lastName>주니어</lastName>
        </First>
    </Map>

    2) json
    {"First":{"mno":111,"firstName":"그루트","lastName":"주니어"}}
    ```

    <br>
    1-5. ResponseEntity 타입 반환

    데이터가 정상적인지 비정상적인지 구분.   
    데이터와 함께 HTTP 헤더의 상태메시지 전달.

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
    - 해당 조건에 따라 Status (502 bad gateway, 200 ok0 등) 가 데이터와 함께 전달된다. 

    <br>
    1-5. @RestController 파라미터   

    사용자가 정의한 타입(클래스)

    * **@PathVariable (REST 방식에서 자주 사용)**   
    URL 경로의 일부를 파라미터로 사용할 때 이용
        ```http://localhost:8080/sample/{sno}/page/{pno}```

        ```java
        @GetMapping(value="/product/{cat}/{pid}")
        public String[] getPath(@PathVariable("cat") String cat, @PathVariable("pid") Integer pid) {
            return new String[] { "category : " + cat, "productid : " + pid };
        }
        ```
        '{ }' 으로 변수명 지정, @PathVariable 을 이용하여 지정 이름의 변수값을 얻을 수 있다.    
        *값을 얻을 때에는 기본 자료형 불가 (int, double 등)*

        ```
        http://localhost:8080/sample/product/bags/1234 호출 시, 

        <Strings>
            <item>category : bags</item>
            <item>productid : 1234</item>
        </Strings>

        경로 값이 변수 값으로 처리됨!
        ```
    * **@RequestBody**   
        > - 전달된 요청의 내용을 이용해서 해당 파라미터의 타입으로 변환을 요구.   
        > - 보통 JSON데이터를 서버에 보내서 원하는 타입의 객체로 변환하는 용도.   
        > - 원하는 포맷의 데이터를 보내고, 이를 해석해 원하는 타입으로 사용하기도 함.
        
        ```java
        // @RequestBody가 요청한 내용(Body)을 처리하기 때문에 일반적인 파라미터 전달방식을 사용할 수 없어 Post방식으로 지정.
        @PostMapping("/ticket")
        public Ticket convert(@RequestBody Ticket ticket) {
            log.info("convert............ ticket " + ticket);
            return ticket;
        }
        ```
        *REST 방식의 테스트*   
        0-1. JUnit (Tomcat 구동 없이도 가능.)
        ```java
        @RunWith(SpringJUnit4ClassRunner.class)
        @WebAppConfiguration	// Test for Controller
        @ContextConfiguration({	"file:src/main/webapp/WEB-INF/spring/root-context.xml",
                                "file:src/main/webapp/WEB-INF/spring/appServlet/servlet-context.xml"})
        //@ContextConfiguration(classes = { org.zerock.config.RootConfig.class, org.zerock.config.ServletConfig.class})	// JAVA Config
        @Log4j
        public class SampleControllerTests {

            @Setter(onMethod_ = {@Autowired})
            private WebApplicationContext ctx;
            
            private MockMvc mockMvc;
            
            @Before
            public void setup() {
                this.mockMvc = MockMvcBuilders.webAppContextSetup(ctx).build();
            }
            
            /**
            * JSON 전달 데이터를 Ticket 타입으로 변환
            * @throws Exception
            */
            @Test
            public void testConvert() throws Exception {
                
                Ticket ticket = new Ticket();
                ticket.setTno(123);
                ticket.setOwner("Admin");
                ticket.setGrade("AAA");
                
                // Gson : Java 객체를 JSON 문자열로 변환하기 위해 사용 
                String jsonStr = new Gson().toJson(ticket);
                
                log.info(jsonStr);
                
                mockMvc.perform(post("/sample/ticket")
                            .contentType(MediaType.APPLICATION_JSON)
                            .content(jsonStr))
                            .andExpect(status().is(200));
            }
            
        }
        ```

        0-2. Chrome 브라우저 앱스토어 (chrome://apps/)
        
        'Boomerang - SOAP & REST Client' 확장프로그램 사용 

       ![1](https://user-images.githubusercontent.com/22673024/78144043-0e178800-746a-11ea-923c-6e2a574725c4.PNG)

    1-6. 전송방식   
    HTTP 방식은 GET/POST 등의 전송방식이라면,    
    REST 방식은 URI와 결합하므로 전송방식을 결합하면 아래와 같다.

    |작업	 |전송방식  | URI
    |------------------ |-----------------------|--------
    |등록    |POST      |/members/new
    |조회	 |GET       |/members/{id}
    |수정	 |PUT       |/members/{id} + body(json데이터 등)
    |삭제	 |DELETE    |/members/{id}
    



    
