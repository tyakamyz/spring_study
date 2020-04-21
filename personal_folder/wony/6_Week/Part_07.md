# **Part 07** Spring Web Security를 이용한 로그인 처리

## **Chapter 30** Spring Web Security소개

 - 스프링 시큐리티의 기본 동작 방식
    - 서블릿의 여러 종류의 필터와 인터셉터를 이용해 처리
        - 필터 : 서블릿에서 말하는 단순한 필터
        - 인터셉터(Interceptor) : 스프링에서 필터와 유사한 역할.
 
            |구분|공통점|차이점
            |---|----|----
            |필터| 특정한 서블릿이나 컨트롤러 접근에 관여|스프링과 무관한 서블릿 자원
            |인터셉터| |스프링의 빈으로 관리되며 스프링 컨텍스트에 속한다. </br>따라서 컨텍스트 내의 모든 자원 사용 가능

### 30.1 Spring Web Security 설정

 - spring-security-web, spring-security-config, spring-security-core를 같은버전으로 dependency 추가한다.
 - Spring Securtiy TagLib 사용을 위해 spring-security-taglibs또한 추가한다.

### 30.1. security-context.xml 생성
 - 스프링 시큐리티는 단독으로 설정할 수 있기에 별도로 작성하는게 좋다.(/WEB-INF/spring/security-context.xml)

 - web.xml 설정
 
 ```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app version="2.5" xmlns="http://java.sun.com/xml/ns/javaee"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://java.sun.com/xml/ns/javaee https://java.sun.com/xml/ns/javaee/web-app_2_5.xsd">

	<!-- The definition of the Root Spring Container shared by all Servlets and Filters -->
	<context-param>
		<param-name>contextConfigLocation</param-name>
		<param-value>/WEB-INF/spring/root-context.xml
		/WEB-INF/spring/security-context.xml
		</param-value>
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
	
	<filter>
		<filter-name>springSecurityFilterChain</filter-name>
		<filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
	</filter>
	
	<filter-mapping>
		<filter-name>springSecurityFilterChain</filter-name>
		<url-pattern>/*</url-pattern>
	</filter-mapping>
</web-app>
 ```

 - security-context.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:security="http://www.springframework.org/schema/security"
	xsi:schemaLocation="http://www.springframework.org/schema/security http://www.springframework.org/schema/security/spring-security.xsd
		http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
		
		<security:http>
			<security:form-login/>
		</security:http>
		
		<security:authentication-manager>
		
		</security:authentication-manager>
</beans>
```
 - 스프링 시큐리티가 동작하기 위해서는 authentication-manager 라는 존재와 스프링 시큐리티의 시작지점이 필요하기 때문에 위와같이 최소한의 설정을 지정한 후에 실행해야한다.


### 30.2 인증(Authentication)과 권한 부여(Authorization - 인가)

 - 인증 : 본인을 증명 하여 자격을 얻는 것
 - 권한 부여 : 다른 사람에 의해 자격이 부여되는 것

#### 30.2.1 AuthenticationManager(인증 매니저)
 - AuthenticationManager(인증 매니저) : 다양한 방식의 인증을 처리할 수 있도록 하는 스프링 시큐리티에서 중요한 존재


 ![AuthenticationManager](../img/AuthenticationManager.png)

 
 - ProviderManager : 인증에 대한 처리를 AuthenticationProvider라는 타입의 객체를 이용해 처리를 위임한다.

![ProviderManager](../img/ProviderManager.png)
 
 - AuthenticationProvider(인증 제공자) 는 실제 인증 작업을 진행한다.
    - 인증된 정보에는 권한에 대한 정보를 같이 전달 하게 되는데 이 처리는 UserDetailsService라는 존재와 관련 있다.
    - UserDetailsService 인터페이스의 구현체는 실제로 사용자의 정보와 사용자가 가진 권한의 정보를 처리해서 반환하게 된다.

![UserDetailsService](../img/UserDetailsService.png)

- 스프링 시큐리티를 커스터마이징 하는방식은 크게 AuthenticationManager를 직접 구현하는 방식과 실제 처리를 담당하는 UserDetailsService를 구현하는 방식으로 나뉜다.
- 대부분의 경우 UserDetailsService를 구현하는 형태를 사용하는 것으로 충분하지만, **새로운 프로토콜** 이나 **인증 구현방식** 을 직접 구현하는 경우 AuthenticationManager 인터페이스를 직접 구현해서 사용한다.

## **Chapter 31** 로그인과 로그아웃 처리