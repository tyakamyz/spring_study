 - ## Part01 - Chapter 02 - 4. Spring이 동작하며 생기는 일

 	- Spring FrameWork가 시작 되면 Spring이 사용하는 메모리 영역을 만듦 - **Context** 라  하며 **ApplicationContext**라는 이름의 객체가 만들어진다.
 	- Spring은 자신이 객체를 생성하고 관리해야 하는 객체들에 대한 설정이 필요 - **root-context.xml**
 	- root-context.xml에 설정되어있는 **<<context:component-scan>>** 태그의 내용을 통해서 **org.zerock.sample** 패키지를 스캔한다.
 	- 해당 패키지에 있는 클래스들 중 Spring이 사용하는 @Component라는 어노테이션이 존재하는 클래스의 인스턴스를 생성
 	- Restaurant 객체는 Chef 객체가 필요하다는 어노테이션 설정이 있으므로 , Spring은 Chef 객체의 레퍼런스를 Restaurnat 객체에 주입
  ---

- ## Part02 - Chapter 05 - 2. 스프링 MVC 프로젝트 내부 구조
    - 스프링 MVC 프로젝트를 구성해서 사용한다는 의미는 내부적으로 root-context.xml로 사용하는 일반 Java 영역(POJO)과 servlet-context.xml로 설정하는 Web 관련 영역을 같이 연동해서 구동한다.
---

- ## Part02 - Chapter 05 - 6. 예제 프로젝트 로딩 구조

    - 프로젝트 구동시 관여하는 XML은 web.xml, root-context.xml, servlet-context.xml 파일 이다.
    - web.xml은 Tomcat 구동과 관련 된 설정 파일.
    - root-context.xml, servlet-context.xml은 스프링과 관련된 설정 파일.

    #### (1). 프로젝트 구동은 web.xml에서 시작. web.xml의 상단에 가장 먼저 구동되는 Context Listener가 등록되있음.
    ```xml
    <!-- The definition of the Root Spring Container shared by all Servlets and Filters -->
	<context-param>
		<param-name>contextConfigLocation</param-name>
		<param-value>/WEB-INF/spring/root-context.xml</param-value>
	</context-param>
	
	<!-- Creates the Spring Container shared by all Servlets and Filters -->
	<listener>
		<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
	</listener>
    ```
    - < context-param > 에는 root-context.xml의 경로가 설정되어있다.
    - < listener >에는 스프링 MVC의 ContextLoaderListener가 등록되어 있는 것을 볼 수 있다.
    - ContextLoaderListener는 해당 웹 어플리케이션 구동 시 같이 동작하므로 해당 프로젝트를 실행하면 가장 먼저 로그를 출력하면서 기록된다.  
    #### (2) root-context.xml이 처리되면 파일에 있는 빈(Bean) 설정들이 동작하게 된다.
    - root-context.xml에 정의된 객체(Bean)들은 스프링의 영역(context)안에 생성되고, 객체들 간의 의존성이 처리된다.
    #### (3) root-context.xml이 처리된 후에는 스프링 MVC에서 사용하는 DispatcherServlet이라는 서블릿과 관련된 설정이 동작한다.
    - org.springframework.web.servlet.DispatcherServlet 클래스는 스프링 MVC의 구조에서 가장 핵심적인 역할을 하는 클래스이다. 내부적으로 웹 관련 처리의 준비작업을 진행하는데 이 때 사용하는 파일이 servlet-context.xml이다.
    - DispatcherServlet에서 XmlWebApplicationContext를 이용해서 servlet-context.xml을 로딩하고 해석하기 시작한다. 이 과정에서 동륵된 객체(Bean)들은 기존에 만들어진 객체(Bean)들과 같이 연동되게 된다.
