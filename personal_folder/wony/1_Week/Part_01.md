# **Part 01** 개발을 위한 준비


## **Chapter 01** 개발 환경 설정


+ ### 1. Java Configuration 을 하는 경우 필요한 작업
	- web.xml의 파일 삭제 및 스프링 관련 파일 삭제  
(web.xml, servlet-context.xml, root-context.xml)
	- pom.xml의 수정 및 스프링 버전 변경
	- Java 설정 관련 패키지 생성


## **Chapter** 02 스프링의 특징과 의존성 주입

+ ### 1. 스프링의 주요 특징
	- POJO 기반의 구성
 	- 의존성 주입(DI)을 통한 객체 간의 관계 구성
 	- AOP(Aspect-Oriented-Programming) 지원
 	- 편리한 MVC 구조
 	- WAS의 종속적이지 않은 개발 환경

+ ### 2. @Component
  - Spring에게 해당 클래스가 Spring에서 관리해야하는 대상임을 표시

+ ### 3. @Setter(onMethd = @__({@Autowired}))

 |속성명| 의미|
 |-----------------|------------------|
 |value|접근 제한 속성을 의미 </br> 기본값은  lombok.AccessLevel.PUBLC|
 |onMethod|setter 메서드의 생성시 메서드에 추가할 어노테이션 지정 </br> JDK 버전에 따라 '_' 표기차이가 있음  </br>up to JDK7 : @Setter(onMethod = @__({@AnnotationsGoHere})  </br>from JDK8 : @Setter(onMethod_={@AnnotationsGohere})|
 |onParam           |setter 메서드의 파라미터에 어노테이션을 사용하는경우 적용|
 
 	- 자동으로 SetChef()를 컴파일 시 생성
	- onMethod 속성은 생성되는 setChef()에 @Autowired 를 추가
 	- @Autowired 는 상황에 맞는 타입(Class)의 Bean을 가져와서 주입
 	- @Resource 는 상황에 맞는 이름(Id)의 Bean을 가져와서 주입
 

 + ### 4. Spring이 동작하며 생기는 일
 	- Spring FrameWork가 시작 되면 Spring이 사용하는 메모리 영역을 만듦 - **Context** 라  하며 **ApplicationContext**라는 이름의 객체가 만들어진다.
 	- Spring은 자신이 객체를 생성하고 관리해야 하는 객체들에 대한 설정이 필요 - **root-context.xml**
 	- root-context.xml에 설정되어있는 **<<context:component-scan>>** 태그의 내용을 통해서 **org.zerock.sample** 패키지를 스캔한다.
 	- 해당 패키지에 있는 클래스들 중 Spring이 사용하는 @Component라는 어노테이션이 존재하는 클래스의 인스턴스를 생성
 	- Restaurant 객체는 Chef 객체가 필요하다는 어노테이션 설정이 있으므로 , Spring은 Chef 객체의 레퍼런스를 Restaurnat 객체에 주입

 + ### 5. Test
 	- 현재 테스트 코드가 스프링을 실행하는 역할을 할 것이라는 것을 **@RunWith** 로 표현
 	- 지정된 클래스나 문자열을 이용해 필요한 객체들을 스프링 내에 객체로 등록(스프링 빈으로 등록한다),  사용하는 문자열은 **'classpath:'** 나 **'file:'** 을 이용한다. 자동으로 상성된 **root-context.xml의 경로** 를 지정 - **@ContextConfiguration**
 	- Junit 진행중 ApplicationContext를 만들고 관리하는 작업을 진행, 테스트 별로 객체 생성하더라도 싱글톤 ApplicationContext 를 보장
 
 #### * 중요사항 
 >	1. 테스트 코드가 실행되기 위해 Spring FrameWork가 동작
 >	2. 동작하는 과정에서 필요한 객체들이 Spring에 등록
 >	3. 의존성 주입이 필요한 객체는 자동으로 주입이 이루어짐


 #### * 코드에 사용된 어노테이션
 |Lombok 관련       |  Spring 관련      | Test 관련
 |-----------------|------------------|---------
 |@Setter          |@Autowired        |@RunWith
 |@Data            |@Component        |@ContextConfiguration
 |@Log4j           |                  |@Test

</br>


+ ### 6. 생성자 자동주입
 	- 인스턴스 변수로 선언된 모든 것을 파라미터로 받는 생성자 작성 - **@AllArgsConstructor**
 	- 여러 개의 인스턴스 변수들 중 특정한 변수에 대해서만 생성자를 작성 -  **@NonNull** 과 **@RequiredArgsConstructor** 

## **Chapter 03** 스프링과 Oracle Database 연동

+ ### 1. Port 확인 및 변경

	- 확인 : **select dbms_xdb.gethttpport() from dual;**
	- 변경 : **exec dbms_xdb.sethttpport({Port Number});**

+ ### 2. 커넥션 풀

	- 일반적으로 다수의 사용자를 동시에 처리해야 하는 웹 어플리케이션의 경우 데이터 베이스 연결을 이용할 때는 커넥션풀을 이용.
	- Java에서는 DataSource라는 인터페이스를 통해 커넥션 풀을 사용.
	- 여러 종류중 최근 유행하는 HikariCP사용(2018년도)


## **Chapter 04** MyBatis와 스프링 연동

+ ### 1. MyBatis 관련 라이브러리
	- spring-jdbc/spring-tx : 스프링에서 데이터베이스 처리와 트랜잭션 처리(MyBatis 와 유관하므로 반드시 추가)
	- mybatis/mybatis-spring : MyBatis 와 스프링 연동용 라이브러리

+ ### 2. SQLSessionFactory
	- SQLSession을 통해 Connection을 생성하거나 원하는 SQL을 전달하고, 결과를 리턴 받는 구조로 작성
	- SqlSessionFactory를 등록 하는 작업은 SqlSessionFactoryBean을 이용. 패키지명을 확인해보면 MyBatis의 패키지가 아니라 스프링과 연동 작업을 처리하는 mybatis-spring 라이브러리의 클래스임을 알수 있다.

```xml
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
		<property name="dataSource" ref="dataSource"></property>
</bean>
```

+ ### 3. Mapper
	- SQL을 어떻게 처리할 것인지를 별도의 설정을 분리해 주고, 자동으로 처리되는 방식
	- XML과 인터페이스 + 어노테이션의 형태로 작성 가능

+ #### 3.1 Mppaer 인터페이스
	
	```java
	public interface TimeMapper {
	
	@Select("SELECT sysdate FROM dual")
	public String getTime();

	}
	```

	- Mapper 작성한 후 MaBatis가 동작할 때 Mapper를 인식할 수 있도록 **root-context.xml**에 추가 설정 필요. 가장 간단한 방식은 **<<mybatis:scan>>** 태그 사용

	- root-context.xml 에 아래 내용 추가
	```xml
	<mybatis-spring:scan base-package="org.zerock.mapper"/>
	```
	- 지정된 패키지의 모든 MyBatis 관련 어노테이션을 찾아서 처리한다.
	- Mapper 설정 작업은 각각의 XML이나 Mapper 인터페이스를 설정할 수도 있지만, 매번 너무 번잡하기에 예제는 자동으로 org.zerock.mapper 패키지를 인식하는 방식으로 작성하는 것이 가장 편리하다.
	- SQL이 복잡하거나 길어지는 경우 어노테이션 보다 XML을 이용하는 방식을 더 선호, MyBatis-Spring의 경우 Mapper 인터페이스와 XML을 동시에 사용 가능

+ #### 3.2 Mapper XML
	- 사용시 XML의 파일 위치와 XML 파일에 지정하는 namespace속성이 중요.
	- XML 파일 위치의 경우 Mapper 인터페이스가 있는 곳에 같이 작성 하거나, **src/main/resources 구조에 XML을 저장할 폴더를 생성 가능.
	- XML의 이름 생성 규칙은 별도로 없지만 Mapper 인터페이스와 같은 이름을 갖는 것이 가독성을 높여준다.
	- XML 파일에는 MyBatis의 XML 매퍼에서 이용하는 태그에 대한 설정이 필요  
	**(http://www.mybatis.org/mybatis-3/ko/sqlmap-xml.html)** 참조
	- Mapper 인터페이스와 XML을 인터페이스의 이름과 namespace의 속성값을 가지고 판단, 아래의 경우 메서드 선언은 인터페이스에 존재하고 SQL에 대한 처리는 XML을 이용하는 방식이다.
	```java
	package org.zerock.mapper;

	public interface TimeMapper
	```
	```xml
	<mapper namespace = "org.zerock.mapper.TimeMapper">
	
+ ### 4. log4jdbc-log4j2
	
	- 복잡한 형태의 SQL, 입력된 파라미터의 확인 등의 SQL 로그를 제대로 보기위해 설정하는 라이브러리
	- pom.xml 라이브러리 설정
	```xml
	<!-- https://mvnrepository.com/artifact/org.bgee.log4jdbc-log4j2/log4jdbc-log4j2-jdbc4 -->
		<dependency>
			<groupId>org.bgee.log4jdbc-log4j2</groupId>
			<artifactId>log4jdbc-log4j2-jdbc4</artifactId>
			<version>1.16</version>
		</dependency>
	```
	- src/main/resource 에 log4jdbc.log4j2.properties 파일 추가
	```properties
	log4jdbc.spylogdelegator.name=net.sf.log4jdbc.log.slf4j.Slf4jSpyLogDelegator
	```
	- JDBC 드라이버와 URL 정보 수정
	> - 변경 전
	```xml
	<property name="driverClassName" value="oracle.jdbc.driver.OracleDriver"/>
	<property name="jdbcUrl" value="jdbc:oracle:thin:@localhost:1521:xe"/>
	```

	> - 변경 후 
	``` xml
	<property name="driverClassName" value="net.sf.log4jdbc.sql.jdbcapi.DriverSpy"/>
		<property name="jdbcUrl" value="jdbc:log4jdbc:oracle:thin:@localhost:1521:xe"/>
	```
 - ### 5 로그의 레벨 설정
	- 테스트 코드가 실핼 될 때의 로그 설정 **src/test/resources** 밑에 **log4j.xml** 을 이용  
	- 로그 레벨 설정 참고
	**(https://logging.apache.org/log4j/2.x/manual/customloglevels.html)**		