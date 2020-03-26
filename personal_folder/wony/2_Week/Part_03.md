# **Part 03** 기본적인 웹 게시물 관리

## **Chapter 07** 스프링 MVC 프로젝트의 기본 구성

### 1. 일반적으로 웹 프로젝트는 3-Tier(티어) 방식으로 구성
    - Presentation Tier(화면 계층)
    - Business Tier(비즈니스 계층)
    - Persistence Tier(영속 계층 혹은 데이터 계층)

- Presentation Tier(화면계층)
    - 화면에 보여주는 기술을 사용하는 영역. Servlet/JSP나 스프링 MVC가 담당하는 영역
    - 프로젝트 성격에 맞춰 앱이나 CS(Client-Server)로 구성되는 경우가 있다.
    - 이전 파트에서 학습한 스프링 MVC와 JSP를 이용한 화면구성이 이에 속한다.
    
- Business Tier(비즈니스 계층)
    - 순수한 비즈니스 로직을 담고있는 영역
    - 고객이 원하는 요구 사항을 반영하는 계층으로 **중요한 영역** 이다
    - 주로 'xxxService'와 같은 이름으로 구성, 메스드의 이름 역시 고객들이 사용하는 용어를 그대로 사용
- Persistence Tier(영속 계층 혹은 데이터 계층)
    - 데이터를 어떤 방식으로 보관하고, 사용하는가에 대한 설계가 들어가는 계층
    - 일반적인 경우 데이터베이스를 많이 사용하지만, 경우에 따라 네트워크 호출이나 원격 호출등의 기술이 접목
    - 해당 영역은 MyBatis와 mybatis-spring을 이용해서 구성했던 파트 1을 이용

### 2. 각 영역의 Naming Convention(명명 규칙)

    - 프로젝트를 위의 3-Tier 방식으로 구성하는 가장 일반적인 설명은 '유지보수' 에 대한 필요성 떄문이다.
    - 따라서 각 영역은 설계당시부터 영역을 구분하고, 해당 연결 부위는 인터페이스를 이용해서 설계하는 것이 일반적인 구성방식이다.

|네이밍|설명
|----|--
|xxxController|스프링 MVC 에서 동작하는 Controller 클래스를 설계할 때 사용
|xxxService, xxxServiceimpl| 비즈니스 영역을 담당하는 인터페이스는 'xxxServie'라는 방식을 사용하고, 인터페이스를 구현한 클래스는 'xxxServiceimpl'이라는 이름을 사용
|xxxDAO,xxxRepository|DAO(Data-Access-Object)나 Repository(저장소)라는 이름으로 영역을 따로 구성하는 것이 보편적이다. 다만 이 책에서는 별도의 DAO를 구성하는 대신 MyBatis의 Mapper 인터페이스를 활용한다.
|VO, DTO| - VO와 DTO는 일반적으로 유사한 의미로 사용되는 용어로 데이터를 담고 있는 객체를 의미한다는 공통점이 있다. </br> - VO는 Read Only의 목적이 강하고, 데이터 자체도 Immutable(불변)하게 설계하는 것이 정석이다 </br> - DTO는 주로 데이터 수집의 용도가 좀더 강하다. 예로 웹하면에서 로그인하는 정보를 DTO로 처리하는 방식을 사용한다.

#### 2.1. 패키지의 Naming Convention
    - 패키지의 구성은 프로젝트의 크기나 구성원들의 성향으로 결정한다.
    - 규모가 작은 프로젝트는 Controller 영역을 별도의 패키지로 설계하고, Service 영역 등을 하나의 패키지로 설계할 수 있다.
    - 규모가 큰 프로젝트는 Service 클래스와 Controller들이 혼재할수 있다면 비즈니스를 단위별로 구성하고(비즈니스 단위 별로 패키지 작성) 다시 내부에서 Controller 패키지, Service 패키지 등으로 다시 나누는 방식을 이용한다.

### 3. 화면 설계

    - 화면설계 시 주로 Mock-up(목업) 툴을 이용하는 경우가 많다.
    - 대표적으로 PowerPoint나 Balsamiq studio, Pencil Mockup 등의 SW를 이용해서 작성한다.

### 4. 테이블 생성
```
regdate date default sysdate,
updatedate date default sysdate
``` 
 - default sysdate를 해줌으로써 기본값으로 현재 시간이 들어갈 수 있도록 만들어준다.

 ## **Chapter 08** 영속/비즈니스 계층의 CRUD 구현
