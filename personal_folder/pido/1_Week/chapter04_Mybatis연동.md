
[Part5] - chapter 19
=========================

스프링의 트랜잭션 관리
-----------------
* 트랜잭션(Transaction)   
'한번에 이루어지는 작업의 단위'

* 트랜잭션 성격
    #### ACID 원칙
    |용어                       | 의미	 
    |---------------------------|----------------------
    |원자성(Atomicity)          |하나의 트랜잭션은 하나의 단위로 처리되어야한다.어떤 작업이 잘못되는 경우 모든것은 다시 원점으로 돌아가야한다.
    |일관성(Consistency)	    |트랜잭션이 성공했다면 데이터베이스의 모든 데이터는 일관성을 유지해야한다.
    |격리(Isolation)	        |트랜잭션으로 처리되는 중간에 외부에서의 간섭은 없어야한다.
    |영속성(Durability)	        |트랜잭션이 성공적으로 처리되면, 결과는 영속적으로 보관되어야한다. 

* '트랜잭션으로 관리한다', '트랜잭션으로 묶는다' → 'AND' 연산과 비슷하다.

* **정규화**(중복된 데이터를 제거 해서 데이터 저장의 효율을 올린다.   
(테이블 증가, 각 테이블의 데이터 양 감소))가 진행될 수록 '트랜잭션처리' 대상에서 멀어진다.   
그러나 조인이나 서브쿼리를 이용하여 처리해야하는 작업의 경우, 성능이슈로 **'반정규화'** 를 하게 된다.   
(ex. 게시물의 댓글, tbl_reply 테이블에 insert하고 tbl_board 테이블에 update)   
이때 , 하나의 트랜잭션으로 관리되어야 하는 작업이다. 

<hr />

트랜잭션 실습
---------------
1-1. pom.xml 추가
```xml
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

<dependency>
    <groupId>com.zaxxer</groupId>
    <artifactId>HikariCP</artifactId>
    <version>2.7.8</version>
</dependency>

<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>3.4.6</version>
</dependency>
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis-spring</artifactId>
    <version>1.3.2</version>
</dependency>

<dependency>
    <groupId>org.bgee.log4jdbc-log4j2</groupId>
    <artifactId>log4jdbc-log4j2-jdbc4</artifactId>
    <version>1.16</version>
</dependency>
```
1-2. root-context.xml 추가
```xml
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

<!-- Mybatis-spring Connection -->
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
    <property name="dataSource" ref="dataSource"></property>
</bean>

<!-- Transaction -->
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource"></property>
</bean>

<tx:annotation-driven />
```

```<bean>```으로 등록된 transactionManager와 ```<tx:annotation-driven>``` 설정이 추가 된 후에는 트랜잭션이 필요한 상황을 만들어 어노테이션을 추가하는 방식으로 설정한다. 

<br>

**java 설정 시**   
> ```java
> @Configuration
> @ComponentScan(basePackages= {"org.zerock.service"})
> @ComponentScan(basePackages="org.zerock.aop")
> @EnableAspectJAutoProxy
> @EnableTransactionManagement
> @MapperScan(basePackages = {"org.zerock.mapper"})
> public class RootConfig {
>     ...
>     @Bean
> 	public DataSourceTransactionManager txManager() {
> 		return new DataSourceTransactionManager(dataSource());
> 	}
> }    
> ```

* TransactionManager를 @Bean 으로 설정하는 작업과 'aspectj-autoproxy' 설정을 추가해서 처리한다. 
* @EnableTransactionManagement 설정은 'aspectj-autoproxy'에 대한 설정이고 txManager()는 ```<bean>``` 설정을 대신하게 된다.


1-3. 트랜잭션 실행   

SampleTxServiceImpl.java
```java
@Transactional
@Override
public void addData(String value) {
    
    log.info("mapper1..........................");
    mapper1.insertCol1(value);
    
    log.info("mapper2..........................");
    mapper2.insertCol2(value);
    
    log.info("end..............................");
}
```
* 위와 같이 사용할 메서드 위에 ```@Transactional``` 어노테이션을 추가해주면 된다. 

#### 트랜잭션 어노테이션 속성들

1. 전파(Propagation)속성   

    |용어                   | 의미	 
    |-----------------------|----------------------
    |PROPAGATION_MADATORY   | 작업은 반드시 특정한 트랜잭션이 존재한 상태에서만 가능 
    |PROPAGATION_NESTED     | 기존에 트랜잭션이 있는 경우, 포함되어서 실행
    |PROPAGATION_NEVER	    | 트랜잭션 상황하에 실행되면 예외발생
    |PROPAGATION_NOT_SUPPORTED | 트랜잭션이 있는 경우 트랜잭션이 끝날때까지 보류된 후 실행
    |PROPAGATION_REQUIRED   | 트랜잭션이 있으면 그 상황에서 실행, 없으면 새로운 트랜잭션 실행(기본설정)
    |PROPAGATION_REQUIRED_NEW | 대상은 자신만의 고유한 트랜잭션으로 실행
    |PROPAGATION_SUPPORTS   | 트랜잭션을 필요로 하지 않으나, 트랜잭션 상황하에 있다면 포함되어서 실행

2. 격리(Isolation)레벨

    |용어                  | 의미	 
    |----------------------|----------------------
    |DEFAULT               | DB설정, 기본 격리수준(기본 설정)
    |SERIALIZABLE          | 가장 높은 격리(성능저하 우려가 있음)
    |READ_UNCOMMITED       | 커밋되지 않은 데이터에 대한 읽기 허용
    |READ_COMMITED         | 커밋된 데이터에 대해 읽기 허용
    |REPEATEABLE_READ      | 동일 필드에 대해 다중 접근 시 모두 동일한 결과를 보장

3. Read-only 속성
    * true 인 경우, insert, update, delete 실행 시 예외 발생, 기본 설정 false

4. Rollback-for-예외
    * 특정 예외가 발생 시 강제로 Rollback

5. No-rollback-for-예외
    * 특정 예외의 발생 시에는 Rollback 처리되지않음

1-4. 트랜잭션의 우선순위(메서드 > 클래스 > 인터페이스) 
* 메서드의 @Transactional
* 클래스의 @Transactional
* 인터페이스의 @Transactional

→ **인터페이스**에는 가장 **기준**이되는 @Transactional 과 같은 설정을 지정하고, **클래스나 메서드**에 **필요한 어노테이션**을 처리하는 것이 좋다. 