---
- ## Part 02 - Chapter 05 - 8. 모델 2와 스프링 MVC
    - 모델2방식은 **'로직과 화면을 분리'** 하는 스티일의 개발방식
    >   (1) Request -> **Controller** (요청)  
    >   (2) **Controller** -> **Model** (데이터 처리요청)  
    >   (3) **Model** -> **Controller** (데이터 처리 후 전달)  
    >   (4) **Controller** -> **View**  (처리된 데이터 가공 후 전달)  
    >   (5) **View** -> Response  (전달)
    - 스프링 MVC 기본 구조
    >   #### (1) Request -> **DispatcherServlet** <br/>(Front-Controller 인 DispatcherServlet을 통해 사용자 Request 처리)  
    ```xml
    <servlet>
		<servlet-name>appServlet</servlet-name>
		<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
		<init-param>
			<param-name>contextConfigLocation</param-name>
			<param-value>/WEB-INF/spring/appServlet/servlet-context.xml</param-value>
		</init-param>
		<load-on-startup>1</load-on-startup>
	</servlet>
		
	<servlet-mapping>
		<servlet-name>appServlet</servlet-name>
		<url-pattern>/</url-pattern>
	</servlet-mapping>
    ```
    >   #### (2) **DispatcherServlet** <-> **HandlerMapping**
    - HandlerMapping은 Request의 처리를 담당하는 컨트롤러를 찾기 위해 존재
    - HandlerMapping 인터페이스를 구현한 여러 객체들 중 RequestMappingHandlerMaping같은 경우 @RequestMapping 어노테이션이 적용된 것을 기준으로 판단하게 된다.
    >   #### (3) **DispatcherServlet** -> **HandlerAdapter**  
    - HandlerMapping을 통해 @RequestMapping로 컨트롤러를 찾게 되었다면 HandlerAdapter를 이용해서 해당 컨트롤러를 동작한다.
    >   #### (4) **HandlerAdapter** <-> **Controller**  
    - Controller는 개발자가 작성하는 클래스로 실제 Request를 처리하는 로직을 작성하게 된다.
    - View에 전달해야 하는 데이터는 주로 Model이라는 객체에 담아서 전달한다.
    - Controller는 다양한 타입의 결과를 반환하는데 이에 대한 처리는 ViewResolver를 이용한다.
    >   #### (5) **HandlerAdapter** -> **DispatcherServlet**  
    >   #### (6) **DispatcherServlet** <-> **ViewResolver**  
    - ViewResolver는 Controller가 반환한 결과를 어떤 View를 통해서 처리하는 것이 좋을지 해석하는 역할.
    - 가장 흔하게 사용되는 설정은 servlet-context.xml에 정의된 InternalResourceViewResolver이다.
    ```xml
    <beans:bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
		<beans:property name="prefix" value="/WEB-INF/views/" />
		<beans:property name="suffix" value=".jsp" />
	</beans:bean>
    ```
    >   #### (7) **DispatcherServlet** -> **View**  
     - View는 실제로 응답 보내야 하는 데이터를 Jsp등을 이용해서 생성하는 역할을 하게된다.
    - 만들어진 응답은 DispatcherServlet을 통해서 전달된다.
    >   #### (8) **View** -> Jsp 및 기타

    ---
    - 위의 과정을 확인하면 모든 Request는 DispatcherServlet을 통하도록 설계되는데 이런 방식을 **Front-Controller 패턴**이라 한다.
---
## Part 02 - Chapter 06 - 2. @RequestMapping의 변화

 - 추가 속성 부여 가능하다. 가장 많이 사용하는 속성은 **method** 속성으로 흔히 GET/POST 방식을 구분해서 사용할때 이용된다.
 - **스프링 4.3** 버전부터 이러한 @RequestMapping을 줄여서 사용할 수 있는 @GetMapping, @PostMapping이 등장했다.
 - @PutMapping, @DeleteMapping 또한 추가로 등장했다는 것을 확인 할 수있다.
---
 ## Part 02 - Chapter 06 - 3. DTO

  - Lombok의 @Data 어노테이션을 이용해서 처리하게 되면 getter/setter, equals(), toString() 등의 메서드를 자동 생성하기에 편리하다.
  ```java
  @Data
public class SampleDTO {
	
	private String name;
	private int age;
}
```
 - DTO객체 여러개를 받을 리스트가 필요할 때 별도의 DTOList 클래스 생성해서 사용
 ```java
 @Data
public class SampleDTOList {
	
	private List<SampleDTO> list;
	
	public SampleDTOList() {
		list = new ArrayList<>();
	}
}
```
 - GET 방식의 URL
```
http://localhost:18080/sample/ex02Bean?list[0].name=aaa&list[2].name=bbb
```

- Tomcat은 버전에 따라 위아 같은 문자열에서 '[]' 문자를 특수문자로허용하지 않을 수 있다.
   - javascript의 경우 encodeURLComponent()와 같은 방법으로 해결 가능
   - 그 외 '[' 는 '%5B' 로 ']'는 '%5D'로 변경해서 사용한다.
