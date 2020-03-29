# 3장. 기본적인 웹 게시물 관리물 관리
> ## ch08. 영속/비즈니스 계층의 CRUD 구현
- VO 클래스 정의
    - lombok 사용
    ```java
    package org.zerock.controller;

    import java.util.Date;

    import lombok.Data;

    @Data
    public class BoardVO {
        private Long bno;
        private String title;
        private String content;
        private String writer;
        private Date regdate;
        private Date updatedate;
    }
    ```
- org.zerock.mapper 패키지를 스캔하도록 설정
- root-context.xml
```xml
<mybatis-spring:scan base-package="org.zerock.mapper"/>
```

- BoardMapper.java
```java
package org.zerock.mapper;

import java.util.List;

import org.apache.ibatis.annotations.Select;
import org.zerock.domain.BoardVO;

public interface BoardMapper {
	
    @Select("select * from tbl_board where bno > 0")
	public List<BoardVO> getList();
	
}
```
- xml의 파일명은 가독성을 위해 클래스와 동일하게 하는 것을 추천
- \<![CDATA[]]> 를 사용하면 '>' 부등호 사용 가능
- BoardMapper.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org/DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="org.zerock.mapper.BoardMapper">

	<select id="getList" resultType="org.zerock.domain.BoardVO">
	<![CDATA[
	select * from tbl_board where bno > 0
	]]>
	</select>

</mapper>
```
- @Select 구절 대신에 xml에 정의된 mybatis 사용이 가능해짐
- BoardMapper.java
```java
package org.zerock.mapper;

import java.util.List;

import org.apache.ibatis.annotations.Select;
import org.zerock.domain.BoardVO;

public interface BoardMapper {
	
