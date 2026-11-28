#typescript 

*any*: 모르니깐 신경 끄자
*unknown*: 모르니깐 조심하자
> 어떤 타입의 데이터가 담길지 모른다는 의미의 자료형

### 01 any
변수에 어떤 값을 넣든 컴파일 오류가 나지 않음
타입스크립트 사용 의미를 상실함으로서 지양해야 할 코드

```typescript
let any: any = 10;        // Number
any = "Hello";            // String
any = true;               // Boolean
any = [1, 2, 3];          // Boolean
any = { name: "john" };   // Boolean
```

## 01-1 any 사용의 위험성
오류가 컴파일 (코드 작성 시점) 단계에서 걸러지지 않고 런타임 단계에서야 오류가 발생함

```typescript
let anyString: any = 123;
// 타입이 any기 때문에 어떤 값이든 들어감
// => anyString 이라는 변수명의 의도와 달리 숫자가 값으로 할당되어도 컴파일에서 걸러지지 않음

console.log(anyString.toUpperCase()); // toUpperCase()는 String의 메소드지만
//  변수 타입이 any기 때문에 컴파일러가 오류를 발생시키지 않음
console.log(anyString.noneMethod()); // noneMethod()는 존재하는 메소드가
// 아니지만 변수 타입이 any기 때문에 어떤 타입인지 알 수 없으므로 컴파일 오류 생성 X
```

## 01-2 any 사용 예시
- 외부의 라이브러리
- 네트워크에서 받아오는 데이터
> 다루어야 할 데이터의 타입을 미리 알 수 없는 경우

### 02 unknown
타입을 모르는 값에 대한 모든 걸 금지시킴

```typescript
let anyVar: any = 10;
let unKnownVar: unknown = 10;

let anyNumber: number = anyVar;
// any 변수의 값은 숫자 자료형으로 지정된 다른 변수에 할당하는 것이 가능함
// => '무슨 값이 들었는지 모르지만 개발자가 알아서 잘 넣엇겟지' 마인드
anyVar.toFixed(2);
// 또한 어떤 메소드든 호출 가능

let unknownNumber: number = unKnownVar;
// unknown 변수의 값은 숫자 자료형으로 지정된 다른 변수에 할당하는 것이 불가능함
// => 무슨 값이 담겼을지 모르기 때문에 값을 넘겨주지 못하게 함
unKnownVar.toFixed(2);
// 또한 어떤 메소드도 호출하지 못하도록 막음
```

### 02-1 타입 가드
만약 이 타입의 값이 ~이라면.. 과 같은 조건들을 두고 unknown 타입의 값을
조건식들로 할당시킴

`if문`: 변수의 타입이 문자열임을 확신할 수 없을 때 사용 (일반적으로 바람직한 방법 )
`as`: 변수의 타입이 문자열임을 확신할 수 있을 때 사용

```typescript

function processValue(val: unknown): string {
    if (typeof val === 'string') { // val의 타입이 문자열임을 전제함으로
        return val.toUpperCase(); // 문자열 메소드인 toUpperCase()의 호출이 허용됨
    }

    if (typeof val === 'number') { // val의 타입이 숫자열임을 전제함으로
        return val.toFixed(2); // 숫자열 메소드인 toFixed()의 호출이 허용됨
    }

    return String(val); // 문자열도 숫자도 아닐 경우엔 String() 함수에 인자로 전달한 결과를 내보냄
}

console.log(processValue('hello')); // HELLO
console.log(processValue(42)); // 42.00
console.log(processValue(true)); // "true"

let unknownValue: unknown = "Hello, TypeScript!";

// 변수의 타입이 문자열임을 확신할 수 있을 때
let stringLength = (unknownValue as string).length
// unknownValue의 타입이 문자열이라고 보장한다는 의미
// => 타입스크립트는 이를 신뢰하고 컴파일 단계에서 오류를 발생시키지 않음

// 변수의 타입이 문자열임을 확신할 수 없을 때
if (typeof unknownValue === 'string') {
    let length = unknownValue.length
}
// => 일반적으론 이 방법이 더 바람직한 방법 
```

### 02-2 Unknown, any 활용 객체 예제

```typescript
function processUserData(user: unknown): string {
    if (typeof user === 'object' && user !== null) { // user 객체의 타입이 object고, 값이 null이 아니먄
        if ('name' in user && typeof (user as any).name === 'string') { // user 객체 안에 'name'이라는 속성이 존재하고, 그 속성의 값이 문자열일 때
            return (user as any).name.toUpperCase(); // 그 'name' 속성의 값을 대문자로 변환하여 반환
        }
    }
    return 'Invalid user data'; // 만약 위 조건식을 만족하지 않으면 문자열 반환
}
```