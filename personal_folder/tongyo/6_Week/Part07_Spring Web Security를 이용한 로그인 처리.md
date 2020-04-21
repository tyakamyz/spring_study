# 7장. Part07_Spring Web Security를 이용한 로그인 처리
> ## ch30. Spring Web Security 소개
- 스프링 시큐리티의 기본 동작 방식
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
------------
- Spring Web Security의 설정
  - pom.xml
  ```xml
  <!-- Security -->
  <!-- https://mvnrepository.com/artifact/org.springframework.security/spring-security-core -->
  <dependency>
      <groupId>org.springframework.security</groupId>
      <artifactId>spring-security-web</artifactId>
      <version>5.0.6.RELEASE</version>
  </dependency>
  
  <dependency>
      <groupId>org.springframework.security</groupId>
      <artifactId>spring-security-config</artifactId>
      <version>5.0.6.RELEASE</version>
  </dependency>
  
  <dependency>
      <groupId>org.springframework.security</groupId>
      <artifactId>spring-security-core</artifactId>
      <version>5.0.6.RELEASE</version>
  </dependency>
  
  <dependency>
      <groupId>org.springframework.security</groupId>
      <artifactId>spring-security-taglibs</artifactId>
      <version>5.0.6.RELEASE</version>
  </dependency>
  ```
  ↑↓↑↓←→←→←→↑↓ 
-------------
- 인증(Authentication)과 권한(Authorization)
  - AuthenticationManager(인증 매니저)
    - 스프링 시큐리티에서 가장 중요한 역할을 하는 존재
    - ProviderManager는 인증에 대한 처리를 AuthenticationProvider라는 타입의 객체를 이용해서 처리를 위임함
      <pre>
      [AuthenticationManager]
                ↑
         [ProviderManager] ←→ [AuthenticationProvider]
      </pre>
    - AuthenticationProvider(인증 제공자)는 실제 인증 작업을 진행
    - 이때 인증된 정보에는 권한에 대한 정보를 같이 전달하게 되는데 이 처리는 UserDetailsService 인터페이스에서 사용자의 정보와 사용자가 가진 권한의 정보를 처리해서 반환해줌
      <pre>
      [AuthenticationManager]
                ↑
         [ProviderManager] ←→ [AuthenticationProvider] ⇠⇢ [UserDetailsService]
      </pre>
    - 개발자가 스프링 시큐리티를 커스터마이징 하는 방식은 크게 AuthenticationProvider를 직접 구현하는 방식과 실제 처리를 담당하는 UserDetailsService를 구현하는 방식으로 나누어짐
    - 대부분의 겨우 UserDetailsService를 구현하는 형태를 사용하는 것 만으로도 충분하지만, 새로운 프로토콜이나 인증 구현 방식을 직접 구현하는 경우에는 AuthenticationProvider 인터페이스를 직접 구현해서 사용함