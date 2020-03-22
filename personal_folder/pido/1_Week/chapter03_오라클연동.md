[Part1] - chapter 03
=========================

오라클 설치
----

#### 1. 설치   
Oracle database   
[Download]: https://www.oracle.com/database/technologies/xe-downloads.html

SQL Developer   
[Download]: https://www.oracle.com/kr/tools/downloads/sqldev-v192-downloads.html


#### 2. 오라클 계정 생성
계정명 System. 호스트명 localhost. 포트 1521. SID xe 접속   

* 사용자 생성
    > CREATE USER book_ex IDENTIFIED BY book_ex
    DEFAULT TABLESPACE USERS
    TEMPORARY TABLESPACE TEMP;
* 권한 처리
    > GRANT CONNECT, DBA TO BOOK_EX;


계정명 sys. SYSDBA 권한으로 접속    

* 8080 포트변경   
    > -- 현재포트확인   
    > select dbms_xdb.gethttpport() from dual;   
    > -- 포트 변경   
    > exec dbms_xdb.sethttpport(9090);

<hr />

프로젝트의 JDBC 연결
-----

* 프로젝트 선택 후 'Build Path' Libraries에 ojdbc8.jar 추가   
* Deployment Assemply 에도 Add - Java Build Path 선택 후 ojdbc8.jar 추가 

#### 1. 커넥션 풀 설정

여러명의 사용자를 동시에 처리해야하는 경우 '커넥션 풀(Connection Pool)을 이용하며, DataSource 라는 인터페이스를 통해 커넥션 풀을 사용.

HikariCP   
[Download]: https://github.com/brettwooldridge/HikariCP

* pom.xml 추가 
    >       <dependency>
    >       <groupId>com.zaxxer</groupId>
    >		    <artifactId>HikariCP</artifactId>
    >		    <version>2.7.4</version>
    >		</dependency>


**- XML 기반 설정 -**  
root-context.xml 
```
<!-- Root Context: defines shared resources visible to all other web components -->
	<bean id="hikariConfig" class="com.zaxxer.hikari.HikariConfig">
		<property name="driverClassName"  value="oracle.jdbc.driver.OracleDriver"></property>
		<property name="jdbcUrl"  value="jdbc:oracle:thin:@localhost:1521:XE"></property>
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
		hikariConfig.setDriverClassName("oracle.jdbc.driver.OracleDriver");
		hikariConfig.setJdbcUrl("jdbc:oracle:thin:@localhost:1521:XE");
		hikariConfig.setUsername("book_ex");
		hikariConfig.setPassword("book_ex");
		
		HikariDataSource dataSource = new HikariDataSource(hikariConfig);
		
		return dataSource;
	}
```

