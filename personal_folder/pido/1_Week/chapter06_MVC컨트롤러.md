
[Part2] - chapter 06
=========================

MVC Controller
--------
- @Controller
    ```java
    @Controller
    @RequestMapping("/sample/*")
    public class SampleController {

    }
    ```
    servlet-context.xml 을 통해 자동으로 Bean 등록   
    ```<context:component-scan>```태그를 통해 스캔후 객체로 생성

- @RequestMapping   
현재 클래스의 모든 메서드들의 기본적인 URL경로   
    > - method 속성 추가(GET / POST 방식 구분시 이용)   
    > - 스프링 4.3버전 이후 축약형으로 @GetMapping, @PostMapping 등장
    ```java
    // GET, POST 모두 지원할 때 배열로 처리하여 지정 가능 
    @RequestMapping(value="/basic", method= {RequestMethod.GET, RequestMethod.POST})
	public void basicGet() {
		log.info("basic get ...........................");
	}
	
    // 간편하지만 get 만 지원하여 제한이 많은편 
	@GetMapping("/basicOnlyGet")
	public void basicGet2() {
		log.info("basic get only get .................................");
	}
    ```    

#### < Controller의 파라미터 수집 >
1. Controller의 메서드가 DTO를 파라미터로 사용하게 되면 자동으로 setter 메서드 동작   
2. 파라미터가 자동으로 수집. request.getParameter()를 사용하지 않아도 됨!
3. 파라미터 추가 호출 *http://localhost:8080/sample/ex01?name=AAA&age=20*    
    이때 기본 path 경로가 '/' 인지 확인

    ```java
    // DTO 클래스
    @Data
    public class SampleDTO {
        private String name;
        private int age;
    }

    // Controller 클래스
    @GetMapping("/ex01")
    public String ex01(SampleDTO dto) {
        log.info("" + dto);
        return "ex01";
    }
    ```
4. @RequestParam 을 이용한 전달    
    파라미터로 사용된 변수의 이름과 전달되는 파라미터의 이름이 다른 경우 유용 
    
    ```java
    @GetMapping("/ex02")
	public String ex02(@RequestParam("name") String name, @RequestParam("age") int age) {
		log.info("name : " + name);
		log.info("age : " + age);
		return "ex02";
	}
    ```
5. 동일한 이름의 파라미터가 여러개 전달 되는 경우 ArrayList<> 처리   
    파라미터 타입을 보고 객체를 생성하므로 실제적인 클래스 타입 지정   
    배열도 동일 처리 (String[] ids)
    ```java
    @GetMapping("/ex02List")
	public String ex02List(@RequestParam("ids") ArrayList<String> ids) {
		log.info("ids : " + ids);
		return "ex02List";
	}
    ```
   > http://localhost:8080/sample/ex02List?ids=111&ids=222&ids=333 을 호출할 경우,    
    INFO : org.zerock.controller.SampleController - ids : [111, 222, 333] 

6. 객체 리스트 전달 (DTO)    
    ```java
    // DTO 클래스
    @Data
    public class SampleDTOList {
        private List<SampleDTO> list;
        
        public SampleDTOList() {
            list = new ArrayList<>();
        }
    }

    // Controller 클래스 
    @GetMapping("/ex02Bean")
	public String ex02Bean(SampleDTOList list) {
		log.info("list dtos : " + list);
		return "ex02Bean";
	}
    ```
    > http://localhost:8080/sample/ex02Bean?list[0].name=aaa&list[2].name=bbb 호출 할 경우, (encodeURIComponent() 미사용 시 []는 유니코드로 바꿔줘야함. )   
    INFO : org.zerock.controller.SampleController - list dtos : SampleDTOList(list=[SampleDTO(name=aaa, age=0), SampleDTO(name=null, age=0), SampleDTO(name=bbb, age=0)])

7. 파라미터 변환   
    @InitBinder   
    변수 타입이 Date 일 경우 '2018-01-01'과 같은 형태의 데이터 일 때 사용하여 정상적인 파라미터를 수집
    ```java
    @InitBinder
	public void initBinder(WebDataBinder binder) {
		SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd");
		binder.registerCustomEditor(java.util.Date.class, new CustomDateEditor(dateFormat, false));
	}
    ```
    > INFO : org.zerock.controller.SampleController - todo : TodoDTO(title=test, dueDate=Mon Jan 01 00:00:00 KST 2018)   

    혹은 DTO의 파라미터로 사용되는 인스턴스 변수에    
    @DateTimeFormat(pattern = "yyyy/MM/dd")    로 적용, 변환 

8. Model  
    JSP에 생성된 데이터를 전달하는 역할   
    (MVC2의 requiest.setAttribute()와 유사)
    ```java
    public String home(Model model){
        model.addAttribute(String, String);
        return "";
    }
    ```

    * @ModelAttribute  
    기본 자료형 파라미터는 화면까지 전달되지 않는 것을 보완.   
    타입에 관계없이 강제로 Model에 담아서 파라미터를 전달.
        ```java
        public String ex(@ModelAttribute("page") int page){
            return "/page";
        }
        ```

    * @RedirectAttributes
    일회성으로 데이터를 전달하는 용도.
    (response.sendRedirect()와 동일한 용도)
        ```java
        public String home(RedirectAttributes rttr){
            rttr.addFlashAttribute("name", "AAA");
        }
        ```
