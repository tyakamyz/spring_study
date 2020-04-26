
[Part7] - chapter 35
=========================

자동로그인
----------------

```<security:remember-me>```
|용어                     |  설명	 
|-------------------------|----------------------
|key                      | 쿠키에 사용되는 값을 암호화 하기위한 키 값
|data-source-ref          | DataSource를 지정하고 테이블을 이용해서 기존 로그인 정보를 기록
|remember-me-cookie	      | 브라우저에 보관되는 쿠키의 이름을 지정한다. 기본값은 'remember-me'
|remember-me-parameter    | 웹 화면에서 로그인 할 때 'remember-me'는 대부분 체크박스를 이용해 처리하는데, 이떄 체크박스 태그의 name 속성을 의미
|token-validity-seconds   | 쿠키의 유효시간 지정

### 사용 예제

1. sql 작성   
스프링 시큐리티에서 제공하는 로그인 정보 유지 테이블

```sql
create table persistent_logins (
username varchar2(64) not null,
series varchar2(64) primary key,
token varchar2(64) not null,
last_used timestamp not null);
```

2. security-context.xml
```xml
<security:http auto-config="true" use-expressions="true">
	...
	<security:logout logout-url="/customLogout" invalidate-session="true" delete-cookies="remember-me, JSESSION_ID"/>
	
	<security:remember-me data-source-ref="dataSource" token-validity-seconds="604800" />
</security:http>
```

3. 로그인화면 jsp
```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"  pageEncoding="UTF-8"%>
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>

<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>Insert title here</title>
</head>
<body>
	<h1>Custom Login Page</h1>
	<h2><c:out value="${error}" /></h2>
	<h2><c:out value="${logout}" /></h2>
	
	<form method="post" action="/login">
		
		<div>
			<input type="text" name="username" value="admin">
		</div>
		<div>
			<input type="password" name="password" value="admin">
		</div>
		<div>
			<input type="checkbox" name="remember-me"> Remember Me
		</div>
		
		<div>
			<input type="submit">
		</div>
	
		<input type="hidden" name="${_csrf.parameterName}" value="${_csrf.token }" />
	</form>	
	
</body>
</html>
```

* 실행 후 쿠키를 확인해보면 'remember-me' 쿠키가 생성된 것을 확인할 수 있다.   
또한 데이터 베이스 테이블에도 사용자가 로그인한 정보가 남아있는 것을 확인 할 수 있다.
* 로그아웃 시 쿠키 삭제는 security-context.xml 의 delete-cookies 를 활용한다.

