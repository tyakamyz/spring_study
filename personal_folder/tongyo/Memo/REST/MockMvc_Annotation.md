# MockMvc의 @Annotation 

<details markdown="1">
<summary>@RunWith(SpringRunner.class)</summary>

- ## @RunWith(SpringRunner.class)
- 테스트를 진행할 때 JUnit에 내장된 실행자 외에 다른 실행자를 실행시킴
- 여기서는 SpringRunner라는 스프링 실행자를 사용
- 즉, 스프링 부트 테스트와 JUnit 사이에 연결자 역할을 함

</details>

--------

<details markdown="1">
<summary>@WebMvcTest</summary>

- ## @WebMvcTest
- 여러 스프링 테스트 어노테이션 중, Web(Spring MVC)에 집중할 수 있는 어노테이션
- 선언할 경우 @Controller, @ControllerAdvice 등을 사용할 수 있음
- 단 @Service, @Component, @Repository 등은 사용할 수 없음

</details>

--------

<details markdown="1">
<summary>@Autowired</summary>

## @Autowired
- 스프링이 관리하는 빈(Bean)을 주입 받음

</details>