---
## Part 02 - Chapter 06 - 4. @RequestParam
-   스프링은 파라미터의 **'타입'** 을 보고 객체를 생성하므로 파라미터의 타입은 List<>와같은 **인터페이스** 타입이아닌 실제적인 **클래스** 타입으로 지정한다 ex) ArrayList<> 타입
---
## Part 02 - Chapter 06 - 5. @InitBinder

 - DTO에서와의 같이 파라미터값의 바인딩이 자동으로 되는 경우가 있지만. 그렇지 않은 경우 역시 존재한다. 예로 '2018-01-01'과 같은 날짜 데이터를 java.util.Date 타입으로 변환하는 작업이 그러하다.
 > (1) 스프링 Controller에서는 파라미터를 바인딩할 때 자동으로 호출되는 @InitBinder를 이용해서 이러한 변환을 처리할 수 있다.
 ```java
 @InitBinder
	public void initBinder(WebDataBinder binder) {
		
		SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd");
		binder.registerCustomEditor(   java.util.Date.class, 
                                    new CustomDateEditor(dateFormat, false));
		
	}
```
> (2) @DatetimeFormat 어노테이션 사용, 해당 어노테이션 사용 시 @initBinder는 제거 해야한다.
```java
@DateTimeFormat(pattern = "yyyy/MM/dd")
	private Date dueDate;
```
---
## Part 02 - Chapter 06 - 7. @ModelAttribute

 - **스프링 MVC의 Controller는 기본적으로 Java Beans 규칙에 맞는 객체는 다시 화면으로 객체를 전달한다.**
 - 좁은 의미에서 Java Beans의 규칙은 단순히 생성자가 없거나 빈 생성자를 가져야 하며, getter/setter를 가진 클래스의객체들을 의미한다.
 - 전달될 때에는 클래스명의 앞글자는 소문자로 처리된다.
 - 반면에 기본 자료형의 경우는 파라미터로 선언하더라도 기본적으로 화면까지 전달되지 않는다. 때문에 강제로 전달받은 파라미터를 Model에 담아서 전달하고자 할때 필요한 어노테이션이다.
 - @ModelAttribute 가 걸린 파라미터는 타입에 관계없이 무조건 Model에 담아서 전달되므로, 파라미터로 전달된 데이터를 다시 화면에서 사용해야 할 경우에 유용하게 사용된다.
 ---
 ## Part 02 - Chapter 06 - 9. Controller의 리턴 타입

  |타입|설명|
  |---|---|
  |String| jsp를 이용하는 경우에는 jsp 파일의 경로와 파일이름으르 나타내기 위해서 사용
  |void|호출하는 URL과 동일한 이름의 jsp를 의미
  |VO, DTO|주로 JSON 타입의 데이터를 만들어서 반환하는 용도로 사용
  |ResponseEntity|response 할 때 Http 헤더 정보와 내용을 가공하는 용도로  사용
  |Model, ModelAndView| Model로 데이터를 반환하거나 화면까지 같이 지정하는 경우에 사용<br/>(최근에는 많이 사용하지않는다.)
  |HttpHeaders|응답에 내용 없이 Http 헤더 메시지만 전달하는 용도로 사용
  ---
  ## Part 02 - Chapter 06 - 10. 파일 업로드 처리
 - 파일 업로드를 하기위해서는 파일 데이터를 분석해야 한다. 이를 위해 Servlet 3.0 전 까지는 commons의 파일 업로드를 이용하거나 cos.jar 등을 이용해서 처리를 해왔다.
 - Servlet 3.0이후 (Tomcat7.0)에는 기본적으로 업로드되는 파일을 처리할 수 있는 기능이 추가되어 있으므로 더이상 추가적인 라이브러리가 필요하지 않다.
 - 다만 **'Spring Legacy Project'** 로생성되는 프로젝트의 경우 Servlet 2.5를 기준으로 생성되기에 3.0 이후에 지원대는 설정을 사용하기 어렵다. 따라서 일반적으로 많이 사용하는 **commons-fileupload** 라이브러리를 이용

    #### 10.1 servlet-context.xml 설정
     - servlet-context.xml은 스프링 MVC의 특정한 객체(Bean)를 설정해서 파일을 처리한다.
     - 다른 객체(Bean)을 설정하는 것과 달리 파일 업로드의 경우 반드시 id값이 **'multipartResolver'** 로 정확하게 지정해야한다.
     > - xml 형식
    ```xml
    <beans:bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
	    <beans:property name="defaultEncoding" value="utf-8"></beans:property>
        <!-- 		1024 * 1024 * 10 bytes 10MB -->
        <beans:property name="maxUploadSize" value="104857560"></beans:property>
        <!-- 		1024 * 1024 * 2 bytes 2MB -->
        <beans:property name="maxUploadSizePerFile" value="2097152"></beans:property>
        <beans:property name="uploadTempDir" value="file:/Users/yun-wonhui/Desktop/upload/tmp"></beans:property>
        <beans:property name="maxInMemorySize" value="10485756"></beans:property>		
	</beans:bean>
    ```

    > - java 형식
    ```java
    @Bean(name = "multipartResolver")
	public CommonsMultipartResolver getResoler() throws IOException{
		
		CommonsMultipartResolver resolver = new CommonsMultipartResolver();
		
		// 10MB
		resolver.setMaxUploadSize(1024 * 1024 * 10);
		
		// 2MB
		resolver.setMaxUploadSizePerFile(1024 * 1024 * 2);
		
		//1MB
		resolver.setMaxInMemorySize(1024 * 1024);
		
		// temp upload
		resolver.setUploadTempDir(new FileSystemResource("/Users/yun-wonhui/Desktop/upload/tmp"));
		
		resolver.setDefaultEncoding("UTF-8");
		
		return resolver;
	}
    ```
    |속성|설명
    |--|--
    |maxUploadSize| 한번의 Request로 전달될 수 있는 최대 크기를 의미
    |maxUploadSizePerFile| 하나의 파일 최대 크기
    |maxInMemorySize| 메모리상에서 유지하는 최대의 크기, 만일 이크기 이상의 데이터는 uploadTempDir에 임시파일로 저장 된다.
    |uploadTempDir| 절대경로를 이용하려면 URL형태로 제공해야 하기에 **'file:/'** 로시작한다.
    |defaultEncoding| 업로드하는 파일의 이름이 한글일 경우 깨지는 문제 처리
    ---