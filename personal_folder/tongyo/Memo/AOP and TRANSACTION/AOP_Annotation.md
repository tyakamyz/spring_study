># AOP의 @Annotation 

<details markdown="1">
<summary>주로 사용되는 설정</summary>

- ## 주로 사용되는 설정

    - 과거의 스프링에서는 별도의 인터페이스로 구현되고 이를 클래스로 구현하는 방식으로 제작했으나, 스프링 3버전 이후에는 어노테이션만으로도 모든 설정이 가능
    - Target에 어떤 Advice를 적용할 것인지 XML 설정이 가능하지만, 어노테이션을 통해서도 사용이 가능함
    - AOP에서 Target은 Pointcut에 의해서 자신에게는 없는 기능들을 가지게 됨.
        |구분|설명|
        |---|---|
        |execution(@execution)|메서드를 기준으로 Pointcut을 설정|
        |within(@within)|특정한 타입(클래스)을 기준으로 Pointcut을 설정|
        |this|주어진 인터페이스를 구현한 객체를 대상으로 Pointcut으로 설정|
        |args(@args)|특정한 파라미터를 가지는 대상들만을 Pointcut으로 설정|
        |@annotation|특정한 어노테이션이 적용된 대상들만을 Pointcut으로 설정|
</details>

-------------

<details markdown="1">
<summary>@Component</summary>

- ## @Component
    - AOP와 직접적인 관계는 없지만 스프링에서 bean으로 인식하기 위해 사용
</details>

-------------

<details markdown="1">
<summary>@Aspect</summary>

- ## @Aspect
    - Aspect : 부가기능을 정의한 코드인 어드바이스(Advice)와 어드바이스를 어디에 적용하지를 결정하는 포인트컷(PointCut)을 합친 개념
    - 해당 Class가 횡단관심사(부가기능) Class임을 알려주는 Annotation
    - Annotation이 부여되었다고 해서 자동으로 Bean으로 등록되는것이 아니므로 따로 Bean으로 등록을 해주는 작업이 필요 (@Component등의 Annotation을 이용 가능)
    ```java
    @Aspect
    @Log4j
    @Component
    public class LogAdvice {
        ...
    }
    ```
</details>

-------------

<details markdown="1">
<summary>@Around</summary>

- ## @Around
    - Advice한 종류로 핵심관심사의 실패여부와 상관없이 전 후로 실행되도록 하는 Advice (Joinpoint 앞과 뒤에서 실행되는 Advice)
    - Annotation의 속성값으로 PointCut을 전달해주어야 함
    ```java
    @Aspect
    @Log4j
    @Component
    public class LogAdvice {
        @Around("execution(* org.zerock.service.SampleService*.*(..))")
        public Object logTime(ProceedingJoinPoint pjp) {
            long start = System.currentTimeMillis();
            
            log.info("Target : " + pjp.getTarget());
            log.info("Param : " + Arrays.deepToString(pjp.getArgs()));
            
            //invoke method
            Object result = null;
            
            try {
                result = pjp.proceed();
            }catch(Throwable e) {
                e.printStackTrace();
            }
            
            long end = System.currentTimeMillis();
            
            log.info("TIME : " + (end - start));
            
            return result;
        }
    }
    ```
</details>

-------------

<details markdown="1">
<summary>@Pointcut</summary>

- ## @Pointcut
    - 횡단관심사(부가기능)이 적용될 JoinPoint들을 정의한 것
    - @Pointcut의 표현식 execution(접근제어자,반환형 패키지를 포함한 클래스 경로 메소드파라미터 )
</details>

-------------

<details markdown="1">
<summary>@Pointcut의 표현식(excution)</summary>

