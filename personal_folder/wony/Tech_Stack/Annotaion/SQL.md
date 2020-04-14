# Annotation

 1. [@Select](#Select)
 1. [@Param](#Param)
 1. [@MapperScan](#MapperScan)

# Explanation

## @Select

 - xml 파일이아닌 어노테이션만으로도 select, update, insert delete 등의 쿼리를 실행이 가능하다.
 - 예 ) 
 ```java
    @Select("SELECT * FROM employee WHERE id = #{id}")
    @ResultMap("employeeResultMap")
    Employee findById(long id);
 ```
 - Select 의 결과값을 @ResultMap 으로 받을 수 있다.

**[⬆ go to top](#Annotation)**

----
## @Param
 - MyBatis는 두개 이상의 데이터를 쿼리에 전달하기 위해서는 별도의 객체 사용, Map 과같은 인터페이스 또는 @Param을 이용한다.

 - 예)
 ```java
 public List<ReplyVO> getListWithPageing(@Param("cri") Criteria cri,@Param("bno") Long bno);
 ```
 - xml 쿼리에서의 사용은 '#{cri}', '#{bno}'로 사용 가능하다.

**[⬆ go to top](#Annotation)**

----
## @MapperScan

 - Java Config 를 통한 Mapper 설정 어노테이션
 - xml config 방식의 \<mybatis:scan\> 역할을 한다.

**[⬆ go to top](#Annotation)**

----
