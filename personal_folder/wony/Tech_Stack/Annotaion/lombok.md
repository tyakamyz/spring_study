# Annotation

 1. [@Getter](#Getter)
 1. [@Setter](#Setter)
 1. [@ToString](#ToString)
 1. [@EqualsAndHashCode](#EqualsAndHashCode)
 1. [@Data](#data)
 1. [@Log4j](#Log4j)
 1. [@AllArgsConstructor](#AllArgsConstructor)
 1. [@RequiredArgsConstructor](#RequiredArgsConstructor)
 1. [@NonNull](#NonNull)
 1. [@NoArgsConstructor](#NoArgsConstructor)
 1. [@Cleanup](#Cleanup)
 1. [@Value](#Value)
 1. [@Builder](#Builder)

# Explanation

## @Getter
 
  - getter() 메서드 생성하지 않도록 지원하는 어노테이션
  - lazy 속성을 통해 동기화를 사용한 설정가능
  - @Getter(lazy=true)
    - 동기화를 이용해서 최초 한번만 getter 호출

**[⬆ go to top](#Annotation)**

----
## @Setter

 - setter() 메서드 생성하지 않도록 지원하는 어노테이션

**[⬆ go to top](#Annotation)**

----
## @ToString

 - 적용된 클래스의 모든 필드들을 key=value 형태로 출력하는 어노테이션
 - exclude 속성을 통해 특정 필드를 제외 할 수 있다.
 - 적용 예)
    ```java
    @ToString(exclude = "password")
    public class User {
    private Long id;
    private String username;
    private String password;
    private int[] scores;
    }
    ```
 -  결과)
    ```java
    System.out.print(user);
    User(id=1, username=1234, scores=[80, 70, 100])
    ```

**[⬆ go to top](#Annotation)**

----
## @EqualsAndHashCode

 - equals, hashCode 자동 생성 어노테이션
    - equals : 두 객체의 내용이 같은지, 동등성(equality)를 비교하는 연산자
    - hashCode : 두 객체가 같은 객체인지, 동일성(identity)를 비교하는 연산자
 - exclude 속성을 통해 특정필드르 제외
 - of 속성을 통해 특정 필드를 포함

**[⬆ go to top](#Annotation)**

----
## @Data

 - @ToString, @EqualsAndHashCode, @Getter, @Setter, @RequiredArgsConstructor를 모두 합쳐놓은 어노테이션

**[⬆ go to top](#Annotation)**

----
## @Log4j

 - 로그 라이브러리중 하나

**[⬆ go to top](#Annotation)**

----
## @AllArgsConstructor

 - 모든 인자를 가진 생성자를 생성

**[⬆ go to top](#Annotation)**

----
## @RequiredArgsConstructor

 - 필수 인자만을 가진 생성자 생성
 - @NonNull과 final 필드와 조합

**[⬆ go to top](#Annotation)**

----
## @NonNull

 - Null값이 될 수 없음을 명시하는 어노테이션
 - NullPointerException에 대한 대비책으로 사용이 가능하다.
 - @RequiredArgsConstructor 어노테이션과 조합하여 사용 가능하다.

**[⬆ go to top](#Annotation)**

----

## @NoArgsConstructor

 - 인자 없는 생성자 생성

**[⬆ go to top](#Annotation)**

----
## @Cleanup

 - 자동 리소스 관리 어노테이션으로 try-catch-finally 문에서 finally쪽에서 close메서드를 통해 닫지 않아도 자동으로 현재 스코프에서 벗어날시 close 된다
 - 예제)
    ```java
    @Cleanup Connection con = DriverManager.getConnection(url, user, password);
    ```

**[⬆ go to top](#Annotation)**

----
## @Cleanup

 - 자동 리소스 관리 어노테이션으로 try-catch-finally 문에서 finally쪽에서 close메서드를 통해 닫지 않아도 자동으로 현재 스코프에서 벗어날시 close 된다
 - 예제)
    ```java
    @Cleanup Connection con = DriverManager.getConnection(url, user, password);
    ```

**[⬆ go to top](#Annotation)**

----
## @Value

 - 프로퍼티에 값을 설정한 후 가져올 경우사용
 - 예) 1. 프로퍼티에 선언
    ```properties
    img.path=/home/upload/images/
    ```
 - 예) 2. application-servlet.xml 파일에 프로퍼티 파일 선언
    ```xml
    <beans xmlns:util="http://www.springframework.org/schema/util"></beans>
    ```
    ```xml
     <util:properties id="config" location="classpath:/config/config.properties"/>
    ```
 - 예) 3. controller에서 선언한 프로퍼티 사용
    ```java
    @Value("#{config['img.path']}")
    String imgPath;
    ```

 *출처 : https://steady-snail.tistory.com/44

**[⬆ go to top](#Annotation)**

----
## @Builder

 - Builde 패턴이란
    - 인스턴스를 생성할 때 생성자만을 사용해 생성하는데는 어려움이있다. 예로 다수의 인자를 필요로 하는 생성자를 생성할 떄 어떠한 인자가 어떠한 값을 나타내는지 확인하기 힘들다. 또한 특정 인자만을 필요로 할 경우에 그 외값을 Null로 주어야하는데 해당 코드는 가독성 면에서도 좋지 않아 Build 패턴을 사용한다.
    - 일반 예)
    ```java
    PersonInfo personInfo = new PersonInfo("Mommoo", 12, 119);
    ```
    - Builde 패턴 예)
    ```java
    PersonInfo personInfo = new Builder("Mommoo").setAge(12).setPhonNumber(119).build( );
    ```

**[⬆ go to top](#Annotation)**

----