- 영속 계층의 작업은 항상 다음과 같은 순서로 진행
    - 테이블의 칼럼 구조를 반영하는 VO(Value Object)클래스의 생성
    - MyBatis의 Mapper 인터페이스의 작성/XML 처리
    - 작성한 Mapper 인터페이스의 테스트

### 1. Mapper Interface
```java
public interface BoardMapper {
	
	@Select("select * from tbl_board where bno > 0")
	public List<BoardVO> getList();

}
```
- **'where bno > 0'** 와 같은 조건은 테이블을 검색하는데 bno라는 컬럼 조건을 주어서 Primary Key를 이용하도록 유도하는 조건이다.

### 2. 영속 영역의 CRUD 구현
- MyBatis는 내부적으로 JDBC의 PreparedStatment를 활용하고 필요한 파라미터를 처리하는 '?' 에 대한 치환은 '#{속성}'을 이용해서 처리한다.

#### 2.1 create(insert) 처리
```sql
<insert id="insert">
		insert into tbl_board(bno, title, content, writer)
		values (seq_board.nextval, #{title}, #{content}, #{writer})
	</insert>
	
	<insert id="insertSelectKey">
		<selectKey keyProperty="bno" order="BEFORE" resultType="long">
			select seq_board.nextval from dual
		</selectKey>
		
		insert into tbl_board (bno, title, content, writer)
		values (${bno}, ${title}, ${content}, ${writer})
	</insert>
```
 - **insertSelectKey** 는 @SelectKey라는 MyBatis의 어노테이션을 이용한다.
 - @SelectKey는 주로 PK값을 미리(Before) SQL을 통해서 처리해 두고 특정한 이름으로 결과를 보관하는 방식이다.
 - @Insert 할때 SQL문을 보면 #{bno}와 같이 이미 처리된 결과를 이용하는 것을 볼 수있다.

 ```java
 @Test
	public void testInsert() {
		
		BoardVO boardVO = new BoardVO();
		boardVO.setTitle("새로 작성하는 글");
		boardVO.setContent("새로 작성하는 내용");
		boardVO.setWriter("newbie");
		
		mapper.insert(boardVO);
		
		log.info(boardVO); 
        //lombok 이 만들어주는 toString()을 이용해서 bno 멤버 변수(인스턴스 변수)의 값을 알아보기 위함이다.
		
	}
```

 - @SelectKey를 이용하는 방식은 SQL을 한 번 더 실행하는 부담이 있기는 하지만 자동으로 추가되는 PK값을 확인해야 하는 상황에서는 유용하게 사용될 수 있다.
 #### 2.2 read(select) 처리
 
 ```sql
<select id="read" resultType="org.zerock.domain.BoardVO">
    select * from tbl_board where bno = ${bno}
</select>
```
 - MyBatis는 bno라는 칼럼이 존재하면 인스턴스의 'setBno()'를 호출 하게된다.
 - MyBatis의 모든 파라미터와 리턴 타입의 처리는 get 파라미터명(), set 칼럼명()의 규칙으로 호출 된다.
 - **다만 위에같이 #{속성}이 1개만 존재하는 경우 별도의 get 파라미터명()을 사용하지 않고 처리한다.**
 ```java
 @Test
	public void testRead() {
		
		BoardVO board = mapper.read(5L);
		
		log.info(board);
	}
```
- mapper.read(5L) long형을 파라마티로 사용하는데 숫자뒤에 '**L**'을 사용하므로써 Long형임을 표시해서 알려준다.

```sql
select * from tbl_board where bno = #{bno}
```
```sql
select * from tbl_board where bno = ${value}
```

 - 예제 작성 중 매개변수 입력 형식을 **${bno}** 로 잘못 작성해서 JUnit 테스트틀 진행하였더니 오류가났었다. 오류사항을 확인도중 입력 받는형식이 **$**, **#** 두가지 모두 가능한것을 확인했다.  
 
 ||#{속성}|${속성}|
 |------|------|------|
 |Long(단일)|#{속성}, ${value}|${value}
 |Long,String 등(복수)|#{속성}| String일 경우 '${속성}' ,number타입은 ${속성}

 #### 2.3 delete 처리
  - 등록, 삭제, 수정과 같은 DML 작업은 '몇 건의 데이터가 삭제(혹은 수정)되었는지'를 반환할 수 있다.

## **Chapter 09** 비즈니스 계층

 - 고객의 요구사항을 반영하는 계층으로 프레젠테이션 계층과 영속 계층의 중간다리 역할
 - 영속 계층은 데이터베이스를 기준으로 해서 설계를 나눠 구현하지만, **비즈니스 계층** 은 로직을 기준으로 해서 처리한다.
 - 설계 시 계층 간의 연결은 인터페이스를 이용해서 느슨한(loose) 연결(결합)을 한다.  
