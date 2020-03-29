> ## 오라클 통계정보
- ANALYZE
    - ANALYZE는 인덱스, 테이블, 클러스터의 통계정보를 생성
    - ANALYZE가 생성한 통계정보들은 비용기준(Cost-based)의 옵티마이저가 가장 효율적인 실행계획을 수립하기 위해 최소비용을 계산할 때 사용
    - 각 오브젝트의 구조를 확인하는 것과 체인(Chain) 생성 여부를 확인할 수 있으므로 시스템의 저장공간 관리를 도와줌
--------
- 생성되는 통계정보
    - 테이블 : 총 로우의수, 총 블럭의 수, 비어있는 블럭에 쓰여질 수 있는 빈 공간의 평군, 체인이 발생된 로우의 수, 로우의 평균 길이
    - 인덱스 : 인덱스의 깊이(Depth), Leaf block의 개수, Distinct Key의 수, Leaf Blocks/Key의 평균, Data blocks/key의 평균, Clustering Factor, 가장 큰 key 값, 가장 작은 key 값
    - 컬럼 : Distinct한 값의 수, 히스토그램 정보
    - 클러스터 : Cluster Key당 길이의 평균
-------
- 문법
    - object-clause : TABLE, INDEX, CLUSTER중에서 해당하는 오브젝트를 기술하고 처리할 오브젝트 명을 기술
    - operation : operation 옵션에는 다음 3가지중 한가지 기능을 선택
        - COMPUTE : 각각의 값들을 정확하게 계산 함. 가장 정확한 통계를 얻을 수 있지만 처리 속도가 가장 느림
        - ESTIMATE : 자료사전의 값과 데이터 견본을 가지고 검사해서 통계를 예상 함. COMPUTE보다 덜 정확 하지만 처리속도가 훨씬 빠름.
        - DELETE : 테이블의 모든 통계 정보를 삭제
        ```sql
        ANALYZE object-clause operation STATISTICS
        [VALIDATE STRUCTURE[CASCADE]]
        [LIST CHAINED ROWS[INTO table]]
        ```
-----
- 정보수집 예제
    - 주기적인 ANALYZE 작업을 수행하는 것을 권장
        - 테이블을 재생성한 경우
        - 새로 클러스터링을 한 경우
        - 인덱스를 추가하거나 재생성한 경우
        - 다량의 데이터를 SQL이나 배치 애플리케이션을 통해 작업한 경우
    - 사용자는 USER_TABLES, USER_COLUMNS, USER_INDEXS, USER_CLUSTER 등의 자료사전 뷰를 통해 정보 확인 가능
    - 테이블을 ANALYZE 시킨다면 거기에 따르는 인덱스들도 같이 실시하는 것이 좋음
        > 테이블 정보 수집 예제
        ```sql
        ANALYZE TABLE 테이블명 COMPUTE STATISTICS;
        ```
        > 새로운 정보를 구하기 전에 기존 정보를 삭제
        ```sql
        ANALYZE TABLE 테이블명 DELETE STATISTICS;
        ```
        > 특정 column에 대한 data 분포 수집
        ```sql
        ANALYZE TABLE 테이블명 COMPUTE STATISTICS FOR ALL INDEXED COLUMNS;
        ```
        > 통계정보 확인 예제
        ```sql
        SELECT num_rows, blocks, empty_blocks, avg_space, chain_cnt, 
            avg_row_len, sample_size, last_analyzed
        FROM USER_TABLES
        WHERE table_name='CMS_CATEGORY';


         SELECT num_distinct, density, low_value, high_value, last_analyzed,column_name
        FROM USER_TAB_COL_STATISTICS
        WHERE table_name='CMS_CATEGORY';
        ```
-------
- 출처 : http://www.gurubee.net/lecture/1740