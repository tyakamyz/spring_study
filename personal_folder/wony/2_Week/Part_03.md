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

 ```xml
 <beans:bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
		<beans:property name="prefix" value="/WEB-INF/views/" />
		<beans:property name="suffix" value=".jsp" />
	</beans:bean>
 ```
  - 스프링 MVC의 설정에서 화면 설정은 ViewResolver라는 객체를 통해서 이루어진다.
  - 위의 servlet-context.xml의 설정을 보면 '/WEB-INF/views'폴더를 이용하는 것을 볼 수 있다.
  - '/WEB-INF' 경로는 브라우저에서 직접 접근할 수 업슨 경로이므로 반드시 Controller를 이용하는 모델 2방식에서는 기본적으로 사용하는 방식이다.

  ### 11.1 resources

  ```xml
  <resources mapping="/resources/**" location="/resources/" />
  ```   
  - servlet-context.xml의 일부로 CSS나 JS 파일과 같이 정적인(static) 자원들의 경로를 'resources'라는 경로로 지정하고있다.

  ### 11.2 JSTL fmt(형식화)

  ```js
  <td><fmt:formatDate value="${board.updateDate }" pattern="yyyy-mm-dd"/></td>
  ```
   - fmt(형식화)에서 formatDate태그로 yyyy/mm/dd 날짜포맷을 yyyy-mm-dd로 포맷 변환을 해준다.

 ### 11.3 한글 문제와 UTF-8 필터 처리
- 브라우저에서 전송되는 데이터는 개발자도구의 **NetWork** 탭의 Form Data를 통해서 확인할 수 있다.
이 때, Post방식으로 제대로 전송되었는지, 한글이 깨진 상태로 전송된 것인지를 확인할 수 있다.
- 브라우저에서의 전송된 데이터가 한글 데이터로 정상으로 보내졌을 경우 Lombok의 로그를 확인해보면 Controller에 전달 될 때 이미 한글이 깨진 상태로 처리되있을 확률이 대부분이다.
- 해결방안으로는 아래의 소스를 추가한다.
>   - xml Version
```xml
<filter>
		<filter-name>encoding</filter-name>
		<filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
		<init-param>
			<param-name>encoding</param-name>
			<param-value>UTF-8</param-value>
		</init-param>
	</filter>
	
	<filter-mapping>
		<filter-name>encoding</filter-name>
		<servlet-name>appServlet</servlet-name>
	</filter-mapping>
```
>   - Java Version (getServletFilters @Override)
```java
@Override
	protected Filter[] getServletFilters() {
		// TODO Auto-generated method stub
		CharacterEncodingFilter characterEncodingFilter = new CharacterEncodingFilter();
		characterEncodingFilter.setEncoding("UTF-8");
		characterEncodingFilter.setForceEncoding(true);
		
		return new Filter[] {characterEncodingFilter};
	}
```

### 11.4 일회성 데이터 redirect
 - RedirectAttributes라는 타입의 addFlashattribute() 함수는 보관덴 데이터는 단 한번만 사용할 수 있게 보관된다 (내부적으로는 HttpSession을 이용해서 처리된다.)

 ### 11.5 목록 페이지와 뒤로가기 문제
  - 최근의 웹페이지들은 사용자들의 트래픽을 고려해 목록페이지에서 새창을 띄워서 조회페이지로 이동하는 방식을 선호하지만, 전통적인 방식에서는 현재창 내에서 이동하는 방식을 사용한다.
  - 해당 예제에서 등록 -> 목록 -> 조회 화면 이동후 뒤로가기 버튼 사용하면 게시물이 등록 되었다는 모달창을 다시한번 볼 수 있다.
  #### - 위의 뒤로가기 문제가 발생하는 이유로는 '뒤로가기', '앞으로가기'를 하면 서버를 다시 호출하는게 아니라 과거에 자신이 가진 모든 데이터를 활용하기 때문이다. (위의 RedirectAttributes 또한 예외없이 사용되는 것을 확인 할 수 있다.)

  >  - 해결방안  
  >  스택 구조로 동학하는 '**window 의 history**' 객체 사용
  > 추가로 Teck_Stack -> Spring -> History(Back,Forward).md 파일확인

