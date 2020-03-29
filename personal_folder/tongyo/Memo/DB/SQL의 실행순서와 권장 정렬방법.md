> ## SQL 실행 순서
1. SQL 파싱
    - SQL 구문에 오류가 있는지 SQL을 실행해야 하는 대상 객체(테이블, 제약 조건, 권한 등)가 존재하는지 검사
2. SQL 최적화
    - SQL 파싱을 통해 계산된 결과 값을 기초로 실행계획(execuion plan)을 세우게 됨
3. SQL 실행
    - SQL 최적화 단계에서 세워진 실행 계획을 통해서 메모리상에서 데이터를 읽거나 물리적인 공간에서 데이터를 로딩하는 등의 작업을 함
    - SQL Plus 등을 이용하여 특정한 SQL에 대한 실행 계획을 알아볼 수 있음<br>
    (SQL Developer에서는 단축키 F10(실행계획))
> ## 정렬 권장 방법
- order by를 통해 정렬할 경우 시간이 많이 소요됨
- 인덱스를 통해 정렬하는 것을 권장
    - 인덱스는 색인을 통해 데이터가 이미 정렬되어 있음
    - 순번을 통해 접근
- 인덱스를 활용한 hint 사용
    - hint
        - 개발자가 데이터베이스에 어떤 방식으로 실행해 줘야 하는지를 명시하기 때문에 조금 강제성이 부여되는 방식
        - 힌트 구문에서 에러가 나도 SQL 실행에 지장을 주지 않으므로, 반드시 실행 계획을 통해 원하는 방향으로 SQL이 실행되었는지 확인해야 함
    -------
    > 힌트 사용 문법
    ```sql
    SELECT /*+ 힌트명(param) */ *
    FROM 테이블명;
    ```
    > INDEX_ASC 힌트 사용
     - param 값에 테이블명과 pk명이 함께 사용됨
    ```sql
    select /*+ INDEX_ASC(tbl_board pk_board) */ *
    from tbl_board;
    ```
    > INDEX_DESC 힌트 사용
     - param 값에 테이블명과 pk명이 함께 사용됨
    ```sql
    select /*+ INDEX_DESC(tbl_board pk_board) */ *
    from tbl_board;
    ```
    > FULL 힌트 사용 (인덱스 사용X)
     - param 값에 테이블명만 사용됨
    ```sql
    select /*+ FULL(tbl_board) */ *
    from tbl_board
    order by bno desc;
    ```
-------
> ## Hint를 통한 ROWNUM 응용 방법
- 일반적으로 rownum 사용 시 정렬전에 이미 rownum 번호가 부여되고 정렬되기 때문에 원하는 대로 값이 나오지않음
- hint를 사용하여 정렬 시 index를 통해 이미 정렬되어 있는 값에 접근한 뒤 rownum 번호 부여<br>
(FULL 힌트 제외. FULL의 경우 index를 통해 접근하지 않기 때문)
```sql
select /*+ INDEX_DESC(tbl_board pk_board) */ rownum rn, bno, title
from tbl_board;
```