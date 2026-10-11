## 1. this
> 자기 자신 (인스턴스 / 지역변수 구분)

- 형태: `this.인스턴스변수;`
```java
class Person {
	String name;
	public void setName(String name) {
		this.name = name;
		// this.name = 클래스 내의 name 변수 (인스턴스 변수)
		// name = 매개변수로 받은 전달값
	}
}

public static void main(String[] args) {
	Person person = new Person();
	person.setName("시후");
	System.out.println(person.name); // 시후
}
```

## 2. this, this()

**this**
- 객체 자신을 가리키는 레퍼런스 변수로, **자신의 객체에 접근할 때 사용**
- 주[[Java] 자바 - this와 this( )의 용도 및 사용예제](https://kadosholy.tistory.com/92)로 멤버변수와 매개변수의 이름이 동일할 때, 이를 구분하기 위해 사용

**this()** ^9ed6be
- 같은 클래스에서 **생성자가 다른 생성자를 호출할 때 사용**
- 주로 코드의 중복을 줄일 목적으로 사용
- 생성자 코드에서만 사용할 수 있음
- 생성자 코드 안에서 사용될 때 **첫번째 문장으로 다른 코드보다 가장 윗줄에 위치**해야 함

**예제 코드**
```java
public class Book {
    private String title;
    private int price;
 
    public Book(String title, int price) {
        this.title = title;
        this.price = price;
    }
 
    public Book(String title) {
        this(title, 0);
    }
}
```

참고 자료
- [자바 - this와 this()의 용도 및 사용예제](https://kadosholy.tistory.com/92)
- [this와 this()에 대해](https://rlaehddnd0422.tistory.com/181)