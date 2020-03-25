
[Part2] - chapter 06
=========================

MVC Controller
--------
- @Controller
    ```java
    @Controller
    @RequestMapping("/sample/*")
    public class SampleController {

    }
    ```
    servlet-context.xml 을 통해 자동으로 Bean 등록   
    ```<context:component-scan>```태그를 통해 스캔후 객체로 생성

- @RequestMapping   
현재 클래스의 모든 메서드들의 기본적인 URL경로   
    > - method 속성 추가(GET / POST 방식 구분시 이용)   
    > - 스프링 4.3버전 이후 축약형으로 @GetMapping, @PostMapping 등장
    ```java
    // GET, POST 모두 지원할 때 배열로 처리하여 지정 가능 
    @RequestMapping(value="/basic", method= {RequestMethod.GET, RequestMethod.POST})
	public void basicGet() {
		log.info("basic get ...........................");
	}
	
    // 간편하지만 get 만 지원하여 제한이 많은편 
	@GetMapping("/basicOnlyGet")
	public void basicGet2() {
		log.info("basic get only get .................................");
	}
    ```    

#### Controller의 파라미터 수집
1. Controller의 메서드가 DTO를 파라미터로 사용하게 되면 자동으로 setter 메서드 동작   
2. 파라미터가 자동으로 수집. request.getParameter()를 사용하지 않아도 됨!
3. 파라미터 추가 호출 *http://localhost:8080/sample/ex01?name=AAA&age=20*    
    이때 기본 path 경로가 '/' 인지 확인

    ```java
    // DTO 클래스
    @Data
    public class SampleDTO {
        private String name;
        private int age;
    }

    // Controller 클래스
    @GetMapping("/ex01")
    public String ex01(SampleDTO dto) {
        log.info("" + dto);
        return "ex01";
    }
    ```
4. @RequestParam 을 이용한 전달    
    파라미터로 사용된 변수의 이름과 전달되는 파라미터의 이름이 다른 경우 유용 
    
    ```java
    @GetMapping("/ex02")
	public String ex02(@RequestParam("name") String name, @RequestParam("age") int age) {
		log.info("name : " + name);
		log.info("age : " + age);
		return "ex02";
	}
    ```
5. 동일한 이름의 파라미터가 여러개 전달 되는 경우 ArrayList<> 처리   
    파라미터 타입을 보고 객체를 생성하므로 실제적인 클래스 타입 지정   
    배열도 동일 처리 (String[] ids)
    ```java
    @GetMapping("/ex02List")
	public String ex02List(@RequestParam("ids") ArrayList<String> ids) {
		log.info("ids : " + ids);
		return "ex02List";
	}
    ```
   > http://localhost:8080/sample/ex02List?ids=111&ids=222&ids=333 을 호출할 경우,    
    INFO : org.zerock.controller.SampleController - ids : [111, 222, 333] 

6. 객체 리스트 전달 (DTO)    
    ```java
    // DTO 클래스
    @Data
    public class SampleDTOList {
        private List<SampleDTO> list;
        
        public SampleDTOList() {
            list = new ArrayList<>();
        }
    }

    // Controller 클래스 
    @GetMapping("/ex02Bean")
	public String ex02Bean(SampleDTOList list) {
		log.info("list dtos : " + list);
		return "ex02Bean";
	}
    ```
    > http://localhost:8080/sample/ex02Bean?list[0].name=aaa&list[2].name=bbb 호출 할 경우, (encodeURIComponent() 미사용 시 []는 유니코드로 바꿔줘야함. )   
    INFO : org.zerock.controller.SampleController - list dtos : SampleDTOList(list=[SampleDTO(name=aaa, age=0), SampleDTO(name=null, age=0), SampleDTO(name=bbb, age=0)])

7. @InitBinder   
변수 타입이 Date 일 경우 '2018-01-01'과 같은 형태의 데이터 일 때 사용하여 정상적인 파라미터를 수집
    > INFO : org.zerock.controller.SampleController - todo : TodoDTO(title=test, dueDate=Mon Jan 01 00:00:00 KST 2018)   

    혹은 DTO의 파라미터로 사용되는 인스턴스 변수에    
    @DateTimeFormat(pattern = "yyyy/MM/dd")    로 적용, 변환 