### 11.6 수정/삭제
 - 1) 조회 페이지에서 직접처리하는 방식
 - 2) 별도의 수정/삭제 페이지를 만들어 해당 페이지에서 수정/삭제를 처리하는 방식
 - 최근에는 게시물의 조회 페이지에서 댓글 등에 대한 처리가 많아지면서 '2)'의 수정/삭제를 별개의 페이지에서 하는 것이 일반적이다.

 ### 11.7 다수의 Button 처리
 ```js
 $(document).ready(function(){
 		var formObj = $("form");
 		
 		$('button').on('click', function(e){
 			e.preventDefault();
	 		var operation = $(this).data("oper");
	 		
	 		console.log(operation);
	 		
	 		if(operation === 'remove'){
	 			formObj.attr("action", "/board/remove");
	 		}else if(operation === 'list'){
	 			self.location="/board/list";
	 			return ;
	 		}
	 		
	 		formObj.submit();
 		});
 	})
```

 - 여러개의 \<Button> 태그 사용시 속성값으로 위에서는 'data-oper'를 주어 버튼을 구분한다.
 - <form> 태그의 모든 버튼은 기본적으로 submit으로 처리하기 때문에 'e.preventDefault()'로 기본동작을 막고 해당 버튼에 대한 속성을 지정 후에 마지막으로 submit()을 수행한다.  
- $(form).empty() 로 폼 내부의 내용을 삭제할 수 있다.

## **Chapter 12** 오라클 데이터베이스 페이징 처리

 ### 12.1 Order by 문제

- 수백 만 데이터가 주어졋을 때 빠르게 동작하는 SQL을 위해서는 먼저 order by 를 이용하는 작업을 하면 좋지 않다.

 ### 12.2 SQL 실행 계획 과 order by
 - 데이터 베이스에 전달된 SQL문은 아래와 같은 과정을 거친다.
 >  1. SQL 파싱  
    - SQL 구문에 오류가 있는지 검사   
    - SQL을 실행 해야 하는 대상 객체(테이블, 제약 조건, 권한 등)가 존재 하는지 검사
 >  2. SQL 최적화  
    - SQL이 실행되는데 필요한 비용(cost) 계산  
    - 계산된 값을 기초로 어떤 방식으로 실행하는 것이 좋은지 판단하는 '**실행 계획(execuion plan)**' 을 세운다.
 >  3. SQL 실행  
    - 세워진 실행 계획을 통해 메모리상에서 데이터를 읽거나 물리적인 공간에서 데이터를 로딩하는 등의 작업 진행

 - 데이터가 많은상태에서의 정렬작업경우 가장 일반적인 해결책은 '인덱스(index)를 이용해서 정렬을 생략하는 방법이다.
 - **인덱스** 라는 존재가 이미 정렬된 구조 이므로 이를 이용해서 별도의 정렬을 하지 않는 방법이다.
 >  - index 사용 SQL
 ```sql
 select * from tbl_board order by bno desc;
 ```
 >  - hint 사용 SQL
 ```sql
 select /*+INDEX_DESC (tbl_board pk_board) */ *
from tbl_board
where bno >0;
```
 - index 사용 쿼리같은 경우 현재 index를 제대로 타지않는 결과를 보여준다. (확인필요)
 - hint 사용쿼리 경우 index가 제대로 적용되어 빠른 속도의 효과를 적용할 수 있다.
 - SQL의 실행 계획에서 주의해서 봐야 하는 부분
    -  sort를 하지 않았다는 점
    - tbl_board를 바로 접근하는 것이 아니라 pk_board(pk index)를 이용해서 접근한 점
    - RANGE SCAN DESCRENDING, BY INDEX ROWID로 접근한 점

### 12.3 Hint 사용 문법
 - select문을 작성할 때 힌트는 잘못 작성되어도 실핼할 때는 무시되기만 하고 별도의 에러는 발생하지 않는다.
 - 힌트 사용 문법
 ```sql
 SELECT
 /*+ Hint name (param...) */ column name, .....
 FROM
  table name
 ```
  > - 힌트 구문은 '**/*+**' 로 시작하고 '***/**'로 마무리 된다.
  > - 힌트 자체는 SQL로 처리되지 않기 때문에 위의 그림처럼 뒤에 컬럼명이 나오더라도 별도으 ','로 처리되지 않습니다.

 ### 12.4 FULL 힌트
  - 힌트 중에는 해당 select문을 실행할 때 테이블 전체를 스캔할 것으로 명시하는 FULL힌트가 있다.
  - FULL 힌트는 테이블의 모든 데이터를 스캔하기 때문에 데이터가 많을 떄는 상당히 느리게 실행된다.
  ```SQL
  SELECT
  /*+ FULL(table name) */ column name, .....
  FROM 
   table name
```

