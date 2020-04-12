
Transaction
-----------

## ◈ Transaction 사용

```java
@Transactional
@Override
public void addData(String value) {
    
    log.info("mapper1..........................");
    mapper1.insertCol1(value);
    
    log.info("mapper2..........................");
    mapper2.insertCol2(value);
    
    log.info("end..............................");
}
```
* 위와 같이 사용할 메서드 위에 ```@Transactional``` 어노테이션을 추가해주면 된다. 

<hr>

## ◈ 트랜잭션 어노테이션 속성들

|속성                   | 설명	 
|-----------------------|----------------------
|propagation            |트랜잭션 개시할지 등 전파행위에 관한 속성. 
|isolation              |트랜잭션 격리레벨에 관한 속성으로 기본값은 Default레벨이며 실제 사용하는 데이터베이스(JDBC) 등의 기본값을 따릅니다. 
|readOnly               |트랜잭션을 읽기전용으로 지정하는 속성. 최적화 관점에서 지원되는 프로터티이므로 현재 트랜잭션 상태에따라 다르게 동작할 수 있습니다. 
|timeout                |트랜잭션의 타임아웃(초단위)을 지정하는 속성으로 지정하지 않을 경우 사용하는 트랜잭션 시스템의 타임아웃을 따릅니다. 
|rollbackFor           |Checked 예외 발생시에 롤백을 수행할 예외를 지정하는 속성.   
|rollbackForClassName   |rollbackFor와 동일하지만 문자열로 클래스명을 지정하는 속성.    
|noRollbackFor          |Spring의 트랜잭션은 기본적으로 Runtime예외만 롤백처리를 수행하지만 Runtime예외중 특정 예외는 롤백을 수행하지 않아야 할 경우 사용하는 속성.   
|noRollbackForClassName |noRollbackFor와 동일하지만 문자열로 클래스명을 지정하는 속성.

tx 어노테이션 사용법
```java
@Transactional(propagation = Propagation.REQUIRED, rollbackFor = Exception.class, isolation=..., timeout=...) 
public class SomeService { 
    
    @Transactional(propagation = Propagation.REQUIRES_NEW, isolation = Isolation.DEFAULT, timeout = 10) 
    public void operateSome() {

    } 
    ... 
}
```
##### 출처: https://reiphiel.tistory.com/entry/understanding-of-spring-transaction-management-practice 

<br>

1. 전파(Propagation)속성   

    |용어                   | 의미	 
    |-----------------------|----------------------
    |PROPAGATION_MADATORY   | 작업은 반드시 특정한 트랜잭션이 존재한 상태에서만 가능 
    |PROPAGATION_NESTED     | 기존에 트랜잭션이 있는 경우, 포함되어서 실행
    |PROPAGATION_NEVER	    | 트랜잭션 상황하에 실행되면 예외발생
    |PROPAGATION_NOT_SUPPORTED | 트랜잭션이 있는 경우 트랜잭션이 끝날때까지 보류된 후 실행
    |PROPAGATION_REQUIRED   | 트랜잭션이 있으면 그 상황에서 실행, 없으면 새로운 트랜잭션 실행(기본설정)
    |PROPAGATION_REQUIRED_NEW | 대상은 자신만의 고유한 트랜잭션으로 실행
    |PROPAGATION_SUPPORTS   | 트랜잭션을 필요로 하지 않으나, 트랜잭션 상황하에 있다면 포함되어서 실행

    ##### 참고예제 : http://wonwoo.ml/index.php/post/966

2. 격리(Isolation)레벨

    |용어                  | 의미	 
    |----------------------|----------------------
    |DEFAULT               | DB설정, 기본 격리수준(기본 설정)
    |SERIALIZABLE          | 가장 높은 격리(성능저하 우려가 있음)
    |READ_UNCOMMITED       | 커밋되지 않은 데이터에 대한 읽기 허용
    |READ_COMMITED         | 커밋된 데이터에 대해 읽기 허용
    |REPEATEABLE_READ      | 동일 필드에 대해 다중 접근 시 모두 동일한 결과를 보장

3. Read-only 속성
    * true 인 경우, insert, update, delete 실행 시 예외 발생, 기본 설정 false

4. Rollback-for-예외
    * 특정 예외가 발생 시 강제로 Rollback

5. No-rollback-for-예외
    * 특정 예외의 발생 시에는 Rollback 처리되지않음

6. timeout 속성
    * 지정한 시간 내에 해당 메소드 수행이 완료되이 않은 경우 rollback 수행. -1일 경우 no timeout(Default=-1)


<hr>

## ◈ 트랜잭션의 우선순위

* 1순위 메서드의 @Transactional
* 2순위 클래스의 @Transactional
* 3순위  인터페이스의 @Transactional

→ **인터페이스**에는 가장 **기준**이되는 @Transactional 과 같은 설정을 지정하고, **클래스나 메서드**에 **필요한 어노테이션**을 처리하는 것이 좋다. 
