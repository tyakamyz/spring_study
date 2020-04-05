**Lombok 라이브러리 설치**    
[Download]: https://projectlombok.org/download   

> * cmd 실행 후, 다운로드 된 jar 파일 경로에서 'java -jar lombok.jar' 명령어로 실행   
> * 실행 후 Specify location 선택 후 설치한 STS 실행경로를 추가(C:\sts\sts.exe) , Quit Installer 실행 
> * 설치가 끝나면 STS 실행 경로에 **lombok.jar** 파일이 추가된다.
> * 만약 STS 실행이 되지 않을경우, **sts.ini** 파일에   
        *-vmargs*   
        *-javaagent:lombok.jar*   
    를 추가해준다.  

<hr />

1. 접근자 / 설성자 자동생성 (**@Getter @Setter**)
```java
// 자동으로 getPageNum과 setPageNum을 만들어 준다. 
@Getter @Setter
public class Criteria {
    private int pageNum;
}
```

2. 생성자 자동생성
```java
@NoArgsConstructor
@RequiredArgsConstructor
@AllArgsConstructor
public class ReplyPageDTO {
	private int replyCnt;
	private List<ReplyVO> list;
    @NonNull
    private int replyNum;
}
```
```java
// @NoArgsConstructor
ReplyPageDTO page = new ReplyPageDTO();
// @RequiredArgsConstructor
ReplyPageDTO page2 = new ReplyPageDTO(replyNum);
// @AllArgsConstructor
ReplyPageDTO page3 = new ReplyPageDTO(replyCnt, list, replyNum);
```
@NoArgsConstructor : 파라미터가 없는 기본 생성자를 생성   
@RequiredArgsConstructor : final이나 @NonNull인 필드 값만 파라미터로 받는 생성자를 생성(특정 변수에 대해서만 생성자를 작성하고 싶을 경우)   
@AllArgsConstructor : 모든 필드 값을 파라미터로 받는 생성자를 생성   


3. ToString 메소드 자동생성
```java
// 자동으로 .toString()을 생성한다.
@ToString
public class Criteria {
    // paging 
	private int pageNum;
	private int amount;
}    
```    

4. @Data
```java
@Data
public class ReplyVO {
	private Long bno;
	private Long rno;
}

```
@Getter, @Setter, @RequiredArgsConstructor, @ToString, @EqualsAndHashCode을 한꺼번에 설정해주는어노테이션

5. @Log4j   
```java
@Log4j
@AllArgsConstructor
public class ReplyController {
    ...
}    
```
자동으로 log 필드를 만들고, 해당 클래스의 이름으로 로거 객체를 생성하여 할당

6. Null 체크   
```java
public class ReplyPageDTO {
	private int replyCnt;
	private List<ReplyVO> list;
    @NonNull
    private int replyNum;
}
```
@NonNull 어노테이션을 변수에 붙이면 자동으로 null 체크를 하며,   
해당 변수가 null로 넘어온 경우, NullPointerException 예외를 일으켜 준다.