### 12.5 INDEX_ASC, INDEX_DESC 힌트
 - 흔히 목록 페이지에서 가장 많이 사용하는 힌트는 인덱스와 관련된 'INDEX_ASC, INDEX_DESC'힌트이다.
 - 이름에서도 보이는것과 같이 주로 'order by'를 위해서 사용한다.
 - 인덱스 자체가 정렬을 해둔 상태이므로 이를 통해 SORT과정을 생략하기 위한 용도이다.
 - 테이블 이름과 인덱스 이름을 같이 파라미터로 사용한다.
 ```sql
 SELECT 
 /*+ INDEX_ASC or INDEX_DESC(table name index name) */ column name, ....
 FROM
  table name
 ```
  - 예제
  ```sql
 select /*+INDEX_DESC (tbl_board pk_board) */ *
from tbl_board
where bno >0;
```

### 12.6 ROWNUM
 - ROWNUM 이라는 것은 데이터를 가져올 때 적용되는 것이다.
 - 이 후 에 정렬되는 과정에서는 ROWNUM이 변경되지않는다. 정렬은 나중에 처리된다는 의미.
 - Table Alias를 사용했을 경우와 사용하지 않을 경우의 RowNum 맺어지는 결과값이 다르가 작용된다. 해당 사용법 **확인필요**

 ### 12.7 인덱스를 이용한 접근시 ROWNUM

  - 인덱스를 통해 접근한다면 다음과 같은 과정으로 접근한다.  
    1. 인덱스를 통해서 테이블에 접근
    2. 접근한 데이터에 ROWNUM 부여
  - 위의 1의 과정에서 이미 정렬이 되어있기 떄문에 ROWNUM은 전혀 다른 값을 가지게 된다.

### 12.8 페이지 번호 1,2의 데이터
 - 한 페이지당 10개의 데이터를 출력한다고 가정할 때 ROWNUM 조건을 WHERE 구문에 추가해 다음과 같이 작성 가능하다
 ```sql
 select /*+INDEX_DESC(tbl_board pk_board) */
rownum rn, bno, title, content
from tbl_board
where rownum <= 10;
 ```

 - 위의 1페이지 결과 이후 2페이지 결과를 동일한 쿼리로 가져올 경우 결과값이 나오지 않음을 확인할 수 있다.
 ```sql
 select /*+INDEX_DESC(tbl_board pk_board) */
rownum rn, bno, title, content
from tbl_board
where rownum <= 20 and rownum > 10;
```

   - 위의 실행계획을 보면
        - ROWNUM > 10 인데이터를 찾는다
        - TBL_BOARD에 처음으로 나오는 ROWNUM의 값은 1이다. where의 조건에 의해 1값은 무시된다.
        - ROWNUM의 값은 계속 1로 만들어지고 where 조건에 의해 없어지는 과정이 반복데므로 테이블의 모든 데이터를 스캔하지만 결과는 아무것도 나오지 않는다.
        - 위의 결과에서 rownum이 1이 포함되게 조건을 수정해야 결과를 얻을 수 있다.

#### 12.8.1 인라인 뷰(In-Line View) 처리
- 'Select 문 안쪽 From에 다시 Select문'으로 처리하는 방법
- 어떠한 결과를 구하는 Select 문이 잇고, 그 결과를 다시 대상으로 삼아 Select를 하는 것이다.
```sql
SELECT
    rn,
    bno,
    title,
    content
FROM
    (
        SELECT /*+INDEX_DESC(tbl_board pk_board) */
            ROWNUM rn,
            bno,
            title,
            content
        FROM
            tbl_board
        WHERE
            ROWNUM <= 20
    )
WHERE
    rn > 10;
```
 - 위의 과정
    - 필요한 순서로 정렬된 데이터에 ROWNUM을 붙인다.
    - 처음부터 해당 페이지의 데이터를 'ROWNUM <= 30'과 같은 조건을 이용해서 구한다.
    - 구해놓은 데이터를 하나의 테이블처럼 간주하고 인라인뷰로 처리한다.
    - 인라인뷰에서 필요한 데이터만을 남긴다.


 - 페이징 처리를 위해 필요한 파라미터
    - 페이지 번호(pageNum)
    - 한 페이지당 몇개의 데이터(amount)를 보여줄 것인지

