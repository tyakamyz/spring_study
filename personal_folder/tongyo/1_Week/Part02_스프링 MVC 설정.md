# 2장. 스프링 MVC 설정
>## ch05. 스프링 MVC의 기본 구조
- web.xml
    - Tomcat 구동과 관련된 설정
- root-context.xml, servlet-context.xml
    - 스프링과 관련된 설정
-------------
- web.xml 설정
    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <web-app version="2.5" xmlns="http://java.sun.com/xml/ns/javaee"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://java.sun.com/xml/ns/javaee https://java.sun.com/xml/ns/javaee/web-app_2_5.xsd">

        <!-- The definition of the Root Spring Container shared by all Servlets and Filters -->
        <context-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>/WEB-INF/spring/root-context.xml</param-value>
        </context-param>
        
        <!-- Creates the Spring Container shared by all Servlets and Filters -->
        <listener>
            <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
        </listener>

        <!-- Processes application requests -->
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

    </web-app>
    ```
    - \<context-param> 
        - root-context.xml의 경로 설정
    - \<listener>
        - 스프링 MVC의 ContextLoaderListener 등록
        - ContextLoaderListener는 해당 웹 어플리케이션 구동 시 같이 동작하므로 해당 프로젝트를 실행하면 가장 먼저 로그를 출력
    - org.springframework.web.servlet.DispatcherServlet
        - 스프링 MVC의 구조에서 가장 핵심적인 역할을 하는 클래스
        - DispatcherServlet
            - root-context.xmㅣ에 정의된 객체(Bean)들은 스프링의 영역(context)안에 생성되고, 객체들 간이 의존성이 처리됨
            - root-context.xml이 처리된 후 스프링 MVC에서 사용하는 DispatcherServlet이라는 서블릿과 관련된 설정이 동작됨
        - servlet-context.xml
            - 내부적으로 웹 관련 처리의 준비작업을 진행하는데 사용하는 파일
            - DispatcherServlet에서 XmlWebApplicationContext를 이용해서 servlet-context.xml을 로딩하고 해석함
            - 이 과정에서 등록된 객체(Bean)들은 기존에 만들어진 객체(Bean)들과 같이 연동됨
------------
- 스프링 MVC의 기본 사상
    - model2 방식에서는 Servlet/JSP 기술을 사용
    - 스프링 MVC는 내부적으로 Servlet/JSP 처리
    - HttpServletRequest/HttpServletResponse 등과 같이 Servlet/JSP API 사용 필요성이 현저히 줄어듬(스프링이 중간 역할을 하여 코드를 작성하지 않고도 원하는 기능 구현 가능)
- 스프링 MVC의 기본 구조
    - DispatcherServlet
        - 사용자의 Request는 Front-Controller인 DispatcherServlet을 통해서 처리
        - 생성된 프로젝트의 web.xml을 보면 모든 Request를 DispatcherServlet이 받도록 처리
    - HandlerMapping / HandlerAdapter
        - HandlerMapping은 Request의 처리를 담당하는 컨트롤러를 찾기 위해서 존재
        - HandlerMapping 인터페이스를 구현한 여러 객체들 중 RequestMappingHandlerMap-ping 같은 경우 개발자가 @RequestMapping 어노테이션이 적용된 것을 기준으로 판단
        - 적절한 컨트롤러가 찾아졌다면 HandlerMapping을 이용하여 해당 컨트롤러를 동작
    - Controller
        - Controller는 개발자가 작성하는 클래스로 실제 Request를 처리하는 로직을 작성
        - View에 전달해야 하는 데이터는 주로 Model이라는 객체에 담아서 전달
        - Controller는 다양한 타입의 결과를 반환하는데 이에 대한 처리는 ViewResolver를 이용
    - ViewResolver
        - ViewResolver는 Controller가 반환한 결과를 어떤 View를 통해서 처리하는 것이 좋을지 해석
        - 가장 흔하게 사용하는 설정은 servlet-context.xml에 정의된 Inter-nalResourceViewResolver
    - View
        - View는 실제로 응답 보내야 하는 데이터를 Jsp 등을 이용해서 생성하는 역할
        - 만들어진 응답은 DispatcherServlet을 통해 전송
-----------------------
>## ch06. 스프링 MVC의 Controller
- 스프링 MVC의 Controller 특징
    - HttpServletRequest, HttpServletResponse를 거의 사용할 필요 없이 필요한 기능 구현
    - 다양한 타입의 파라미터 처리, 다양한 타입의 리턴 타입 사용 가능
    - Get / Post 방식 등 전송 방식에 대한 처리를 어노테이션으로 처리 가능
    - 상속/인터페이스 방식 대신에 어노테이션만으로도 필요한 설정 가능
------------
- @Controller, @RequestMapping
    - @RequestMapping은 현재 클래스의 모든 메서드들의 기본적인 URL경로가 됨<br>
    (/sample/*에 대한 URL은 모두 SampleController에서 처리 됨)
    - SampleController.java
    ```java
    package org.zerock.controller;

    import org.springframework.stereotype.Controller;
    import org.springframework.web.bind.annotation.GetMapping;
    import org.springframework.web.bind.annotation.RequestMapping;
    import org.springframework.web.bind.annotation.RequestMethod;

    import lombok.extern.log4j.Log4j;

    @Controller
    @RequestMapping("/sample/*")
    @Log4j
    public class SampleController {
        
        @RequestMapping("")
        public void basic() {
            
            log.info("basic get...................");

        }

    }
    ```
    - SampleController 클래스는 스프링의 객체(Bean)으로 자동 등록<br>
    (servlet-context.xml에 설정해두었기 때문)
    ```xml
    <context:component-scan base-package="org.zerock.controller" />
    ```
------------
- @RequestMapping의 변화
    - @RequestMapping의 경우 몇 가지의 속성 추가 가능
    - 가장 많이 사용하는 속성은 method 속성
    - Method 속성은 흔히 GET / POST 방식을 구분해서 사용할 때 이용 함
    - 스프링 4.3버젼부터는 @RequestMapping을 줄여서 사용할 수 있는 @GetMapping, @PostMapping 축약형 표현 가능
    - 일반적인 경우에는 GET / POST 방식만을 사용하지만 최근에는 PUT, DELETE 방식 등도 점점 많이 사용 중
    - SampleController.java
    ```java
    @RequestMapping(value = "/basic", method = { RequestMethod.GET, RequestMethod.POST })
    public void basicGet() {
        log.info("basic get...................");
    }

    @GetMapping("/basicOnlyGet")
    public void basicGet2() {
        log.info("basic get only get...................");
    }
    ```
------------
- Lombok의 @Data 어노테이션을 이용하여 처리<br>
(getter/setter, equals(), toStirng()등의 메서드를 자동으로 생성)
- SampleDTO.java
```java
package org.zerock.domain;

