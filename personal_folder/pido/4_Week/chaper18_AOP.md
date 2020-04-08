
[Part5] - chapter 18
=========================

AOP란?
-----------------
- 관점 지향 프로그래밍(Aspect Oriented Programing)
- 관심사의 분리를 추구한다.('관심사' == 주변로직)
- '관심사 + 비즈니스 로직'을 분리해서 별도의 코드로 작성하도록 하고, 실행할 때 이를 결합하는 방식으로 접근한다.
- 스프링은 AOP를 지원한다.   

==> <U>AOP는 **핵심 기능과 공통 기능을 분리** 시켜놓고, 공통 기능을 필요로 하는 핵심 기능들에서 사용하는 방식</U>

<br>

## AOP용어들
> *AOP는 기존의 코드를 수정하지 않고도 관심사들을 엮을 수 있다.*     

==> <U>외부에서 호출은 Proxy 객체를 통해서 Target 객체의 JoinPoint를 호출하는 방식이다.</U>

|용어           | 의미	 
|---------------|----------------------
|Aspect         |공통기능(관심사)
|Advice	        |Aspect의 기능 자체
|Jointpoint	    |Advice를 적용해야 되는 부분(ex : 필드, 메소드 / 스프링에서는 메소드만 해당)
|Pointcut	    |Jointpoint의 부분으로 실제로 Advice가 적용된 부분
|Weaving        |Advice를 핵심기능에 적용하는 행위
###### 참고 : https://hongku.tistory.com/114


### **타깃(Target)**   
    순수한 비즈니스 로직으로 순수한 코어(core). 어떠한 관심사들과도 관계를 맺지 않는다.    
### **프록시(Proxy)**    
    Target을 전체적으로 감싸고 있는 존재. 내부적으로 Target을 호출.    
    중간에 필요한 관심사들을 거쳐 Target을 호출하도록 자동 혹은 수동으로 작성된다.   
    이때 대부분의 경우 스프링 AOP 기능을 이용, 자동으로 생성되는(auto-proxy) 방식을 이용한다.
### **조인포인트(JoinPoint)**   
    Target의 객체가 가진 메서드.  
### **포인트커트(PointCut)**   
    어떤 메서드(JointPoint)에 관심사(Advice)를 결합할 것인지의 결정. 관심사와 비즈니스 로직이 결합되는 지점을 결정하는 것.
    
|구분                   | 설명	 
|-----------------------|----------------------
|execuition(@execution) | "메서드" 를 기준으로 PointCut을 설정
|within(@within)        | "특정한 타입(클래스)" 을 기준으로 PointCut을 설정
|this           	    | "주어진 인터페이스를 구현한 객체" 를 대상으로 PointCut을 설정
|args(@args)    	    | "특정한 파라미터를 가지는" 대상들만을 PointCut으로 설정
|@annotation            | "특정한 어노테이션이 적용된" 대상들만을 PointCut으로 설정

* Target은 결과적으로 PointCut에 의해서 자신에게는 없는 기능들을 가지게 된다.

### **애스팩트(Aspect)**   
    관심사 자체   
### **어드바이스(Advice)**   
    Aspect를 구현한 코드

|정의            |구분                      | 설명	 
|----------------|--------------------------|----------------------
|이전            |Before Advice             |Target의 JoinPoint를 호출하기 전에 실행되는 코드(코드의 실행 자체에는 관여 불가능)
|반환 이후       |After Returning Advice	|모든 실행이 정상적으로 이루어진 후 동작하는 코드
|예외발생 이후   |After Throwing Advice	    |예외가 발생한 뒤 동작하는 코드
|이후            |After Advice   	        |정상적으로 실행되거나 예외가 발생했을 때 구분없이 실행되는 코드
|주위            |Around Advice             |메서드의 실행 자체를 제어할 수 있는 가장 강력한 코드. 직접 대상 메서드를 호출하고 결과나 예외를 처리할 수 있다.

* Advice는 과거에 별도의 인터페이스로 구현되고, 이를 클래스로 구현하는 방식으로 제작했으나 스프링3 버전 이후에는 **어노테이션만으로** 모든 설정이 가능하다.
* Target에 어떤 Advice를 적용할 것인지는 **XML을 이용한 설정**을 이용할 수 있고, **어노테이션을 이용하는 방식**을 이용할 수 있다.

<hr />

AOP 실습
---------------

* AOP 기능은 주로 일반적인 JavaAPI를 이용하는 클래스(POJO)들에 적용한다.
* 서비스 계층에 AOP를 적용
    - 서비스 계층의 메서드 호출 시 모든 파라미터들을 로그로 기록
    - 메서드들간의 실행 시간을 기록

1-1. 스프링 , AOP 버전 업   
```xml
<properties>
    <java-version>1.8</java-version>
    <org.springframework-version>5.0.7.RELEASE</org.springframework-version>
    <org.aspectj-version>1.9.0</org.aspectj-version>
    <org.slf4j-version>1.7.25</org.slf4j-version>
</properties>
```   
    AOP는 AspectJ 라이브러리의 도움을 받기 때문에 스프링버전에 맞춰 올려준다.

