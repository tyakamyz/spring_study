
[Part1] - chapter 04
=========================

MyBaties 
----
- SQL 매핑 프레임워크 
- 자동으로 Connection close() 기능
- MyBaties 내부적으로 PreparedStatement 처리
- #{prop} 와 같이 속성을 지정하면 내부적으로 자동 처리
- 리턴타입을 지정하는 경우 자동으로 객체 생성 및 ResultSet 처리

#### pom.xml 추가 
>       <dependency>
>		    <groupId>org.mybatis</groupId>
>		    <artifactId>mybatis</artifactId>
>		    <version>3.4.6</version>
>		</dependency>
>				
>		<dependency>
>		    <groupId>org.mybatis</groupId>
>		    <artifactId>mybatis-spring</artifactId>
>		    <version>1.3.2</version>
>		</dependency>
>		
>		<dependency>
>		    <groupId>org.springframework</groupId>
>		    <artifactId>spring-tx</artifactId>
>		    <version>${org.springframework-version}</version>
>		</dependency>
>		
>		<dependency>
>		    <groupId>org.springframework</groupId>
>		    <artifactId>spring-jdbc</artifactId>
>		    <version>${org.springframework-version}</version>
>		</dependency>

*spring-jdbc/spring-tx : 데이터베이스 처리와 트랜잭션 처리*   
*mybatis/mybatis-spring : 스프링과 Mybatis 연동 라이브러리* 

#### 1. SQLSessionFactory
* 내부적으로 SQLSession 을 생성해내는 존재   
Connection을 생성하거나 원하는 SQL을 전달, 결과를 리턴받는 구조

**[스프링 SQLSessionFactory 등록 작업]** 

**- XML 기반 설정 -**  
root-context.xml 
```
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
		<property name="dataSource" ref="dataSource"></property>
</bean>
```

**- JAVA 기반 설정 -**
RootConfig.java
```
@Bean
public SqlSessionFactory sqlSessionFactory() throws Exception {
	SqlSessionFactoryBean sqlSessionFactory = new SqlSessionFactoryBean();
	sqlSessionFactory.setDataSource(dataSource());
	return (SqlSessionFactory) sqlSessionFactory.getObject();
}
```

#### 2. Mybatis + 스프링 연동
* MyBatis의 Mapper   
SQL 을 어떻게 처리할 것인지 별도의 설정을 분리해주고, 자동으로 처리되는 방식. SQL과 그에 대한 처리를 지정하는 역할.

**[Mapper 설정 작업]**    
Mybatis 가 동작할 때 Mapper를 인식할 수 있도록 설정하는 작업

**- XML 기반 설정 -**  
root-context.xml 파일 아래 Namespaces 항목에서 **mybatis-spring** 탭 선택
root-context.xml Source 항목에 추가
```
<mybatis-spring:scan base-package="org.zerock.mapper"/>	
```

**- JAVA 기반 설정 -**  
RootConfig.java
```
@MapperScan(basePackages = {"org.zerock.mapper"})
public class RootConfig {
```

**[Mapper 매퍼 사용]**    
1. 어노테이션 방식
```
public interface TimeMapper {

	@Select("SELECT sysdate FROM dual")
	public String getTime();
	
	// XML Mapper 처리
	public String getTime2();
}
```

2. XML 방식
```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="org.zerock.mapper.TimeMapper">

	<select id="getTime2" resultType="string">
		SELECT sysdate FROM dual
	</select>
</mapper>
```

> * XML 파일 생성 시 Mapper인터페이스의 구조와 같게 설정하는 것이 가독성을 높여준다. 
> * ```<mapper>``` 태그의 namespace 속성값은 Mapper인터페이스와 XML을 인터페이스의 이름과 namespace 속성값을 가지고 판단한다. 메서드 선언은 동일한 "org.zerock.mapper.TimeMapper"의 인터페이스에 존재하고 SQL 에 대한 처리는 XML을 이용하는 방식.
> * ```<select>``` 태그의 id속성의 값은 메서드 이름과 동일해야한다. 
> * resultType 속성의 경우 인터페이스에 선언된 메서드의 리턴타입과 동일해야한다. 
> * XML 매퍼에서 이용하는 태그에 대한 설정 필요   
    [참고]http://www.mybatis.org/mybatis-3/ko/sqlmap-xml.html

    
log4jdbc-log4j 
--------------------
정확한 파라미터의 확인을 위한 라이브러리 사용

#### 1. pom.xml 추가 
>      <dependency>
>        <groupId>org.bgee.log4jdbc-log4j2</groupId>
>       <artifactId>log4jdbc-log4j2-jdbc4</artifactId>
>        <version>1.16</version>
>     </dependency>

#### 2. 로그 설정 파일을 추가
* src/main/resources 밑에 log4jdbc.log4j2.properties 파일 생성 후 하단 내용추가
    >log4jdbc.spylogdelegator.name=net.sf.log4jdbc.log.slf4j.Slf4jSpyLogDelegator

#### 3. log4jdbc 이용을 위해 JDBC드라이버와 URL정보 수정
**- XML 기반 설정 -**  
root-context.xml 
```
<!-- Root Context: defines shared resources visible to all other web components -->
	<bean id="hikariConfig" class="com.zaxxer.hikari.HikariConfig">
		<property name="driverClassName"  value="net.sf.log4jdbc.sql.jdbcapi.DriverSpy"></property>
		<property name="jdbcUrl"  value="jdbc:log4jdbc:oracle:thin:@localhost:1521:XE"></property>
		<property name="username"  value="book_ex"></property>
		<property name="password"  value="book_ex"></property>
	</bean>
	
	<!-- HikariCP configuration -->
	<bean id="dataSource" class="com.zaxxer.hikari.HikariDataSource" destroy-method="close">
		<constructor-arg ref="hikariConfig" />
	</bean>
```

**- JAVA 기반 설정 -**
RootConfig.java
```
/**
	 @Configuration
    @ComponentScan(basePackages= {"org.zerock.sample"})	
    @MapperScan(basePackages = {"org.zerock.mapper"})
    public class RootConfig {
	
	@Bean
	public DataSource dataSource() {
		HikariConfig hikariConfig = new HikariConfig();
		hikariConfig.setDriverClassName("net.sf.log4jdbc.sql.jdbcapi.DriverSpy");
        hikariConfig.setJdbcUrl("jdbc:log4jdbc:oracle:thin:@localhost:1521:XE");
		hikariConfig.setUsername("book_ex");
		hikariConfig.setPassword("book_ex");
		
		HikariDataSource dataSource = new HikariDataSource(hikariConfig);
		
		return dataSource;
	}
```


#### 4. 로그 레벨 설정 
log4j.xml 파일의 설정 변경   
[참고] https://logging.apache.org/log4j/2.x/manual/customloglevels.html

