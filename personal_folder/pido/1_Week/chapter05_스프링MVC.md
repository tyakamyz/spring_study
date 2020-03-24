[Part1] - chapter 05
=========================

스프링 MVC 
--------
- 스프링 서브(sub) 프로젝트
- root-context.xml 로 사용하는 JAVA영역(POJO)과 servlet-context.xml 로 설정하는 Web 관련 영역을 연동하여 구동하는 방식. 

#### pom.xml 변경 
> - java-version(1.6 → 1.8)   
> - springframework-version(3.1.1 → 5.0.7)   
> - spring-test(springVersion) 와 lombok(1.18.0) 라이브러리 추가
> - javax.servlet-api(2.5 → 3.1.0) 서블릿 버전  
> - maven-compiler(1.6 → 1.8) 컴파일 옵션  
*project - Maven - update project *   

#### JAVA 기반 설정   
> - web.xml, servlet-context.xml, root-context.xml 제거   
> - pom.xml 에 web.xml 미사용 설정 추가 
> - servlet-context.xml을 대신할 RootConfig.java, WebConfig.java, servletConfig.jvav(Spring MVC 이용시) 클래스 생성 

[☝ 관련 설정 소스 참고](https://github.com/tyakamyz/spring_study/blob/master/personal_folder/pido/1_Week/chapter01_%EA%B0%9C%EB%B0%9C%ED%99%98%EA%B2%BD%EC%84%A4%EC%A0%95.md)
