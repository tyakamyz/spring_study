
[Part3] - chapter 12
=========================

오라클 데이터베이스 페이징처리
-----------------

### **order by**

1. test용 재귀 복사
    ```
    insert into tbl_board (bno, title, content, writer)
    (select seq_board.next_val, title, content, writer from tbl_board);
    ```
2. 정렬 
    ```
    select * from tbl_board order by bno +1 desc;
    ```    
    실행계획을 살펴보면,  (안쪽에서 바깥쪽으로, 위에서 아래로)  
        - 테이블 전체를 스캔 ==> TBL_BOARD FULL   
        - 바깥쪽으로 가면서 SORT
        - 가장 오랜 시간을 소모하는 작업은 SORT 로 확인됨

    >    **인덱스**
    >    ```
    >    select * from tbl_board order by bno desc;
    >    ```
    >    ```
    >    select /*+ INDEX_DESC(tbl_board pk_board) */
    >    from tbl_board 
    >    where bno > 0;
    >    ```
    >    두 쿼리의 실행계획을 살펴보면,   
    >    - PK_BOARD 로 FULL SCAN DESCENDING 으로 SORT를 사용하지 않음   
    >    - TBL_BOARD 는  BY INDEX ROWID 인덱스 사용 하는 것을 확인할 수 있음   
    >
    >    처음 테이블 생성시, 
    >    ```
    >    alter table tbl_board add constrant pk_board
    >    primary key (bno);
    >    ```
    >    위와 같이 제약조건으로 PK를 생성하였는데, 이는 '식별자'의 의미와 '인덱스'의 의미를 가진다. 
    >    - bno 라는 컬럼으로 인덱스가 생성된 것
    >    - 인덱스는 '이미 정렬이 되어있다.' 
    >    - PK_BOARD 인덱스를 이용해 조건 데이터의 ROWID를 먼저 찾고, 이후 ROWID를 통해 테이블에 접근하는 방식
    
    




