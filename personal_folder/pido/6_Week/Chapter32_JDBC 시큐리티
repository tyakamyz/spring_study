
[Part7] - chapter 32
=========================

JDBC 인증 / 권한 처리 
------------------

UserDetailsService 인터페이스    


1) 지정된 형식으로 테이블을 생성해서 사용하는 방식   
스프링시큐리티에서 지정한 SQL 을 사용한다. 
```sql
create table users(
    username varchar2(50) not null primary key,
    password varchar2(50) not null,
    enabled char(1) default '1' );
    
create table authorities (
    username varchar2(50) not null,
    authority varchar2(50) not null,
    constraint fk_authorities_users foreign key(username) references users(username));
    
create unique index ix_auth_username on authorities(username, authority);

insert into users (username, password) values ('user00', 'pw00');
insert into users (username, password) values ('member00', 'pw00');
insert into users (username, password) values ('admin00', 'pw00');

insert into authorities (username, authority) values ('user00', 'ROLE_USER');
insert into authorities (username, authority) values ('member00', 'ROLE_MANAGER');
insert into authorities (username, authority) values ('admin00', 'ROLE_MANAGER');
insert into authorities (username, authority) values ('admin00', 'ROLE_ADMIN');
```
security-context.xml
```xml
<security:authentication-manager>
	<security:authentication-provider>
		<security:jdbc-user-service data-source-ref="dataSource"/>
    </security:authentication-provider>
</security:authentication-manager>
```

위와 같이 설정하면 WAS 실행 시 별도의 처리 없이 자동으로 필요한 쿼리들이 호출되는 것을 확인 할 수 있다. 

### PasswordEncoder 문제 
임시로 '{noop}' 처리를 하였지만 스프링 시큐리티 5부터는 기본적으로 PasswordEncoder를 지정해야한다.    
(4버전까지는 NoOpPasswordEncoder를 사용하였지만 5버전부터는 Deprecated 되었다.)   

<br>

PasswordEncoder 커스터마이징을 위한 CustomNoOpPasswordEncoder.java 생성
```java
package org.zerock.security;

import org.springframework.security.crypto.password.PasswordEncoder;

import lombok.extern.log4j.Log4j;

/**
 * 암호화가 없는 PasswordEncoder 구현 
 * @author Sunny
 *
 */
@Log4j
public class CustomNoOpPasswordEncoder implements PasswordEncoder{@Override
	public String encode(CharSequence rawPassword) {
		
	log.warn("before encode : " + rawPassword);
		return rawPassword.toString();
	}

	@Override
	public boolean matches(CharSequence rawPassword, String encodedPassword) {
		
		log.warn("matches : " + rawPassword + ":" + encodedPassword);
		return rawPassword.toString().equals(encodedPassword);
	}

}
```
security-context.xml
```xml
<!-- Bean 처리 -->
<bean id="customPasswordEncoder" class="org.zerock.security.CustomNoOpPasswordEncoder"></bean>
...
<security:authentication-manager>
	<security:authentication-provider>
		<security:jdbc-user-service data-source-ref="dataSource"/>
        <security:password-encoder ref="customPasswordEncoder"/>
    </security:authentication-provider>
</security:authentication-manager>

```

2) 기존에 작성된 데이터베이스 이용하는 방식 

<security:jdbc-user-service> 태그 속성   
users-by-username-query   
authorities-by-username-query    
cache-ref   
group-authorities-by-username-query   
id   
role-prefix   

인증/권한 테이블 생성
```sql
create table tbl_member (
    userid varchar2(50) not null primary key,
    userpw varchar2(100) not null,
    username varchar2(100) not null,
    regdate date default sysdate,
    updatedate date default sysdate, 
    enabled char(1) default '1');
    
create table tbl_member_auth (
    userid varchar2(50) not null,
    auth varchar2(50) not null,
    constraint fk_member_auth foreign key(userid) references tbl_member(userid)
    );  
```

### BCryptPasswordEncoder 클래스를 이용한 패스워드 보호    

※ bcrypt : 패스워드를 저장하는 용도로 설계된 해시 함수. 특정문자열을 암호화 하고, 체크하는 쪽에서는 암호회를 다시 원문으로는 되돌릴 수 없다. 

security-context.xml
```xml
<!-- Bean 처리 -->
<bean id="bcryptPasswordEncoder" class="org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder"></bean>
...

<security:authentication-manager>
	<security:authentication-provider>
		<security:jdbc-user-service data-source-ref="dataSource"/>
        <security:password-encoder ref="bcryptPasswordEncoder"/>
    </security:authentication-provider>
</security:authentication-manager>

```

[TEST] 인코딩된 패스워드를 가진 사용자 생성
```java
package org.zerock.security;

import java.sql.Connection;
import java.sql.PreparedStatement;

import javax.sql.DataSource;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

import lombok.Setter;
import lombok.extern.log4j.Log4j;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration({"file:src/main/webapp/WEB-INF/spring/root-context.xml",
					   "file:src/main/webapp/WEB-INF/spring/security-context.xml"})
@Log4j
public class MemberTests {
	
	@Setter(onMethod_ = @Autowired)
	private PasswordEncoder pwencoder;
	
	@Setter(onMethod_ = @Autowired)
	private DataSource ds;

    @Test
	public void testInsertMember() {
		
		String sql = "insert into tbl_member(userid, userpw, username) values (?,?,?)";
		
		for (int i = 0; i < 100; i++) {
			
			Connection con = null;
			PreparedStatement pstmt = null;
			
			try {
				con = ds.getConnection();
				pstmt = con.prepareStatement(sql);
				
				pstmt.setString(2, pwencoder.encode("pw" + i));
				
				if(i < 80) {
					pstmt.setString(1,  "user" + i);
					pstmt.setString(3, "일반사용자" + i);
				}else if(i < 90) {
					pstmt.setString(1, "manager" + i);
					pstmt.setString(3, "운영자"+i);
				}else {
					pstmt.setString(1, "admin"+i);
					pstmt.setString(3, "관리자"+i);
				}
				pstmt.executeUpdate();
				
			} catch (Exception e) {
				// TODO: handle exception
				e.printStackTrace();
			} finally {
				if(pstmt != null) {try {pstmt.close();} catch(Exception e) {}}
				if(con != null) {try {con.close();} catch(Exception e) {}}
			}
			
		}
	}
```
위와 같이 PasswordEncoder 를 이용하여 문자열을 추가하는 과정을 통해 코드를 실행하고 나면 BcryptPasswordEncodoer를 이용해 생성된 암호화된 패스워드를 확인할 수 있다. 

<img src ="https://user-images.githubusercontent.com/22673024/80277013-27d58380-8727-11ea-9e47-27d77ae013d9.png" width=60%>

security-context.xml
```xml
<security:authentication-manager>
	<security:authentication-provider>
		<security:jdbc-user-service data-source-ref="dataSource" 
		users-by-username-query="select userid, userpw, enabled from tbl_member where userid= ? " 
		authorities-by-username-query="select userid, auth from tbl_member_auth where userid= ? " />
		
		<security:password-encoder ref="bcryptPasswordEncoder"/>
    </security:authentication-provider>
</security:authentication-manager>
```
인증을 하는데 필요한 쿼리(users-by-username-query)와 권한을 확인하는데 필요한 쿼리(authorities-by-username-query)를 이용하여 처리한다. 

