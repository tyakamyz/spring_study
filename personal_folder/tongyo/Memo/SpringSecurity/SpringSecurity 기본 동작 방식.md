# 스프링 시큐리티의 기본 동작 방식
- 서블릿의 여러가지 필터와 인터셉터를 이용하여 처리
    |필터|인터셉터|
    |---|---|
    |서블릿에서 말하는 단순한 필터를 의미|스프링에서 필터와 유사한 역할|
    |스프링과 무관하게 서블릿 자원|스프링의 빈으로 관리되면서 스프링의 컨텍스트 내에 속함|
    |Dispatcher servlet의 앞단에서 정보를 처리|Dispatcher servlet에서 Handler(Controller)로 가기 전에 정보를 처리|
    |J2EE 표준 스펙에 정의 되어 있는 기능|Spring Framework에서 자체적으로 제공하는 기능|
    |인코딩이나 보안 관련 처리와 같은 web app의 전역적으로 처리해야 하는 로직은 필터로 구현|클라이언트에서 들어오는 디테일한 처리(인증, 권한 등)에 대해서는 주로 인터셉터에서 처리|
    -------
- 필터와 인터셉터의 차이를 그림으로 표현

    <img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=http%3A%2F%2Fcfile24.uf.tistory.com%2Fimage%2F2564124C588F496C01B966">

- 출처
    - https://goddaehee.tistory.com/154
    - https://www.leafcats.com/39