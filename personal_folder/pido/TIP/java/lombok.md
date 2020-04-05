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
