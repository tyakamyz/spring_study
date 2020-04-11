> ## Spring 에서 Was를 실행 하지 않고 Web URL을 테스트 하는 방법
 - 웹을 개발할 때 매번 URL을 테스트하기 위해 Tomcat과 같은 WAS를 실행하는 불편한 단계를 생략하기 위해서 스프링의 테스트 기능을 활용한다.
 ------------
 > 기본설정
 - 테스트 클래스의 선언부에는 @WebAppConfiguration 을 사용<br>
	(Servlet의 ServletContext를 이용하기 위해서인데 스프링에서는 WebApplicationContext라는 존재를 이용하기 위함)
 - @Before 가 적용된 setUp()에서는 import 할 때 JUnit을 이용<br>
 	해당 어노테이션이 적용된 메서드는 모든 테스트 전에 매번 실행되는 메서드가 됨
 - MockMvc는 말 그대로 **'가짜 mvc'** 로생각하면된다.  
 - 가짜로 URL과 파라미터 등을 브라우저에서 사용하는 것처럼 만들어서 Controller 실행 가능
	- MockMvcRequestBuilders라는 존재를 이용해서 GET 방식의 호출
	- BoardController의 getList()에서 반환된 결과를 이용해 Model에 어떠한 데이터들이 담겨 있는지 확인
 - Tomcat을 통해 실행되는 방식이아니라 기존의 JUnit Test로 실행
```java
import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
import org.springframework.test.context.web.WebAppConfiguration;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.request.MockMvcRequestBuilders;
import org.springframework.test.web.servlet.setup.MockMvcBuilders;
import org.springframework.web.context.WebApplicationContext;

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

> 등록 테스트
- MockMvcRequestBuilder의 post()를 이용하면 POST방식으로 전달 가능
- param()을 이용해서 전달해야 하는 파라미터들을 지정 가능 (Input 태그와 유사한 역할)
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

> 조회 테스트
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

> 수정 테스트
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

> 삭제 테스트
```java
@Test
	public void testRemove() throws Exception{
		
		String resultPage = mockMvc.perform(MockMvcRequestBuilders.post("/board/remove")
				.param("bno", "6"))
				.andReturn().getModelAndView().getViewName();
		
		log.info(resultPage);
	}
```