- ## @Pointcut의 표현식
    - execution(접근제어자,반환형 패키지를 포함한 클래스 경로 메소드파라미터 )
    > 예제1
    - 퍼블릭형의 반환형이 없는(public void) 형태의 메소드 중 get으로 시작하는(get*) 모든 메소드중 파라미터가 존재하지 않는(()) 메소드들에게 적용
        ```java
        @Pointcut("execution(public void get*())")
        ```
    > 예제2
    - 첫번째 *기호는 접근제어자와 반환형 모두를 상관하지 않고 적용하겠다는 의미
    - 두번째 *기호는 어떠한 경로에 존재하는 클래스도 상관하지 않고 적용하겠다는 의미
    - 마지막으로 (..)은 메소드의 파라미터가 몇개가 존재하던지 상관없이 실행하는 경우를 의미
        ```java
        @Pointcut("execution(* *(..))")
        ```
    >예제3
    - 첫번째 *기호는 역시 접근제어자, 반환형을 상관하지 않는다는 의미
    - org.zerock.service.SampleService*.*(..)는 해당 SampleService가 포함된 해당 클래스의 메소드가 호출될 때 실행하는 경우를 의미
        ```java
        @Pointcut("execution(* org.zerock.service.SampleService*.*(..))")
        ```
    >예제4
    - 첫번째 *기호는 역시 접근제어자, 반환형을 상관하지 않는다는 의미
    - org.zerock.service.SampleService*.doTest()는 해당 Class의 doTest()(파라미터가 없는) 메소드가 호출될 때 실행하는 경우를 의미
        ```java
        @Pointcut("execution(* org.zerock.service.SampleService*.doTest())")
        ```
    >예제5
    - 첫번째 *기호는 역시 접근제어자, 반환형을 상관하지 않는다는 의미
    - org.zerock.service.SampleService*.doAdd(String, String)는 해당 Class의 doAdd(String, String)(String,String 파라미터를 가진) 메소드가 호출될 때 실행하는 경우를 의미
    - args(str1, str2)를 통해 str1, str2 파라미터를 추적
    - 간단한 파라미터를 찾아서 기록할 때에는 유용하지만 다른 여러종류의 메서드에 적용할 수 없는 단점 (@Around와 ProceedingJoinPoint를 사용하여 여러종류의 메서드에 적용할 수 없는 문제를 해결할 수 있음)
        ```java
        @Pointcut("execution(* org.zerock.service.SampleService*.doAdd(String, String)) && args(str1, str2)")
        ```
    >예제6
    - 위와 거의 비슷하지만 com.java..부분 즉, ..부분이 조금 다른것을 볼 수 있으며, ..의 경우 해당 패키지를 포함한 모든 하위패키지에 적용을 한다는 의미
    - 해석한다면 접근제어자, 반환형을 상관하지 않고 org.zerock 패키지를 포함한 모든 하위디렉토리의 모든 Class의 모든 파라미터가 존재하지 않는 메소드에 적용한다는 의미
        ```java
        @Pointcut("execution(* org.zerock..*.*())")
        ```
</details>

-------------

<details markdown="1">
<summary>@Pointcut의 표현식(within)</summary>

- ## @Pointcut의 표현식(within)
    - 패키지내의 모든 메소드에 적용할 때 사용
    >예제1
    - com.java.ex.하위의 모든 클래스의 모든 메소드에 적용하겠다는 의미
        ```java
        @Pointcut("within(com.java.ex.*)")
        ```
    >예제2
    - com.java.ex.패키지의 하위 패키지를 포함한 모든 클래스의 모든 메소드에 적용하겠다는 의미
        ```java
        @Pointcut("within(com.java.ex..*)")
        ```
    >예제3
    - com.java.ex.Car 클래스안의 모든 메소드에 적용하겠다는 의미
        ```java
        @Pointcut("within(com.java.ex.Car)")
        ```
</details>

-------------

<details markdown="1">
<summary>@Pointcut의 표현식(bean)</summary>

- ## @Pointcut의 표현식(bean)
    >예제
    - 해당 bean id를 가지고있는 bean의 모든 메소드에 적용한다는 의미
        ```java
        @Pointcut("bean(bean_id)")
        ```
</details>

-------------
<details markdown="1">
<summary>@Before</summary>

- ## @Before
    - 타겟의 메서드가 실행되기 이전(before) 시점에 처리해야 할 필요가 있는 부가기능을 정의 (Jointpoint 앞에서 실행되는 Advice)
    ```java
    @Before("execution(* org.zerock.service.SampleService*.*(..))")
	public void logBefore() {
		log.info("========================");
	}
    ```
