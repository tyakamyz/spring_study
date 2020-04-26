
[Part7] - chapter 37
=========================

어노테이션을 이용한 스프링 시큐리티
-------------------------

@Secured   
() 안에 'ROLE_ADMIN' 과 같은 문자열 혹은 문자열 배열을 이용한다.

@PreAuthorize, @PostAuthorize : 3버전부터 지원. ()안에 표현식을 사용할 수 있어 최근 많이 사용된다.

1. servlet-context.xml 관련 설정 추가

<img src="https://user-images.githubusercontent.com/22673024/80308331-604d8e00-8809-11ea-8ec8-cb16e63fe3a2.png" width=60%>

네임스페이스 추가 시 5.0 버전으로 추가되는 사항은 버전을 없애준다. 


```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans:beans xmlns="http://www.springframework.org/schema/mvc"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:beans="http://www.springframework.org/schema/beans"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:security="http://www.springframework.org/schema/security"
	xsi:schemaLocation="http://www.springframework.org/schema/security http://www.springframework.org/schema/security/spring-security.xsd"
    ... >
...

<security:global-method-security pre-post-annotations="enabled" secured-annotations="enabled" />
```
global-method-security 은 기본적으로 disabled 되어있으므로 enabled 로 변경한다.

2. 어노테이션 사용 
```java
/**
* 스프링 시큐리티 어노테이션
*/
@PreAuthorize("hasAnyRole('ROLE_ADMIN', 'ROLE_MEMBER')")
@GetMapping("/annoMember")
public void deMember2() {
    log.info("logined annotation member");
}

@Secured({"ROLE_ADMIN"})
@GetMapping("/annoAdmin")
public void daAdmin2() {
    log.info("admin annotation only");
}
```


### 자바 설정 시 
ServletConfig.java

```java
package org.zerock.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.security.config.annotation.method.configuration.EnableGlobalMethodSecurity;
import org.springframework.web.multipart.MultipartResolver;
import org.springframework.web.multipart.support.StandardMultipartHttpServletRequest;
import org.springframework.web.multipart.support.StandardServletMultipartResolver;
import org.springframework.web.servlet.config.annotation.EnableWebMvc;
import org.springframework.web.servlet.config.annotation.ResourceHandlerRegistry;
import org.springframework.web.servlet.config.annotation.ViewResolverRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;
import org.springframework.web.servlet.view.InternalResourceViewResolver;
import org.springframework.web.servlet.view.JstlView;

@EnableWebMvc
@ComponentScan(basePackages = {"org.zerock.controller"})
@EnableGlobalMethodSecurity(prePostEnabled = true, securedEnabled = true)
public class ServletConfig implements WebMvcConfigurer {

    ...
    
```