9. Controller 리턴 타입
- String : jsp를 이용하는 경우, 파일 경로와 파일 이름을 리턴   
    return "ex05"일 경우   
    -> '/WEB-INF/views/sample/ex05.jsp
- void : 호출하는 URL과 동일한 이름의 jsp 
    public void ex05() 일경우   
    -> '/WEB-INF/views/sample/ex05.jsp
- 객체(VO, DTO) : JSON데이터를 만드는 용도 
    ```xml
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-databind</artifactId>
        <version>2.9.4</version>
    </dependency>
    ```
    ```java
    public @ResponseBody SampleDTO ex06(){
        SampleDTO dto = new SampleDTO();
        dto.set(10);
        dto.set("박새로이");
        return dto;
    }

    ==> 브라우저
    {"name":"박새로이", "age":10}
    ```
- ResponseEntity : HTTP프로토콜 헤더 정보 전달   
    원하는 메시지 가공 가능
    ```java
    @GetMapping("/ex07")
    public ResponseEntity<String> ex07(){
        String msg = "{\"name\" : \"홍길동\"}";
        HttpHeaders header = new HttpHeaders();
        header.add("Content-Type", "application/json;charset=UTF-8");
        return new ResponseEntity<>(msg, header, HttpStatus.OK);
    }
    ```
#### < 파일 업로드 처리 >
**- XML 기반 설정 -**  
pom.xml 
```xml
<dependency>
    <groupId>commons-fileupload</groupId>
    <artifactId>commons-fileupload</artifactId>
    <version>1.3.3</version>
</dependency>
```
임시 업로드 폴더 추가   
```ex) C:\upload\tmp```

servlet-context.xml   
```xml
<beans:bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
    <beans:property name="defaultEncoding" value="utf-8"></beans:property>
    <!-- 1024 * 1024 * 10 bytes		10MB (전달 최대사이즈)-->
    <beans:property name="maxUploadSize" value="104857560"></beans:property>
    <!-- 1024 * 1024 * 2 bytes		2MB (하나의 파일 최대 크기)-->
    <beans:property name="maxUploadSizePerFile" value="2097152"></beans:property>
    <beans:property name="uploadTempDir" value="file:/C:/upload/tmp"></beans:property>
    <beans:property name="maxInMemorySize" value="10485756"></beans:property>
</beans:bean>
```
*id 값은 반드시 multipartResolver 지정*

MultipartFile의 배열   
전달되는 동일한 이름의 파라미터가 여러개 존재하면 배열로 처리
```java
@GetMapping("/exUploadPost")
public void exUploadPost(ArrayList<MultipartFile> files) {
    files.forEach(file -> {
        log.info("-------------------------------------");
        log.info("name : " + file.getOriginalFilename());
        log.info("size : " + file.getSize());
    });
}
```

**- JAVA 기반 설정 -**    
ServletConfig.java  
```java
@Bean(name= "multipartResolver")
public CommonsMultipartResolver getResolver() throws IOException {
    CommonsMultipartResolver resolver = new CommonsMultipartResolver();
    
    // 10MB
    resolver.setMaxUploadSize(1024 * 1024 * 10);
    
    // 2MB
    resolver.setMaxUploadSizePerFile(1024 * 1024 * 2);
    
    // 1MB
    resolver.setMaxInMemorySize(1024 * 1024);
    
    // temp upload
    resolver.setUploadTempDir(new FileSystemResource("C:\\upload\\tmp"));
    
    resolver.setDefaultEncoding("UTF-8");
    
    return resolver;
}
```

#### < Controller의 Exception처리 >
**- XML 기반 설정 -**  
@ControllerAdvice   
AOP를 이용한 공통적인 예외사항 처리 
CommonExceptionAdvice.java
```java
// 스프링 컨트롤러에서 발생하는 예외를 처리하는 존재임을 명시 
@ControllerAdvice	
@Log4j
public class CommonExceptionAdvice {

// 해당메서드로 예외타입을 처리
@ExceptionHandler(Exception.class)		
public String except(Exception ex, Model model) {
    ...
    return "error_page";
}
```
이 때 새로 생성한 exception 패키지는 인식을 위해 servlet-context.xml에 추가   
```<context:component-scan base-package="org.zerock.exception" />```

**- JAVA 기반 설정 -**    
ServletConfig.java
```java
@EnableWebMvc
@ComponentScan(basePackages = {"org.zerock.controller", "org.zerock.exception"})
public class ServletConfig implements WebMvcConfigurer {
...
}
```

**404에러페이지 출력**   

**- XML 기반 설정 -**  
WAS 내부에서 발생하는 에러 처리   
web.xml
```xml
<servlet>
    <servlet-name>appServlet</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>/WEB-INF/spring/appServlet/servlet-context.xml</param-value>
    </init-param>
    <init-param>
        <param-name>throwExceptionIfNoHandlerFound</param-name>
        <param-value>true</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
</servlet>
```
CommonExceptionAdvice.java   
```java
@ExceptionHandler(NoHandlerFoundException.class)
@ResponseStatus(HttpStatus.NOT_FOUND)
public String handle404(NoHandlerFoundException ex) {
    return "custom404";
}
```

**- JAVA 기반 설정 -**  
서블릿 3.0 이상 이용 필수    
WebConfig.java    
```java
@Override
protected void customizeRegistration(ServletRegistration.Dynamic registration) {
    registration.setInitParameter("throwExceptionIfNoHandlerFound", "true");
}
```
    