</details>

-------------

<details markdown="1">
<summary>@After</summary>

- ## @After
    - 타겟의 메서드가 실행된 이후(after) 시점에 처리해야 할 필요가 있는 부가기능을 정의 (Jointpoint 메서드 호출이 종료된 뒤 실행되는 Advice)
    - 예외상황(실패) 여부와 상관없이 실행되기 때문에 정상 실행과 예외상황 발생을 나눠서 작성해야 하는 경우에는 @AfterReturning, @AfterThrowing 를 사용
    ```java
    @After("execution(* org.zerock.service.SampleService*.*(..))")
	public void logAfter() {
		log.info("========================");
	}
    ```
</details>

-------------
<details markdown="1">
<summary>@AfterReturning</summary>

- ## @AfterReturning
    - 타겟의 메서드가 정상적으로 실행된 이후(after) 시점에 처리해야 할 필요가 있는 부가기능을 정의 (Jointpoint 메서드 호출이 정상적으로 종료된 뒤에 실행되는 Advice)
    ```java
     @AfterReturning(value = "execution(* org.zerock.service.SampleService*.*(..))", returning = "returnValue")
    public void logSuccess(ResponseObject returnValue) throws RuntimeException {
        //logging
        //returnValue 는 해당 메서드의 리턴객체를 그대로 가져올 수 있다.
    }
    ```
</details>

-------------

<details markdown="1">
<summary>@AfterThrowing</summary>

- ## @AfterThrowing
    - 타겟의 메서드가 예외를 발생된 이후(after) 시점에 처리해야 할 필요가 있는 부가기능을 정의 (예외가 던져 질때 실행되는 Advice)
    ```java
    @AfterThrowing(pointcut = "execution(* org.zerock.service.SampleService*.*(..))", throwing="exception")
	public void logException(Exception exception) {
		log.info("Exception....!!!!");
		log.info("exception : " + exception);
	}
    ```
</details>

-------------

<details markdown="1">
<summary>@ProceedingJoinPoint</summary>

- ## ProceedingJoinPoint
    - Around Advice에서 사용할 공통 기능 메서드는 대부분 파라미터로 전달 받은 ProceedingJoinPoint의 proceed() 메서드만 호출하면 됨

    > ProceedingJoinPoint 인터페이스의 제공 메서드

    |메서드|설명|
    |---|---|
    |Signature getSignature()|호출되는 메서드에 대한 정보를 구함|
    |Object getTarget()|대상 객체를 구함|
    |Object[] getArgs()|파라미터의 목록을 구함|
    -----
    > org.aspectj.lang.Signature 인터페이스는 호출되는 메서드

    |메서드|설명|
    |---|---|
    |String getName|메서드의 이름을 구함|
    |String toLongString()|메서드를 완전하게 표현한 문장을 구함<br>(메서드의 리턴 타입, 파라미터 타입이 모두 표시됨)|
    |String toShortString()|메서드를 축약해서 표현한 문장을 구함<br>(기본 구현은 메서드의 이름만을 구함)|

    >예제
    ```java
    @Around("execution(* org.zerock.service.SampleService*.*(..))")
	public Object logTime(ProceedingJoinPoint pjp) {
		long start = System.currentTimeMillis();
		
		log.info("Target : " + pjp.getTarget());
		log.info("Param : " + Arrays.deepToString(pjp.getArgs()));
		/*
        INFO : org.zerock.aop.LogAdvice - Target : org.zerock.service.SampleServiceImpl@25243bc1
        INFO : org.zerock.aop.LogAdvice - Param : [123, ABC]
        */

		//invoke method
		Object result = null;
		
		try {
			result = pjp.proceed();
		}catch(Throwable e) {
			e.printStackTrace();
		}
		
		long end = System.currentTimeMillis();
		
		log.info("TIME : " + (end - start));
        // INFO : org.zerock.aop.LogAdvice - TIME : 4
		
		return result;
	}
    ```
</details>