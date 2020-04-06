
[Part5] - chapter 18
=========================

AOP란?
-----------------
- 관점 지향 프로그래밍(Aspect Oriented Programing)
- 관심사의 분리를 추구한다.('관심사' == 주변로직)
- '관심사 + 비즈니스 로직'을 분리해서 별도의 코드로 작성하도록 하고, 실행할 때 이를 결합하는 방식으로 접근한다.
- 스프링은 AOP를 지원한다.   

==> <span style="color:orange">AOP는 **핵심 기능과 공통 기능을 분리** 시켜놓고, 공통 기능을 필요로 하는 핵심 기능들에서 사용하는 방식</span>

<br>

## AOP용어들
> *AOP는 기존의 코드를 수정하지 않고도 관심사들을 엮을 수 있다.*   
- <span style="color:blue">Target</span> : 순수한 비즈니스 로직으로 순수한 코어(core). 어떠한 관심사들과도 관계를 맺지 않는다.    
- <span style="color:blue">Proxy</span> : Target을 전체적으로 감싸고 있는 존재. 내부적으로 Target을 호출.    
중간에 필요한 관심사들을 거쳐 Target을 호출하도록 자동 혹은 수동으로 작성된다.   
 이때 대부분의 경우 스프링 AOP 기능을 이용, 자동으로 생성되는(auto-proxy) 방식을 이용한다.
- <span style="color:blue">JoinPoint</span> : Target의 객체가 가진 메서드.   

==> <span style="color:orange">외부에서 호출은 Proxy 객체를 통해서 Target 객체의 JoinPoint를 호출하는 방식이다.</span>
 
 






