# JavaBeans
> JavaBean의 정의
- 데이터를 표현하는 것을 목적으로 하는 자바 클래스
- 컴포넌트와 비슷한 의미로도 사용됨
- JavaBean 규격서에 따라 작성된 자바 클래스를 가리킴
- EJB(Enterprise Java Beans) 와 혼동해선 안됨
----
> JavaBeans의 관례
- 클래스는 패키지화 하여야 한다.
- 클래스는 직렬화 되어야 한다. (클래스의 상태를 지속적으로 저장 혹은 복원시키기 위해)
    ```java
    implements java.io.Serializable
    ```
- 클래스는 기본 생성자를 가지고 있어야한다.
    ```java
    public TestClass() {}
    ```
- 멤버변수는 프로퍼티(Property)라 칭한다.
- 프로퍼티의 접근자는 private이다.
- 프로퍼티마다 getter/setter 가 존재해야 하며, 그 이름은 각각 get/set으로 시작해야 한다.
- boolean 형의 getter 는 is로 시작한다.
- 위의 프로퍼티 getter/setter 메서드의 접근자는 public이어야 한다.
- 외부에서 프로퍼티에 접근은 메서드를 통해서 접근한다.
- 프로퍼티는 반드시 읽기/쓰기가 가능해야 하지만, 읽기 전용인 경우 getter만 정의할 수 있다.
- getter의 경우 파라미터가 존재하지 않아야 하고, setter의 경우 한 개 이상의 파라미터가 존재한다.
- 클래스는 필요한 이벤트 처리 메서드를 포함하고 있어야 한다.
- 영속성이 있어야한다.(?) ⇒ Bean의 현재 상태를 비휘발성의 저장소에 저장하고 나중에 꺼낼 수 있어야 한다.
> 예제1
```java
/***********************************
* * 
* PersonBean.java *
* *
************************************/
public class PersonBean implements java.io.Serializable 
{
    private String name;
    private boolean coding;

    // 기본 생성자 (인자가 없는).
    public PersonBean() 
    {

    }

    public String getName() 
    {
        return this.name;
    }
    public void setName(String name) 
    {
        this.name = name;
    }
    
    // Different semantics for a boolean field (is vs. get)
    
    public boolean isCoding() 
    {
        return this.coding;
    }

    public void setCoding(boolean coding) 
    {
        this.coding = coding;
    }
}
```
>예제2
```java
/***********************************
* * 
* TestPersonBean.java *
* *
************************************/

public class TestPersonBean 
{
    public static void main(String[] args) 
    {

        PersonBean person = new PersonBean();
        person.setName("Bob");
        person.setCoding(true);

        // Output: "Bob [coding]"
        System.out.print(person.getName());
        System.out.println(person.isCoding() ? " [coding]" : "");
    }
}
```
> 예제3
```java
package 패키지_명;

[import 패키지_명;]

public class Bean_ClassName [ implements java.io.Serializable ] {
	private String name;    // 값을 저장하는 속성 정의(필드)
	public Bean_ClassName() { }    // 기본 생성자

	public String getName() {    // 필드 값을 읽어오는 메소드 
		return name; 
	}
	public void setName(String name) {    // 필드 값을 저장하는 메소드
		this.name = name;
	}
}
```