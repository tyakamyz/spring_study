# Annotation
 1. [@RequestMapping](#RequestMapping)
 1. [@PostMapping](#PostMapping)
 1. [@GetMapping](#GetMapping)
 1. [@PutMapping](#PutMapping)
 1. [@PatchMapping](#PatchMapping)
 1. [@DeleteMapping](#DeleteMapping)
 1. [@RestController](#RestController)
 1. [@RequestBody](#RequestBody)
 1. [@ResponseBody](#ResponseBody)
 1. [@PathVariable](#PathVariable)
 1. [@CrossOrigin](#CrossOrigin)
 1. [@Consumes](#Consumes)
 1. [@Produces](#Produces)

# Explanation

## @RequestMapping
 
- URL 에 따른 기본적인 요청 처리 어노테이션
```java
@RequestMapping(value = "/test", method = RequestMethod.GET) //
```

**[⬆ go to top](#Annotation)**

----
## @PostMapping

 - Spring 4.3 이후 부터 사용 할 수 있게된 URL 요청처리
 - 등록에 사용

```java
 @PostMapping("/post")
public @ResponseBody ResponseEntity<String> post() {
    return new ResponseEntity<String>("POST Response", HttpStatus.OK);
}
```
 
**[⬆ go to top](#Annotation)**

----
## @GetMapping

  - Spring 4.3 이후 부터 사용 할 수 있게된 URL 요청처리
 - 조회에 사용
```java
@GetMapping(value = "/test2") public void TestCase2(HttpServletRequest request, HttpServletResponse response) { // do something }
```
**[⬆ go to top](#Annotation)**

----
## @PutMapping
 
  - Spring 4.3 이후 부터 사용 할 수 있게된 URL 요청처리
 - 수정에 사용
```java
@PutMapping("/put")
public @ResponseBody ResponseEntity<String> put() {
    return new ResponseEntity<String>("PUT Response", HttpStatus.OK);
}
```
**[⬆ go to top](#Annotation)**

----
## @PatchMapping

 - Spring 4.3 이후 부터 사용 할 수 있게된 URL 요청처리
 - 수정에 사용
 - @PutMapping이 자원의 전제 교체라면(전달해야할 필드들 모두 Null or 초기값)  
- @PatchMapping은 자원의 부분 교체(전달할 값만 교체)

```java
@PatchMapping("/patch")
public @ResponseBody ResponseEntity<String> patch() {
    return new ResponseEntity<String>("PATCH Response", HttpStatus.OK);
}
```
 
**[⬆ go to top](#Annotation)**

----
## @DeleteMapping
 
 - Spring 4.3 이후 부터 사용 할 수 있게된 URL 요청처리
 - 삭제에 사용
 ```java
 @DeleteMapping("/delete")
public @ResponseBody ResponseEntity<String> delete() {
    return new ResponseEntity<String>("DELETE Response", HttpStatus.OK);
}
 ```
**[⬆ go to top](#Annotation)**

----
## @RestController

 - View 를 전달하는 @Controller 와다르게 문자열, 객체, JSON 등의 데이터 자체를 전달함에 목적이 있다.
 
**[⬆ go to top](#Annotation)**

----
## @RequestBody

 - HTTP 요청 객체를 자바 객체로 전달 받을 수 있다.
 - 요청 예)
 ```javascript
var obj = {"name": "kim", "age": 30};

$.ajax({
    url: "/test",
    type: "post",
    data: JSON.stringify(obj),
    contentType: "application/json",
    success: function(data) {
        alert("성공");
    },
    error: function(errorThrown) {
        alert(errorThrown.statusText);
    }
});
 ```
 - 변환 예 1)
 ```java
 @Controller
public class MainController {
	
    @ResponseBody
    @RequestMapping("/test")
    public void init(@RequestBody HashMap<String, Object> map) {
    
    	System.out.println(map);
    	// {name=kim, age=30} 출력
    }
}
 ```
 - 변환 예 2)
 ```java
 @Controller
public class MainController {
	
    @ResponseBody
    @RequestMapping("/test")
    public void init(@RequestBody UserVO userVO) {
        
    	userVO.getName(); // "kim"
        userVO.getAge(); // 30
    }
}
 ```
 - 이 떄 UserVO 클래스의 프로퍼티는 전송된 JSON 객체와 프로퍼티명이 일치해야 하고 getter, setter 가 있어야 한다.

**[⬆ go to top](#Annotation)**

----
## @ResponseBody

 - 자바 객체를 HTTP 응답 객체로 변환하여 전송 가능 하다.
 - @RestController 사용할 경우 자동으로 @ReponseBody가 붙기에 별도로 사용 안하여도 된다.
 - 응답 예 1)
 ```java
 @Controller
public class MainController {
	
    @ResponseBody
    @RequestMapping("/test")
    public HashMap<String, Object> init(@RequestBody HashMap<String, Object> map) {
    	
        map.put("phone", "0000-0000");
    	return map;
        // {"name": "kim", "age": 30, "phone": "0000-0000"}가 data로 바인딩
    }
}
 ```
 - 응답 예 2)
```java
@Controller
public class MainController {
	
    @ResponseBody
    @RequestMapping("/test")
    public HashMap<String, Object> init(@RequestBody UserVO userVO) {
        
    	HashMap<String, Object> map = HashMap<String, Object>();
        map.put("userVO", userVO);
        
        return map;
        // {"userVO": {name: "kim", age: 30}}가 data로 바인딩
    }
}
```
 
**[⬆ go to top](#Annotation)**

----
## @PathVariable
 
 - URL 상에 변수를 가져오는 어노테이션
 - @RequestParam 의 REST 화?
 - 예)
```java
@RequestMapping(value = "user/email/{email:.+}", method = RequestMethod.GET)
public ModelAndView getUserByEmail(@PathVariable("email") String email) {
```

**[⬆ go to top](#Annotation)**

----
## @CrossOrigin

 - CORS란
    - CORS(Cross-origin resource sharing)이란, 웹 페이지의 제한된 자원을 외부 도메인에서 접근을 허용해주는 메커니즘이다.
    - API에서의 외부 접근을 제한시키는 Same-origin policy와 반대의 개념으로 일부 외부의 접근을 허용을 설정해주는 개념이다.
 - Default 로는 '모든 도메인, 모든 요청방식'에 대해 허용한다.
 - 예 1)
 ```java
 @RestController
@RequestMapping("/account")
public class AccountController {

	@CrossOrigin
	@RequestMapping("/{id}")
	public Account retrieve(@PathVariable Long id) {
		// ...
	}

	@RequestMapping(method = RequestMethod.DELETE, value = "/{id}")
	public void remove(@PathVariable Long id) {
		// ...
	}
}
 ```
- @CrossOrigin이 설정된 retrieve()에 한해 모든 도메인, 오청방식에 대해 허용한다.
- 에 2)
```java
@CrossOrigin(origins = "http://domain1.com, http://domain2.com")
@RestController
@RequestMapping("/account")
public class AccountController {

	@RequestMapping("/{id}")
	public Account retrieve(@PathVariable Long id) {
		// ...
	}

	@RequestMapping(method = RequestMethod.DELETE, value = "/{id}")
	public void remove(@PathVariable Long id) {
		// ...
	}
}
```
- AccountController 클래스에 대해 http://domain1.com, http://domain2.com의 도메인만을 허용한다.
 
**[⬆ go to top](#Annotation)**

----

## @Consumes
 

 - 수신하고자 하는 데이터 포맷 정의
 - 예 1)
 ```java
 @Consumes(MediaType.APPLICATION_JSON)
 public void test(){

 }
 ```
 - 에 2)
 ```java
 @Consumes({"application/json","text/html"})
 public void test(){

 }
 ```

 **[⬆ go to top](#Annotation)**
 
----

## @Produces
 
  - 출력하고자 하는 데이터 포맷 정의
  - 예 1)
  ```java
  @Produces("text/plain")
  public class test{

      @Produces("text/html")
      public String goTest(){}

  }
  ```
   - 예 2)
  ```java
  @Produces({MediaType.TEXT_PLAIN_VALUE, MediaType.APPLICATION_JSON}))
  public class test{  }
  ```

**[⬆ go to top](#Annotation)**

----

- 출처 : https://www.baeldung.com/spring-new-requestmapping-shortcuts
- 출처 : https://webcoding.tistory.com/entry/Spring-스프링-RequestBody-ResponseBody-사용하기-1
- 출처 : http://jmlim.github.io/spring/2018/12/11/spring-boot-crossorigin/
- 출처 : https://m.blog.naver.com/PostView.nhn?blogId=writer0713&logNo=220922066642&proxyReferer=https:%2F%2Fwww.google.com%2F