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
