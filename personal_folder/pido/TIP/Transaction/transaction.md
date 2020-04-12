
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

    ```java
    @Transactional(propagation = Propagation.NEVER)
    public void transactionService() {
        ...
    }    
    ```

    참고예제 : http://wonwoo.ml/index.php/post/966

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
