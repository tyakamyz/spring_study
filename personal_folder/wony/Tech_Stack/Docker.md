# Docker

## 1. 특성
   - 컨테이너 기반의 오픈소스 가상화 플랫폼
 - 기존의 VM의 가상화 는 추가적인 OS를 설치 해야 하기에 성능상의 문제가 발생 - 이를 개선하고자 프로세스를 격리하는 Container 방식이 등장.
 - 초기 컨테이너 엔진은 **LXC(Linux Container)**를 기반으로 한다.



## 2. Docker와 VM 비교
   - Docker : 애플리케이션 실행하는 것과 마찬가지로 호스트 OS 위에 애플리케이션의 실행 패키지인 **이미지**를 배포하기만 하면 됨
- VM : 항상 게스트 OS가 필요함, 애플리케이션 실핼 할때에도 먼저 VM 을 띄우고 자원 할당 후 , 게스트 OS를 부팅하여 애플리케이션을 실행 시켜야 한다.

## 3. Oracle 설치
 - docker 로그인
```bash
$ docker login
```
 - 다운로드
```bash
$ docker search oracle-xe-11g
$ docker pull jaspeen/oracle-xe-11g
```
 - 이미지 실행(만들면서 실행)
```bash
$ docker run --name oracle11g -d -p 8080:8080 jaspeen/oracle-xe-11g
```
 - 컨테이너 확인
```bash
$ docker ps (실행중인 컨테이너)
$ docker ps -a (모든 컨테이너)
```
 - SQLPlus 실행
```bash
$ docker exec -it oracle11g sqlplus
```
user-name/password = system/oracle
 - 시작
```bash
$ docker start oracle11g
```
 - 종료
```bash
$ docker stop oracle11g
```
 - 삭제
```bash
$ docker rm oracle11g
```
