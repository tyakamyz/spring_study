# Annotation

 1. [@ModelAttribute](#ModelAttribute)
 1. [@RedirectAttributes](#RedirectAttributes)
 1. [@Autowired](#Autowired)
 1. [@Configuration](#Configuration) 
 1. [@Component](#Component)
 1. [@Bean](#Bean)
 1. [@RequestParam](#RequestParam)
 1. [@InitBinder](#InitBinder)
 1. [@DateTimeFormat](#DateTimeFormat)
 1. [@ExceptionHandler](#ExceptionHandler)
 1. [@ControllerAdvice](#ControllerAdvice)
 1. [@ResponseStatus](#ResponseStatus) 
 1. [@ComponentScan](#ComponentScan)
 1. [@Transactional](#Transactional)
 1. [@RequestHeader](#RequestHeader)
 1. [@Scheduled](#Scheduled)


# Explanation

## @ModelAttribute

- Spring에서 JSP파일에 반환되는 Model 객체에 속성값을 주입하거나, 바인딩할 때 사용되는 어노 테이션

1. 메서드에 사용하는 방식
```java
@ModelAttribute("serverTime")
public String getServerTime(Locale locale) {
    Date date = new Date();
    DateFormat dateFormat = DateFormat.getDateTimeInstance(DateFormat.LONG, DateFormat.LONG, locale);
    return dateFormat.format(date);
}
```

 - 위에서 serverTime 이름에 dateFormat.format(date)값을 바인딩한다. 해당 값은 ${serverTime} 형태로 JSP파일에서 사용 가능하다.

 2. 메서드 인수에 사용하는 방식
 ```java
 @RequestMapping(value="/memJoin", method=RequestMethod.POST)
public String memJoin(@ModelAttribute("mem") Member member) {
    service.memberRegister(member);
    return "memJoinOk";
}
 ```

 - HTTP요청에 들어있는 속성값들을 Member 객체에 자동으로 바인딩하게 된다. @ModelAttribute("[Name]") 형태로 사용할 경우 JSP 파일에서 ${[Name].property} 형태로 Model 객체 값을 사용 할 수 있다.
 - 전달될 때에는 클래스명의 앞글자는 소문자로 처리된다.
 - 반면에 기본 자료형의 경우는 파라미터로 선언하더라도 기본적으로 화면까지 전달되지 않는다. 때문에 강제로 전달받은 파라미터를 Model에 담아서 전달하고자 할때 필요한 어노테이션이다.
 - @ModelAttribute 가 걸린 파라미터는 타입에 관계없이 무조건 Model에 담아서 전달되므로, 파라미터로 전달된 데이터를 다시 화면에서 사용해야 할 경우에 유용하게 사용된다.

**[⬆ go to top](#Annotation)**

----
## @RedirectAttributes

- Model 타입과 더불어 스프링 MVC가 자동으로 전달해 주는 타입 중에는 RedirectAttributes타입이 존재한다.
- 일회성으로 데이터를 전달하는 용도로 사용
- 기존의 Servlet에서 response.sendRedirect()를 사용할 떄와 동일한 용도로 사용된다.


**[⬆ go to top](#Annotation)**

----
## @Autowired

 - IoC컨테이너의 빈을 자동으로 주입하는 어노테이션
 - 예)
 ```java
 public class test{
	 @Autowried
	 public test2 test;
 }
 ```
 - test2 test = new test2(); 를 주입한 결과와 같다.

**[⬆ go to top](#Annotation)**

----
## @Configuration

 - Ioc 컨테이너에게 선언된 클래스가 Bean 구성 클래스임을 명시한다.
 - 하나이상의 @Bean을 가지고 있다.
 - 예)
 ```java
 @Configuration
 public class test{
	 @Bean
	 public ArrayList<String> testS(ArrayList<String> array){

		 return new array;
	 }
 }
 ```
**[⬆ go to top](#Annotation)**

----
## @Component

 - 컨트롤이 불가능한 외부라이브러리등을 Bean으로 등록하고자 하는 @Bean과 다르게 개발자가 컨트롤이 가능한 Class에 선언한다.
 - 예)
```java
@Component
@Aspect
public class CheckSessionValid {

    @Autowired
    private SessionTokenRedisRepository sessionTokenRedisRepository;

    @Pointcut("execution(public * com.finder.genie_ai.controller.ShopController(..)) && args(token)")
    public void controllerClassMethods(String token) {}

    //Todo search @AspectJ
    @Before("controllerClassMethods(token)")
    public void checkSessionValid(String token) {
        if (!sessionTokenRedisRepository.isSessionValid(token)) {
            throw new UnauthorizedException();
        }
    }

}
```
**[⬆ go to top](#Annotation)**

----
## @Bean

 - 컨트롤이 불가능한 외부 라이브러리등을 Bean으로 등록하고자 할 때 사용한다.
 - 예)
 ```java
 public class RedisConfig {

    private @Value("${spring.redis.host}") String redisHost;
    private @Value("${spring.redis.port}") int redisPort;
    private @Value("${spring.redis.password}") String password;

    @Bean
    public JedisPoolConfig jedisPoolConfig() {
        JedisPoolConfig jedisPoolConfig = new JedisPoolConfig();
        jedisPoolConfig.setMaxTotal(30);
        jedisPoolConfig.setTestOnBorrow(true);
        jedisPoolConfig.setTestOnReturn(true);
        return jedisPoolConfig;
    }
 }
 ```

**[⬆ go to top](#Annotation)**

----
## @RequestParam

 - 스프링은 파라미터의 타입을 보고 객체를 생성하므로 파라미터의 타입은 List<>와같은 **인터페이스** 타입이아닌 실제적인 **클래스** 타입으로 지정한다 ex) ArrayList<> 타입

**[⬆ go to top](#Annotation)**

----
## @InitBinder
 - DTO에서와의 같이 파라미터값의 바인딩이 자동으로 되는 경우가 있지만. 그렇지 않은 경우 역시 존재한다. 예로 '2018-01-01'과 같은 날짜 데이터를 java.util.Date 타입으로 변환하는 작업이 그러하다.
    - 스프링 Controller에서는 파라미터를 바인딩할 때 자동으로 호출되는 @InitBinder를 이용해서 이러한 변환을 처리할 수 있다.
 ```java
 @InitBinder
	public void initBinder(WebDataBinder binder) {
		
		SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd");
		binder.registerCustomEditor(java.util.Date.class, new CustomDateEditor(dateFormat, false));
		
	}
```

**[⬆ go to top](#Annotation)**

----
## @DateTimeFormat

 - @InitBinder 와 동일한 작업(날짜 데이터를 java.util.Date 타입으로 변환)
 - @DatetimeFormat 사용 시 @initBinder는 제거 해야한다.(충돌 에러 발생)
```java
@DateTimeFormat(pattern = "yyyy/MM/dd")
	private Date dueDate;
```

**[⬆ go to top](#Annotation)**

----
## @ExceptionHandler
 - 해당 메서드가 () 들어가는 예외 타입을 처리한다는 것을 의미
 - 속성으로 Exception 클래스 타입을 지정할 수 있다.
 - 아래의 예제로는 Exception.class를 지정하였으므로 모든 예외에 대한 처리가 except()만을 이용해서 처리할 수 있다.
```java
@ControllerAdvice
@Log4j
public class CommonExceptionAdvice {
	
	@ExceptionHandler(Exception.class)
	public String except(Exception ex, Model model) {
		
		log.error("Exception ......." + ex.getMessage());
		model.addAttribute("exception", ex);
		log.error(model);
		
		return "erro_page";
	}

}
```

**[⬆ go to top](#Annotation)**

----
## @ControllerAdvice
 - 해당 객체가 스프링의 컨트롤러에서 발생하는 예외를 처리하는 존재임을 명시하는 용도
 - @ExceptionHandler 보다 큰 범위에서 사용
 ```java
@ControllerAdvice
@Log4j
public class CommonExceptionAdvice {
	
	@ExceptionHandler(Exception.class)
	public String except(Exception ex, Model model) {
		
		log.error("Exception ......." + ex.getMessage());
		model.addAttribute("exception", ex);
		log.error(model);
		
		return "erro_page";
	}

}
```

**[⬆ go to top](#Annotation)**

----
## @ResponseStatus

 - HTTP 상태 코드를 제어하는 어노테이션
 - ExceptionHandler와 같이 조합해서 사용가능

 ```java
 @ExceptionHandler(DataFormatException.class)
 @ResponseStatus(value = HttpStatus.NOT_FOUND, reason = "My Response Status Change….!!")
 public void handleDataFormatException(DataFormatException ex,
     HttpServletResponse response) {
 
     logger.info("Handlng DataFormatException - Catching: " + ex.getClass().getSimpleName());
 
 ```
- ReponsCode의 변환가능

**[⬆ go to top](#Annotation)**

----
## @ComponentScan

 - xml에서 일일히 Bean등록을 하지않고 @ComponentScan으로 자동으로 빈등록 처리한다.

**[⬆ go to top](#Annotation)**

----
## @Transactional
 
  - 트랜잭션 설정 어노테이션

  - 전파 속성

|속성|설명|
|--|--|
|PROPAGATION_MADATORY|작업은 반드시 특정한 트랜잭션이 존재한 사앹에서만 가능|
|PROPAGATION_NESTED|기존에 트랜잭션이 있는 경우, 포함되어서 실행|
|PROPAGATION_NEVER|트랜잭션 상황하에 실행되면 예외 발생|
|PROPAGATION_NOT_SUPPORTED|트랜잭션이 있는 겨웅엔 트랜잭션이 끝날 때까지 보류된 후 실행
|PROPAGATION_REQUIRED|트랜잭션이 있으면 그 상황에서 실행, 없으면 새로운 트랜 잭션 실행(기본값)
|PROPAGATION_REQUIRED_NEW|대상은 자신만의 고유한 트랜잭션으로 실행
|PROPAGATION_SUPPORTS|트랜잭션을 필요로 하지 않으나, 트랜잭션 상황하에 있다면 포함되어 실행

-  격리 레벨

|레벨|설명
|--|--
|DEFAULT|DB 설정, 기본 격리 수준(기본 설정)
|SERIALIZABLE| 가장 높은 격리, 성능 저하의 우려가 있음
|READ_UNCOMMITED|커밋되지 않는 데이터에 대한 읽기를 허용
|READ_COMMITED| 커밋된 데이터에 대해 읽기 허용
|REPEATEABLE_READ|동일 필드에 대해 다중 접근 시 모두 동일한 결과 봐장

- Read-only 속성
    - true인 경우 insert, update, delete 실행 시 예외 발생, 기본 설정은 false

- Rollback-for-에외
    - 즉정 예외가 발생 시 강제로 Rollback

- No_rollback-for-예외
    - 특정 예외의 발생 시에는 Rollback 처리되지 않음

 -  @Transactional 적용 순서
	 - 위와 같은 설정 외에도 '클래스'나 '인터페이스'에 선언하는 것 역시 가능하다.
	 - 어노테이션의 우선순위는 다음과 같다
		- **1순위** : 메서드의 @Transactional
		- **2순위** : 클래스의 @Transactional
		- **3순위** : 인터페이스의 @Transactional
 	- 위의 우선순위를 통해 인터페이스에는 가장 기준이 되는 @Transactional을 설정하고, 클래스나 메서드에 필요한 어노테이션을 처리하는 것이 좋다.

**[⬆ go to top](#Annotation)**

----
## @RequestHeader

 - HttpHeader 정보를 가져온다.

```java
@GetMapping(value = "/download", produces = MediaType.APPLICATION_OCTET_STREAM_VALUE)
@ResponseBody
public ResponseEntity<Resource> downloadFile(@RequestHeader("User-Agent") String userAgent, String fileName){
```

**[⬆ go to top](#Annotation)**

----
## @Scheduled

 - 스케줄러 어노테이션

**[⬆ go to top](#Annotation)**

----
- 출처 : https://engkimbs.tistory.com/694 [새로비]
- 출처 : https://effectivesquid.tistory.com/entry/Bean-%EA%B3%BC-Component%EC%9D%98-%EC%B0%A8%EC%9D%B4