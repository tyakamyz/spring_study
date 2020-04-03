# Oracle Index 사용

## Index

 - SQL Index 안잡힐 경우

```sql
analyze table tbl_board(table name) compute statistics;
 ```
## Hint

### 유의 사항

 - Tabble Alias 쓸 경우 Hint의 사용 테이블과 Alias명을 동일시해야 작동한다.
```sql
select /*+INDEX_DESC(tbl_board pk_board) */
rownum rn, bno, title, content
from tbl_board
where rownum <= 10;
```

```sql
select /*+INDEX_DESC(tbl_board pk_board) */
rownum rn, bno, title, content
from tbl_board a
where rownum <= 10;
```