import lombok.Data;

@Data
public class SampleDTO {
    private String name;
    private int age;
}
```
-----------
- Controller의 파라미터 수집
    - Controller를 작성할 때 가장 편리한 기능은 파라미터가 자동으로 수집되는 기능
    - 이 기능을 통해 매번 request.getParameter()를 이용하는 불편함을 없앨 수 있음
    - SampleController의 경로가 '/sample/*'이므로 ex01을 호출하는 경로는 '/sample/ex01'
    - 필요한 파라미터를 URL뒤에 '?name=AAA&age=10'과 같은 형태로 추가해서 호출
     - SampleController.java
    ```java
    package org.zerock.controller;

    import org.springframework.stereotype.Controller;
    import org.springframework.web.bind.annotation.GetMapping;
    import org.springframework.web.bind.annotation.RequestMapping;
    import org.springframework.web.bind.annotation.RequestMethod;
    import org.zerock.domain.SampleDTO;

    import lombok.extern.log4j.Log4j;

    @Controller
    @RequestMapping("/sample/*")
    @Log4j
    public class SampleController {
        
        @RequestMapping("")
        public void basic() {
            log.info("basic get...................");
        }
        
        @RequestMapping(value = "/basic", method = { RequestMethod.GET, RequestMethod.POST })
        public void basicGet() {
            log.info("basic get...................");
        }

        @GetMapping("/basicOnlyGet")
        public void basicGet2() {
            log.info("basic get only get...................");
        }
        
        @GetMapping("/ex01")
        public String ex01(SampleDTO dto) {
            log.info("" + dto);
            
            return "ex01";
        }
    }

    ```
- 만일 기본 자료형이나 문자열 등을 이용한다면 파라미터의 타입만을 맞게 선언해주는 방식을 사용할 수 있음
    - SampleController.java
    ```java
    @GetMapping("/ex02")
	public String ex02(@RequestParam("name") String name, @RequestParam("age") int age) {
		log.info("name: " + name);
		log.info("age: " + age);
		return "ex02";
	}
    ```
-------------------
- List
    - 스프링은 파라미터의 타입을 보고 객체를 생성하므로 파라미터의 타입은 List<>와 같이 인터페이스 타입이 아닌 실제적인 클래스 타입으로 지정함
    - 같은 이름의 파마리터가 여러 개 전달되더라도 ArrayList\<String>이 생성되어 자동으로 수집<br>
    (호출 예시 /sample/ex02List?ids=111&ids=222&ids=333)
     - SampleController.java
    ```java
    @GetMapping("/ex02List")
	public String ex02List(@RequestParam("ids") ArrayList<String> ids) {
		log.info("ids: " + ids);
		return "ex02List";
	}
    ```
- 배열
    - 배열도 List와 동일하게 처리
    - SampleController.java
    ```java
    @GetMapping("/ex02Array")
	public String ex02Array(@RequestParam("ids") String[] ids) {
		log.info("array ids: " + Arrays.deepToString(ids));
		return "ex02Array";
	}
    ```
-----------
- 객체 리스트
    - 전달하는 데이터가 SampleDTO와 같이 객체 타입이고, 여러 개를 처리해야할 때 유용
    - SampleDTOList.java
    ```java
    package org.zerock.domain;

    import java.util.ArrayList;
    import java.util.List;

    import lombok.Data;

    @Data
    public class SampleDTOList {
        private List<SampleDTO> list;
        
        public SampleDTOList() {
            list = new ArrayList<>();
        }
    }
    ```
    - SampleController.java
    ```java
    @GetMapping("/ex02Bean")
	public String ex02Bean(SampleDTOList list) {
		log.info("list dtos: " + list);
		
		return "ex02Bean";
	}
    ```
    - 파라미터는 [인덱스] 형식으로 전달해서 처리<br>
    (호출 예시 : /sample/ex02Bean?list[0].name=aaa&list[1].name=bbb)
    - 톰캣 버젼에 따라 [] 특수문자를 허용하지 않을 수 있음
    - JavaScript를 이용하는 경우 encodeURIComponent()와 같은 방법으로 해결할 수 있음
    - 임시방편으로 %5B, %5D로 처리 가능
------------
- 바인딩
    - 파라미터의 수집을 다른 용어로는 Binding이라고 함
    - 변환이 가능한 데이터는 자동으로 변환되지만 경우에 따라서 파라미터를 변환해서 처리해야하는 경우가 존재함<br>
    ex) 2018-01-01과 같이 문자열로 전달된 데이터를 java.util.Date 타입으로 변환하는 작업
- @initBinder
    - TodoDTO.java
    ```java
    package org.zerock.domain;

    import java.util.Date;

    import lombok.Data;

    @Data
    public class TodoDTO {
        private String title;
        private Date dueDate;
    }
    ```
    - SampleController.java
    ```java
    @InitBinder
	public void initBinder(WebDataBinder binder) {
		SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-mm-dd");
		binder.registerCustomEditor(java.util.Date.class, new CustomDateEditor(dateFormat, false));
	}
	
	@GetMapping("/ex03")
	public String ex03(TodoDTO todo) {
		log.info("todo: " + todo);
		
		return "ex03";
	}
    ```
    - 호출예시 : /sample/ex03?title=test&dueData=2018-01-01

- @DateTimeFormat
    - 파라미터로 사용되는 인스턴스 변수에 적용하여 변환
    - DateTimeFormat 사용 시 @InitBinder는 필요하지 않음
    - TodoDTO.java
    ```java
    package org.zerock.domain;

    import java.util.Date;

    import org.springframework.format.annotation.DateTimeFormat;

    import lombok.Data;

    @Data
    public class TodoDTO {
        private String title;
        
        @DateTimeFormat(pattern = "yyyy/MM/dd")
        private Date dueDate;
    }
    ```
    - SampleController.java
    ```java
    @InitBinder
	public void initBinder(WebDataBinder binder) {
		SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-mm-dd");
		binder.registerCustomEditor(java.util.Date.class, new CustomDateEditor(dateFormat, false));
	}
	
	@GetMapping("/ex03")
	public String ex03(TodoDTO todo) {
		log.info("todo: " + todo);
		
		return "ex03";
	}
    ```
--------------
- Model
    - Model 객체는 JSP에 컨트롤러에서 생성된 데이터를 담아서 전달하는 역할
    - JSP와 같은 뷰(View)에 전달해야 하는 데이터를 담아서 전송
    - SampleController.java
    ```java
    @GetMapping("/ex04")
	public String ex04(SampleDTO dto, @ModelAttribute("page") int page) {
		log.info("dto: " + dto);
		log.info("page: " + page);
		
		return "ex04";
	}
    ```
    - ex04.jsp 페이지에 전달
---------
- RedirectAttributes
    - Model 타입과 더불어서 스프링 MVC가 자동으로 전달해주는 타입 중 하나
    - 일회성으로 데이터를 전달하는 용도
    - 기존 Servlet에서는 response.sendRedirect()와 동일
        - Servlet
        ```java
        response.sendRedirect("/home?name=aaa&age=10");
        ```
        - Spring MVC
        ```java
        rttr.addFlashAttribute("name", "AAA");
            rttr.addFlashAttribute("age", 10);

            return "redirect:/"
        ```
---------
- Controller의 리턴 타입
    - 스프링 MVC의 구조가 기존의 상속과 인터페이스에서 어노테이션을 사용하는 방식으로 변한 이후에 가장 큰 변화 중 하나는 리턴 타입이 자유로워짐
    - 리턴 타입의 종류
        - String
            - jsp를 이용하는 경우에는 jsp 파일의 경로와 파일이름을 나타내기 위해서 사용
            - 키워드를 붙여서 사용 가능
                - redirect : 리다이렉트 방식으로 처리하는 경우
                - forward : 포워드 방식으로 처리하는 경우
        - void
            - 호출하는 URL과 동일한 이름의 jsp를 의미
        - VO, DTO 타입
            - 주로 JSON 타입의 데이터를 만들어서 반환하는 용도로 사용
            - jackson-databind 라이브러리 필요 (스프링3 이전 버전은 별도의 Converter 작성 필요)
                - pom.xml
                ```xml
                <!-- https://mvnrepository.com/artifact/com.fasterxml.jackson.core/jackson-databind -->
                <dependency>
                    <groupId>com.fasterxml.jackson.core</groupId>
                    <artifactId>jackson-databind</artifactId>
                    <version>2.9.4</version>
                </dependency>
                ```
        - ResponseEntity 타입
            - response할 때 Http 헤더 정보와 내용을 가공하는 용도로 사용
            - HttpServletRequest나 HttpServletResponse를 직접 핸들링하지않고 ResponseEntity를 통해 처리 가능
                - SampleController.java
                ```java
                @GetMapping("/ex07")
                public ResponseEntity<String> ex07() {
                    // {"name" : "홍길동"}
                    String msg = "{\"name\" : \"홍길동\"}";
                    
                    HttpHeaders header = new HttpHeaders();
                    header.add("Content-Type", "application/json);charset=UTF-8");
                    
                    return new ResponseEntity<>(msg, header, HttpStatus.OK);
                }
                ```
        - Model, ModelAndView
            - Model로 데이터를 반환하거나 화면까지 같이 지정하는 경우 사용 (최근에는 많이 사용하지 않음)
        - HttpHeaders
            - 응답에 내용 없이 Http 헤더 메시지만 전달하는 용도로 사용
----------
- 파일업로드 처리
    - pom.xml
    ```xml
    <!-- https://mvnrepository.com/artifact/commons-fileupload/commons-fileupload -->
    <dependency>
        <groupId>commons-fileupload</groupId>
        <artifactId>commons-fileupload</artifactId>
        <version>1.3.3</version>
    </dependency>
    ```
    ---
    - servlet-context.xml은 스프링 MVC의 특정한 객체(Bean)를 설정함
    - 다른 객체(Bean)를 설정하는 것과 달리 파일 업로드의 경우, 반드시 id 속성의 값을 'multipartResolver'로 정확하게 지정해야함
        - maxUploadSize : 한 번의 Request로 전달될 수 있는 최대의 크기
        - maxUploadSizePerFile : 하나의 파일 최대 크기
        - maxInMemorySize : 메모리상에서 유지하는 최대의 크기
        - uploadTempDir : 지정된 크기보다 큰 파일을 업로드할 경우, 임시저장 경로
        - defaultEncoding : 업로드 파일명이 한글일 경우 깨지는 문제 처리
    - servlet-context.xml
    ```xml
    <beans:bean id="multipartResolver"
		class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
		<beans:property name="defaultEncoding" value="utf-8"></beans:property>
		<!-- 1024 * 1024 * 10 bytes 10MB -->
		<beans:property name="maxUploadSize" value="104857560"></beans:property>
		<!-- 1024 * 1024 * 2 bytes 2MB -->
		<beans:property name="maxUploadSizePerFile"
			value="2097152"></beans:property>
		<beans:property name="uploadTempDir"
			value="file://Users/tongbook/upload/tmp"></beans:property>
		<beans:property name="maxInMemorySize" value="10485756"></beans:property>
	</beans:bean>
    ```
    - exUpload 페이지 이동
    - SampleController.java
    ```java
    @GetMapping("/exUpload")
	public void exUpload() {
		log.info("/exUpload....");
	}
    ```
    - exUpload.jsp
    ```jsp
    <%@ page language="java" contentType="text/html; charset=UTF-8"
	pageEncoding="UTF-8"%>
    <html>
    <head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
    <title>Insert title here</title>
    </head>
    <body>
        <form action="/sample/exUploadPost" method="post"
            enctype="multiPART/form-data">

            <div>
                <input type='file' name='files'>
            </div>
            <div>
                <input type='file' name='files'>
            </div>
            <div>
                <input type='file' name='files'>
            </div>
            <div>
                <input type='file' name='files'>
            </div>
            <div>
                <input type='file' name='files'>
            </div>
            <div>
                <input type='submit'>
            </div>
        </form>
    </body>
    </html>
    ```
    - 업로드한 파일 정보 확인
    - SampleController.java
    ```java
    @PostMapping("/exUploadPost")
	public void exUploadPost(ArrayList<MultipartFile> files) {
		log.info("/sample/exUploadPost....");
		
		files.forEach(file -> {
			log.info("--------------------");
			log.info("name: " + file.getOriginalFilename());
			log.info("size: " + file.getSize());
		});
	}
    ```
---------
- Controller의 Exception 처리
    - @ExceptionHandler와 @ControllerAdvice를 이용한 처리
    - @ResponseEntity를 이용하는 예외 메시지 구성
- @ControllerAdvice
    - AOP(Aspect-Oriented-Programming)을 이용한 방식
        - CommonExceptionAdvice.java
        ```java
        package org.zerock.exception;

        import org.springframework.ui.Model;
        import org.springframework.web.bind.annotation.ControllerAdvice;
        import org.springframework.web.bind.annotation.ExceptionHandler;

        import lombok.extern.log4j.Log4j;

        @ControllerAdvice
        @Log4j
        public class CommonExceptionAdvice {
            @ExceptionHandler(Exception.class)
            public String except(Exception ex, Model model) {
                log.error("Exception......" + ex.getMessage());
                model.addAttribute("exception", ex);
                log.error(model);
                return "error_page";
            }
        }
        ```
        - @ControllerAdvice
            - 해당 객체가 스프링의 컨트롤러에서 발생하는 예외를 처리하는 존재임을 명시하는 용도
        - @ExceptionHandler 
            - 해당 메서드가 () 들어가는 예외 타입을 처리한다는 거을 의미
            - 어노테이션 속성으로는 Exception 클래스 타입 지정 가능<br>
            예제의 경우 Exception.class를 지정하였으므로 모든 예외에 대한 처리가 except()만을 이용해서 처리할 수 있음
            - servlet-context.xml
            ```xml
            <context:component-scan base-package="org.zerock.exception" />
            ```
            - error_page.jsp
            ```jsp
            <%@ page language="java" contentType="text/html; charset=UTF-8"
            pageEncoding="UTF-8"%>
            <%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>
            <%@ page session="false" import="java.util.*"%>
            <!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
            <html>
            <head>
            <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
            <title>Insert title here</title>
            </head>
            <body>

            <h4><c:out value="${exception.getMessage()}"></c:out></h4>

            <ul>
            <c:forEach items="${exception.getStackTrace() }" var="stack">
                <li><c:out value="${stack}"></c:out></li>
            </c:forEach>
            </ul>

            </body>
            </html>
            ```
- 404 에러 페이지
    - 스프링 MVC의 모든 요청은 DispatcherServlet을 이용해서 처리되므로 404에러도 같이 처리할 수 있도록 web.xml을 수정
    - web.xml
    ```xml
    <!-- Processes application requests -->
    <servlet>
        <servlet-name>appServlet</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>/WEB-INF/spring/appServlet/servlet-context.xml</param-value>
        </init-param>
        
        <!-- 404에러 페이지 설정 -->
        <init-param>
            <param-name>throwExceptionIfNoHandlerFound</param-name>
            <param-value>true</param-value>
        </init-param>
        
        <load-on-startup>1</load-on-startup>
    </servlet>
    ```
    - CommonExceptionAdvice.java
    ```java
    @ExceptionHandler(NoHandlerFoundException.class)
	@ResponseStatus(HttpStatus.NOT_FOUND)
	public String handle404(NoHandlerFoundException ex) {
		return "custom404";
	}
    ```
    - custom404.jsp
    ```jsp
    <%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
    <!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
    <html>
    <head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
    <title>Insert title here</title>
    </head>
    <body>
    <h1>해당 URL은 존재하지 않습니다.</h1>
    </body>
    </html>
    ```