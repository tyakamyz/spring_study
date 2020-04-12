
AOP Annotation
=======================================
Index   
--------

[1. @Aspect](#◈-@Aspect)   
[2. @Component](#◈-@Component)     
[3. @Pointcut](#◈-@Pointcut)   
[4. @Before](#◈-@Before)   
[5. @After](#◈-@After)   
[6. @AfterReturning](#◈-@AfterReturning)    
[7. @Around](#◈-@Around)   
[8. @AfterThrowing](#◈-@AfterThrowing)    

------------------------------

## ◈ @Aspect
* @Aspect를 선언하여 해당 클래스를 AOP클래스로 사용하겠다는 의미
```java
@Aspect
@Log4j
@Component
public class LogAdvice {
    ...
}    
```

<hr>

## ◈ @Component
* @Bean과 동일하게 Spring Bean 등록 어노테이션
```java
@Aspect
@Log4j
@Component
public class LogAdvice {
    ...
}    
```
<hr>

## ◈ @Pointcut
* 어노테이션 설정으로 포인트 컷을 선언할 경우 사용
* Aspect에서 마치 변수와 같이 재사용 가능한 Pointcut을 정의할 수 있다. 이를 사용하여 미리 지정된 메소드명으로 표현식을 그대로 사용할 수 있다.
```java
@Pointcut("execution(* lee..*Impl.*(..))")
public void annotationAOP() {

}
```
* 포인트컷 표현식에는 AND, OR, NOT와 같은 관계연산자를 이용할 수 있다.
```java
@Aspect
public class Performance {

    @Pointcut("execution(* com.blogcode.board.BoardService.getBoards(..))")
    public void getBoards(){}

    @Pointcut("execution(* com.blogcode.user.UserService.getUsers(..))")
    public void getUsers(){}

    @Around("getBoards() || getUsers()")
    public Object calculatePerformanceTime(ProceedingJoinPoint proceedingJoinPoint) {
        ...

```
<hr>

## ◈ @Before
* 비즈니스 메소드 실행 전에 동작 
```java
@Before("annotationAOP()")
public void before(JoinPoint joinPoint){
    System.out.println("----------------------Annotation-----------beforeAOP");
}
```

<hr>

## ◈ @After
* 비즈니스 메소드가  실행된 후 무조건 실행 
```java
@After("annotationAOP()")
public void after(){
    System.out.println("----------------------Annotation-----------afterAOP");
}
```

<hr>

## ◈ @AfterReturning
*  비즈니스 메소드가 성공적으로 리턴되면 동작
```java
@AfterReturning(pointcut="annotationAOP()",  returning="retValue")
public void afterReturningAOP(JoinPoint joinPoint, Object retValue){
    System.out.println("---------------Annotation--------------afterReturningAOP");

}
```

<hr>

## ◈ @Around
*  호출 자체를 가로채 비즈니스 메소드 실행 전후에 처리할 로직 삽입
```java
@Around("annotationAOP()")
public Object measure(ProceedingJoinPoint joinPoint) throws Throwable {
    long start = System.nanoTime(); //현재의 나노시간을 반환
    try {
        System.out.println("---------------Annotation--------------AroundAOP");
        Object result = joinPoint.proceed(); // 대상객체의 메서드 실행(ProceedingJoinPoint 타입은 대상 객체의 메서드를 호출할 때 사용)
        return result;
    } finally {
        System.out.println("---------------Annotation--------------AroundAOP");
        long finish = System.nanoTime();
        Signature sig = joinPoint.getSignature(); //메서드의 시그니쳐
        System.out.printf("%s.%s(%s) 실행 시간 : %d ns\n",
        joinPoint.getTarget().getClass().getSimpleName(),
        sig.getName(),
        Arrays.toString(joinPoint.getArgs()), //인자목록을 반환
        (finish - start));
    }
}
```

<hr>

## ◈ @AfterThrowing
* 비즈니스 메소드 실행 중 예외가 발생하면 동작
```java
@AfterThrowing(pointcut="annotationAOP()", throwing="ex") //예외값 지정
public void after_throwing(Throwable ex){
    System.out.println("---------------Annotation--------------AfterThrowing");
}
```


##### 참고: https://lee-mandu.tistory.com/28 [개발/일상_Mr.lee]
##### 참고: https://jojoldu.tistory.com/69?category=635883