(Service(Interface) + ServiceImpl(Class))


```java
@Service
@AllArgsConstructor
//@Log4j
public class BoardServiceImpl implements BoardService{
	
	//spring 4.3 이상에서 자동 처리 (미해결)
//	private BoardMapper mapper;
	
	@Setter(onMethod_ = @Autowired)
	private BoardMapper mapper;
```
    -  ServiceImpl 작성 중 **@Setter** 방식이 아닌 **@@AllArgsConstructor** 어노테이션 방식으로 진행도중 에러사항 발생(미해결)으로 인해 Setter로 사용해서 진행헀다. 확인 필요!

 - 확인결과
 ```xml
 <dependency>
    <groupId>log4j</groupId>
    <artifactId>log4j</artifactId>
    <version>1.2.15</version>
```
 ```xml
 <dependency>
    <groupId>log4j</groupId>
    <artifactId>log4j</artifactId>
    <version>1.2.17</version>
</dependency>
```
 > 기존의 log4j default version인 1.2.15로 진행 했을 때 위의 문제사항이 발생함을 확인
 > 이후의 **1.2.17** version으로 업그레이드 후 정상 작동됨을 확인

 ### 9.1 스프링의 서비스 객체 설정(root-context.xml)

  - root-context.xml 의 namespace에 context를 추가하면 해당 이름으로 시작하는 태그들을 활용 할 수 있다.
  ```xml
  <context:component-scan base-package="org.zerock.service"></context:component-scan>
  ```
  - 그후 위의 내용을 추가해서 @Service 어노테이션이 있는 **org.zerock.service** 패키지를 스프링의 빈으로 스캔하도록 추가해야한다.

  ### 9.2 삭제/수정

   - 정상적으로 삭제, 수정이 이루어지면 1이라는 값이 반환된다.


## **Chapter 10** 프레젠테이션(웹) 계층의 CRUD 구현

 - BoardController의 분석

 |Task|URL|Method|Parameter|From|URL 이동|
 |----|---|------|---------|----|-------|
 |전체등록|/board/list|GET
 |등록처리|/board/register/POST|모든항목|입력화면 필요|이동
 |조회|/board/read|GET|bno=123|
 |삭제 처리|/board/remove|POST|bno|입력화면 필요|이동
 |수정 처리|/board/modify|POST|모든 항목|입력화면 필요 | 이동

 ### 10.1 BoardController
 ```java
@Controller
@Log4j
@RequestMapping("/board/*")
public class BoardController {

}
```
 - @Controller 어노테이션을 추가해서 스프링의 빈으로 인실할 수 있게한다.
 - @ReuquestMapping을 통해 '/board'로 시작하는 모든처리를 BoardController가 하도록 지정

 ### 10.2 **URL 테스트 방법**

 - 웹을 개발할 때 매번 URL을 테스트하기 위해 Tomcat과 같은 WAS를 실행하는 불편한 단계를 생략하기 위해서 스프링의 테스트 기능을 활용한다.
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

### 10.3 등록 처리와 테스트

- 등록
```java
	@PostMapping("/register")
	public String register(BoardVO board, RedirectAttributes rttr) {
		log.info("register : " + board);
		
		service.register(board);
		
		rttr.addFlashAttribute("result", board.getBno());
		
		return "redirect:/board/list";
	}
```
>   - 등록 작업이 끝난 후 다시 목록 화면으로 이동하기 위해 RedirectAttributes를 파라미터를 사용한다.
>   - 리턴시 'redirect:' 접두어를 사용하면 스프링 MVC가 내부적으로 **response.sendRedirect()** 를 처리해준다

- 등록 테스트
```java
@Test
	public void testRegister() throws Exception{
		
		String resultPage = mockMvc.perform(MockMvcRequestBuilders.post("/board/register")
				.param("title", "테스트 새글 제목")
				.param("content", "테스트 새글내용")
				.param("writer", "user00"))
				.andReturn().getModelAndView().getViewName();
		
		log.info(resultPage);
	}
```

>   - MockMvcRequestBuilder의 post()를 이용하면 POST방식으로 전달할 수 있다.
>   - param()을 이용해서 전달해야 하는 파라미터들을 지정할 수 있다.(Input 태그와 유사한 역할)

### 10.4 조회

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
```

## **Chapter 11** 화면 처리

 - 화면을 개발하기 전에는 반드시 화면의 전체 레이아웃이나 디자인이 반영된 상태에서 개발하는 것을 추천한다.