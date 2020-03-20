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