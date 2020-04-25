
[Part7] - chapter 33
=========================

커스텀 UserDetailsService 
-----------------------

**JDBC 를 이용한 방식**으로는 스프링 시큐리티에서 username 이라고 부르는 사용자 정보만을 이용하기 때문에 실제 프로젝트에서 사용자 이름이나 이메일 등의 다른 정보를 이용할 경우엔 충분하지 못하다는 단점이 있다. 

→ 직접 UserDetailsService 를 구현하는 방식을 이용한다. 

<U>UserDetailsService 인터페이스 </U>   
loadUserByUsername() 하나의 메서드만이 존재한다.    
메서드 리턴타입인 UserDetail 역시 인터페이스로 사용자의 정보, 권한 정보등을 담는다.

1. 회원 도메인, 회원 Mapper 설계

```java
// AuthVO.java
package org.zerock.domain;

import lombok.Data;

@Data
public class AuthVO {
	
	private String userid;
	private String auth;
}

// MemberVO.java
package org.zerock.domain;

import java.util.Date;
import java.util.List;

import lombok.Data;

@Data
public class MemberVO {
	
	private String userid;
	private String userpw;
	private String userName;
	private boolean enabled;
	
	private Date regDate;
	private Date updateDate;
	private List<AuthVO> authList;
}

```

2. Mapper 작성
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="org.zerock.mapper.MemberMapper">

	<resultMap type="org.zerock.domain.MemberVO" id="memberMap">
		<id property="userid" column="userid" />
		<result property="userid" column="userid" />
		<result property="userpw" column="userpw" />
		<result property="userName" column="userName" />
		<result property="regDate" column="regDate" />
		<result property="updateDate" column="updateDate" />
		<collection property="authList" resultMap="authMap"></collection>
	</resultMap>
	
	<resultMap type="org.zerock.domain.AuthVO" id="authMap">
		<result property="userid" column="userid" />
		<result property="auth" column="auth" />
	</resultMap>
	
	<select id="read" resultMap="memberMap">
		SELECT
			mem.userid, userpw, username, enabled, regdate, updatedate, auth 
		FROM
			tbl_member mem LEFT OUTER JOIN tbl_member_auth auth
		ON	mem.userid = auth.userid
		WHERE mem.userid = #{userid}
	</select>
</mapper>

```

* ResultMap 기능 사용    
하나의 데이터가 여러개의 하위 데이터를 포함 하고 있을 때,(JOIN) ResultMap 을 사용하면, 하나의 쿼리로 MemberVO와 내부 AuthVO의 리스트까지 처리가능하다. (1:N 결과 처리)

3. CustomUserDetailsService 구성

시큐리티의 UserDetailsService 를 구현하고, MemberMapper타입의 인스턴스를 주입받아 기능을 구현한다. 

*security-context.xml*
```xml
<!-- Bean 등록 -->
<bean id="customUserDetailsService" class="org.zerock.security.CustomUserDetailsService"></bean>
...
<security:authentication-manager>
	<security:authentication-provider user-service-ref="customUserDetailsService">
		<security:password-encoder ref="bcryptPasswordEncoder"/>
	</security:authentication-provider>
</security:authentication-manager>
```

4. MemberVO 를 UsersDetails 타입으로 변환

기존의 클래스를 수정하지 않고 확장하는 방식이다.

*CustomUser.java*
```java
package org.zerock.security.domain;

import java.util.Collection;
import java.util.stream.Collectors;

import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.core.userdetails.User;
import org.zerock.domain.MemberVO;

import lombok.Getter;

@Getter
public class CustomUser extends User {
	
	private static final long serialVersionUID = 1L;
	private MemberVO member;
	
	/**
	 * User 클래스의 생성자 
	 * @param username
	 * @param password
	 * @param authorities
	 */
	public CustomUser(String username, String password, Collection<? extends GrantedAuthority> authorities) {
		super(username, password, authorities);
		// TODO Auto-generated constructor stub
	}

	/**
	 * MemberVO의 인스턴스를 스프링 시큐리티의 UserDetails 타입으로 변환 
	 * @param vo
	 */
	public CustomUser(MemberVO vo) {
		super(vo.getUserid(), vo.getUserpw(), vo.getAuthList().stream().map(auth -> 
		new SimpleGrantedAuthority(auth.getAuth())).collect(Collectors.toList()));
		
		this.member = vo;
	}
}

```


*CustomUserDetailsService.java*
```java
package org.zerock.security;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.zerock.domain.MemberVO;
import org.zerock.mapper.MemberMapper;
import org.zerock.security.domain.CustomUser;

import lombok.Setter;
import lombok.extern.log4j.Log4j;

@Log4j
public class CustomUserDetailsService implements UserDetailsService {

	@Setter(onMethod_ = @Autowired)
	private MemberMapper memberMapper;
	
	/**
	 * CustomUser 를 반환하도록 한다. 
	 */
	@Override
	public UserDetails loadUserByUsername(String userName) throws UsernameNotFoundException {
		
		log.warn("Load User By UserName : " + userName);
		
		// userName means userid 
		MemberVO vo = memberMapper.read(userName);
		
		log.warn("queried by member mapper : " + vo);
		
		return vo == null ? null : new CustomUser(vo);
	}
}
```

* org.springframework.security.core.userdetails.User 를 상속하기 때문에 부모 클래스의 생성자를 호출 한 후 객체를 생성한다.    
* AuthVO 인스턴스는 GrantedAuthority 객체로 변환해야 하므로 stream(). map()으로 처리한다. 
* MemberVO의 인스턴스를 얻으면 CustomUser타입의 객체로 리턴한다. 

<img src="https://user-images.githubusercontent.com/22673024/80279527-388ef500-8739-11ea-8411-fb847a69a918.png" width=60%>
