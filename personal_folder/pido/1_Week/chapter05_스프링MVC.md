
[Part1] - chapter 05
=========================

스프링 MVC 
--------
- 스프링 서브(sub) 프로젝트
- root-context.xml 로 사용하는 JAVA영역(POJO)과 servlet-context.xml 로 설정하는 Web 관련 영역을 연동하여 구동하는 방식. 

#### pom.xml 변경 
> - java-version(1.6 → 1.8)   
> - springframework-version(3.1.1 → 5.0.7)   
> - spring-test(springVersion) 와 lombok(1.18.0) 라이브러리 추가
> - javax.servlet-api(2.5 → 3.1.0) 서블릿 버전  
> - maven-compiler(1.6 → 1.8) 컴파일 옵션  
*project - Maven - update project *   

#### JAVA 기반 설정   
> - web.xml, servlet-context.xml, root-context.xml 제거   
> - pom.xml 에 web.xml 미사용 설정 추가 
> - servlet-context.xml을 대신할 RootConfig.java, WebConfig.java, servletConfig.java(Spring MVC 이용시) 클래스 생성   
>	* @EnableWebMvc 어노테이션과 WebMvcConfigurer 사용하여 스프링 MVC 설정
>	* 	<details markdown="1">
> 		<summary>ServletConfig 클래스</summary>
>	
>		```java   
>		@EnableWebMvc
>		@ComponentScan(basePackages = {"org.zerock.controller"})
>		 public class ServletConfig implements WebMvcConfigurer {
> 		
>		 	@Override
> 			public void configureViewResolvers(ViewResolverRegistry registry) {
> 		
> 			InternalResourceViewResolver bean = new  InternalResourceViewResolver();
> 			bean.setViewClass(JstlView.class);
> 			bean.setPrefix("/WEB-INF/views/");
>			bean.setSuffix(".jsp");
>			registry.viewResolver(bean);
>			}
>			
>			@Override
>			public void addResourceHandlers(ResourceHandlerRegistry registry) { 
>				registry.addResourceHandler("/resources/**").addResourceLocations("/resources/");
>			}
>		}	
>		``` 
>	</details>	
> 
>		
>	* ServletConfig 클래스의 정상적 실행을 위한 WebConfig 설정	
>	*	```java   
>		// 기본 경로 변경
>		protected String[] getServletMappings() {
>			return new String[] { "/" };
>		}
>		``` 
> -	[☝ 관련 설정 소스 참고](https://github.com/tyakamyz/spring_study/blob/master/personal_folder/pido/1_Week/chapter01_%EA%B0%9C%EB%B0%9C%ED%99%98%EA%B2%BD%EC%84%A4%EC%A0%95.md)

### 프로젝트 로딩 구조
#### - 설정
* 스프링 : root-context.xml, servlet-context.xml
* 톰캣 : web.xml

#### - 구동(XML 기반)
* 1．web.xml dml Context Listener (스프링MVC의 ContextLoaderListener) 동작
* 2．root-context.xml 처리 후 정의된 Bean 동작
* 3．DispatcherServlet 설정 동작(내부적으로 웹 관련 처리의 준비작업을 진행하는데 사용)
* 4．XmlWebApplicationContext를 이용해 servlet-context.xml 로딩 후 등록된 Bean들과 기존 Bean이 같이 연동 

<hr />

스프링 MVC 모델2
--------
> 로직과 화면을 분리   
> Request, Response, Controller, View 로 구성

##### DispatcherServlet
* 사용자의 Request를 스프링 MVC API 인 Front-Controller DispatcherServlet을 통해 처리
##### HandlerMapping
* Request의 처리를 담당하는 Controller 를 찾기 위해 존재.   
RequestMappingHandlerMapping 같은 경우 @RequestMapping 어노테이션이 적용된 것을 기준으로 판단. 
##### HandlerAdapter
* 해당 컨트롤러를 동작
##### ViewResolver
* 개발자가 작성한 Request를 처리하는 로직을 View에 전달, 전달하는 데이터는 Model이란 객체에 담는다. ViewResolver는 Controller가 다양한 타입의 결과를 반환할 때 어떤 View 를 통해 처리할지 해석하는 역할. 
* servlet-context.xml 에 정의된 InternalResourceViewResolver


