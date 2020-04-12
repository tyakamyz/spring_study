
Rest Annotation
=======================================
Index   
--------

[1. @RestController](#◈-@RestController)   
[2. @ResponseBody](#◈-@ResponseBody)     
[3. @PathVariable](#◈-@PathVariable)   
[4. @RequestParam](#◈-@RequestParam)   
[5. @CrossOrign](#◈-@CrossOrign)   
[6. @RequestBody](#◈-@RequestBody) 


------------------------------

## ◈ @RestController
* Controller가 REST 방식을 처리하기 위한 것임을 명시한다.
* @Controller에 @ResponseBody가 추가된 것이다.
* 주용도는 Json/Xml 형태로 객체 데이터를 반환하는 것이다.

    ```java
    @RestController
    @RequestMapping("/sample")
    @Log4j
    public class SampleController {
        ...
    }    
    ```

> Spring MVC @Controller와 RESTful 컨트롤러인 @RestController의 차이점은   
> **HTTP Response Body가 생성되는 방식**이다.   
> @Controller 는 **View Page를 반환**하지만,   
> @RestController는 객체(VO,DTO)를 반환하기만 하면, 객체데이터는 **application/json 형식의 HTTP ResponseBody에 직접 작성**되게 된다.



<hr>

## ◈ @ResponseBody
* 일반적인 JSP와 같은 뷰로 전달되는게 아니라 데이터 자체를 전달하기 위한 용도
* Spring MVC의 @Controller의 메소드에서 @ResponseBody 어노테이션 사용 시 반환값을 변환하여 HTTP Response에 자동으로 작성한다. 
* RestController에서는 내포하고 있는 속성이다.    
    ```java
    // Map 정보를 전송하기
    @Controller
    public class MainController {
        
        @ResponseBody
        @RequestMapping("/test")
        public HashMap<String, Object> init(@RequestBody HashMap<String, Object> map) {
            
            map.put("phone", "0000-0000");
            return map;
            // {"name": "kim", "age": 30, "phone": "0000-0000"}가 data로 바인딩
            // @ResponseBody가 붙은 메서드에서 Map을 반환하면 자동으로 Map 정보가 JSON 객체로 변환되어 전송된다.
        }
    }
    ```    
    ```java
    // 객체 정보를 전송하기 
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
##### 참고 : https://webcoding.tistory.com/entry/Spring-%EC%8A%A4%ED%94%84%EB%A7%81-RequestBody-ResponseBody-%EC%82%AC%EC%9A%A9%ED%95%98%EA%B8%B0-1

<hr>

## ◈ @PathVariable
* URL 경로에 있는 값을 파라미터로 추출하려고 할 때 사용한다.
* URI에서 각 구분자에 들어오는 값을 처리해야 할 때 사용한다.
* HTTP 요청에 대해 매칭되는 request parameter 값이 자동으로 들어간다.
    ```java
    @GetMapping(value="/product/{cat}/{pid}")
	public String[] getPath(@PathVariable("cat") String cat, @PathVariable("pid") Integer pid) {
		return new String[] { "category : " + cat, "productid : " + pid };
	}
    ```
<hr>

## ◈ @RequestParam
* Controller 단에서 클라이언트에서 URL에 파라미터를 같이 전달하는 경우 사용.
* 파라미터의 값과 이름을 함께 전달하는 방식
* @RequestParam 어노테이션의 괄호 안의 문자열이 전달 인자 이름(실제 값을 표시)이다.
* ex) ` http://localhost:8080/home?index=1&page=2`
    ```java
    @GetMapping("/home")
    public String show(@RequestParam("page") int pageNum {
        ...
    }    
    ```

※ <U>[파라미터를 받는 방법]</U> @RequestParam와 @PathVariable 동시 사용 예제
```java
@GetMapping("/user/{userId}/invoices")
public List<Invoice> listUsersInvoices(@PathVariable("userId") int user,
	                                  @RequestParam(value = "date", required = false) Date dateOrNull) {
}
```
위의 경우 GET/user/{userId}invoices?date=190101 와 같이 uri가 전달될 때,   
구분자 {userId}는 @PathVariable(“userId”)로,
뒤에 이어붙은 parameter는 @RequestParam(“date”)로 받아온다.

##### 참고 : https://gmlwjd9405.github.io/2018/12/02/spring-annotation-types.html

<hr>

## ◈ @CrossOrign
* Ajax의 크로스 도메인 문제를 해결해주는 어노테이션   
1. 크로스도메인 이슈   
웹 브라우저에서 Ajax 등을 통해 다른 도메인의 서버에 url(data)를 호출할 경우, 나타나는 보안문제
2. CORS(Cross-origin resource sharing)   
웹 페이지의 제한된 자원을 외부 도메인에서 접근을 허용해주는 메커니즘
3. 스프링에서 CORS 설정하는 방법   
스프링  RESTful Service에서 CORS를 설정은 @CrossOrigin 어노테이션을 사용하여 간단히 해결 할 수 있다.

    ```java
    @CrossOrigin(origins = "*")
    @RestController
    public class crossController {

        @CrossOrigin(origins = "http://localhost:9000")
        @GetMapping("/greeting")
        public Greeting greeting(){
            ...
        }            
    }    
    ```
* RestController를 사용한 클래스 자체에 적용할 수 도 있고, 특정 rest api method에도 설정 가능.
* 특정 도메인만 접속을 허용할 수도 있습니다.    
@CrossOrigin(origins = "허용주소:포트")

##### 참고 : https://ithub.tistory.com/63

<hr>

## ◈ @RequestBody
* HTTP 요청 Body를 추출하여 파라미터로 변환받아 자바 객체로 전달받을 수 있는 방식.
* 반드시 HTTP POST 요청일 경우에만 처리할 수 있다.    
> *GET 메소드 요청의 경우에는 HTTP Body에 요청이 전달되는 것이 아니라, URL의 파라미터 전달 (ex: ` http://localhost:8080/test?id=admin&name=hanq…`) 형식으로 전달되기 때문에 @RequestBody로 받으려고 해도 서로 다른 곳을 보며 데이터가 없다는 결과를 던질 수 밖에 없다.*

```javascript
// 컨트롤러로 보낼 JSON body
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
```java
// JSON을 Map 형태로 변환
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
```java
// JSON을 객체 형태로 변환
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
##### 참고 : https://webcoding.tistory.com/entry/Spring-%EC%8A%A4%ED%94%84%EB%A7%81-RequestBody-ResponseBody-%EC%82%AC%EC%9A%A9%ED%95%98%EA%B8%B0-1

<hr>



