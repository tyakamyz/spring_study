# @RestController 반환타입
- JSP와 달리 순수한 데이터를 반환하는 형태이므로 다양한 포맷의 데이터 전송 가능
    - 주로 많이 사용 형태 : 일반문자열, JSON, XML 등
    - produces는 MIME TYPE을 의미
>일반 문자열 반환
- @Controller에서는 문자열 return값이 jsp 파일의 이름으로 처리되지만, @RestController에서는 순수한 문자열로 처리 됨
- SampleController.java
```java
package org.zerock.controller;

import org.springframework.http.MediaType;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import lombok.extern.log4j.Log4j;

@RestController
@RequestMapping("/sample")
@Log4j
public class SampleController {
    
    @GetMapping(value = "/getText", produces = "text/plain; charset=UTF-8")
    public String getText() {
        log.info("MIME TYPE : " + MediaType.TEXT_PLAIN_VALUE);
        
        return "안녕하세요";
    }
    
}
```
>객체 반환
- SampleVO.java
```java
package org.zerock.domain;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@AllArgsConstructor
@NoArgsConstructor
public class SampleVO {
    private Integer mno;
    private String firstName;
    private String lastName;
}
```
- SampleController.java
    - /sample/getSample 로 호출하는 경우 json 타입의 데이터가 전달되는 것을 확인 할수 있음
    - /sample/getSample.json 으로 호출하는 경우 json 타입의 데이터가 전달되는 것을 확인 할수 있음
```java
@GetMapping(value="/getSample", produces={MediaType.APPLICATION_JSON_UTF8_VALUE, MediaType.APPLICATION_XML_VALUE})
public SampleVO getSample() {
    return new SampleVO(112, "스타", "로드");
}
```
- SampleController.java
    - produces 생략가능 (결과는 위와 같음)
```java
@GetMapping(value="/getSample2")
public SampleVO getSample2() {
    return new SampleVO(112, "스타2", "로드2");
}
```
>컬렉션 타입의 객체 반환
- SampleContoller.java
```java
@GetMapping(value="/getList")
public List<SampleVO> getList(){
    return IntStream.range(1,10).mapToObj(i -> new SampleVO(i, i+"First", i+ " Last")).collect(Collectors.toList());
}

@GetMapping(value="/getMap")
public Map<String, SampleVO> getMap(){
    Map<String,SampleVO> map = new HashMap<>();
    map.put("First", new SampleVO(111,"그루트", "주니어"));
    
    return map;
}
```
> ResponseEntity 타입
- SampleController.java
    - REST 호출 방식의 경우 데이터 자체를 전송하는 방식으로 처리되기 때문에 데이터를 요청한 쪽에서 정상적인 데이터인지 비정산적인 데이터인지 구분할 수 있는 방법이 필요
    - 예제의 경우 height값이 150 미만인 값을 파라미터로 받게되면 502 에러를 같이 보냄(개발자도구 - 네트워크에서 확인 가능)
```java
@GetMapping(value="/check", params= {"height", "weight"})
public ResponseEntity<SampleVO> check(Double height, Double weight){
    SampleVO vo = new SampleVO(0, "" + height, "" + weight);
    
    ResponseEntity<SampleVO> result = null;
    
    if(height < 150) {
        result = ResponseEntity.status(HttpStatus.BAD_GATEWAY).body(vo);
    }else {
        result = ResponseEntity.status(HttpStatus.OK).body(vo);
    }
    
    return result;
}
```