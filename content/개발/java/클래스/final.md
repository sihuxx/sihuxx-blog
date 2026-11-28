> 변경할 수 없게 만드는 키워드

## 1. final 변수
- 한 번 초기화하면 재할당 불가 (값 읽기는 가능)
- 초기화 시점: 선언 시 또는 생성자에서 한 번
- **기본 자료형**: 저장된 값 자체 변경 불가
- **참조 자료형**: 다른 객체로 교체는 불가, 객체 내부 값은 변경 가능
- **상수**: `static final`로 선언

```java
final int age = 18;
age = 20;            // 에러: 재할당 불가

final int[] arr = {1, 2, 3};
arr[0] = 99;         // OK: 내부 값 변경
arr = new int[3];    // 에러: 다른 배열로 교체 불가

static final int MAX_SCORE = 100;  // 상수
```

## 2. final 메서드
- 자식 클래스에서 오버라이딩 불가
```java
class Person {
    public final void introduce() {
        System.out.println("사람입니다");
    }
}
class Student extends Person {
    public void introduce() { }  // 컴파일 에러: final 메서드는 오버라이딩 불가
}
```

## 3. final 클래스
- 다른 클래스가 상속 불가
- 대표 예시: `String`, `Math`

```java
final class Animal { }
class Dog extends Animal { }  // 컴파일 에러: final 클래스는 상속 불가
```
