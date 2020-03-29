
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
    >    - 속도에서 매우 우수한 성능을 보여줌!
    
3. 오라클 힌트(hint)   
데이터베이스에 어떤 방식으로 실행해 줘야 하는지를 명시.     
강제성 부여.

    ```
    select  /*+ INDEX_DESC(tbl_board pk_board) */
    from tbl_board; 
    ```
    order by 조건 없이도 descending 으로 실행된다.   
    ```
    <문법>
    SELECT
        /*+ Hint name (param...) */ column name,...
    FROM table name    
    
    **FULL 힌트**   
    select /*+ FULL(tbl_board) */ * from tbl_board order by desc;

    ***ASC/DESC 힌트**
    select /*+ INDEX_ASC(tbl_board pk_board) */ * from tbl_board whrer bno > 0;
    ```

4. ROWNUM   
- SQL이 실행된 결과에 넘버링한 변수(데이터를 가져올 떄 적용)   
- 정렬 과정에서 ROWNUM은 변경되지 않는다.

    > 인덱스를 통한 ROWNUM 접근시...   
    > 1) PK_BOARD 인덱스를 통해 테이블에 접근
    > 2) 접근한 데이터에 ROWNUM 부여    
    > 3) 게시물을 역순으로 접근한다면, 가장 높은 값의 데이터부터 접근하여 ROWNUM 값이 1이 될수 있도록 설정이 가능하다. 

5. ROWNUM 을 통한 페이징 처리
    ```
    select /*+INDEX_DESC(tbl_board pk_board) */
        rownum rn, bno, title, content
    from tbl_board
    where rownum <= 10;    
    ```
    ==> 역순으로 10개 출력 확인

    ```
    select /*+INDEX_DESC(tbl_board pk_board) */
        rownum rn, bno, title, content
    from tbl_board
    where rownum > 10 and rownum <= 20;    
    ```
    ==> 2페이지 데이터 구현이지만 데이터가 나오지 않는다. 
    * 실행계획 확인 시..
    1. ROWNUM > 10  --> 이 때, 처음나오는 데이터는 rownum이 무조건 1이다.
    2. where 조건에 의해 무효화 됨.
    3. 그 후 다른 데이터를 가져오면 다시 rownum이 1
    4. 결국 rownum이 1로 만들어지고 무효화 되면서 데이터결과가 없는것. 

    **인라인뷰 처리를 통한 2페이지 구현**
    ```
    select 
        bno, title, content
    from (
        select /*+INDEX_DESC(tbl_board pk_board) */
            rownum rn, bno, title, content
        from tbl_board
        where rownum <= 20    
    )
    where rn > 10;
    ```
    1. 20개 데이터 추출
    2. 2페이지에 해당하는 10개만 다시 추출 

=======>

* 필요한 순서로 정렬된 데이터에 ROWNUM을 붙인다
* 처음부터 해당 페이지의 데이터를 'ROWNUM <= 20' 과 같은 조건을 이용해서 구한다.
* 구해놓은 데이터를 하나의 테이블로 간주하고 인라인뷰로 처리한다.
* 인라인뷰에서 필요한 데이터만을 남긴다.




