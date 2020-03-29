##  MyBatis의 동적 태그들
  - 동적 태그
    - if
    - choose(when, otherwise)
    - trim(where, set)
    - foreach
 ### \<if>  

- if 는 test라는 속성과 함께 특정한 조건이 true가 되었을 때 포함된 SQL을 사용하고자 할 때 작성
    - ex)
    - 검색 조건이 'T'면 제목(title)이 키워드(keyword)인 항목 검색
    - 검색 조건이 'C'면 내용(content)이 키워드(keyword)인 항목 검색
    - 검색 조건이 'W'면 작성자(writer)이 키워드(keyword)인 항목 검색
    
        ```xml
        <if test="type == 'T'.toString()">
            (title like '%'||#{keyword}||'%')
        </if>
        <if test="type == 'C'.toString()">
            (content like '%'||#{keyword}||'%")
        </if>
        <if test="type == 'W'.toString()">
            (writer like '%'||#{keyword}||'%")
        </if>
        ```
- if 안에 들어가는 표현식(expression)은 OGNL 표현식이라는 것을 이용한다.
- 좀더 자세한 내용은 https://commons.apache.org/proper/commons-ognl/language-guide.html 참고

### \<choose>
- if와 달리 choose는 여러 상황들 중 하나의 상황에서만 동작한다.
- Java의 'if ~ else' 나 JSTL의 \<choose>와 유사
    ```xml
    <choose>
        <when test="type == 'T'.toString()">
            (title like '%'||#{keyword}||'%')
        </when>
        <when test="type == 'C'.toString()">
            (content like '%'||#{keyword}||'%')
        </when>
        <when test="type == 'W'.toString()">
            (writer like '%'||#{keyword}||'%')
        </when>
        <otherwise>
            (title like '%'||#{keyword}||'%' or content like '%'||#{keyword}||'%')
        </otherwise>
    </choose>
    ```
### \<trim>, \<where>, \<set>
 - trim,where,set은 단독으로 사용되지 않고, \<if>, \<choose>와 같은 태그들을 내포하여 SQL들을 연결해 주고, 앞 뒤에 필요한 구문들(AND, OR, WHERE 등)을 추가하거나 생략하는 역할을 한다.
- \<where>의 경우 태그 안 쪽에서 SQL이 생성될 때는 WHERE구문이 붙고, 그렇지 않은 경우에는 생성되지 않는다.
    ```sql
    select * frmo tbl_board
    <where>
        <if test="bno != null">
            bno = #{bno}
        </if>
    </where>
    ```
    | | |
    |--|--|
    |bno값이 존재하는 경우 | select * from tbl_board WHERE bno = 33
    |bno가 null인 경우|select * from tbl_board

- \<trim>은 태그의 내용을 앞의 내용과 관련되어 원하는 접두/접미를 처리할 수 있다.
    ```sql
    select * from tbl_board
    <where>
        <if test="bno != null">
            bno = #{bno}
        </if>
        <trim prefixOverrides ="and">
            rownum = 1
        </trim>
    </where>
    ```
    - trim은 prefix, suffix, prefixOverrides, suffixOverrides 속성을 지정할 수 있다.  

    | | |
    |--|--|
    |bno값이 존재하는 경우 | select * from tbl_board WHERE bno = 33 and rownum = 1
    |bno가 null인 경우|select * from tbl_board WHERE rownum = 1
 - \<foreach>는 List, 배열, 맵 등을 이용해서 루프를 처리할 수 있다.
    - 주로 IN 조건에서 많이 사용하지만, 경우에 따라서는 복잡한 WHERE 조건을 만들 떄에도 사용할 수 있다.
    - ex) 제목('T')은 'TTTT'로 내용('C')은 'CCCC'라는 값을 이용한다면 Map의 형태로 작성 가능
    ```java
        Map<String, String> map = new HashMap<>();
        map.put("T", "TTTT");
        map.put("C", "CCCC");
    ```
    - 작성된 Map을 파라미터로 전달하고, foreach를 이용
    ```xml
        <trim prefix="where (" suffix=")" prefixOverrides="OR">
            <foreach item="val" index="key" collection="map">
                <trim prefix = "or">
                    <if test="key == 'C'.toString()">
                        content = #{val}
                    </if>
                    <if test="key == 'T'.toString()">
                        title = #{val}
                    </if>
                    <if test="key == 'W'.toString()">
                        writer = #{val}
                    </if>
                </trim>
            </foreach>
        </trim>
    ```
    - foreach를 배열이나 List를 이용하는 경우에는 item 속성만을 이용하면 되고, Map의 형태로 key와 value를 이용해야 할 때는 index와 item 속성을 둘다 이용한다.