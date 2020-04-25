
[Part7] - chapter 30,31
=========================

Spring Web Security 설정
-----------------

```xml
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-web</artifactId>
    <version>5.0.6.RELEASE</version>
</dependency>

<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-config</artifactId>
    <version>5.0.6.RELEASE</version>
</dependency>

<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-core</artifactId>
    <version>5.0.6.RELEASE</version>
</dependency>

<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-taglibs</artifactId>
    <version>5.0.6.RELEASE</version>
</dependency>
```

spring-security.xml 생성
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:security="http://www.springframework.org/schema/security"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/security http://www.springframework.org/schema/security/spring-security.xsd">

</beans>
```
*<U>시큐리티5 버전에서 5.0 네임스페이스 버그가 있기 때문에 spring-security-5.0.xsd 부분을 spring-security.xsd로 변경해준다.</U>*

web.xml 추가 
```xml
<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>
        /WEB-INF/spring/root-context.xml
        /WEB-INF/spring/security-context.xml    
    </param-value>
</context-param>

...

<filter>
    <filter-name>springSecurityFilterChain</filter-name>
    <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
</filter>

<filter-mapping>
    <filter-name>springSecurityFilterChain</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```
security-context.xml 파일 수정 
```xml
<security:http>
	<security:form-login/>
</security:http>

<security:authentication-manager>

</security:authentication-manager>
```
1) context-param 에 security-context.ml 파일을 로딩하도록 설정하는 작업 
2) filter 스프링 동작에 관여하도록 설정 
3) 스프링 시큐리티 동작을 위한 authentication-manager 
4) 스프링 시큐리티의 시작지점 선정 

<hr>

Spring Web Security 활용(로그인, 로그아웃)
-----------------

securiy-context.xml
```xml
<security:http>
	<security:intercept-url pattern="/sample/all" access="permitAll"/>
	<security:intercept-url pattern="/sample/member" access="hasRole('ROLE_MEMBER')"/>	
	<security:form-login/>
</security:http>
```
* ```<security:intercept-url>```   
    - 특정 URL 에 접근 시 인터셉터를 이용해 접근을 제한한다.
    - pattern 속성은 URI 패턴을 의미
    - access 속성은 권한을 체크한다.    
        + 속성값으로 사용되는 문자열 → 1) 표현식 2) 권한명 
        + ```<security:http>```는 표현식을 이용하는게 기본 설정값이다. 
        + 단순 문자열을 이용하고 싶을 경우 <U>use-expressions=false</U> 지정한다. (그러나 표현식 사용이 권장되는 사용법이다.)
        > ```xml 
        > <security:http auto-config="true" use-expressions="false">
        > <security:intercept-url pattern="/sample/member" access="ROLE_MEMBER">
        > <security:intercept-url pattern="/sample/admin" access="ROLE_ADMIN">
        > <security:intercept-url pattern="/sample/puser" access="ROLE_MANAGER">
        > ```
        
### 단순 로그인 처리

**◆ Point**   
일반 시스템 userid == 스프링 시큐리티 username 
User : 인증정보와 권한을 가진 객체   

security-context.xml 의 UserDetailsService 처리 
```xml
<security:authentication-manager>
	<security:authentication-provider>
		<security:user-service>
			<security:user name="member" password="{noop}member" authorities="ROLE_MEMBER"/>
			<security:user name="admin" password="{noop}admin" authorities="ROLE_MEMBER, ROLE_ADMIN"/>
		</security:user-service>
	</security:authentication-provider>
</security:authentication-manager>
```                
* password의 {noop} 처리는 PasswordEncoder 의 지정이 반드시 필요하기 때문에 패스워드의 인코딩 처리 없이 사용을 위하여 {noot}문자열을 추가한다.
* 스프링 시큐리티 5 부터는 반드시 PasswordEncoder를 이용해야하고, 임시로 포맷팅 처리를 지정 하여 사용할 수 있다.(https://spring.io/blog/2017/11/01/spring-security-5-0-0-rc1-released#password-storage-format)


### 로그아웃 
tomcat 의 JSESSIONID 쿠키를 이용하여 세션을 유지시킨다. 


### 접근 제한
1. 특정 url 이용   
security-context.xml    
```<security:access-denied-handler error-page="/accessError"/>```   
error 페이지를 직접 처리한다. 

2. AccessDeninedHandler 인터페이스 구현   
```java
package org.zerock.security;

import java.io.IOException;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.springframework.security.access.AccessDeniedException;
import org.springframework.security.web.access.AccessDeniedHandler;

import lombok.extern.log4j.Log4j;

/**
 * AccessDeniedHandler 커스터마이징 
 * @author Sunny
 *
 */
@Log4j
public class CustomeAccessDeniedHandler implements AccessDeniedHandler {
	
	@Override
	public void handle(HttpServletRequest request, HttpServletResponse response,
			AccessDeniedException accessDeniedException) throws IOException, ServletException {
		
		log.error("Access Denied Handler");
		log.error("Redirect.......");
		
		response.sendRedirect("/accessError");
		
	}
}
```
* 접근 제한이 된 경우 다양한 처리를 하고 싶을 때, 직접 인터페이스를 구현하는 편이 더 좋다.
* 쿠키나 세션에 특정한 작업을 할 경우 
* 특정한 헤더 정보를 추가하는 등 행위 
* 접근 제한이 걸리는 경우 리다이렉트 처리 하여 동작한다. 

security-context.xml
```xml 
<bean id="customAccessDenied" class="org.zerock.security.CustomeAccessDeniedHandler"></bean>