- 페이지 번호와 몇 개의 데이터가 필요한지를 별도의 파라미터로 전달하는 방식도 나쁘지 않지만, 아예 이 데이터들을 하나의 객체로 묶어서 전달하는 방식이 나중을 생각하면 좀 더 확장성이 좋다.

```java
@Getter
@Setter
@ToString
public class Criteria {
	
	private int pageNum;
	private int amount;
	
	public Criteria() {
		this(1,10);
	}
	
	public Criteria(int pageNum, int amount) {
		this.pageNum = pageNum;
		this.amount = amount;
	}

}
```
- this(1,10)으로 생성자를 통해 기본값을 지정할 수 있다.

## **Chapter 14** 페이징 화면 처리

- 페이지를 보여주는 작업 과정
    - 브라우저 주소창에서 페이지 번호를 전달해서 결과를 확인하는 단계
    - JSP에서 페이지 번호를 출력하는 단계
    - 각 페이지 번호에 클릭 이벤트 처리
    - 전체 데이터 개수를 반영해서 페이지 번호 조절
- 페이지에서 조회, 수정/삭제 페이지 까지 페이지 번호가 계소해서 유지되어야만 하기 때문에 끝까지 신경써야할 부분이 많다.

### 14.1 페이징 처리할 때 필요한 정보들
 - 화면에 페이징 처리를 위해 필요한 정보
    - 현재 페이지 번호(page)
    - 이전과 다음으로 이동가능한 링크의 표시 여부(prev, next)
    - 화면에서 보여지는 페이지의 시작 번호와 끝 번호(startPage, endPage)
 - 일반적으로 페이지 계산할 때 시작번호를 먼저하려고 하지만, 오히려 끝 번호를 해 두는 것이 수월하다.
    - 끝 번호 공식(한번에 보여지는 페이지가 10페이지일 경우 10.0으로 나눈다.)
        ```java
        this.endPage = (int)(Math.ceil(페이지번호 / 10.0(한번에보여지는 페이지갯수))) * 10;
        ```
    - 시작 번호 (한번에 보여지는 페이지가 10일 경우 위의 끝번호에서 -9)
        ```java
        this.startPage = this.endPage - 9;
        ```
 - 만일 끝번호와 한페이지당 출력되는 데이터수의 곱이 전체 데이터 수보다 크다면 끝 번호는 다시 total을 이요해서 계산되어야 한다.
    - total을 통한 끝 번호 재계산
        ```java
        realEnd = (int)(Math.ceil(total * 1.0) / amount));

        if(realEnd < this.endPage){
            this.endPage = realEnd;
        })
        ```
#### 14.2 이전(prev) 과 다음(next)

- 이전 계산 ( 시작 번호가 1보다 큰 경우 존재)
    ```java
    this.prev = this.startPage > 1;
    ```
- 다음 계산 (위의 realEnd가 끝 번호(endPage)보다 큰 경우 존재)
    ```java
    this.next = this.endPage < realEnd;
    ```

### 14.2 페이지 번호 이벤트 처리
 - 일반적으로 \<a> 태그의 href 속성을 이용하는 방법을 사용하 수 도 있지만, 직접 링크를 처리하는 방식의 경우 검색 조건이 붙고 난 후에 처리가 복잡하게 되므로 JavaScript를 통해서 처리하는 방식을 이용한다.
 - JavaSciprt Function
    ```js
    var actionForm = $("#actionForm");
	
	$(".paginate_button a").on('click', function(e){
		
		e.preventDefault();
		
		console.log('click');
		
		actionForm.find("input[name='pageNum']").val($(this).attr("href"));
		actionForm.submit();
	});
    ```
     - e.preventDefault() 함수 
        - \<a> 의  href, \<button> 의 submit 과 같은 페이지 이동의 작동을 중지시키는 함수
### 14.3 조회 페이지로 이동
- 1페이지가 아닌 다른 페이지에서 조회 후 다시 목록으로 이동하면 무조건 1페이지로 이동하는 증상이 일어난다. 이를 해결하기 위해 조회 페이지로 갈때 현재 목록 페이지의 pageNum과 amount를 같이 전달해야 한다.

