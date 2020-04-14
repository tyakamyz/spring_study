# Annotation

 1. [@RunWith](#RunWith)
 1. [@ContextConfiguration](#ContextConfiguration)
 1. [@Test](#Test)
 1. [@WebAppConfiguration](#WebAppConfiguration)

# Explanation

## @RunWith
 - Junit 프레임워크의 테스트 실행 방법을 확장할 때 사용하는 어노테이션
 - SpringJUnit4ClassRunner라는 JUnit용 테스트 컨텍스트 프레임워크 확장 클래스를 지정해주면 JUnit이 테스트를 진행하는 중에 테스트가 사용할 애플리케이션 컨텍스트를 만들고 관리하는 작업을 진행해준다.


**[⬆ go to top](#Annotation)**

----
## @ContextConfiguration
 
 - @RunWith를 통해 자동으로 만들어줄 애플리케이션 컨텍스트의 설정파일위치를 지정한 것이다.
 - 경로 미지정시 아래의 xml 파일을 읽는다.(대소문자 구분X)
    - ContextConfigLocationTest-context.xml
    - contextconfiglocationtest-context.xml

**[⬆ go to top](#Annotation)**

----
## @Test

 - JUnit Test작업 대상임을 알려주는 어노테이션

**[⬆ go to top](#Annotation)**

----
## @WebAppConfiguration

 - 웹 어플리케이션 컨텍스트를 사용 가능하게 해주는 어노테이션
 - 별도의 was 기동 없이 테스가 가능하게 끔 한다.

**[⬆ go to top](#Annotation)**

----

- 출처 : https://countryxide.tistory.com/17