> # AOP 용어 정리
- Target
    - 순수한 비즈니스 로직을 의미
    - 어떠한 관심사들과도 관계를 맺지않는 순수한 코어(core)
- Proxy
    - Target을 전체적으로 감싸고 있는 존재
    - Proxy는 내부적으로 Target을 호출하지만, 필요한 관심사들을 거쳐서 Target을 호출하도록 자동 혹은 수동으로 작성됨
    - Proxy는 대부분 Spring AOP 기능을 이용해서 자동으로 생성되는 auto-proxy 방식을 사용
- JoinPoint
    - Target이 가진 여러 메서드 (엄밀하게 스프링 AOP에서는 메서드만이 JoinPoint)
- Pointcut
    - Target에는 여러 메서드(JointPoint)가 존재하기 때문에 어떤 메서드에 관심사를 결합할 것인지를 결정하는 역할
- Aspect
    - 관심사(concern)는 Aspect와 Advice로 표현할 수 있으며, Aspect는 관심사 자체를 의미하는 추상명사
- Advice
    - Advice는 Aspect를 구현한 코드
    - 걱정거리를 분리해 놓은 코드를 의미
        |구분|설명|
        |---|---|
        |Before Advice|Target의 JoinPoint를 호출하기 전에 실행되는 코드.<br>코드의 실행 자체에는 관여할 수 없음|
        |After Returning Advice|모든 실행이 정상적으로 이루어진 후에 동작하는 코드|
        |After Throwing Advice|예외가 발생한 뒤에 동작하는 코드|
        |After Advice|정상적으로 실행되거나 예외가 발생했을 때 구분 없이 동작하는 코드|
        |Around Advice|메서드의 실행 자체를 제어할 수 있는 가장 강력한 코드.<br>직접 대상 메서드를 호출하고 결과나 예외를 처리할 수 있음|