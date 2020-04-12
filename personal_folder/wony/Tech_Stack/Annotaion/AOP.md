# Annotation

 1. [@Before](#Before)
 1. [@After](#After)
 1. [@AfterReturning](#AfterReturning)
 1. [@AfterThrowing](#AfterThrowing)
 1. [@Around](#Around)
 1. [@Aspect](#Aspect)
 1. [@EnableAspectJAutoProxy](#EnableAspectJAutoProxy)
 1. [@Pointcut](#Pointcut)

# Explanation

## @Before
 
- 아래의 targetMethod()로 정의된 pointcut 이전에 수행된다.
```java
@Aspect
public class AspectUsingAnnotation {
    
    @Before("targetMethod()")
    public void beforeTargetMethod(JoinPoint thisJoinPoint) {
        Class clazz = thisJoinPoint.getTarget().getClass();
        String className = thisJoinPoint.getTarget().getClass().getSimpleName();
        String methodName = thisJoinPoint.getSignature().getName();
        
    }
}
```

**[⬆ go to top](#Annotation)**

----
## @After

 - 메소드 수행후 무조건 수행되는 메소드
 - 정상 종료와, 예외 발생 경우를 모두 처리해야하는 경우에 사용된다.(리소스 해제 등)
 - 아래의 targetMethod()로 정의된 pointcut 이후 수행
 ```java
 @Aspect
public class AspectUsingAnnotation {
    ..
    @After("targetMethod()")
    public void afterTargetMethod(JoinPoint thisJoinPoint) {
        System.out.println("AspectUsingAnnotation.afterTargetMethod executed.");
    }
}
 ```
 
**[⬆ go to top](#Annotation)**

----
## @AfterReturning
 
- 정상적으로 메소드가 실핼될 때 수행하는 어노테이션
- 아래의 targetMethod()로 정의된 pointcut 후에 수행된다, 결과는 retVal 변수에 저장된다.
```java
@Aspect
public class AspectUsingAnnotation {
    ..
    @AfterReturning(pointcut = "targetMethod()", returning = "retVal")
    public void afterReturningTargetMethod(JoinPoint thisJoinPoint,
            Object retVal) {
        System.out.println("AspectUsingAnnotation.afterReturningTargetMethod executed." + 
                           " return value is [" + retVal + "]");
 
    }
}
```

**[⬆ go to top](#Annotation)**

----
## @AfterThrowing

 - 메서드가 수행 중 예외사항을 반호나하고 종료하는 경우 수행된다.
 - targetMethod()로 정의된 pointcut에서 예외가 발생한 후에 수행된다.
 - targetMethod() pointcut에서 발생된 예외는 exception 변수에 저장되어 전달된다.
 - 아래 예제로는 전달받은 예외를 사용자가 쉽게 실별될 수있도록 메시지로 설정하여 반환
 ```java
 @Aspect
public class AspectUsingAnnotation {
    ..
    @AfterThrowing(pointcut = "targetMethod()", throwing = "exception")
    public void afterThrowingTargetMethod(JoinPoint thisJoinPoint,
            Exception exception) throws Exception {
        System.out.println("AspectUsingAnnotation.afterThrowingTargetMethod executed.");
        System.out.println("에러가 발생했습니다.", exception);
 
        throw new BizException("에러가 발생했습니다.", exception);
    }
}
 ```
 
**[⬆ go to top](#Annotation)**

----
## @Around

 - 메소드 수행 전/후에 수행된다.
 - 다른 Advice들과는 다르게 파라미터로 ProceedingjoinPoint를 전달하며 proceed() 메소드 호출을 통해 대상의 pointcut을 실행한다, pointcut 수행 결과값 retVal을 Around 내에서 변환하여 변환할 수 있음을 보여준다.
 ```java
 @Aspect
public class AspectUsingAnnotation {
    ..
    @Around("targetMethod()")
    public Object aroundTargetMethod(ProceedingJoinPoint thisJoinPoint)
            throws Throwable {
        System.out.println("AspectUsingAnnotation.aroundTargetMethod start.");
        long time1 = System.currentTimeMillis();
        Object retVal = thisJoinPoint.proceed();
 
        System.out.println("ProceedingJoinPoint executed. return value is [" + retVal + "]");
 
        retVal = retVal + "(modified)";
        System.out.println("return value modified to [" + retVal + "]");
 
        long time2 = System.currentTimeMillis();
        System.out.println("AspectUsingAnnotation.aroundTargetMethod end. Time(" + (time2 - time1) + ")");
        return retVal;
    }
}
 ```
 
**[⬆ go to top](#Annotation)**

----
## @Aspect

 - 관점 정의 하는 어노테이션으로. 설정된 클래스가 관점로직임을 알린다
 - Spring AOP 설정에 자동으로 반영된다.
 
**[⬆ go to top](#Annotation)**

----
## @EnableAspectJAutoProxy
 
- AOP를 사용하고자 하는 Class, Interface에 선언하여 AspectJ의 기능을 ON해준다

**[⬆ go to top](#Annotation)**

----
## @Pointcut
 
 - 결합점(join point)를 지정하여 주변로직(Advice)가 언제 실핼 될지를 지정하는데 사용 하는 어노테이션
 - 지정자
    - execution: 메소드 실행 결합점(join points)과 일치시키는데 사용된다.
    - within: 특정 타입에 속하는 결합점을 정의한다.
    - this: 빈 참조가 주어진 타입의 인스턴스를 갖는 결합점을 정의한다.
    - target: 대상 객체가 주어진 타입을 갖는 결합점을 정의한다.
    - args: 인자가 주어진 타입의 인스턴스인 결합점을 정의한다.
    - @target: 수행중인 객체의 클래스가 주어진 타입의 어노테이션을 갖는 결합점을 정의한다.
    - @args: 전달된 인자의 런타입 타입이 주어진 타입의 어노테이션을 갖는 결합점을 정의한다.
    - @within: 주어진 어노테이션을 갖는 타입 내 결합점을 정의한다.
    - @annotation: 결합점의 대상 객체가 주어진 어노테이션을 갖는 결합점을 정의한다.

**[⬆ go to top](#Annotation)**

----

* 출처 : https://www.egovframe.go.kr/wiki/doku.php?id=egovframework:rte:fdl:aop:aspectj