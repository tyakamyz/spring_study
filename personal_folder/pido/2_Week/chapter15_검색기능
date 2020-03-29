
[Part3] - chapter 15
=========================

검색
-----------------

### SQL

단일 검색 시,    
인라인뷰에서 검색조건을 추가하여 검색하고 그 후 Rownum 조건을 붙인다.

다중 검색 시,   
우선 순위 연산자인 '()' 을 이용해 OR 조건들을 처리한다.    
(AND 연산자가 OR 연산자보다 우선순위가 높기 때문)   
    ```
    select * from (
        select /*+INDEX_DESC(tbl_board pk_board) */
            rownum rn, bno, title, content, writer, regdate, updatedate
        from tbl_board
        where (title like '%TEST%' or content like '%TEST%')
        and rownum <= 20
    )
    where rn > 10;     
    ```
<hr />

### Mybatis

동적 태그 기능을 사용   
* if
* choose(when, otherwise)
* trim(where, set)
* foreach

1. if
    ```xml
    <if test="type == 'W'.toString()">
        (writer like '%' || #{keyword} || '%')
    </if>    
    ```
    if 안에 들어가는 표현식은 OGNL 표현식 사용    

2. choose
<choose>
    <when test="type == 'T'.toString()">
        (title like '%' || #{keyword} || '%')
    </when>
    <when test="type == 'C'.toString()">
        (content like '%' || #{keyword} || '%')
    </when>
    <when test="type == 'W'.toString()">
        (writer like '%' || #{keyword} || '%')
    </when>
    <otherwise>
        (title like '%' || #{keyword} || '%' OR '%' || #{keyword} || '%')
    </otherwise>
</choose>    

3. trim, where, set
    ```xml
    select * from tbl_board
    <where>
        <if test="bno != null">
            bno = #{bno}
        </if>
        <trim prefixOverrides = "and">
            rownum = 1
        </trim>    
    </where>
    ```
    where : 안에서 sql 이 생성되면 where 가 붙고 아니면 붙지 않는다.    
    trim : 앞의 내용과 관련하여 원하는 접두/접미를 처리

4. foreach
List, 배열, 맵 등을 이용해 루프 처리 
    ```xml
    select * from tbl_board
    <trim prefix="(" suffix=") AND " prefixOverrides="OR">
        <foreach item='val' index='key' collection="map">
            <trim prefix="OR">
                <if test="type == 'T'.toString()">
                    title like '%' || #{val} || '%'
                </if>
                <if test="type == 'C'.toString()">
                    content like '%' || #{val} || '%'
                </if>
                <if test="type == 'W'.toString()">
                    writer like '%' || #{val} || '%'
                </if>
            </trim>
        </foreach>
    </trim>
    ```
    배열일경우 item 만 사용 
    map 형태일 경우 item 과 index 사용 

<hr />

### Spring

java (페이징처리했던 공통 클래스 활용)
```java
...

// search
private String type;
private String keyword;

// 검색조건을 배열로 처리
public String[] getTypeArr() {
    return type == null? new String[] {}: type.split("");
}
```

xml 
```xml
<sql id="criteria">
    <trim prefix="(" suffix=") AND " prefixOverrides="OR">
        <foreach item='type' collection="typeArr">
            <trim prefix="OR">
                <choose>
                    <when test="type == 'T'.toString()">
                        title like '%' || #{keyword} || '%'
                    </when>
                    <when test="type == 'C'.toString()">
                        content like '%' || #{keyword} || '%'
                    </when>
                    <when test="type == 'W'.toString()">
                        writer like '%' || #{keyword} || '%'
                    </when>
                </choose>
            </trim>
        </foreach>
    </trim>
</sql>

<select id="getListWithPaging" resultType="org.zerock.domain.BoardVO">
    <![CDATA[
        select 
                bno
            ,	title
            ,	content
            ,	writer
            ,	regdate
            ,	updatedate
        from (
            select /*+ INDEX_DESC(tbl_board pk_board) */
                    rownum rn
                ,	bno
                ,	title
                ,	content
                ,	writer
                ,	regdate
                ,	updatedate
            from tbl_board
            where 
            ]]>
            <include refid="criteria"></include>
        <![CDATA[
            rownum <= #{pageNum} * #{amount}
            )
        where rn > (#{pageNum} -1) * #{amount}
        ]]>
</select>
```
동적 SQL 처리 부분은 ```<sql>``` 태그를 이용하여 별도로 보관하고 필요한 경우 include 시키는 형태로 사용한다.


jsp
```jsp
<form id='searchForm' action="/board/list" method='get'>
    <select name='type'>
        <option value="" 
            <c:out value="${pageMaker.cri.type == null?'selected':''}"/>>--</option>
        <option value="T"
            <c:out value="${pageMaker.cri.type == 'T'?'selected':''}"/>>제목</option>
        <option value="C"
            <c:out value="${pageMaker.cri.type == 'C'?'selected':''}"/>>내용</option>
        <option value="W"
            <c:out value="${pageMaker.cri.type == 'W'?'selected':''}"/>>작성자</option>
        <option value="TC"
            <c:out value="${pageMaker.cri.type == 'TC'?'selected':''}"/>>제목 or 내용</option>
        <option value="TW"
            <c:out value="${pageMaker.cri.type == 'TW'?'selected':''}"/>>제목 or 작성자</option>
        <option value="TWC"
            <c:out value="${pageMaker.cri.type == 'TWC'?'selected':''}"/>>제목 or 내용 or 작성자</option>
    </select>
    <input type='text' 		name='keyword' 	value='<c:out value="${pageMaker.cri.keyword}"/>' />
    <input type='hidden' 	name ='pageNum' value='<c:out value="${pageMaker.cri.pageNum}"/>' />
    <input type='hidden'	name ='amount' 	value='<c:out value="${pageMaker.cri.amount}"/>' />
    <button class='btn btn-default'>Search</button> 
</form>
```
- ```<form>``` 에 담아 검색을 처리한다.   
- 검색 후에는 주소창에 검색 조건과 키워드가 같이 GET 방식으로 처리되므로 ```<select>```태그에 조건을 걸어 해당 조건으로 검색되었다면 selected라는 문자열을 출력하게 하여 선택항목으로 보이게 처리한다. 
- 검색값을 유지하기위해 페이징과 같이 hidden 값 처리
    ```jsp
    <form id="actionForm" action="/board/list" method='get'>
        <input type='hidden' name='pageNum' value='${pageMaker.cri.pageNum }'>
        <input type='hidden' name='amount' value='${pageMaker.cri.amount }'>  
        <input type='hidden' name='type' value='<c:out value="${pageMaker.cri.type }"/>'>
        <input type='hidden' name='keyword' value='<c:out value="${pageMaker.cri.keyword }"/>'>                    	
    </form>
    ```

페이징과 마찬가지로 모든 페이지 이동 과정에서 검색 조건이 따라가야하므로 
동일한 방식으로 java , jsp , javascript 에 type 과 keyword 를 넘겨주어야 한다. 

<br>

**UriComponentsBuilder**   

매번 파라미터를 유지하는일이 번거로울경우...   
web.util.UriComponentsBuilder 는 여러개의 파라미터들을 연결해서 URL 형태로 만들어준다!

```java
/**
* UriComponentsBuilder 를 통해 파라미터들을 손쉽게 추가
* @return GET방식의 인코딩 URL으로 생성
*/
public String getListLink() {
    
    UriComponentsBuilder builder = UriComponentsBuilder.fromPath("")
        .queryParam("pageNum", this.pageNum)
        .queryParam("amount", this.getAmount())
        .queryParam("type", this.getType())
        .queryParam("keyword", this.getKeyword());
    
    return builder.toUriString();
}
```
- queryParam() 메서드로 필요한 파라미터를 추가   
- GET방식에 적합한 URL 인코딩 결과로 만들어진다. 한글처리에 신경쓰지 않아도 됨!

```java
// getListLink() 를 사용하여 간단하게 정리!
@PostMapping("/modify")
public String modify(BoardVO board, @ModelAttribute("cri") Criteria cri, RedirectAttributes rttr) {
    if(service.modify(board)) {
        rttr.addFlashAttribute("result", "success");
    }
    return "redirect:/board/list" + cri.getListLink();
}
```
