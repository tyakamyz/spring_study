> pom.xml
- 스프링 버전과 JAVA 버전 변경
```xml
<properties>
    <java-version>1.8</java-version>
    <org.springframework-version>5.0.7.RELEASE</org.springframework-version>
    <org.aspectj-version>1.6.10</org.aspectj-version>
    <org.slf4j-version>1.6.6</org.slf4j-version>
</properties>
```
- 스프링 관련 라이브러리 추가
```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-test</artifactId>
    <version>${org.springframework-version}</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-jdbc</artifactId>
    <version>${org.springframework-version}</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-tx</artifactId>
    <version>${org.springframework-version}</version>
</dependency>
```
- DataSource 라이브러리 중 HikariCP 추가 (가볍고 간편한 라이브러리)
```xml
<dependency>
    <groupId>com.zaxxer</groupId>
    <artifactId>HikariCP</artifactId>
    <version>2.7.8</version>
</dependency>
```
- MyBatis 라이브러리 추가
```xml
<!-- mybatis 라이브러리 추가 -->
<!-- https://mvnrepository.com/artifact/org.mybatis/mybatis -->
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>3.4.6</version>
</dependency>
<!-- https://mvnrepository.com/artifact/org.mybatis/mybatis-spring -->
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis-spring</artifactId>
    <version>1.3.2</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-tx</artifactId>
    <version>${org.springframework-version}</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-jdbc</artifactId>
    <version>${org.springframework-version}</version>
</dependency>
```
- MyBatis 라이브러리 추가
```xml
<!-- log4jdbc 설정 -->
<!-- https://mvnrepository.com/artifact/org.bgee.log4jdbc-log4j2/log4jdbc-log4j2-jdbc4 -->
<dependency>
    <groupId>org.bgee.log4jdbc-log4j2</groupId>
    <artifactId>log4jdbc-log4j2-jdbc4</artifactId>
    <version>1.16</version>
</dependency>
```
- lombok 라이브러리 추가
```xml
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.0</version>
    <scope>provided</scope>
</dependency>
<dependency>
    <groupId>log4j</groupId>
    <artifactId>log4j</artifactId>
    <version>1.2.17</version>	<!-- lombok 활용을 위해 버젼 1.2.17 사용 -->
    <!-- <exclusions>
        <exclusion>
            <groupId>javax.mail</groupId>
            <artifactId>mail</artifactId>
        </exclusion>
        <exclusion>
            <groupId>javax.jms</groupId>
            <artifactId>jms</artifactId>
        </exclusion>
        <exclusion>
            <groupId>com.sun.jdmk</groupId>
            <artifactId>jmxtools</artifactId>
        </exclusion>
        <exclusion>
            <groupId>com.sun.jmx</groupId>
            <artifactId>jmxri</artifactId>
        </exclusion>
    </exclusions>
    <scope>runtime</scope> -->
</dependency>
```
- junit 버전 변경
```xml
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.12</version>
    <scope>test</scope>
</dependency>  
```
- Servlet3.1(3.0)을 사용하기 위해 버전 변경
```xml
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>servlet-api</artifactId> <!-- javax.servlet-api로 표기되어있을 수 있음 -->
    <version>3.1.0</version>
    <scope>provided</scope>
</dependency>
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId><!-- servlet-api로 표기되어있을 수 있음. 변경해줌 -->
    <version>3.1.0</version>
    <scope>provided</scope>
</dependency>
```
- Servlet3.1(3.0)을 사용하고, JDK1.8의 기능을 활용하기 위해 버전 변경
```xml
 <plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>2.5.1</version>
    <configuration>
        <source>1.8</source>
        <target>1.8</target>
        <compilerArgument>-Xlint:all</compilerArgument>
        <showWarnings>true</showWarnings>
        <showDeprecation>true</showDeprecation>
    </configuration>
</plugin>
```
> Maven Update
- 해당 프로젝트 마우스 우클릭 > Maven > Update Project
> Oracle JDBC Driver(ojdbc8.jar)
- ojdbc8.jar 파일 Build Path에 추가
- ojdbc8.jar 파일 Deployment Assembly에 추가

> root-context.xml

