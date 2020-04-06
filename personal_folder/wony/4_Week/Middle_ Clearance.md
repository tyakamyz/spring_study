# 중간 정리

## Part_01 - Chapter_02

+ ### 1. 스프링의 주요 특징
	- POJO 기반의 구성
 	- 의존성 주입(DI)을 통한 객체 간의 관계 구성
 	- AOP(Aspect-Oriented-Programming) 지원
 	- 편리한 MVC 구조
 	- WAS의 종속적이지 않은 개발 환경
 + ### 2. Spring이 동작하며 생기는 일
 	- Spring FrameWork가 시작 되면 Spring이 사용하는 메모리 영역을 만듦 
        - **Context** 라  하며 **ApplicationContext**라는 이름의 객체가 만들어진다.
 	- Spring은 자신이 객체를 생성하고 관리해야 하는 객체들에 대한 설정이 필요 
        - **root-context.xml**
 	- root-context.xml에 설정되어있는 **<<context:component-scan>>** 태그의 내용을 통해서 **org.zerock.sample** 패키지를 스캔한다.
 	- 해당 패키지에 있는 클래스들 중 Spring이 사용하는 @Component라는 어노테이션이 존재하는 클래스의 인스턴스를 생성
 	- Restaurant 객체는 Chef 객체가 필요하다는 어노테이션 설정이 있으므로 , Spring은 Chef 객체의 레퍼런스를 Restaurnat 객체에 주입

## Part_02 - Chapter_05

-  ### 1. 예제에서 사용 하는 구조  

|       |XML설정|JAVA설정
|------|-------|------|
|Spring MVC|servlet-context.xml|ServletConfig.class
|Spring Core|root-context.xml|RootConfig.class
|MyBatis|root-context.xml|RootConfig.class

-  ### 2. 스프링 MVC 프로젝트 내부 구조
    - 스프링 MVC 프로젝트를 구성해서 사용한다는 의미는 내부적으로 root-context.xml로 사용하는 일반 Java 영역(POJO)과 servlet-context.xml로 설정하는 Web 관련 영역을 같이 연동해서 구동한다.

- ### 3. ServletConfig.class
    - @EnableWebMvc 어노테이션과 WebMvcConfigurer 인터페이스를 구현하는 방식(스프링 5.0 이전은 WebMvcConfigurerAdapter 추상클래스 사용, 이후는 더이상 사용하지 않음) 
    - @Configuration과 WebMvcConfigurationSupport 클래스를 상속하는 방식  
        (일반 @Configuratino 우선순위가 구분되지 않는 경우에 사용)
    - 예제는 @EnableWebMvc 사용
- ### 4. WebMvcConfigurer
    - 스프링 MVC와 관련된 설정을 메서드로 오버라이드 하는 형태를 이용할 때 사용
    - ServletConfig 클래스 역시 @ComponenttScan을 이용해서 다른 패키지에 작성된 스프링의 객체(Bean)를 인식 할 수 있다.
- ### 5. WebConfig.class
    - 작성된 ServletConfig 클래스를 정상적으로 실행하려면 WebConfig의 설정은 아래와 같이 ServletConfig를 이용하고, 스프링 MVC의 기본경로도 "/"로 변경되어야 한다.

    ```java
    @Override
	protected Class<?>[] getServletConfigClasses() {
		// TODO Auto-generated method stub
		return new Class[] {ServletConfig.class};
	}

	@Override
	protected String[] getServletMappings() {
		// TODO Auto-generated method stub
		return new String[] {"/"};
	}
    ```
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

## Part_02 - Chapter_06

- ### 1. Controller의 리턴 타입

  |타입|설명|
  |---|---|
  |String| jsp를 이용하는 경우에는 jsp 파일의 경로와 파일이름으르 나타내기 위해서 사용
  |void|호출하는 URL과 동일한 이름의 jsp를 의미
  |VO, DTO|주로 JSON 타입의 데이터를 만들어서 반환하는 용도로 사용
  |ResponseEntity|response 할 때 Http 헤더 정보와 내용을 가공하는 용도로  사용
  |Model, ModelAndView| Model로 데이터를 반환하거나 화면까지 같이 지정하는 경우에 사용<br/>(최근에는 많이 사용하지않는다.)
  |HttpHeaders|응답에 내용 없이 Http 헤더 메시지만 전달하는 용도로 사용