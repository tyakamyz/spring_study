
[Part3] - chapter 09
=========================

비즈니스 계층
-----------------
Service(인터페이스)
```java
public interface BoardService {
    ...
}    
```
 * 메서드 설계 시 리턴타입 지정(String, boolean, List 등)    

serviceImpl(인터페이스를 구현한 클래스)
```java
@Log4j
@Service
//@AllArgsConstructor  
public class BoardServiceImpl implements BoardService {
    
    // spring 4.3 이상, 자동처리
    // private BoardMapper mapper;
    @Setter(onMethod_ = @Autowired)
	private BoardMapper mapper;
    ...
}    
```    
 * @Service 어노테인션은 계층 구조상 주로 비즈니스 영역을 담당하는 객체임을 표시하기 위해 사용.
 * BoardMapper 객체를 @Setter 어노테이션(Lombok)으로  처리하여 ServicecImpl를 동작
 * @AllArgsConstructor : spring 4.3 이상에서 사용가능하며 , 모든 파라미터를 이용하는 생성자를 만든다.

<br>

 **< 빈 인식을 위한 서비스 객체 설정 >**   

 **- XML 기반 설정 -**   
 root-context.xml
 ```xml
 <context:component-scan base-package="org.zerock.service"></context:component-scan>		
 ```

 **- JAVA 기반 설정 -**  
 RootConfig.java
 ```java
@Configuration
@ComponentScan(basePackages="org.zerock.service")
@MapperScan(basePackages = {"org.zerock.mapper"})
public class RootConfig {
    ...
}    
 ```
<br>

비즈니스 계층 구현
-----------------

1. 등록   
serviceImpl 클래스에서 파라미터로 전달되는 BoardVO 타입의 객체를 BoardMapper 를 통하여 처리 
    ```java
    @Override
    public void register(BoardVO board) {
        mapper.insertSelectKey(board);
    }
    ```    

2. 목록
ServiceImpl 클래스에서 현재 조회한 모든 데이터를 return 
    ```java
    @Override
	public BoardVO get(Long bno) {
		return mapper.read(bno);
	}
    ```

3. 조회
게시물 번호(PK)가 파라미터 이고 BoardVO의 인스턴스가 return     
    ```java
    @Override
	public BoardVO get(Long bno) {
		return mapper.read(bno);
	}
    ```

4. 수정/삭제
정확한 처리를 위해 boolean 타입으로 return 
    ```java
    @Override
    public boolean modify(Long bno) {
		return mapper.update(bno) == 1;
	}
    ```
