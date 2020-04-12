<details markdown="1">
<summary style="font-size:20px;"> @ModelAttribute </summary>

# @ModelAttribute

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

출처: https://engkimbs.tistory.com/694 [새로비]
</details>
<details markdown="1">
<summary style="font-size:20px;"> @RedirectAttributes </summary>

# @RedirectAttributes
- Model 타입과 더불어 스프링 MVC가 자동으로 전달해 주는 타입 중에는 RedirectAttributes타입이 존재한다.
- 일회성으로 데이터를 전달하는 용도로 사용
- 기존의 Servlet에서 response.sendRedirect()를 사용할 떄와 동일한 용도로 사용된다.
</details>

<details markdown="1">
<summary style="font-size:20px;"> @Autowired </summary>

# @Autowired
</details>

<details markdown="1">
<summary style="font-size:20px;"> @Component </summary>

# @Component
</details>

<details markdown="1">
<summary style="font-size:20px;"> @RequestParam </summary>

# @RequestParam
 - 스프링은 파라미터의 타입을 보고 객체를 생성하므로 파라미터의 타입은 List<>와같은 **인터페이스** 타입이아닌 실제적인 **클래스** 타입으로 지정한다 ex) ArrayList<> 타입
</details>
<details markdown="1">
<summary style="font-size:20px;"> @InitBinder </summary>

# @InitBinder
 - DTO에서와의 같이 파라미터값의 바인딩이 자동으로 되는 경우가 있지만. 그렇지 않은 경우 역시 존재한다. 예로 '2018-01-01'과 같은 날짜 데이터를 java.util.Date 타입으로 변환하는 작업이 그러하다.
    - 스프링 Controller에서는 파라미터를 바인딩할 때 자동으로 호출되는 @InitBinder를 이용해서 이러한 변환을 처리할 수 있다.
 ```java
 @InitBinder
	public void initBinder(WebDataBinder binder) {
		
		SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd");
		binder.registerCustomEditor(java.util.Date.class, new CustomDateEditor(dateFormat, false));
		
	}
```
</details>

<details markdown="1">
<summary style="font-size:20px;"> @DateTimeFormat </summary>

# @DateTimeFormat
 - @InitBinder 와 동일한 작업(날짜 데이터를 java.util.Date 타입으로 변환)
 - @DatetimeFormat 사용 시 @initBinder는 제거 해야한다.(충돌 에러 발생)
```java
@DateTimeFormat(pattern = "yyyy/MM/dd")
	private Date dueDate;
```
</details>
<details markdown="1">
<summary style="font-size:20px;"> @ExceptionHandler </summary>

# @ExceptionHandler
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
</details>
<details markdown="1">
<summary style="font-size:20px;"> @ControllerAdvice </summary>

# @ControllerAdvice
 - 해당 객체가 스프링의 컨트롤러에서 발생하는 예외를 처리하는 존재임을 명시하는 용도
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
</details>
<details markdown="1">
<summary style="font-size:20px;"> @ResponseEntity </summary>

# @ResponseEntity
</details>
<details markdown="1">
<summary style="font-size:20px;"> @ExceptionHandler </summary>

</details>
<details markdown="1">
<summary style="font-size:20px;"> @ResponseStatus </summary>

</details>
<details markdown="1">
<summary style="font-size:20px;"> @WebAppConfiguration </summary>

</details>
<details markdown="1">
<summary style="font-size:20px;"> @ComponentScan </summary>

</details>
<details markdown="1">
<summary style="font-size:20px;"> @MapperScan </summary>

</details>
<details markdown="1">
<summary style="font-size:20px;"> @Transactional </summary>

</details>
<details markdown="1">
<summary style="font-size:20px;"> @Bean </summary>

</details>
<details markdown="1">
<summary style="font-size:20px;"> @param </summary>

</details>
<details markdown="1">
<summary style="font-size:20px;"> @ </summary>

</details>
<details markdown="1">
<summary style="font-size:20px;"> @ </summary>

</details>
