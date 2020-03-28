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