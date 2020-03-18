# Part 01 개발을 위한 준비


## Chapter 01 개발 환경 설정


1. Java Configuration 을 하는 경우 필요한 작업
>   + web.xml의 파일 삭제 및 스프링 관련 파일 삭제  
>    (web.xml, servlet-context.xml, root-context.xml)
>   - pom.xml의 수정 및 스프링 버전 변경
>   - Java 설정 관련 패키지 생성


## Chapter 02 스프링의 특징과 의존성 주입

1. 스프링의 주요 특징
 >  - POJO 기반의 구성
 >  - 의존성 주입(DI)을 통한 객체 간의 관계 구성
 >  - AOP(Aspect-Oriented-Programming) 지원
 >  - 편리한 MVC 구조
 >  - WAS의 종속적이지 않은 개발 환경

 2. @Setter(onMethd = @__({@Autowired}))