### 14.4 수정 페이지에서 리스트로 이동
 ```js
 else if(operation === 'list'){
	 			//move to list
	 			formObj.attr("action","/board/list").attr("method","get");
	 			
	 			var pageNumTag = $("input[name='pageNum']").clone();
	 			var amountTag = $("input[name='amount']").clone();
	 			
	 			formObj.empty();
	 			formObj.append(pageNumTag);
	 			formObj.append(amountTag);
	 		}
 ```
 - 기존의 formObj.empty() 에서 리스트로 갈때 다비워둔 채로 가져갔지만 필요한 수정전의 pageNum과 amount을 가지고 이동한다.

 ## **Chapter 15** 검색 처리
  - 검색 기능은 검색조건과 키워드로 나누어 생각할 수 있다.
  - 검색 조건은 일반적으로 \<select> 태그를 이용해서 작성하거나 \<checkbox>를 이용하는 경우가 많다.
  - 근래에 들어서는 일반 사용자들의 경우 \<select> 태그를, 관리자용이나 검색 기능이 강한 경우 \<checkbox>를 사용한다.

  ### 15.1 검색기능과 SQL
  - 게시물의 검색기능 분류
    - 제목/내용/작성자와 같이 단일 항목 검색
    - 제목 or 내용, 제목 or 작성자, 내용 or 작성자, 제목 or 내용 or 작성자와 같은 다중 항목 검색
  - 오라클은 페이징 처리에 인라인 뷰를 이용하기 때문에 실제로 검색 조건에 대한 처리는 인라인뷰의 내부에서 이루어져야 한다.  

  #### 15.1.1 다중 항목 검색
  ```sql
  select /*+INDEX_DESC(tbl_board, pk_board)*/ rownum rn, bno, title, content, writer, regdate, updatedate
from tbl_board
where title like ('%test%') or content like ('%test%') and rownum <= 20
;
```
- 위의 SQL 구문자체에는 이상이 없지만 실제로 동작 시켜보면 20개 의 데이터가아닌 그이상의 데이터를 볼 수 있다.
    - 위의 SQL구문에서는 AND 연산자가 OR연산자보다 우선순위가 높기 때문에 'ROWNUM이 20보다 작거나 같으며서(AND) 내용에 'test'라는 문자열이 있거나(OR) 제목에 'test'라는 문자열이 있는 게시물을 검색한다.
 ### 15.2 MyBatis의 동적 태그들
  - 동적 태그
    - if
    - choose(when, otherwise)
    - trim(where, set)
    - foreach
 #### 15.2.1 \<if>  

- if 는 test라는 속성과 함께 특정한 조건이 true가 되었을 때 포함된 SQL을 사용하고자 할 때 작성
    - ex)
    - 검색 조건이 'T'면 제목(title)이 키워드(keyword)인 항목 검색
    - 검색 조건이 'C'면 내용(content)이 키워드(keyword)인 항목 검색
    - 검색 조건이 'W'면 작성자(writer)이 키워드(keyword)인 항목 검색
    
        ```xml
        <if test="type == 'T'.toString()">
            (title like '%'||#{keyword}||'%')
        </if>
        <if test="type == 'C'.toString()">
            (content like '%'||#{keyword}||'%")
        </if>
        <if test="type == 'W'.toString()">
            (writer like '%'||#{keyword}||'%")
        </if>
        ```
- if 안에 들어가는 표현식(expression)은 OGNL 표현식이라는 것을 이용한다.
- 좀더 자세한 내용은 https://commons.apache.org/proper/commons-ognl/language-guide.html 참고

#### 15.2.2 \<choose>
- if와 달리 choose는 여러 상황들 중 하나의 상황에서만 동작한다.
- Java의 'if ~ else' 나 JSTL의 \<choose>와 유사
    ```xml
    <choose>
        <when test="type == 'T'.toString()">
            (title like '%'||#{keyword}||'%')
        </when>
        <when test="type == 'C'.toString()">
            (content like '%'||#{keyword}||'%')
        </when>
        <when test="type == 'W'.toString()">
            (writer like '%'||#{keyword}||'%')
        </when>
        <otherwise>
            (title like '%'||#{keyword}||'%' or content like '%'||#{keyword}||'%')
        </otherwise>
    </choose>
    ```
#### 15.2.3 \<trim>, \<where>, \<set>
 - trim,where,set은 단독으로 사용되지 않고, \<if>, \<choose>와 같은 태그들을 내포하여 SQL들을 연결해 주고, 앞 뒤에 필요한 구문들(AND, OR, WHERE 등)을 추가하거나 생략하는 역할을 한다.
