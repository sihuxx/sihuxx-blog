## 1. Getter
> 인스턴스 변수의 값 반환

**특징**
- 외부에서 객체의 데이터를 읽을 때 사용하는 메소드
- return 값이 필요하다

**사용하는 이유**
1. 데이터의 무결성이 지켜진다
2. 정보 은닉이 가능하다

```
반환형 get이름() {
	return 반환값
}
```

```java
class Person {
	int age;
	public int getAge() {
		return age;
	}
}
```

## 2. Setter
> 인스턴스 변수의 값 설정

- 외부에서 객체의 데이터를 수정할 때 사용하는 메소드
- 매개변수가 필요하다

```
void set이름(전달값) {

}
```

```java
class Person {
	int age;
	public void setAge(int age) {
		this.age = age;
	}
}

```

## 3. Getter & Setter
- 객체의 데이터는 외부에서 직접 접근하지 못하도록 `private` 접근 제어자로 막는다.
```java
public static void main(String[] args) {
	Person.person = new Person();
	person.setAge(20); // 값 설정하기 (Setter)
	System.out.println(person.getAge()); // 값 가져오기 (Getter)
}
```


참고 자료
- [getter, setter란? (+사용하는 이유, 지양하는 이유) - 야옹카옹](https://velog.io/@falling_star3/getter-setter%EB%9E%80%EC%82%AC%EC%9A%A9%ED%95%98%EB%8A%94-%EC%9D%B4%EC%9C%A0-%EC%A7%80%EC%96%91%ED%95%98%EB%8A%94-%EC%9D%B4%EC%9C%A0)