<security:http auto-config="true" use-expressions="true">
	<security:intercept-url pattern="/sample/all" access="permitAll"/>
	<security:intercept-url pattern="/sample/member" access="hasRole('ROLE_MEMBER')"/>
	<security:intercept-url pattern="/sample/admin" access="hasRole('ROLE_ADMIN')"/>
		
	<security:form-login/>
	
	<!-- <security:access-denied-handler error-page="/accessError"/> -->
	<security:access-denied-handler ref="customAccessDenied"/>
</security:http>
```
이 경우, hocalhost:8080/accessError 와 같이 리다이렉트 되는 것을 확인할 수 있다. 


### 커스텀 로그인 페이지 
security-context.xml
```xml
<!-- <security:form-login/> -->
<security:form-login login-page="/customLogin" />
```
GET 방식으로 접근하는 URI 를 지정한다.    
Controller 처리 
```java
@GetMapping("/customLogin")
public void loginInput(String error, String logout, Model model) {
    log.info("error: " + error);
    log.info("logout : " + logout);
    
    if(error != null) {
        model.addAttribute("error", "Login Error Check Your Account");
    }
    
    if(logout != null) {
        model.addAttribute("logout", "Logout!!");
    }
}
```
로그인화면 jsp 생성
```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"  pageEncoding="UTF-8"%>
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>

<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>Insert title here</title>
</head>
<body>
	<h1>Custom Login Page</h1>
	<h2><c:out value="${error}" /></h2>
	<h2><c:out value="${logout}" /></h2>
	
	<form method="post" action="/login">
		
		<div>
			<input type="text" name="username" value="admin">
		</div>
		<div>
			<input type="password" name="password" value="admin">
		</div>
		<div>
			<input type="submit">
		</div>
	
		<input type="hidden" name="${_csrf.parameterName}" value="${_csrf.token }" />
	</form>	
	
</body>
</html>
```
action="/login"
POST 방식으로 데이터 전송
기본값은 username, password
_csrf


### CSRF(Cross-site request forgery)
"사이트간 위조 방지 토큰"   
CSRF 토큰 : 사용자가 임의로 변하는 특정한 토큰값을 서버에서 체크하는 방식
```
<input type="hidden" name="_csrf" value="24e8abe5-ee7f-46de-833e-95d797c47627">
```
```xml
// 비활성화시 security-context.xml
<security:csrf disabled="true" />
```
일반적으로 CSRF 토큰은 세션을 통해 보관한다.

### 로그인 성공 후 처리 
```java
package org.zerock.security;

import java.io.IOException;
import java.util.ArrayList;
import java.util.List;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.springframework.security.core.Authentication;
import org.springframework.security.web.authentication.AuthenticationSuccessHandler;

import lombok.extern.log4j.Log4j;

/**
 * AuthenticationSuccessHandler 커스터마이징
 * 로그인 성공 후 동작추가 
 * @author Sunny
 *
 */
@Log4j
public class CustomLoginSuccessHandler implements AuthenticationSuccessHandler{@Override
	public void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response,
			Authentication auth) throws IOException, ServletException {
		// TODO Auto-generated method stub
		
		log.warn("Login Success");
		
		List<String> roleNames = new ArrayList<>();
		auth.getAuthorities().forEach(authority -> {
			roleNames.add(authority.getAuthority());
		});
		
		log.warn("ROLE NAMES: " + roleNames);
		
		// 로그인 후 이동경로 설정 
		if(roleNames.contains("ROLE_ADMIN")) {
			response.sendRedirect("/sample/admin");
			return;
		}
		
		if(roleNames.contains("ROLE_MEMBER")) {
			response.sendRedirect("/sample/member");
			return;
		}
		
		response.sendRedirect("/");
	}
}
```
security-context.xml
```xml
<bean id="customLoginSuccess" class="org.zerock.security.CustomLoginSuccessHandler"></bean>
...
<security:form-login login-page="/customLogin" authentication-success-handler-ref="customLoginSuccess" />
```
빈으로 등록 후 로그인 성공 후 처리를 담당하는 핸들러로 지정한다.    
권한에 따라 로그인 후 다른 페이지를 호출하는 것을 확인할 수 있다. 


### 로그아웃
security-context.xml
```xml
<security:logout logout-url="/customLogout" invalidate-session="true"/>
```
특정 URI 를 지정하여 로그아웃을 처리한다.   
세션을 무효화 시키거나 특정한 쿠키를 지우는 작업을 처리한다.   

```jsp
<form method="post" action="/customLogout">
		<input type="hidden" name="${_csrf.parameterName}" value="${_csrf.token }" />
		<button>로그아웃</button>
	</form>	
```
로그아웃 처리 시 action="/customLogout" 으로 처리하고 POST 방식으로 처리한다.    
POST 방식이기 때문에 CRSF 토큰값을 같이 hidden 값으로 지정한다. 