- \<where>의 경우 태그 안 쪽에서 SQL이 생성될 때는 WHERE구문이 붙고, 그렇지 않은 경우에는 생성되지 않는다.
    ```sql
    select * frmo tbl_board
    <where>
        <if test="bno != null">
            bno = #{bno}
        </if>
    </where>
    ```
    | | |
    |--|--|
    |bno값이 존재하는 경우 | select * from tbl_board WHERE bno = 33
    |bno가 null인 경우|select * from tbl_board

- \<trim>은 태그의 내용을 앞의 내용과 관련되어 원하는 접두/접미를 처리할 수 있다.
    ```sql
    select * from tbl_board
    <where>
        <if test="bno != null">
            bno = #{bno}
        </if>
        <trim prefixOverrides ="and">
            rownum = 1
        </trim>
    </where>
    ```
    - trim은 prefix, suffix, prefixOverrides, suffixOverrides 속성을 지정할 수 있다.  

    | | |
    |--|--|
    |bno값이 존재하는 경우 | select * from tbl_board WHERE bno = 33 and rownum = 1
    |bno가 null인 경우|select * from tbl_board WHERE rownum = 1
 - \<foreach>는 List, 배열, 맵 등을 이용해서 루프를 처리할 수 있다.
    - 주로 IN 조건에서 많이 사용하지만, 경우에 따라서는 복잡한 WHERE 조건을 만들 떄에도 사용할 수 있다.
    - ex) 제목('T')은 'TTTT'로 내용('C')은 'CCCC'라는 값을 이용한다면 Map의 형태로 작성 가능
    ```java
        Map<String, String> map = new HashMap<>();
        map.put("T", "TTTT");
        map.put("C", "CCCC");
    ```
    - 작성된 Map을 파라미터로 전달하고, foreach를 이용
    ```xml
        <trim prefix="where (" suffix=")" prefixOverrides="OR">
            <foreach item="val" index="key" collection="map">
                <trim prefix = "or">
                    <if test="key == 'C'.toString()">
                        content = #{val}
                    </if>
                    <if test="key == 'T'.toString()">
                        title = #{val}
                    </if>
                    <if test="key == 'W'.toString()">
                        writer = #{val}
                    </if>
                </trim>
            </foreach>
        </trim>
    ```
    - foreach를 배열이나 List를 이용하는 경우에는 item 속성만을 이용하면 되고, Map의 형태로 key와 value를 이용해야 할 때는 index와 item 속성을 둘다 이용한다.

 ### 15.3 Class 사용 - MyBatis(동적 쿼리)   
 ```java
 public String[] getTypeArr() {
		
		return type == null ? new String[] {} : type.split("");
	}
```
```xml
<trim prefix="where (" suffix=")" prefixOverrides="OR">
    <foreach item="type" collection="typeArr">
        <trim prefix="OR">
            <choose>
                <when test="type =='T'.toString()">
                    title like '%'||#{keyword}||'%'
                </when>
                <when test="type =='C'.toString()">
                    content like '%'||#{keyword}||'%'
                </when>
                <when test="type =='W'.toString()">
                    writer like '%'||#{keyword}||'%'
                </when>
            </choose>
        </trim>
    </foreach>
</trim>
```
- foreach 를 이용해서 검색 조건을 처리하는데 typeArr이라는 속성을 사용한다
- MyBatis는 원하는 속성을 찾을 떄 getTypeArr()과 같이 이름에 기반을 두어서 검색하기 때문에 Class에서 만들어둔 getTypeArr() 결과인 문자열의 배열이 \<foreach>의 대상이 된다.
- MyBatis는 엄격하게 Java Beans의 규칙을 따르지 않고, get/set 메서드만을 활용하는 방식이다.

### 15.4 \<sql> \<include> 와 검색 데이터의 개수 처리
- 동적 SQL을 이용해서 검색 조건을 처리하는 부분은 해당 데이터의 개수를 처리하는 부분에서도 동일하게 적용되어야만 한다. 
    - 가장 간단한 방법으로 동적 SQL을 처리하는 부분을 그대로 복사
    - 하지만 동적 SQL을 수정하는 경우에는 여러번의 수정작업이 필요
 - MyBatis는 \<sql> 이라는 태그를 이용해서 SQL의 일부를 별도로 보관하고, 필요한 경우에 include 시키는 형태로 사용할 수 있다.