    //@Select("select * from tbl_board where bno > 0")
	public List<BoardVO> getList();
	
}
```
------------
- create(insert)
    - BoardMapper.java
    ```java
    // insert문만 처리되고 생성된 PK 값을 알 필요가 없는 경우
    public void insert(BoardVO board);
    
    // insert문이 실행되고 생성된 PK 값을 알아야 하는 경우
    public void insertSelectKey(BoardVO board);
    ```
    - BoardMapper.xml
    ```xml
    <insert id="insert">
		insert into tbl_board (bno, title, content, writer)
		values (seq_board.nextval, #{title}, #{content}, #{writer})
	</insert>
	
	<insert id="insertSelectKey">
		<selectKey keyProperty="bno" order="BEFORE" resultType="long">
			select seq_board.nextval from dual
		</selectKey>
		insert into tbl_board (bno, title, content, writer)
		values (#{bno}, #{title}, #{content}, #{writer})
	</insert>
    ```
    - BoardMapperTests.java
    ```java
    package org.zerock.mapper;

    import org.junit.Test;
    import org.junit.runner.RunWith;
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.test.context.ContextConfiguration;
    import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
    import org.zerock.domain.BoardVO;

    import lombok.Setter;
    import lombok.extern.log4j.Log4j;

    @RunWith(SpringJUnit4ClassRunner.class)
    @ContextConfiguration("file:src/main/webapp/WEB-INF/spring/root-context.xml")
    //Java Config
    //@@ContextConfiguration(classes = {org.zerock.config.RootConfig.class})
    @Log4j
    public class BoardMapperTests {
        
        @Setter(onMethod_ = @Autowired)
        private BoardMapper mapper;
        
        @Test
        public void testGetList() {
            mapper.getList().forEach(board -> log.info(board));
        }
        
        @Test
        public void testInsert() {
            BoardVO board = new BoardVO();
            board.setTitle("새로 작성하는 글");
            board.setContent("새로 작성하는 내용");
            board.setWriter("newbie");
            
            mapper.insert(board);
            
            log.info(board);
        }
        
        @Test
        public void testInsertSelectKey() {
            BoardVO board = new BoardVO();
            board.setTitle("새로 작성하는 글 SelectKey");
            board.setContent("새로 작성하는 내용 SelectKey");
            board.setWriter("newbie");
            
            mapper.insertSelectKey(board);
            
            log.info(board);
        }
    }
    ```
- read(select)
    - BoardMapper.java
    ```java
    public BoardVO read(Long bno);
    ```
    - BoardMapper.xml
    ```xml
    <select id="read" resultType="org.zerock.domain.BoardVO">
		select * from tbl_board where bno = #{bno}
	</select>
    ```
    - BoardMapperTests.java
    ```java
    @Test
	public void testRead() {
		
		// 존재하는 게시물 번호로 테스트
		BoardVO board = mapper.read(1L);
		
		log.info(board);
		
	}
    ```
- delete
    - BoardMapper.java
    ```java
    public int delete(Long bno);
    ```
    - BoardMapper.xml
    ```xml
    <delete id="delete">
		delete from tbl_board where bno = #{bno}
	</delete>
    ```
    - BoardMapperTests.java
    ```java
    @Test
	public void testDelete() {
		log.info("DELETE COUNT : " + mapper.delete(1L));
	}
    ```
- update
    - BoardMapper.java
    ```java
    public int update(BoardVO board);
    ```
    - BoardMapper.xml
    ```xml
    <update id="update">
		update tbl_board
		set title = #{title},
			content = #{content},
			writer = #{writer},
			updateDate = sysdate
			where bno = #{bno}
	</update>
    ```
    - BoardMapperTests.java
    ```java
    @Test
	public void testUpdate() {
		BoardVO board = new BoardVO();
		
		board.setBno(5L);
		board.setTitle("수정된 제목");
		board.setContent("수정된 내용");
		board.setWriter("user00");
		
		int count = mapper.update(board);
		log.info("UPDATE COUNT : " + count);
	}
    ``` 
-------------
> ## ch09. 비즈니스 계층
- 고객의 요구사항을 반영하는 계층
- 프레젠테이션 계층과 영속 계층의 중간 다리 역할
- 비즈니스 계층은 로직을 기준으로 처리 (영속 계층은 데이터베이스를 기준으로 설계를 나눔)
- 일반적으로 비즈니스 영역에 있는 객체들은 서비스 라는 용어를 많이 사용험
    - BoardService.java
    ```java
    package org.zerock.service;

    import java.util.List;

    import org.zerock.domain.BoardVO;

    public interface BoardService {
        
        public void register(BoardVO board);
        
        public BoardVO get(Long bno);
        
        public boolean modify(BoardVO board);
        
        public boolean remove(Long bno);
        
        public List<BoardVO> getList();
        
    }
    ```
    - BoardServiceImpl.java
        - @Service는 계층 구조상 주로 비즈니스 영역을 담당하는 객체임을 표시하기 위해 사용
        - 작성된 어노테이션은 패키지를 읽어 들이는 동안 처리됨. BoardServiceImpl이 정상적으로 동작하기 위해서는 BoardMapper 객체가 필요
        - @AllArgsContstructor는 모든 파라미터를 이용하는 생성자를 만들기 때문에 실제 코드는 BoardMapper를 주입받는 생성자가 만들어지게 됨<br>
        (@Setter(onMethod_ = @Autowired) 대신 사용)
    ```java
    package org.zerock.service;

    import java.util.List;

    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.stereotype.Service;
    import org.zerock.domain.BoardVO;
    import org.zerock.mapper.BoardMapper;

    import lombok.AllArgsConstructor;
    import lombok.Setter;
    import lombok.extern.log4j.Log4j;

    @Log4j
    @Service
    @AllArgsConstructor
    public class BoardServiceImpl implements BoardService{
    //	@Setter(onMethod_ = @Autowired)
    //	private BoardMapper mapper;
        
        // Spring 4.3 이상에서 자동 처리
        private BoardMapper mapper;

        @Override
        public void register(BoardVO board) {
            log.info("register....." + board);
            
            mapper.insertSelectKey(board);
        }

        @Override
        public BoardVO get(Long bno) {
            // TODO Auto-generated method stub
            return null;
        }

        @Override
        public boolean modify(BoardVO board) {
            // TODO Auto-generated method stub
            return false;
        }

        @Override
        public boolean remove(Long bno) {
            // TODO Auto-generated method stub
            return false;
        }

        @Override
        public List<BoardVO> getList() {
            // TODO Auto-generated method stub
            return null;
        }
        
    }
    ```
    - root-context.xml
    ```xml
    <context:component-scan base-package="org.zerock.service"></context:component-scan>
    ```
    - RootConfig.java (java 설정 시])
    ```java
    @Configuration
    @ComponentScan(basePackages="org.zerock.service")
    @MapperScan(basePackages={"org.zerock.mapper"})
    public class RootConfig{
        .......
    ```
    - BoardServiceTests.java
        - 의존성 주입 테스트 (BoardService 객체가 제대로 주입 가능한지 테스트)
    ```java
    package org.zerock.service;

    import static org.junit.Assert.assertNotNull;

    import org.junit.Test;
    import org.junit.runner.RunWith;
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.test.context.ContextConfiguration;
    import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

    import lombok.Setter;
    import lombok.extern.log4j.Log4j;

    @RunWith(SpringJUnit4ClassRunner.class)
    @ContextConfiguration("file:src/main/webapp/WEB-INF/spring/root-context.xml")
    //Java Config
    //@@ContextConfiguration(classes = {org.zerock.config.RootConfig.class})
    @Log4j
    public class BoardServiceTests {
        
        @Setter(onMethod_ = @Autowired)
        private BoardService service;
        
        @Test
        public void testExist() {
            log.info(service);
            assertNotNull(service);
        }
        
    }
    ```
----------------
- 비즈니스 계층 CRUD 테스트
    - BoardServiceimpl.java
    ```java
    package org.zerock.service;

    import java.util.List;

    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.stereotype.Service;
    import org.zerock.domain.BoardVO;
    import org.zerock.mapper.BoardMapper;

    import lombok.AllArgsConstructor;
    import lombok.Setter;
    import lombok.extern.log4j.Log4j;

    @Log4j
    @Service
    @AllArgsConstructor
    public class BoardServiceImpl implements BoardService{
    //	@Setter(onMethod_ = @Autowired)
    //	private BoardMapper mapper;
        
        // Spring 4.3 이상에서 자동 처리
        private BoardMapper mapper;

        @Override
        public void register(BoardVO board) {
            log.info("register....." + board);
            
            mapper.insertSelectKey(board);
        }

        @Override
        public BoardVO get(Long bno) {
            log.info("get.........." + bno);
            
            return mapper.read(bno);
        }

        @Override
        public boolean modify(BoardVO board) {
            log.info("modify....." + board);
            
            return mapper.update(board) == 1;
        }

        @Override
        public boolean remove(Long bno) {
            log.info("remove....." + bno);
            
            return mapper.delete(bno) == 1;
        }

        @Override
        public List<BoardVO> getList() {
            log.info("getList.........");
            
            return mapper.getList();
        }
        
    }
    ```
    - BoardServiceTests.java
    ```java
    package org.zerock.service;

    import static org.junit.Assert.assertNotNull;

    import org.junit.Test;
    import org.junit.runner.RunWith;
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.test.context.ContextConfiguration;
    import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
    import org.zerock.domain.BoardVO;

    import lombok.Setter;
    import lombok.extern.log4j.Log4j;

    @RunWith(SpringJUnit4ClassRunner.class)
    @ContextConfiguration("file:src/main/webapp/WEB-INF/spring/root-context.xml")
    //Java Config
    //@@ContextConfiguration(classes = {org.zerock.config.RootConfig.class})
    @Log4j
    public class BoardServiceTests {
        
        @Setter(onMethod_ = @Autowired)
        private BoardService service;
        
        @Test
        public void testExist() {
            log.info(service);
            assertNotNull(service);
        }
        
        @Test
        public void testRegister() {
            BoardVO board = new BoardVO();
            board.setTitle("새로 작성하는 글");
            board.setContent("새로 작성하는 내용");
            board.setWriter("newbie");
            
            service.register(board);
            
            log.info("생성된 게시물의 번호 : " + board.getBno());
        }
        
        @Test
        public void testGetList() {
            service.getList().forEach(board -> log.info(board));
        }
        
        @Test
        public void testGet() {
            log.info(service.get(6L));
        }
        
        @Test
        public void testDelete() {
            log.info("REMOVE RESULT: " + service.remove(7L));
        }

        @Test
        public void testUpdate() {
            BoardVO board = service.get(6L);
            
            if(board == null) {
                return;
            }
            
            board.setTitle("제목 수정합니다");
            log.info("MODIFY RESULT: " + service.modify(board));
        }
    }
    ```
---------------
> ## ch10. 프레젠테이션(웹) 계층의 CRUD 구현
- BoardController.java
```java
package org.zerock.controller;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.servlet.mvc.support.RedirectAttributes;
import org.zerock.domain.BoardVO;
import org.zerock.service.BoardService;

import lombok.AllArgsConstructor;
import lombok.extern.log4j.Log4j;

@Controller
@Log4j
@RequestMapping("/board/*")
@AllArgsConstructor
public class BoardController {
	
	private BoardService service;
	
	@GetMapping("/list")
	public void list(Model model) {
		log.info("list");
		
		model.addAttribute("list", service.getList());
	}
	
	@PostMapping("/register")
	public String register(BoardVO board, RedirectAttributes rttr) {
		log.info("register: " + board);
		
		service.register(board);
		
		rttr.addFlashAttribute("result", board.getBno());
		
		return "redirect:/board/list";
	}
	
	@GetMapping("/get")
	public void get(@RequestParam("bno") Long bno, Model model) {
		log.info("/get");
		
		model.addAttribute("board", service.get(bno));
	}
	
	@PostMapping("/modify")
	public String modify(BoardVO board, RedirectAttributes rttr) {
		log.info("modify: " + board);
		
		if(service.modify(board)) {
			rttr.addFlashAttribute("result", "success");
		}
		
		return "redirect:/board/list";
	}
	
	@PostMapping("/remove")
	public String remove(@RequestParam("bno") Long bno, RedirectAttributes rttr) {
		log.info("remove....." + bno);
		
		if(service.remove(bno)) {
			rttr.addFlashAttribute("result", "success");
		}
		
		return "redirect:/board/list";
	}
}
```
---------------
> ## ch10. 화면처리
- servlet-context.xml
    - 스프링 MVC의 화면 설정은 ViewResolver라는 객체를 통해 이루어지며, "/WEB-INF/views/"로 설정되어있음
```xml
<!-- Resolves views selected for rendering by @Controllers to .jsp resources in the /WEB-INF/views directory -->
	<beans:bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
		<beans:property name="prefix" value="/WEB-INF/views/" />
		<beans:property name="suffix" value=".jsp" />
	</beans:bean>
```
--------------
> ## ch12. 오라클 데이터베이스 페이징 처리
- SQL 실행 순서
    1. SQL 파싱
        - SQL 구문에 오류가 있는지 SQL을 실행해야 하는 대상 객체(테이블, 제약 조건, 권한 등)가 존재하는지 검사
    2. SQL 최적화
        - SQL 파싱을 통해 계산된 결과 값을 기초로 실행계획(execuion plan)을 세우게 됨
    3. SQL 실행
        - SQL 최적화 단계에서 세워진 실행 계획을 통해서 메모리상에서 데이터를 읽거나 물리적인 공간에서 데이터를 로딩하는 등의 작업을 함
        - SQL Plus 등을 이용하여 특정한 SQL에 대한 실행 계획을 알아볼 수 있음<br>
        (SQL Developer에서는 단축키 F10(실행계획))
- 정렬 권장 방법
    - order by를 통해 정렬할 경우 시간이 많이 소요됨
    - 인덱스를 통해 정렬하는 것을 권장
    ```sql
    select /*+ INDEX_DESC(tbl_board pk_board) */
    *
    from tbl_board
    where bno > 0;
    ```
-------
> ## ch13. MyBatis와 스프링에서 페이징 처리
- Criteria.java
```java
package org.zerock.domain;

import lombok.Getter;
import lombok.Setter;
import lombok.ToString;

@Getter
@Setter
@ToString
public class Criteria {
	private int pageNum;
	private int amount;
	
	public Criteria() {
		this(1,10);
	}
	
	public Criteria(int pageNum, int amount) {
		this.pageNum = pageNum;
		this.amount = amount;
	}
}
```
- BoardMapper.java
```java
public List<BoardVO> getListWithPaging(Criteria cri);
```
- BoardMapper.xml
```java
<select id="getListWithPaging" resultType="org.zerock.domain.BoardVO">
    <![CDATA[
    select bno, title, content, writer, regdate, updatedate
    from
    (
        select /*+INDEX_DESC(tbl_board, pk_board) */ 
                rownum rn, bno, title, content, writer, regdate, updatedate
        from tbl_board
        where rownum < #{pageNum} * #{amount}
    )
    where rn > (#{pageNum}-1) * #{amount}
    ]]>
</select>
```
- BoardService.java
```java
//public List<BoardVO> getList();
	
public List<BoardVO> getList(Criteria cri);
```
- BoardServiceImpl.java
```java
//	@Override
//	public List<BoardVO> getList() {
//		log.info("getList.........");
//		
//		return mapper.getList();
//	}

@Override
public List<BoardVO> getList(Criteria cri) {
    log.info("get List with criteria : " + cri);
    
    return mapper.getListWithPaging(cri);
}
```
- BoardControler.java
```java
@GetMapping("/list")
public void list(Criteria cri ,Model model) {
    log.info("list");
    
    model.addAttribute("list", service.getList(cri));
}
```
> ## ch14. 페이징 화면처리
- '../Memo/Spring or Java/Paging공식.md' 파일 참조
----------
> ## ch15. 검색처리
- MyBatis의 동적 태그들
    - if
    - choose (when, otherwise)
    - trim (where, set)
    - foreach
- Criteria.java
```java
package org.zerock.domain;

import lombok.Getter;
import lombok.Setter;
import lombok.ToString;

@Getter
@Setter
@ToString
public class Criteria {
	private int pageNum;
	private int amount;
	
	private String type;
	private String keyword;
	
	public Criteria() {
		this(1,10);
	}
	
	public Criteria(int pageNum, int amount) {
		this.pageNum = pageNum;
		this.amount = amount;
	}
	
	public String[] getTypeArr() {
		return type == null ? new String[] {} : type.split("");
	}
}
```
- mapper.xml (sql 정의 전)
```xml
<select id="getListWithPaging" resultType="org.zerock.domain.BoardVO">
    <![CDATA[
    select bno, title, content, writer, regdate, updatedate
    from
    (
        select /*+INDEX_DESC(tbl_board, pk_board) */ 
                rownum rn, bno, title, content, writer, regdate, updatedate
        from tbl_board
        where
    ]]>
        <trim prefix="(" suffix=") AND " prefixOverrides="OR">
            <foreach item='type' collection="typeArr">
                <trim prefix="OR">
                    <choose>
                        <when test="type == 'T'.toString()">
                            title like '%'||#{keyword}||'%'
                        </when>
                        <when test="type == 'C'.toString()">
                            content like '%'||#{keyword}||'%'
                        </when>
                        <when test="type == 'W'.toString()">
                            writer like '%'||#{keyword}||'%'
                        </when>
                    </choose>
                </trim>
            </foreach>
        </trim> 
    <![CDATA[
        rownum < #{pageNum} * #{amount}
    )
    where rn > (#{pageNum}-1) * #{amount}
    ]]>
</select>
```
- mapper.xml (sql 정의 후)
    - 중복되는 조건 쿼리는 따로 \<sql>이라는 태그를 통해 재사용이 가능
    ```xml
    <sql id="criteria">
		<trim prefix="(" suffix=") AND " prefixOverrides="OR">
			<foreach item='type' collection="typeArr">
				<trim prefix="OR">
					<choose>
						<when test="type == 'T'.toString()">
							title like '%'||#{keyword}||'%'
						</when>
						<when test="type == 'C'.toString()">
							content like '%'||#{keyword}||'%'
						</when>
						<when test="type == 'W'.toString()">
							writer like '%'||#{keyword}||'%'
						</when>
					</choose>
				</trim>
			</foreach>
		</trim> 
	</sql>

    <select id="getListWithPaging" resultType="org.zerock.domain.BoardVO">
		<![CDATA[
		select bno, title, content, writer, regdate, updatedate
		from
		(
			select /*+INDEX_DESC(tbl_board, pk_board) */ 
					rownum rn, bno, title, content, writer, regdate, updatedate
			from tbl_board
			where
		]]>
		
			<include refid="criteria"></include>
			
		<![CDATA[
			rownum <= #{pageNum} * #{amount}
		)
		where rn > (#{pageNum}-1) * #{amount}
		]]>
	</select>
	
	<select id="getTotalCount" resultType="int">
		select count(*) from tbl_board 
		where 
		
		<include refid="criteria"></include>
		
		bno > 0
	</select>
    ```
- list.jsp (문제점 해결)
    - 검색 시 페이지가 그대로 유지되는 문제
    - 검색 후 페이지를 이동하면 검색 조건 초기화되는 문제
    - 검색 후 화면에서 어떤 검색 조건과 키워드를 사용했는지 알 수 없는 문제
```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
	pageEncoding="UTF-8"%>
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>
<%@ taglib uri="http://java.sun.com/jsp/jstl/fmt" prefix="fmt" %>
	
<%@include file="../includes/header.jsp"%>

<script type="text/javascript">
$(document).ready(function(){
	var result = '<c:out value="${result}" />';
	
	/* 검색 버튼 이벤트 처리 */
	var searchForm = $("#searchForm");
	
	$("#searchForm button").on("click", function(e){
		if(!searchForm.find("option:selected").val()){
			alert("검색종류를 선택하세요");
			return false;
		}
		
		if(!searchForm.find("input[name='keyword']").val()){
			alert("키워드를 입력하세요");
			return false;
		}
		
		searchForm.find("input[name='pageNum']").val("1");
		e.preventDefault();
		
		searchForm.submit();
	});
	/* 검색 버튼 이벤트 처리 종료 */
	
	checkModal(result);
	
	history.replaceState({},null,null);
	
	function checkModal(result){
		if(result === '' || history.state){
			return;
		}
		
		if(parseInt(result) > 0){
			$(".modal-body").html("게시글 " + parseInt(result) + "번이 등록되었습니다.");
		}
		
		$("#myModal").modal("show");
	}
	
	$("#regBtn").on("click", function(){
		self.location = "/board/register";
	});
	
	/* 페이징 처리 */
	var actionForm = $("#actionForm");
	
	$(".paginate_button a").on("click", function(e){
		e.preventDefault();
		
		console.log("click");
		
		actionForm.find("input[name='pageNum']").val($(this).attr("href"));
		actionForm.submit();
	});
	/* 페이징 처리 종료 */
	
	/* 조회 페이지 이동 */
	$(".move").on("click", function(e){
		e.preventDefault();
		
		actionForm.append("<input type='hidden' name='bno' value='"+ $(this).attr("href")+ "'>");
		actionForm.attr("action", "/board/get");
		actionForm.submit();
	});
	/* 조회 페이지 이동 종료 */
	
});
</script>
<!-- /.row -->
<div class="row">
	<div class="col-lg-12">
		<h1 class="panel-heading">Tables</h1>
	</div>
</div>
<!-- /.row -->
<div class="row">
	<div class="col-lg-12">
		<div class="panel panel-default">
			<div class="panel-heading">Board List Page
				<button id="regBtn" type="button" class="btn btn-xs pull-right">Register New Board</button>
			</div>
			<!-- /.panel-heading -->
			<div class="panel-body">
				<table class="table table-striped table-bordered table-hover">
					<thead>
						<tr>
							<th>#번호</th>
							<th>제목</th>
							<th>작성자</th>
							<th>작성일</th>
							<th>수정일</th>
						</tr>
					</thead>
					<tbody>
						<c:forEach items="${list}" var="board">
						<tr>
							<td><c:out value="${board.bno}" /></td>
							<td><a class="move" href="<c:out value='${board.bno}' />"><c:out value="${board.title}" /></a></td>
							<td><c:out value="${board.writer}" /></td>
							<td><fmt:formatDate pattern="yyyy-MM-dd" value="${board.regdate}" /></td>
							<td><fmt:formatDate pattern="yyyy-MM-dd" value="${board.updateDate}" /></td>
						</tr>
						</c:forEach>
					</tbody>
				</table>
				<!-- 검색 조건 추가 -->
				<form id='searchForm' action="/board/list" method='get'>
					<select name='type'>
						<option value=""
							<c:out value="${pageMaker.cri.type == null?'selected':''}"/>>--</option>
						<option value="T"
							<c:out value="${pageMaker.cri.type eq 'T'?'selected':''}"/>>제목</option>
						<option value="C"
							<c:out value="${pageMaker.cri.type eq 'C'?'selected':''}"/>>내용</option>
						<option value="W"
							<c:out value="${pageMaker.cri.type eq 'W'?'selected':''}"/>>작성자</option>
						<option value="TC"
							<c:out value="${pageMaker.cri.type eq 'TC'?'selected':''}"/>>제목
							or 내용</option>
						<option value="TW"
							<c:out value="${pageMaker.cri.type eq 'TW'?'selected':''}"/>>제목
							or 작성자</option>
						<option value="TWC"
							<c:out value="${pageMaker.cri.type eq 'TWC'?'selected':''}"/>>제목
							or 내용 or 작성자</option>
					</select> 
					<input type='text' name='keyword' value='<c:out value="${pageMaker.cri.keyword}"/>' />
					<input type='hidden' name='pageNum' value='<c:out value="${pageMaker.cri.pageNum}"/>' />
					<input type='hidden' name='amount' value='<c:out value="${pageMaker.cri.amount}"/>' />
					<button class='btn btn-default'>Search</button>
				</form>
				<!-- Paging 추가 -->
				<div class="pull-right">
					<ul class="pagination">
						<c:if test="${pageMaker.prev}">
							<li class="paginate_button previous"><a href="${pageMaker.startPage - 1}">Previous</a></li>
						</c:if>
						<c:forEach var="num" begin="${pageMaker.startPage}" end="${pageMaker.endPage}">
							<li class="paginate_button ${pageMaker.cri.pageNum==num ? 'active':''} "><a href="${num}">${num}</a></li>
						</c:forEach>
						<c:if test="${pageMaker.next}">
							<li class="paginate_button next"><a href="${pageMaker.endPage + 1}">Next</a></li>
						</c:if>
					</ul>
				</div>
				<form id="actionForm" action="/board/list" method="get">
					<input type="hidden" name="pageNum" value="${pageMaker.cri.pageNum}">
					<input type="hidden" name="amount" value="${pageMaker.cri.amount}">
					<input type='hidden' name='type' value='<c:out value="${pageMaker.cri.type}"/>' />
					<input type='hidden' name='keyword' value='<c:out value="${pageMaker.cri.keyword}"/>' />
				</form>
				<!-- Modal 추가 -->
				<div class="modal fade" id="myModal" tabindex="-1" role="dialog" aria-labelledby="myModalLabel" aria-hidden="true">
                  <div class="modal-dialog">
                      <div class="modal-content">
                          <div class="modal-header">
                              <button type="button" class="close" data-dismiss="modal" aria-hidden="true">&times;</button>
                              <h4 class="modal-title" id="myModalLabel">Modal title</h4>
                          </div>
                          <div class="modal-body">
                              처리가 완료되었습니다.
                          </div>
                          <div class="modal-footer">
                              <button type="button" class="btn btn-default" data-dismiss="modal">Close</button>
                          </div>
                      </div>
                      <!-- /.modal-content -->
                  </div>
                  <!-- /.modal-dialog -->
              </div>
              <!-- /.modal -->
			</div>
		</div>
	</div>
</div>
<%@include file="../includes/footer.jsp"%>
```
- get.jsp, modify.jsp
    - form안에 추가해줘야 함
```jsp
<input type='hidden' name='type' value='<c:out value="${cri.type}"/>' />
<input type='hidden' name='keyword' value='<c:out value="${cri.keyword}"/>' />
```
- BoardController.java
    - redirect 시 검색조건을 함께 넘겨줘야 함
```java
@PostMapping("/modify")
public String modify(BoardVO board, @ModelAttribute("cri") Criteria cri, RedirectAttributes rttr) {
    log.info("modify: " + board);
    
    if(service.modify(board)) {
        rttr.addFlashAttribute("result", "success");
    }
    
    rttr.addFlashAttribute("pageNum", cri.getPageNum());
    rttr.addFlashAttribute("amount", cri.getAmount());
    rttr.addFlashAttribute("type", cri.getType());
    rttr.addFlashAttribute("keyword", cri.getKeyword());
    
    return "redirect:/board/list";
}

@PostMapping("/remove")
public String remove(@RequestParam("bno") Long bno, @ModelAttribute("cri") Criteria cri, RedirectAttributes rttr) {
    log.info("remove....." + bno);
    
    if(service.remove(bno)) {
        rttr.addFlashAttribute("result", "success");
    }
    
    rttr.addFlashAttribute("pageNum", cri.getPageNum());
    rttr.addFlashAttribute("amount", cri.getAmount());
    rttr.addFlashAttribute("type", cri.getType());
    rttr.addFlashAttribute("keyword", cri.getKeyword());
    
    return "redirect:/board/list";
}
```
- modify.jsp
    - 파라미터 전송 부분 역시 수정이 필요
```jsp
<script type="text/javascript">
$(document).ready(function(){
	var formObj = $("form");
	
	$('button').on("click", function(e){
		e.preventDefault();
		
		var operation = $(this).data("oper");
		
		console.log(operation);
		
		if(operation === 'remove'){
			formObj.attr("action", "/board/remove");
		}else if(operation === 'list'){
			formObj.attr("action", "/board/list").attr("method", "get");
			var pageNumTag = $("input[name='pageNum']").clone();
			var amountTag = $("input[name='amount']").clone();			
			var typeTag = $("input[name='type']").clone();			
			var keywordTag = $("input[name='keyword']").clone();			
			
			formObj.empty();
			formObj.append(pageNumTag);
			formObj.append(amountTag);
			formObj.append(typeTag);
			formObj.append(keywordTag);
		}
		formObj.submit();
	});
});
</script>
```
------------
- UriComponentsBuilder를 이용하는 링크 생성
    - 여러개의 파라미터들을 연결해서 uri의 형태로 만들어 주는 기능
    - Criteria.java
    ```java
    package org.zerock.domain;

    import org.springframework.web.util.UriComponentsBuilder;

    import lombok.Getter;
    import lombok.Setter;
    import lombok.ToString;

    @Getter
    @Setter
    @ToString
    public class Criteria {
        private int pageNum;
        private int amount;
        
        private String type;
        private String keyword;
        
        public Criteria() {
            this(1,10);
        }
        
        public Criteria(int pageNum, int amount) {
            this.pageNum = pageNum;
            this.amount = amount;
        }
        
        public String[] getTypeArr() {
            return type == null ? new String[] {} : type.split("");
        }
        
        public String getListLink() {
            UriComponentsBuilder builder = UriComponentsBuilder.fromPath("")
                .queryParam("pageNum", this.pageNum)
                .queryParam("amount", this.amount)
                .queryParam("type", this.getType())
                .queryParam("keyword", this.getKeyword());
            return builder.toUriString();
        }
    }
    ```
    - BoardController.java
    ```java
    @PostMapping("/modify")
	public String modify(BoardVO board, @ModelAttribute("cri") Criteria cri, RedirectAttributes rttr) {
		log.info("modify: " + board);
		
		if(service.modify(board)) {
			rttr.addFlashAttribute("result", "success");
		}
		
		//rttr.addFlashAttribute("pageNum", cri.getPageNum());
		//rttr.addFlashAttribute("amount", cri.getAmount());
		//rttr.addFlashAttribute("type", cri.getType());
		//rttr.addFlashAttribute("keyword", cri.getKeyword());
		
		return "redirect:/board/list" + cri.getListLink();
	}
	
	@PostMapping("/remove")
	public String remove(@RequestParam("bno") Long bno, @ModelAttribute("cri") Criteria cri, RedirectAttributes rttr) {
		log.info("remove....." + bno);
		
		if(service.remove(bno)) {
			rttr.addFlashAttribute("result", "success");
		}
		
		//rttr.addFlashAttribute("pageNum", cri.getPageNum());
		//rttr.addFlashAttribute("amount", cri.getAmount());
		//rttr.addFlashAttribute("type", cri.getType());
		//rttr.addFlashAttribute("keyword", cri.getKeyword());
		
		return "redirect:/board/list" + cri.getListLink();
	}
    ```