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
<summary></summary>
</details>

-------------

<details markdown="1">
<summary></summary>
</details>

-------------

<details markdown="1">
<summary></summary>
</details>

-------------

<details markdown="1">
<summary></summary>
</details>

-------------

<details markdown="1">
<summary></summary>
</details>

-------------

<details markdown="1">
<summary></summary>
</details>

-------------

<details markdown="1">
<summary></summary>
</details>

-------------

<details markdown="1">
<summary></summary>
</details>

-------------

<details markdown="1">
<summary></summary>
</details>

-------------

<details markdown="1">
<summary></summary>
</details>

-------------

<details markdown="1">
<summary></summary>
</details>

-------------

<details markdown="1">
<summary></summary>
</details>

-------------

<details markdown="1">
<summary></summary>
</details>

-------------