- \<sql> 변환
 ```xml
 <sql id="criteria">
		<trim prefix="where (" suffix=") AND" prefixOverrides="OR">
            <foreach item="type" collection="typeArr">
            	<trim prefix="OR">
            		<choose>
            			<when test="type =='T'.toString()">
            				title like '%'||#{keyword}||'%'
            			</when>
            			<when test="type =='C'.toString()">
            				content like '%'||#{keyword}||'%'
            			</when>
            			<when test="type =='W'.toString()">
            				writer like '%'||#{keyword}||'%'
            			</when>
            		</choose>
            	</trim>
            </foreach>
        </trim>
	</sql>
 ```  
- \<sql> -> \<include> 적용

 ```xml
 <select id="getListWithPaging" resultType="org.zerock.domain.BoardVO">
		<![CDATA[
		select
			bno, title, content, writer, regdate, updatedate
		from
			(
			select /*+INDEX_DESC(tbl_board pk_board) */
			 rownum rn, bno, title, content, writer, regdate, updatedate
		 	from
			  tbl_board
		]]>

		<include refid="criteria"></include>

		<![CDATA[
			rownum <= #{pageNum} * #{amount}
			)
		where rn > (#{pageNum} -1) * #{amount}
		]]>
	</select>
 ```

 ### 15.5 화면에서 검색 조건 처리
 - 주의 사항
    - 페이지 번호가 파라미터로 유지되었던 것처럼 검색 조건과 키워드 역시 항상 화면 이동 시 같이 전송되어야 한다.
    - 화면에서 검색 버튼을 클릭하면 새로 검색을 한다는 의미이므로 1페이지로 이동한다.
    - 한글의 경우 GET 방식으로 이동하는 경우 문제가 생길 수 있으므로 주의 한다.
 - 검색이 처리된 후 문제점들
    - 3페이지를보다 검색을 하면 3페이지로 이동하는 문제
    - 검색 후 페이지를 이동하면 검색 조건이 사라지는 문제
    - 검색 후 화면에서는 어떤 검색 조건과 키워들 이용했는지 알 수 없는 문제

#### 15.5.1 UriComponentsBuilder를 이용하는 링크 생성
 - 웹페이지에서 매번 파라미터를 유지하는 일이 번거롭고 힘들다면 한 번쯤 UriComponentsBuilder라는 클래스를 이용해볼 필요가 있다.
 - org.springframework.web.util.UriComponentsBuilder는 여러 개의 파라미터들을 연결해서 URL의 형태로 만들어주는 기능을 가지고 있다.
- URL을 만들어주면 redirect를 하거나, \<form> 태그를 사용하는 상황을 많이 줄여줄 수 있다.
```java
public String getListLink() {
    
    UriComponentsBuilder builder = UriComponentsBuilder.fromPath("")
            .queryParam("pageNum", this.pageNum)
            .queryParam("amount", this.amount)
            .queryParam("type", this.type)
            .queryParam("keyword", this.keyword);
    
    return builder.toUriString();
}
```
 - GET방식에 적합한 URL 인코딩된 결과로 만들수있다.
 - 한글 처리에 신경을 않써도 된다.

 - 사용 예제
    - 사용 전
    ```java
    @PostMapping("/modify")
	public String modify(BoardVO board, @ModelAttribute("cri") Criteria cri, RedirectAttributes rttr) {
		
		log.info("modify : " + board);
		
		if(service.modify(board)) {
			rttr.addFlashAttribute("result", "success");
		}
		
		rttr.addAttribute("pageNum", cri.getPageNum());
		rttr.addAttribute("amount", cri.getAmount());
		rttr.addAttribute("type", cri.getType());
		rttr.addAttribute("keyword", cri.getKeyword());
		
		return "redirect:/board/list";
	}
    ```
    - 사용 후
    ```java
    @PostMapping("/modify")
	public String modify(BoardVO board, @ModelAttribute("cri") Criteria cri, RedirectAttributes rttr) {
		
		log.info("modify : " + board);
		
		if(service.modify(board)) {
			rttr.addFlashAttribute("result", "success");
		}
		
		return "redirect:/board/list" + cri.getListLink();
	}
    ```

 - UriComponentsBuidler로 생성된 URL은 화면에서도 유용하게 사용될 수 있는데, 주로 JavaScript를 사용할 수 없는 상황에서 링크를 처리해야하는 상황에서 사용된다.