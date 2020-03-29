
[Part3] - chapter 07, 08
=========================

스프링 MVC
---------------
- Spring MVC : Presentation Tier 데이터를 어떤 방식으로 보관하고, 사용하는가에 대한 설계 계층   - *root-context.xml , servlet-context.xml*
- Spring Core : POJO(Plain Old Java Object) 의존성 주입을 이용, 객체간의 연관구조를 완성해서 사용하는 영역   
- Mybatis : mybatis-spring 을 이용해서 구성. SQL 처리 담당 

### Naming 
- Controller : 컨트롤러단
- Service, ServiceImpl : 비즈니스 영역을 담당하는 인터페이스와 인터페이스를 구현한 클래스 
- DAO, Repository 
- VO, DTO : 데이터를 담고 있는 객체    
            (VO : ReadOnly 목적, 데이터 자체도 불변하게 설계   
            DTO : 데이터 수집의 용도. 예로 로그인 정보 처리)

예제 프로젝트 생성 설정
-----------------
**- XML 기반 설정 -**     
pom.xml, root-context.xml
> [☝ XML 관련 설정 소스 참고](https://github.com/tyakamyz/spring_study/blob/master/personal_folder/tongyo/Memo/Spring%20Legacy%20Project_%EC%85%8B%ED%8C%85(xml%20version).md)


**- JAVA 기반 설정 -**  
pom.xml, rootConfig.java, ServletConfig.java, WebConfig.java
> [☝ JAVA 관련 설정 소스 참고](https://github.com/tyakamyz/spring_study/blob/master/personal_folder/tongyo/Memo/Spring%20Legacy%20Project_%EC%85%8B%ED%8C%85(java%20version).md)


**- ORACLE -**  
1. 시퀀스   
```sql
 create sequence seq_board; 
```
2. 테이블 생성
```sql
create table tbl_board (
    bno number(10,0),
    ...
)
```
3. PK 설정
```sql
alter table tbl_board add constraint pk_board 
primary key (bno);
```
4. 더미데이터 생성
```sql
insert into tbl_board (bno, title, content, writer)
values (seq_board.nextval, '테스트t', '테스트c', 'user00');
```

영속/비즈니스 계층의 CRUD 구현
----------------
1. VO 클래스 작성   
@Data    
Lombok을 이용하여 생성자, getter/setter, toString()을 만들어낸다. 
    ```java 
    @Data
    public class BoardVO {
        private Long bno; 
        ...
    }
    ```

2. Mapper 인터페이스   
root-context.xml 활용   
```<mybatis-spring:scan base-package="org.zerock.mapper"/>```   

    org.zerock.mapper 내 BoardMapper 인터페이스   
    어노테이션 방식의 List 확인 
    ```java 
    public interface BoardMapper {
        @Select("select * from tbl_board where bno > 0")
        public List<BoardVO> getList();
    ```

3. Mapper XML 파일 
    ```xml
    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="org.zerock.mapper.BoardMapper">

        <select id="getList" resultType="org.zerock.domain.BoardVO">
            <![CDATA[
            select * from tbl_board where bno > 0
            ]]>
        </select>
    ```
    > ```<mapper>```의 namespace 속성값은 Mapper 인터페이스와 동일   
    > ```<select>```의 id 속성값은 메서드 이름과 일치   
    > resultType 속성값은 select 쿼리의 결과를 객체로 만들기 위해 설정   
    > ```<![CDATA[```는 XML 부등호 사용을 위한 태그    

4. Mybatis의 CRUD   
Mybatis는 내부적으로 JDBC의 PreparedStatement를 활용하고, 필요한 파라미터를 처리하는 방식은 **'?' -> '#{속성}** 으로 처리

**** Create(Insert)
* @SelectKey   
 Pk값을 미리 SQL을 통해서 처리해 두고 특정한 이름으로 결과를 보관
    ```xml
    <insert id = "insertSelectKey">
        <selectKey keyProperty="bno" order="BEFORE" resultType="long">
            select seq_board.nextval from dual
        </selectKey>
        insert into tbl_board (bno, title, content, writer)
        values (#{bno}, #{title}, #{content}, #{writer})
    </insert>
    ```
* 여러개의 파라미터 전달 시 VO에 담아 보낸다
    ```java
    BoardVO board = new BoardVO();
    mapper.insertSelectKey(board);
    ```

**** Read(select)   
* PK인 BoardVO 클래스의 bno 타입 정보를 이용하여 처리
    ```xml
    <select id="read" resultType="org.zerock.domain.BoardVO">
		select * from tbl_board where bno = #{bno}
	</select>
    ```
* resultType의 BoardVO의 속성값으로 리턴 타입이 select 결과 처리     

**** delete   
* PK인 bno 타입 정보를 이용하여 처리
* 메서드 리턴 타입은 int 로 지정하여 정상적으로 데이터 삭제 시 1 이상의 값을 가지도록 작성 
    ```java
    BoardMapper 인터페이스 
    public int delete(Long bno);
    ```

**** update   
* 메서드 리턴 타입은 int 로 지정하여 몇개의 데이터가 수정되었는가를 처리 할 수 있도록 작성
    ```java
    BoardMapper 인터페이스 
    public int update(BoardVO board);
    ```
