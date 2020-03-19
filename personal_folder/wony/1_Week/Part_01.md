# **Part 01** 개발을 위한 준비


## **Chapter 01** 개발 환경 설정


### 1. Java Configuration 을 하는 경우 필요한 작업
>   + web.xml의 파일 삭제 및 스프링 관련 파일 삭제  
>    (web.xml, servlet-context.xml, root-context.xml)
>   - pom.xml의 수정 및 스프링 버전 변경
>   - Java 설정 관련 패키지 생성


## **Chapter** 02 스프링의 특징과 의존성 주입

### 1. 스프링의 주요 특징
 >  - POJO 기반의 구성
 >  - 의존성 주입(DI)을 통한 객체 간의 관계 구성
 >  - AOP(Aspect-Oriented-Programming) 지원
 >  - 편리한 MVC 구조
 >  - WAS의 종속적이지 않은 개발 환경

### 2. @Component
 >   - Spring에게 해당 클래스가 Spring에서 관리해야하는 대상임을 표시

### 3. @Setter(onMethd = @__({@Autowired}))

 |속성명| 의미|
 |-----------------|------------------|
 |value|접근 제한 속성을 의미 </br> 기본값은  lombok.AccessLevel.PUBLC|
 |onMethod|setter 메서드의 생성시 메서드에 추가할 어노테이션 지정 </br> JDK 버전에 따라 '_' 표기차이가 있음  </br>up to JDK7 : @Setter(onMethod = @__({@AnnotationsGoHere})  </br>from JDK8 : @Setter(onMethod_={@AnnotationsGohere})|
 |onParam           |setter 메서드의 파라미터에 어노테이션을 사용하는경우 적용|


 >  - 자동으로 SetChef()를 컴파일 시 생성
 >  - onMethod 속성은 생성되는 setChef()에 @Autowired 를 추가
 >  - @Autowired 는 상황에 맞는 타입(Class)의 Bean을 가져와서 주입
 >  - @Resource 는 상황에 맞는 이름(Id)의 Bean을 가져와서 주입

 ### 4. Spring이 동작하며 생기는 일
 >  - Spring FrameWork가 시작 되면 Spring이 사용하는 메모리 영역을 만듦 - **Context** 라  하며 **ApplicationContext**라는 이름의 객체가 만들어진다.
 >  - Spring은 자신이 객체를 생성하고 관리해야 하는 객체들에 대한 설정이 필요 - **root-context.xml**
 >  - root-context.xml에 설정되어있는 **<<context:component-scan>>** 태그의 내용을 통해서 **org.zerock.sample** 패키지를 스캔한다.
 >  - 해당 패키지에 있는 클래스들 중 Spring이 사용하는 @Component라는 어노테이션이 존재하는 클래스의 인스턴스를 생성
 >  - Restaurant 객체는 Chef 객체가 필요하다는 어노테이션 설정이 있으므로 , Spring은 Chef 객체의 레퍼런스를 Restaurnat 객체에 주입

 ### 5. Test
 >  - 현재 테스트 코드가 스프링을 실행하는 역할을 할 것이라는 것을 **@RunWith** 로 표현
 >  - 지정된 클래스나 문자열을 이용해 필요한 객체들을 스프링 내에 객체로 등록(스프링 빈으로 등록한다),  사용하는 문자열은 **'classpath:'** 나 **'fine:'**을 이용한다. 자동으로 상성된 **root-context.xml의 경로** 를 지정 - **@ContextConfiguration**
 >  - Junit 진행중 ApplicationContext를 만들고 관리하는 작업을 진행, 테스트 별로 객체 생성하더라도 싱글톤 ApplicationContext 를 보장
 ---------
 #### * 중요사항 
 >  1. 테스트 코드가 실행되기 위해 Spring FrameWork가 동작
 >  2. 동작하는 과정에서 필요한 객체들이 Spring에 등록
 >  3. 의존성 주입이 필요한 객체는 자동으로 주입이 이루어짐


 #### * 코드에 사용된 어노테이션
 |Lombok 관련       |  Spring 관련      | Test 관련
 |-----------------|------------------|---------
 |@Setter          |@Autowired        |@RunWith
 |@Data            |@Component        |@ContextConfiguration
 |@Log4j           |                  |@Test

### 6. 생성자 자동주입
>   - 인스턴스 변수로 선언된 모든 것을 파라미터로 받는 생성자 작성 - **@AllArgsConstructor**
>   - 여러 개의 인스턴스 변수들 중 특정한 변수에 대해서만 생성자를 작성 -  **@NonNull** 과 **@RequiredArgsConstructor** 

## **Chapter 03** 스프링과 Oracle Database 연동