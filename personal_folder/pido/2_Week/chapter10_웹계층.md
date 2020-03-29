
[Part3] - chapter 10
=========================

프레젠테이션(웹) 계층
-----------------

Controller   
하나의 클래스 안에서 여러 메서드의 분기를 이용하는 구조


|Task		|Method		|parameter   |URL 이동
|-----------|-----------|------------|---------
|전체목록	|GET		|            | 
|등록		|POST		|모든항목    |이동(화면필요)
|조회		|GET		|bno         |
|삭제		|POST		|bno         |이동(화면필요)
|수정	    |POST		|모든항목    |이동(화면필요)

<hr />

Controller 작성
```java
@Controller
@Log4j
@RequestMapping("/board/*")
@AllArgsConstructor
public class BoardController {
	
	private BoardService service;
    ...
}    
```
* @Controller : 스프링의 빈으로 인식
* @RequestMapping : '/board'로 시작하는 모든 처리를 할수 있도록 지정
* servlet-context.xml 에 해당 controller에 대한 context:component-scan 설정 처리 
* BoardService에 대해 의존적이므로 **@AllArgsConstructor** 를 통해 생성자를 만들고 자동으로 주입 

1. 등록 
String을 리턴타입으로 지정하여 Redirect Attributes를 파라미터로 지정
    ```java
    @PostMapping("/register")
    public String register(BoardVO board, RedirectAttributes rttr) {
        service.register(board);
        // addFlashAttribute 는 일회성으로만 데이터를 전달하여 중복 전달을 방지한다. 
        rttr.addFlashAttribute("result", board.getBno());
        
        return "redirect:/board/list";
    }
    ```
    * @PostMapping : POST 방식 처리
    * redirect : responce.sendRedirect() 처리 

2. 조회 
    ```java
    @GetMapping({"/get"})
    public void get(@RequestParam("bno") Long bno, Model model) {
        model.addAttribute("board", service.get(bno));
    }
    ```    
    * @GetMapping : GET 방식 처리
    * 화면쪽으로 데이터 전달을 위해 Model 을 파라미터로 지정 하여 사용 

3. 수정
    ```java
    @PostMapping("/modify")
	public String modify(BoardVO board, RedirectAttributes rttr) {
		if(service.modify(board)) {
			rttr.addFlashAttribute("result", "success");
		}
		return "redirect:/board/list";
	}
    ```
    * 실제 작업은 POST 방식이기 때문에 @PostMapping 사용
    * service의 메서드의 리턴타입이 boolean 이므로 성공일때만 RedirectAttributes에 추가한다. 

4. 삭제
    ```java
    @PostMapping("/remove")
	public String remove(@RequestParam("bno") Long bno, RedirectAttributes rttr) {
		if(service.remove(bno)) {
			rttr.addFlashAttribute("result", "success");
		}
		return "redirect:/board/list";
	}
    ```    
    * 삭제는 반드시 POST 방식으로만 처리

<hr />

테스트
-----------
@WebAppConfiguration : Servlet의 ServletContext를 이용하기 위함.    
@Before : 모든 테스트 전에 매번 실행되는 메서드    
MockMvc : 가짜 mvc. Tomcat을 이용하지 않고 Controller를 호출할 수 있는 테스트용. (MockMvcRequestBuilders로 GET방식 호출을 한다.)

```java
@RunWith(SpringJUnit4ClassRunner.class)
//Test for Controller(ServletContext를 이용하기 위한 적용)
@WebAppConfiguration	

@ContextConfiguration({"file:src/main/webapp/WEB-INF/spring/appServlet/servlet-context.xml",
						"file:src/main/webapp/WEB-INF/spring/root-context.xml"})
@Log4j
public class BoardControllerTests {

	@Setter(onMethod_ = @Autowired)
	private WebApplicationContext ctx;
	private MockMvc mockMvc;
	
	@Before	// 모든 테스트 전에 매번 실행되는 메서드 
	public void setup() {
		// 가짜 MVC. 가짜 url과 파라미터 등을 생성 
		this.mockMvc = MockMvcBuilders.webAppContextSetup(ctx).build();
	}
	
	@Test
	public void testList() throws Exception {
		log.info(mockMvc.perform(MockMvcRequestBuilders.get("/board/list"))
				.andReturn()
				.getModelAndView()
				.getModelMap());
	}
    ...
}    
```

