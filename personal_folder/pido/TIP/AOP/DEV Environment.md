AOP 사용 방법
=======================================
### pom.xml
```xml
<properties>
    <java-version>1.8</java-version>
    <org.springframework-version>5.0.7.RELEASE</org.springframework-version>
    <org.aspectj-version>1.9.0</org.aspectj-version>
    <org.slf4j-version>1.7.25</org.slf4j-version>
</properties>
```
> AOP는 AspectJ 라이브러리의 도움을 받기 때문에 스프링버전에 맞춰 올려준다.

```xml
<!-- aspectj 추가 -->
<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjrt</artifactId>
    <version>${org.aspectj-version}</version>
</dependency>

<!-- https://mvnrepository.com/artifact/org.aspectj/aspectjweaver -->
<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjweaver</artifactId>
    <version>${org.aspectj-version}</version>
</dependency>
```
> 스프링은 AOP 처리가 된 객체를 생성할 때 AspectJ Weaver 라이브러리의 도움을 받아 동작한다.
--------------------------------------------------
### Advice 작성    

```java
package org.zerock.aop;

import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.springframework.stereotype.Component;

import lombok.extern.log4j.Log4j;

@Aspect
@Log4j
@Component
public class LogAdvice {

	@Before( "execution(* org.zerock.service.SampleService*.*(..))")
	public void logBefore() {
		log.info("==============================");
	}
}
```

------------------------------------
### root-context.xml 

<img src="https://user-images.githubusercontent.com/22673024/78792542-32d0aa00-79ec-11ea-9c73-fea13298d88e.png" width="50%">

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:aop="http://www.springframework.org/schema/aop"
	xmlns:context="http://www.springframework.org/schema/context"
	xsi:schemaLocation="http://www.springframework.org/schema/beans https://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.3.xsd
		http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-4.3.xsd">
	
	<!-- Root Context: defines shared resources visible to all other web components -->

<context:annotation-config></context:annotation-config>

<context:component-scan base-package="org.zerock.service"></context:component-scan>
<context:component-scan base-package="org.zerock.aop"></context:component-scan>

<aop:aspectj-autoproxy></aop:aspectj-autoproxy>
		
</beans>
```   

* ```<component-scan>```으로 service와 aop 패키지를 스캔한다.
* 위 과정에서 SampleServiceImpl 클래스와 LogAdvice는 스프링의 빈으로 등록된다.
* ```<aop:aspectj-autoproxy>```를 이용하여 LogAdvice에 설정한 @Before가 동작한다.   
<img src="https://user-images.githubusercontent.com/22673024/78793440-6233e680-79ed-11ea-96e0-9cb7a7f2e4cc.png" width="50%">

* SampleServiceImpl 클래스에 아이콘이 추가된 것을 확인할 수 있다.
