# 자동 로그인(remember-me) 설정
- 자동 로그인 기능을 처리하는 방식 중에서 가장 많이 사용되는 방식은 로그인 되었던 정보를 데이터베이스에 저장해 두었다가 사용자가 재방문 시 세션에 정보가 없으면 데이터베이스를 조회하는 방식
- 데이터베이스에 정보가 공유되기 때문에 좀 더 안정적인 운영이 가능함
-----
> 스프링 시큐리티의 공식 문서에 나오는 로그인 정보를 유지하는 테이블
```sql
create table persistent_logins(
  username varchar2(64) not null,
  series varchar2(64) primary key,
  token varchar2(64) not null,
  last_used timestamp not null
);
```
> 주로 사용되는 속성

|속성값|설명|
|---|---|
|key|쿠키에 사용되는 값을 암호화하기 위한 키 값|
|data-source-ref|DataSource를 지정하고 테이블을 이용해서 기존 로그인 정보를 기록(옵션)|
|remember-me-cookie|브라우저에 보관되는 쿠키의 이름을 지정. 기본값은 'remember-me'|
|remember-me-parameter|웹 화면에서 로그인할 때 'remember-me'는 대부분 체크박스를 사용하여 처리. 이때 체크박스 태그는 name 속성을 의미|
|token-validity=seconds|쿠키의 유효시간을 지정|
> security-context.xml
```xml
<security:http>
		
  <security:remember-me data-source-ref="dataSource" token-validity-seconds="604800" />
  
</security:http>
```
> customLogin.jsp
```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
	pageEncoding="UTF-8"%>
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c"%>

<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>Insert title here</title>
</head>
<body>

	<h1>Custom Login Page</h1>
	<h2>
		<c:out value="${error}" />
	</h2>
	<h2>
		<c:out value="${logout}" />
	</h2>

	<form method='post' action="/login">

		<div>
			<input type='text' name='username' value='admin'>
		</div>
		<div>
			<input type='password' name='password' value='admin'>
		</div>
		<div>
			<input type="checkbox" name="remember-me"> Remember Me
		</div>

		<div>
			<input type='submit'>
		</div>
		<input type="hidden" name="${_csrf.parameterName}" value="${_csrf.token}" />
	</form>

</body>
</html>
```
-----
# 로그아웃 시 쿠키 삭제
> security-context.xml
```xml
<security:http>

    <!-- JSESSION_ID는 Tomcat에서 발행하는 쿠키의 이름 -->
    <security:logout logout-url="/customLogout" invalidate-session="true" delete-cookies="remember-me, JSESSION_ID"/>
    
    <security:remember-me data-source-ref="dataSource" token-validity-seconds="604800" />
    
</security:http>
```