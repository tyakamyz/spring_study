# **Part 02** 스프링 MVC 설정

## **Chapter 05** 스프링 MVC의 기본 구조

 ### 1. 예제에서 사용 하는 구조  

|       |XML설정|JAVA설정
|------|-------|------|
|Spring MVC|servlet-context.xml|ServletConfig.class
|Spring Core|root-context.xml|RootConfig.class
|MyBatis|root-context.xml|RootConfig.class

 ### 2. 스프링 MVC 프로젝트 내부 구조
    - 스프링 MVC 프로젝트를 구성해서 사용한다는 의미는 내부적으로 root-context.xml로 사용하는 일반 Java 영역(POJO)과 servlet-context.xml로 설정하는 Web 관련 영역을 가티 연동해서 구동한다.