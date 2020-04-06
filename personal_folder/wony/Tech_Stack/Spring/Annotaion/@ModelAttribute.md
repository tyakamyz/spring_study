# ModelAttribute

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

출처: https://engkimbs.tistory.com/694 [새로비]