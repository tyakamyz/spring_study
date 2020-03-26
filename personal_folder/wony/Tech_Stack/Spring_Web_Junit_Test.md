- ## Spring 에서 Was를 실행 하지 않고 Web URL을 테스트 하는 방법

    - ### 프로젝트를 진행하는 멤버들의 경험치가 낮을 수록 테스트를 먼저 진행하는 습관을 가지는 것이 좋다!


    - 웹을 개발할 때 매번 URL을 테스트하기 위해 Tomcat과 같은 WAS를 실행하는 불편한 단계를 생략하기 위해서 스프링의 테스트 기능을 활용한다.



 ### **1. 기본설정**
 ```java
 @RunWith(SpringJUnit4ClassRunner.class)
//Test for Controller
@WebAppConfiguration
@ContextConfiguration({"file:src/main/webapp/WEB-INF/spring/appServlet/servlet-context.xml",
	"file:src/main/webapp/WEB-INF/spring/root-context.xml"}) // XML Version
//@ContextConfiguration(classes = {RootConfig.class, ServletConfig.class}) // Java Version
@Log4j
public class BoardControllerTests {

	@Setter(onMethod_ = @Autowired)
	private WebApplicationContext ctx;
	
	private MockMvc mockMvc;
	
	
	@Before
	public void setup() {
		this.mockMvc = MockMvcBuilders.webAppContextSetup(ctx).build();
	}
	
	@Test
	public void testList() throws Exception{
		log.info(
				mockMvc.perform(MockMvcRequestBuilders.get("/board/list"))
				.andReturn()
				.getModelAndView()
				.getModelMap()
			);
		
	}
}
```
>   - 테스트 클래스의 선언부에는 @WebAppConfiguration 을 사용한다. Servlet의 ServletContext를 이용하기 위해서인데 스프링에서는 WebApplicationContext라는 존재를 이용하기 위해서 이다.
>   - @Before 가 적용된 setUp()에서는 import 할 때 JUnit을 이용해야한다. 해당 어노테이션이 적용된 메서드는 모든 테스트 전에 매번 실행되는 메서드가 된다.
>   - MockMvc는 말 그대로 **'가짜 mvc'** 로생각하면된다.  
>   가짜로 URL과 파라미터 등을 브라우저에서 사용하는 것처럼 만들어서 Controller를 실행 해 볼 수 있다.
>   1. MockMvcRequestBuilders라는 존재를 이용해서 GET 방식의 호출을 한다.
>   2. BoardController의 getList()에서 반환된 결과를 이용해 Model에 어떠한 데이터들이 담겨 있는지 확인한다.
>   - Tomcat을 통해 실행되는 방식이아니라 기존의 JUnit Test로 실행하면된다.

## **2. 등록**
- 등록 테스트
```java
@Test
	public void testRegister() throws Exception{
		
		String resultPage = 
        mockMvc.perform(MockMvcRequestBuilders.post("/board/register")
				.param("title", "테스트 새글 제목")
				.param("content", "테스트 새글내용")
				.param("writer", "user00"))
				.andReturn().getModelAndView().getViewName();
		
		log.info(resultPage);
	}
```

>   - MockMvcRequestBuilder의 post()를 이용하면 POST방식으로 전달할 수 있다.
>   - param()을 이용해서 전달해야 하는 파라미터들을 지정할 수 있다.(Input 태그와 유사한 역할)

## **3. 조회**
- 조회 테스트
```java
@Test
	public void testGet() throws Exception{
		
		log.info(mockMvc.perform(MockMvcRequestBuilders
				.get("/board/get")
				.param("bno", "2"))
				.andReturn()
				.getModelAndView().getModelMap());
	}
````

## **4. 수정**
- 수정
```java
@PostMapping("/modify")
	public String modify(BoardVO board, RedirectAttributes rttr) {
		
		log.info("modify : " + board);
		
		if(service.modify(board)) {
			rttr.addFlashAttribute("result", "success");
		}
		
		return "redirect:/board/list";
	}
```

- 수정 테스트
```java
@Test
	public void testModify() throws Exception{
		
		String resultPaage = 
        mockMvc.perform(MockMvcRequestBuilders.post("/board/modify")
				.param("bno", "1")
				.param("title", "수정된 테스트 새글 제목")
				.param("content", "수정된 테스트 새글 내용")
				.param("writer", "user00"))
			.andReturn().getModelAndView().getViewName();
		
		log.info(resultPaage);
	}
```

## **5. 삭제**
- 삭제 테스트
```java
@Test
	public void testRemove() throws Exception{
		
		String resultPage = mockMvc.perform(MockMvcRequestBuilders.post("/board/remove")
				.param("bno", "6"))
				.andReturn().getModelAndView().getViewName();
		
		log.info(resultPage);
	}
```