<img width='70%' src='./img/root-context Namespaces.png'>

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:mybatis-spring="http://mybatis.org/schema/mybatis-spring"
	xsi:schemaLocation="http://mybatis.org/schema/mybatis-spring http://mybatis.org/schema/mybatis-spring-1.2.xsd
		http://www.springframework.org/schema/beans https://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.3.xsd">
	
	<!-- Root Context: defines shared resources visible to all other web components -->
	<bean id="hikariConfig" class="com.zaxxer.hikari.HikariConfig">
		<property name="driverClassName" value="oracle.jdbc.driver.OracleDriver"></property>
		<property name="jdbcUrl" value="jdbc:oracle:thin:@tongracle.cnsqtfo00xiq.ap-northeast-2.rds.amazonaws.com:1521:orcl"></property>
		<property name="username" value="book_ex"></property>
		<property name="password" value="book_ex"></property>
	</bean> 
	
	<!-- HikariCP configuration -->
	<bean id="dataSource" class="com.zaxxer.hikari.HikariDataSource" destroy-method="close">
		<constructor-arg ref="hikariConfig" />
	</bean>
	
	<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
		<property name="dataSource" ref="dataSource"></property>
	</bean>
	
	<mybatis-spring:scan base-package="org.zerock.mapper"/>
		
</beans>
```
> log4jdbc.log4j2.properties
- src/main/resources/log4jdbc.log4j2.properties
```properties
log4jdbc.spylogdelegator.name=net.sf.log4jdbc.log.slf4j.Slf4jSpyLogDelegator
```
> root-context.xml
- log4jdbc 사용할 수 있도록 변경
- sql 결과가 보기좋게 표시됨
```xml
<bean id="hikariConfig" class="com.zaxxer.hikari.HikariConfig">
		<!-- <property name="driverClassName" value="oracle.jdbc.driver.OracleDriver"></property> -->
		<!-- <property name="jdbcUrl" value="jdbc:oracle:thin:@tongracle.cnsqtfo00xiq.ap-northeast-2.rds.amazonaws.com:1521:orcl"></property> -->
		<property name="driverClassName" value="net.sf.log4jdbc.sql.jdbcapi.DriverSpy"></property>
		<property name="jdbcUrl" value="jdbc:log4jdbc:oracle:thin:@tongracle.cnsqtfo00xiq.ap-northeast-2.rds.amazonaws.com:1521:orcl"></property>
		<property name="username" value="book_ex"></property>
		<property name="password" value="book_ex"></property>
	</bean> 
```
> DataSourceTests.java
- DataSource 테스트
- src/test/java/....../DataSourceTests.java
```java
package org.zerock.persistence;

import static org.junit.Assert.fail;

import java.sql.Connection;

import javax.sql.DataSource;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

import lombok.Setter;
import lombok.extern.log4j.Log4j;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("file:src/main/webapp/WEB-INF/spring/root-context.xml")
/*
 * JAVA 설정을 사용하는 경우
 * @ContextConfiguration(class = {RootConfig.class})
 */
@Log4j
public class DataSourceTests {
	@Setter(onMethod_= {@Autowired})
	private DataSource dataSource;
	
	@Test
	public void testConnection() {
		try(Connection con = dataSource.getConnection()){
			log.info(con);
		}catch(Exception e) {
			fail(e.getMessage());
		}
	}
}
```
> JDBCTests.java
- JDBC 테스트
- src/test/java/....../JDBCTests.java (DataSourceTests.java와 동일한 위치)
```java
package org.zerock.persistence;

import static org.junit.Assert.fail;

import java.sql.Connection;
import java.sql.DriverManager;

import org.junit.Test;

import lombok.extern.log4j.Log4j;

@Log4j
public class JDBCTests {
	static {
		try {
			Class.forName("oracle.jdbc.driver.OracleDriver");
		} catch(Exception e) {
			e.printStackTrace();
		}
	}

	@Test
	public void testConnection() {
		try(Connection con = DriverManager.getConnection("jdbc:oracle:thin:@tongracle.cnsqtfo00xiq.ap-northeast-2.rds.amazonaws.com:1521:orcl","book_ex","book_ex")) {
			log.info(con);
		}catch(Exception e) {
			fail(e.getMessage());
		}
	}
}
```
> web.xml
- UTF-8 필터 설정
- 데이터 전송 시 한글깨짐 방지
```xml
<filter>
    <filter-name>encoding</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
        <param-name>encoding</param-name>
        <param-value>UTF-8</param-value>
    </init-param>
</filter>

<filter-mapping>
    <filter-name>encoding</filter-name>
    <servlet-name>appServlet</servlet-name>
</filter-mapping>
```