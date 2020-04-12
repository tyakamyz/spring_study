
AOP
-----------
<img src="https://user-images.githubusercontent.com/22673024/79061388-c21add80-7cca-11ea-80db-3244afc2f249.png" width="60%">

붉은색 계열의 글자들인 로그나 시간체크 등의 중복코드들은 **횡단 관심사**라고 하며,   
푸른계열의 글자들을 실제 비즈니스 로직을 수행하는 부분이므로 **핵심 관심사**라고 한다.   
이러한 <U>횡단 관심사를 분리하여 핵심 관심사에만 집중하게 해주는 것</U>이 **AOP** 이다. 

##### 출처 : https://blog.azulpintor.io/entry/spring4-aop-usage-example-simply

<hr>

## ◈ AOP 용어

<img src="https://user-images.githubusercontent.com/22673024/79060763-00f96500-7cc4-11ea-9c76-5332267ebe63.png" width="60%">

→ AOP에서 **공통 모듈들을 모듈화하는 것**이 Aspect

<hr>

## ◈ 스프링이 제공하는 어드바이스

|형태            | 설명	 
|----------------|------------------------------------------------
|Before          | 조인포인트 앞에서 실행할 어드바이스
|Aftere     	 | 조인포인트 뒤에서 실행할 어드바이스
|AfterReturning	 | 조인포인트가 완전히 정상종료한 다음 실행되는 어드바이스
|Around          | 조인포인트 앞뒤에서 실행되는 어드바이스
|AfterThrowing   | 조인포인트에서 예외가 발생했을 때 실행되는 어드바이스 

<br>
<details markdown="1">
<summary> 각각의 어드바이스 형태보기 </summary>
<img src="https://user-images.githubusercontent.com/22673024/79061003-70705400-7cc6-11ea-94c9-1a9197e63d96.jpg" width="60%">

##### 출처 : 스프링4 입문 웹 애플리케이션의 기초부터 클라우드 네이티브 입문까지
</details>

<hr>

## ◈ AOP 사용

```java
package org.zerock.aop;

import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.springframework.stereotype.Component;

import lombok.extern.log4j.Log4j;

@Aspect
@Log4j
@Component
public class LogAdvice {

	@Before( "execution(* org.zerock.service.SampleService*.*(..))")
	public void logBefore() {
		log.info("==============================");
	}
}
```
 * @Aspect : 어노테이션을 사용하여 AOP임을 알려준다.   
 * @Component : bean으로 등록한다.
 * AOP를 수행하는 지점을 Pointcut이라 하고, @Pointcut을 사용하여 범위를 지정할 수 있는데, 이 패턴은 "AspectJ pointcut 문법"을 기본적으로 사용한다.


### "AspectJ pointcut 문법"
- 어노테이션 괄호 안에 <U>execution(* findProduct(String))</U> 형식으로 기술한다.
- execution에서는 호출되는 쪽의 메서드를 조건으로 포인트컷을 기술한다.
```
- execution(* *(..)) : 모든 메소드 수행   
- execution(public * *(..)) : 접근 제어 지정자가 public인 모든 메소드 수행   
- execution(public !static * *(..)) : 접근 제어 지정자가 public이며, static이 아닌 메소드 수행   
- execution(* 패키지..controller.*.*(..)) : 패키지 하위에 있는 controller 패키지의 하위에 존재하는 모든 메소드 수행
 
문법이 틀렸을 때에는 java.lang.IllegalArgumentException: Pointcut is not well-formed과 같은 에러를 뱉는다.   
문법은 틀리지 않았으나 없는 패키지나 메소드를 지정 시 AOP가 동작하지 않으니 주의.
```

<hr>

## ◈ Proxy 

AOP에는 여러가지 방식이 있는데, <aop:aspectj-autoproxy /> 이 방식은 proxy를 두는 방식입니다.

proxy란 어떤 핵심 기능을 수행 하기 전과 후에 공통 기능을 수행한다고 할 때, aspect가 곧바로 핵심 기능에서 실행되는 것이 아니라,   
proxy(대행자)에서 공통 기능이 수행하도록 하는 것입니다.

즉 proxy는 핵심 기능을 하는 메서드에서 핵심 기능을 수행하고 다시 돌아와서 공통 기능을 수행하는 로직을 거치게 됩니다.


개발을 하면서 어디에 횡단 관심 모듈을 삽입할 지 알 수 없으므로 spring-servlet.xml 파일에도 <aop:aspectj-autoproxy /> 를 추가하는 것이 좋습니다.

참고 사이트 : https://victorydntmd.tistory.com/178
