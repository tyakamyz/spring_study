# 중간 정리

## **Part_01** - Chapter_02

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

## **Part_02** - Chapter_05

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

## **Part_02** - Chapter_06

- ### 1. Controller의 리턴 타입

  |타입|설명|
  |---|---|
  |String| jsp를 이용하는 경우에는 jsp 파일의 경로와 파일이름으르 나타내기 위해서 사용
  |void|호출하는 URL과 동일한 이름의 jsp를 의미
  |VO, DTO|주로 JSON 타입의 데이터를 만들어서 반환하는 용도로 사용
  |ResponseEntity|response 할 때 Http 헤더 정보와 내용을 가공하는 용도로  사용
  |Model, ModelAndView| Model로 데이터를 반환하거나 화면까지 같이 지정하는 경우에 사용<br/>(최근에는 많이 사용하지않는다.)
  |HttpHeaders|응답에 내용 없이 Http 헤더 메시지만 전달하는 용도로 사용

## **Part 03** - Chapter 07

 - ### 1. 일반적으로 웹 프로젝트는 3-Tier(티어) 방식으로 구성
    - Presentation Tier(화면 계층)
    - Business Tier(비즈니스 계층)
    - Persistence Tier(영속 계층 혹은 데이터 계층)

- Presentation Tier(화면계층)
    - 화면에 보여주는 기술을 사용하는 영역. Servlet/JSP나 스프링 MVC가 담당하는 영역
    - 프로젝트 성격에 맞춰 앱이나 CS(Client-Server)로 구성되는 경우가 있다.
    - 이전 파트에서 학습한 스프링 MVC와 JSP를 이용한 화면구성이 이에 속한다.
    
- Business Tier(비즈니스 계층)
    - 순수한 비즈니스 로직을 담고있는 영역
    - 고객이 원하는 요구 사항을 반영하는 계층으로 **중요한 영역** 이다
    - 주로 'xxxService'와 같은 이름으로 구성, 메스드의 이름 역시 고객들이 사용하는 용어를 그대로 사용
- Persistence Tier(영속 계층 혹은 데이터 계층)
    - 데이터를 어떤 방식으로 보관하고, 사용하는가에 대한 설계가 들어가는 계층
    - 일반적인 경우 데이터베이스를 많이 사용하지만, 경우에 따라 네트워크 호출이나 원격 호출등의 기술이 접목
    - 해당 영역은 MyBatis와 mybatis-spring을 이용해서 구성했던 파트 1을 이용

## **Part 04** - Chapter 16

 - ### 1. 다양한 전송방식

 |작업|전송방식|URI
|---|----|-----
|등록|POST|/members/new
|조회|GET|/members/{id}
|수정|PUT/PATCH|/member/{id} + body (json 데이터 등)
|삭제|DELETE|/member/{id}

## **Part 05** - Chapter 18

 - ### 1. 용어
 |구분|설명
 |---|---
|Proxy|**대리인**, 함수 호출자는 주요업무가아닌 보조업무를 프록시에게 맡기고, 프록시 내부적으로 보조업무를 제어,처리
|Target| **핵심 비즈니스 로직 객체**, 부가기능을 부여할 대상
|Advice| **부가기능을 정의한 코드**, 타겟에 제공할 부가기능을 담고있는 모듈
|JoinPoint| **Advice를 적용 가능한 지점**, Spring에서 관리하는 Bean들의 모든 메서드
|pointcut| **관심사와 비즈니스 로직이 결합되는 지점을 결정하는 것**, JoinPoint의 부분집합으로도 표현
|Aspect| **Advice + pointcut**, Advice의 추상화, AOP의 기본 모듈
- ### 2. pointcut 시점
|구분|어노테이션|설명|
|--|--|--|
|Before Advice|@Before| Target의 JoinPoint를 호출하기 전에 실행되는 코드이다.</br> 코드의 실행 자체에는 관여할 수 없다.
|After Returning Advice |@AfterReturning| 모든 실행이 정상적으로 이루어진 후 에 동작하는 코드|
|After Throwing Advice |@AfterThrowing| 예외가 발생한 뒤에 동작하는 코드|
|After Advice |@After| 정상적으로 실행되거나 예외가 발생했을 때 구분 없이 실행되는 코드|
|Around Advice|@Around| 메서드의 실행 자체를 제어할 수 있는 가장 강력한 코드</br>직접 대상메서드를 호출하고 결과나 예외를 처리할 수 있다.

- ### 3. pointcut 선언형태
 |구분|설명|사용예
 |--|---|---
 |execution(@execution)| '**메서드**'를 기준으로 Pointcut을 설정한다|execution([수식어] [리턴타입] [클래스이름] [이름](\[파라미터]\)
 |within(@within)|특정한 '**타입(클래스)**'을 기준으로 Pointcut을 설정한다|
 |this|주어진 '**인터페이스를 구현한 객체**'를 대상으로 Pointcut을 설정한다|
 |arg(@args)|'**특정한 파라미터**'를 가지는 대상들만을 Pointcut으로 설정한다|
 |@annotation|'**특정한 어노테이션**'이 적용된 대상들만을 Pointcut으로 설정한다.|

 - #### 3.1 pointcut 선언 속성
 |구분|설명
|--|--
|수식어|생략가능, public,protected 등
|리턴타입|메서드의 리턴타입 지정
|클래스이름, 이름| 클래스의 이름 및 메서드 이름 지정
|파라미터 | 메서드 파라미터 지정
|'*'| 모든 값
|'..'| 0개 이상

- ### 4.@Around와 ProceedingJoinPoint
    - AOP 를 이용하여 좀더 구체적인 처리에 사용
    - ProceedingJoinPoint 인터페이스의 메서드

 |메서드|설명
 |--|--
 |Signature getSignature()|호출되는 메서드에 대한 정보
 |Object getTarget() | 대상 객체(Target)
 |Object[] getArgs() | 파라미터의 목록
 |proceed()| 의존 함수 실행
 |proceed(Object[])| 의존 함수 실행(매게변수는 의존 함수 파라미터와 같은 타입을 가져야한다)

 - ProceedingJoinPoint.getSignature() 로 정의된 Signature 인터페이스는 호출되는 메서드와 관련된 정보를 제공하며 다음과 같은 메서드를 정의한다.

 |메서드|설명
 |--|--
 |String getName | 메서드의 이름
 |String toLongString()| 메서드를 완전하게 표현한 문장(메서드의 리턴 타입, 파라미터 타입이 모두 표시)
 |String toShortString()|메서드의 축약 표현 문장(기본 구현은 메서드의 이름만을 구한다)

## **Part 05** - Chapter 19

 - ### 1. @Transactional 적용 순서

    - **1순위** : 메서드의 @Transactional
    - **2순위** : 클래스의 @Transactional
    - **3순위** : 인터페이스의 @Transactional
    - 위의 우선순위를 통해 인터페이스에는 가장 기준이 되는 @Transactional을 설정하고, 클래스나 메서드에 필요한 어노테이션을 처리하는 것이 좋다.