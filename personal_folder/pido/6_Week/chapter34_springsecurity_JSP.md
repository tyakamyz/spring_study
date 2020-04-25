
[Part7] - chapter 34
=========================

스프링 시큐리티를 JSP 에서 사용하기
---------------

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"  pageEncoding="UTF-8"%>

<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>
<%@ taglib uri="http://www.springframework.org/security/tags" prefix="sec" %>

<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">

<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>Insert title here</title>
</head>
<body>
<h1>/sample/admin page</h1>


<p>principal : <sec:authentication property="principal"/></p>
<p>MemberVO : <sec:authentication property="principal.member"/></p>
<p>사용자이름 : <sec:authentication property="principal.member.userName"/></p>
<p>사용자 아이디 : <sec:authentication property="principal.username"/></p>
<p>사용자 권한 리스트 : <sec:authentication property="principal.member.authList"/></p>


<a href="/customLogout">Logout</a>


</body>
</html>
```

<sec:authentication property="principal"/> : UserDetailsService에서 반환된 객체(CustomUserDetailsService를 이용하면 반환된 CustomUser 객체)

<img src="https://user-images.githubusercontent.com/22673024/80280665-662b6c80-8740-11ea-8198-5c7ccd80d607.png" width=60%>

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"  pageEncoding="UTF-8"%>
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>
<%@ taglib uri="http://www.springframework.org/security/tags" prefix="sec" %>

<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>Insert title here</title>
</head>
<body>
<h1>/sample/all page</h1>

	<sec:authorize access="isAnonymous()">
		<a href="/customLogin">로그인</a>
	</sec:authorize>
	
	<sec:authorize access="isAuthenticated()">
		<a href="/customLogout">로그아웃</a>
	</sec:authorize>

</body>
</html>
```

<hr>


### 스프링 시큐리티 표현식 

|표현식                     |  설명	 
|---------------------------|----------------------
|hasRole([role]) , hasAuthority([authority])   | 해당 권한이 있으면 true
|hasAnyRole([role,role2]) , hasAuthority([authority])    | 여러 권한들 중에서 하나라도 해당하는 권한 있으면 true
|principal	        | 현재 사용자 정보를 의미
|permitAll	        | 모든 사용자에게 허용
|denyAll            | 모든 사용자에게 거부
|isAnonymous()      | 익명의 사용자의 경우(로그인을 하지않은 경우도 해당)
|isAuthenticated()   | 인증된 사용자면 true
|isFullyAuthenticated() | Remember-me 로 인증된 것이 아닌 인증된 사용자인 경우 true

