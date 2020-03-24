[Part1] - chapter 01
=========================

#### 1. 개발환경설정 
**java** : openjdk 1.8   
**FrameWork** : STS3(Spring Tool Suite)    
**Server** : tomcat 9  
**DBMS** : Oracle 11g Express Edition  
**SQL Mapper** : Mybatis

-------------------------

1. JDK(openjdk)   
[Download]: https://github.com/ojdkbuild/ojdkbuild   
: 다운로드 받은 파일은 원하는 위치에 압축 해제 
* 환경변수 설정   
    > * 설정 위치 : 내PC 우클릭 속성 - 고급시스템 설정 - 환경변수   
    > * JAVA_HOME : 시스템 변수 새로만들기에서 변수 이름은 JAVA_HOME, 변수 값은 jdk설치경로를 지정
    > * PATH : 시스템 변수 path에 추가로 JDK의 bin 디렉토리를 지정 → %JAVA_HOME%bin
    > * 확인 : cmd 창에서 'javac' 명령어가 제대로 동작하는지 확인 

2. STS3     
[Download]: https://github.com/spring-projects/toolsuite-distribution/wiki/Spring-Tool-Suite-3   
: 다운로드 받은 파일은 압축 해제 후 STS.exe 실행
* 인코딩 설정
    > * 설정 위치 : Window > Preferences > General > Workspace의 
    Text file encoding 부분을 모두 'UTF-8'로 변경
    > * Web 아래 HTML, CSS, JSP 의 Encoding 도 'UTF-8'로 변경

3. tomcat 9    
[Download]: https://tomcat.apache.org/download-90.cgi   
: 다운로드 받은 파일은 원하는 위치에 압축 해제
* 설정
    > * 설정 위치 : Window > Preferences > Server > Runtime Environments > Add 후 Apache 선택 , tomcat 9 경로 설정 

</br>   
</br>   

#### 2. 스프링 프로젝트 생성

<hr />
* File > New > Spring Legacy Project    
스프링 기반 프로젝트를 Maven 기반으로 생성한다.   
프로젝트 최초 생성 시 필요한 코드와 라이브러리 다운 → .m2 > repository 폴더에 저장된다. 프로젝트 라이브러리 관련 에러 발생 시 repository 삭제처리 한다.  
<hr />

1. Lombok 라이브러리 설치    
[Download]: https://projectlombok.org/download   
: Lombok 이용 시 자주 사용하는 getter/setter, toString(), 생성자 등을 자동으로 생성해준다.    
* 실행
    > * cmd 실행 후, 다운로드 된 jar 파일 경로에서 'java -jar lombok.jar' 명령어로 실행   
    > * 실행 후 Specify location 선택 후 설치한 STS 실행경로를 추가(C:\sts\sts.exe) , Quit Installer 실행 
    > * 설치가 끝나면 STS 실행 경로에 **lombok.jar** 파일이 추가된다.
    > * 만약 STS 실행이 되지 않을경우, **sts.ini** 파일에   
        *-vmargs*   
        *-javaagent:lombok.jar*   
     를 추가해준다.    

2. Spring 환경설정

    **- XML 기반 설정 -**   

    * pom.xml 스프링 프레임워크 버전 변경(3.1.1 → 5.0.7)
        > ```<org.springframework-version>5.0.7</org.springframework-version>```   
        > *프로젝트 구조 내 Maven Dependencies 항목을 통해 변경 확인*
    * pom.xml JAVA 버전 변경(1.6 → 1.8)
        > ```<plugin>``` 태그 중 ```<groupId>maven-compiler-plugin</groupId>```의 ```<source>```와 ```<target>```의 버전을 1.8로 변경한다.  

        *이후 프로젝트 우클릭, 'Maven > Update Project' 실행하여 환경 업데이트 완료*

    **- JAVA 기반 설정 -**
    * web.xml 파일 삭제 및 스프링 관련 파일 삭제
        > * web.xml, servlet-context.xml, root-context.xml 삭제
        
    * pom.xml 하단부에 추가 

        >       <plugin>
        >            <groupId>org.apache.maven.plugins</groupId>
        >            <artifactId>maven-war-plugin</artifactId>
        >            <version>3.2.0</version>
        >            <configuration>
        >                <failOnMissingWebXml>false</>failOnMissingWebXml>
        >            </configuration>
        >        </plugin>

    * pom.xml 스프링 프레임워크 버전 변경(3.1.1 → 5.0.7)
        > ```<org.springframework-version>5.0.7</org.springframework-version>```   
        > *프로젝트 구조 내 Maven Dependencies 항목을 통해 변경 확인*

    * pom.xml JAVA 버전 변경(1.6 → 1.8)
        > ```<plugin>``` 태그 중 ```<groupId>maven-compiler-plugin</groupId>```의 ```<source>```와 ```<target>```의 버전을 1.8로 변경한다.  

        *이후 프로젝트 우클릭, 'Maven > Update Project' 실행하여 환경 업데이트 완료*

    * @Configuration 
        > * xml 대신 설정파일을 직접 작성한다.    
        > * @Configuration 어노테이션으로 해당 클래스의 인스턴스를 이용해 설정 파일을 대신할 수 있다. 
        > * RootConfig.java 파일 생성     

    * web.xml 대신하는 WebConfig.java 생성
        > * AbstractAnnotationConfigDispatcherServletInitializer 추상 클래스를 상속받는다. 
        > * 추상 메서드 3개를 오버라이드 하도록 작성되는데, 이때 getRootConfig() 클래스가 'root-context.xml' 역할.
        
