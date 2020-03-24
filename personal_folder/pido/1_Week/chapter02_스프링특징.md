[Part1] - chapter 02
=========================

스프링의 특징
-------

1. 스프링프레임워크   
   * 경량 프레임워크 : 특정 기능을 위주로 간단한 jar 파일 등을 이용하여 모든 개발이 가능하도록 구성된 프레임워크
   * 서버중심, Heavy weight, 확장성, 유지보수 
   → 클라이언트중심, Light weight, 모바일중심, 생산성, 안정성, 다양한개발언어

2. POJO 기반 구성(Plain Old Java Object)
   * 객체간의 관계를 구성할 때 별도의 API를 사용하지 않는 구성.
   * 이는 특정 라이브러리나 컨테이너의 기술에 종속적이지 않다는 말! 

3. 스프링의 의존성(DI) 주입 : 
   * ApplicationContext 라는 존재가 필요한 객체들을 생성
필요한 객체들을 주입하는 역할을 해주는 구조
   * 객체와 객체를 분리하여 생성. 객체들을 역는 작업
   * ApplicationContext가 관리하는 객체 → 빈(Bean)
   * 빈과 빈 사이의 의존관계 처리 → XML, 어노테이션, 자바 설정 이용
      >- 생성자를 이용한 주입과 setter 메서드를 이용한 주입으로 의존성 주입을 구현. 주로 XML이나 어노테이션을 이용하여 처리
      >- XML 방식의 의존성 주입 시 , root-context.xml 설정파일을 사용, 스프링 프레임워크에서 관리해야하는 객체(Bean)을 설정 

4. AOP지원(Aspect Oriented Programming)   
   * 반드시 처리가 필요한 부분(횡단 관심사 cross-consern)을 분리해서 제작하는 것이 가능. AOP 는 이러한 횡단 관심사를 모듈로 분리하는 프로그래밍의 패러다임   
   * AspectJ의 문법을 통하여 작성. 
      >- 핵심 비즈니스 로직에만 집중하여 코드 개발 
      >- 코드의 수정을 최소화
      >- 유지보수가 수월한 코드 구성 

5. 트랜잭션 지원 
   * 어노테이션이나 XML 으로 설정할 수 있음.

<hr />

의존성 주입 
-----

**- XML 기반 -** 
   - root-context.xml 파일의 NameSpaces 탭에서 context 항목 체크   
   - Source 탭에서   
      ```<context:component-scan base-package="org.zerock.sample"></context:component-scan>```    
   추가 
   
**- JAVA 기반 -**    
   > ```@ComponentScan``` : XML기반의 root-context.xml 에서 설정한 의존성 주입을 해당 어노테이션을 이용하여 처리 가능   
   
   - RootConfig.java 파일에서   
   ```@ComponentScan(basePackages= {"org.zerock.sample})```   
   어노테이션 추가

.

스프링 실행 시 ```<context:component-scan>``` 태그를 통해 관리해야하는 페이지를 스캔하고 스프링이 사용하는 ```@Component``` 라는 어노테이션이 존재하는 클래스의 인스턴스를 생성한다. 객체가 필요하다는 의미의 어노테이션 ```@Autowired``` 설정으로 필요한 객체의 레퍼런스를 를 해당 객체에 주입한다.   


<hr />

어노테이션
-------

#### Lombok 

> ```@Data``` : Lombok의 setter를 생성하는 기능, 생성자, toString() 등을 자동 생성 (@ToString + @EqualsAndHashCode + @Getter/@Setter + @RequiredArgsConstructor)   
> ```@Setter``` :  자동으로 set함수를 컴파일 시 생성
 > onMethod : 생성되는 set함수에 @Autowired 어노테이션을 추가하도록 한다.      
> ```@Log4j``` : 로그 객체 생성

#### Spring

> ```@Component``` : Spring에게 해당 클래스가 객체로 만들어서 관리해야하는 대상임을 명시   
> ```@Autowired``` : 자신이 특정한 객체에 의존적이므로 자신에게 해당 타입의 빈을 주입해주라는 표시