1-2. pom.xml 추가
```xml
<!-- 테스트 library 추가 -->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-test</artifactId>
    <version>${org.springframework-version}</version>
</dependency> 

<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.0</version>
    <scope>provided</scope>
</dependency>  

<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.12</version>
    <scope>test</scope>
</dependency>

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
    스프링은 AOP 처리가 된 객체를 생성할 때 AspectJ Weaver 라이브러리의 도움을 받아 동작한다.

1-3. 서비스 계층 설계

SampleService 인터페이스
```java
package org.zerock.service;

public interface SampleService {
	
	public Integer doAdd(String str1, String str2) throws Exception;
}
```

SampleServiceImpl 클래스
```java
package org.zerock.service;

import org.springframework.stereotype.Service;

@Service
public class SampleServiceImpl implements SampleService {

	@Override
	public Integer doAdd(String str1, String str2) throws Exception {
		
		return Integer.parseInt(str1) + Integer.parseInt(str2);
	}
}
```

1-4. Advice 작성    
* 로그를 기록하는 일 ==> 관심사(Aspect) 
* Advice 는 '관심사(Aspect)'를 실제로 구현한 코드이므로,   
 예제로 <U>로그를 기록하는 LogAdvice</U>를 설계한다. 

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
* @Aspect : 해당 클래스의 객체가 Ascpect로 구현한 것임을 나타낸다.
* @Component : 스프링에서 빈(Bean)으로 인식하기 위해 사용한다
* @Before : BeforeAdvice를 구현한 메서드에 추가한다.    
    > **execuition**    
    > "메서드" 를 기준으로 PointCut을 설정   
    > Aspect의 표현식으로, 접근제한자와 특정 클래스의 메서드를 지정할 수 있다.    
    > 맨 앞의 * 는 **접근제한자**를 의미하고 맨 뒤의 * 는 **클래스의 이름과 메서드의 이름**을 의미한다. 

* Advice와 관련된 어노테이션들은 내부적으로 PointCut을 지정한다.   
(PointCut은 별도의 @PountCut으로도 지정해서 사용 가능)   


1-5. AOP 설정

스프링2 버전 이후 자동으로 Proxy 객체를 만들어주는 설정을 추가한다.   

<img src="https://user-images.githubusercontent.com/22673024/78792542-32d0aa00-79ec-11ea-9c73-fea13298d88e.png" width="50%">

root-context.xml 
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

* SampleServiceImpl 클래스에 아이콘이 추가된 것을 확인할 수 있다. (완벽한 동작은 아니지만 도움이 됨...!)

<br>

**java 설정 시**   
> ```java
> @Configuration
> @ComponentScan(basePackages = {"org.zerock.service"})
> @ComponentScan(basePackages = "org.zerock.aop")
> @EnableAspectJAutoProxy
> 
> @MappperScan(basePackages= {"org.zerock.mapper"})
> public class RootConfig {
> ```
> 

1-6. AOP 테스트
```java
package org.zerock.service;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

import lombok.Setter;
import lombok.extern.log4j.Log4j;

@RunWith(SpringJUnit4ClassRunner.class)
@Log4j
@ContextConfiguration({"file:src/main/webapp/WEB-INF/spring/root-context.xml"})
// @ContextConfiguration(classes= {RootConfig.class}) /* java 설정의 경우 */
public class SampleServiceTests {

	@Setter(onMethod_ = @Autowired)
	private SampleService service;
	
	@Test
	public void testClass() {
		log.info(service);
		log.info(service.getClass().getName());
	}
}
```
* AOP 설정을 한 Target에 대해서 Proxy 객체가 정상적으로 만들어져 있는지 확인한다.
* ```<aop:aspectj-autoproxy>``` 가 정상적으로 모든 동작(LogAdvice에 설정한 @Before가 동작)을 하고, LogAdvice에 설정문제가 없다면   
→ **service변수의 클래스**는 단순히 org.zerock.service.SampleServiceImpl의 인스턴스가 아닌 생성된 **<U>Proxy 클래스의 인스턴스가 된다.</U>**   
<img src="https://user-images.githubusercontent.com/22673024/78796071-d2903700-79f0-11ea-9d6c-828cfc8accd1.png" width="60%">

* 단순히 service 변수 출력 시 SampleServiceImpl 클래스의 인스턴스처럼 보이지만, 
getClass() 이용시 JDK의 다이나믹 프록시 기법이 적용된 결과(com.sun.proxy.$Proxy)를 볼 수 있다. 

1-6-1. 프록시 기법을 적용한 테스트코드 
```java
@Test
public void testAdd() throws Exception {
    log.info(service.doAdd("123", "456"));
}
```
    INFO : org.zerock.aop.LogAdvice - ==============================
    INFO : org.zerock.service.SampleServiceTests - 579


