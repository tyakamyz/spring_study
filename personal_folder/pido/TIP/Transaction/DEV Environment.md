트랜잭션 사용 방법
=======================================
### root-context.xml

<img src="https://user-images.githubusercontent.com/22673024/79064729-76762d00-7ce6-11ea-84bf-f85a6b377796.png" width="50%">

> tx 항목 체크 

```xml
<!-- Transaction -->
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource"></property>
</bean>

<tx:annotation-driven />
```
> 트랜잭션을 관리하는 빈(객체)를 등록하고, 어노테이션 기반으로 트랜잭션을 설정할 수 있도록 ```<tx:annotation-driven />``` 태그를 등록한다.   
> 이후에는 트랜잭션이 필요한 상황을 만들어서 어노테이션을 추가하는 방식으로 설정하게 된다. 
