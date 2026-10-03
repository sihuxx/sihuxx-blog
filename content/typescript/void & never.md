**void**: 만나도 줄 선물이 없는 산타
**never**: 절대 만날 수 없는 산타
함수나 메소드의 반환 값에 사용되며 '반환되는 값을 받을 수 없음'을 의미

## 01 void
해당 함수에 'return'문이 없거나, 'return' 뒤에 아무 값이 없을 때 사용됨

### 01-1 void 사용 예시 (1)
> 반환값을 사용하지 않는 함수
```typescript
function printLength(text: string | null): void { // 함수가 반환하는 값의 타입 지정
    if (text === null) { // 위에서 매개변수가 null인 경우가 처리되었음으로
        console.log('No text provided');
        return;
    }
    
    console.log(`Text length: ${text.length}`);
    // 아래선 매개변수가 무조건 문자열임을 타입스크립트가 알 수 있어서 오류가 나지 않음
    // 위의 if문을 없애면 text의 타입이 null인지 string인지 알 수 없기 때문에 컴파일 오류가 발생함
}

printLength(null);
printLength('Hello, world!');
// 위에서 함수의 매개변수에 string | null 이라는 유니온 타입을 설정했기에 두 타입 모두 매개변수로 설정 가능
```


### 01-2 void 사용 예시 (2)
> 반환값을 갖는 함수

void를 반환값 타입으로 갖는 함수는 자바스크립트에서와 마찬가지로 undefined를 반환함
단 그 값을 쓰지 않을 거라는 의미를 void로 나타냄
```typescript
function logMessage(message: string): void {
    console.log(message);
    return undefined; // 'undefinded'를 반환하면 의미 없는 일이지만 컴파일 오류를 발생시키진 않음
    return null; // 'null'을 반환하면 오류가 뜸
    // undefined와 null은 엄연히 다른 타입
}
```

### 01-3 void 사용 예시 (3) 
> 콜백 함수
```typescript
const numbers = [1, 2, 3, 4, 5];

numbers.forEach((num: number): void => {
    console.log(num * 2); // 무언가 값을 반환하지 않고 출력만함
    // => 반환값의 타입을 void로 설정
})
```

## 02 never
함수가 절대 값을 반환하지 않는 경우에 사용

### 02-1 never 사용 예시 (1)
> 특정 오류를 던지는 용도의 함수
```typescript
function thorwError(message: string): never {
    throw new Error(message);
    // 실행부에서 오류를 던짐
    // => 정상 종료되지 않음, 값을 반환하는 과정에 도달하지 못함 >> never 사용
}
```

### 02-2 never 사용 예시 (2)
> 무한 루프를 도는 함수
```typescript
let i = 0;

function infiniteLoop(): never {
    while(true) {
        i++; // 무한 루프 도는 함수
        // => 한번 실행 시 정상적으로 종료되지 않음 >> never 사용
    }
}
```

### 02-3 never 사용 예시 (3)
> 유니온 타입을 사용하는 함수

*매개변수가 문자열도, 숫자도, 불리언도 아닌 케이스는 발생하지 않기 때문에 타입 never로 지정*

x의 유니온 타입 목록에 object가 추가 되면, 위의 주석처리한 if문 처럼 
그 object에 대한 타입 가드를 설정하지 않았을 경우엔 오류가 발생함

**사용하는 이유**
유니온 타입에 object를 추가했을 때 else문으로 boolean 처리를 해버렸다고 가정했을 때,
else문에서 불리언과 객체가 구분되지 않았음을 모르고 지나칠 수 있는 위험성 존재
```typescript
function handleValue(x: string | number | boolean | object): void { 
    // 매개변수로 문자열, 숫자, 불리언 중 하나의 값을 받을 수 있음
    if (typeof x === "string") { // 문자열 타입 가드
        console.log("문자열입니다:", x.toUpperCase());
    } else if (typeof x === "number") { // 숫자 타입 가드
        console.log("숫자입니다:", x.toFixed(2));
    } else if (typeof x === "boolean") { // 불리언 타입 가드
        console.log("불리언입니다", x ? "true" : "false");
    } /* else if (typeof x === "object") {
        console.log("객체입니다", x.toString());
    }  */
    else { // 매개변수의 타입이 위의 조건 중 아무것에도 해당되지 않을 때
        const unreachable: never = x;
        throw new Error(`그 외 타입: ${x}`);
    }
}

handleValue({ name: 'john' });
```

## 03 내로잉(Narrowing)
**조건문으로 타입을 거르고 걸러서, 가장 확실한 하나의 타입만 남기는 과정**
유니온 타입처럼 여러 타입이 올 수 있는 변수의 타입을 특정 코드 블록 안에서 더 구체적인 하나의 타입으로 좁혀주는 기능

```typescript
function printLength(str: string | null) { // 이 시점의 str은 string | null
    if (str === null) { // null 검증
        return; 
    }

    // 여기부터 str을 오직 'string'으로만 인식함 (null이 제거됨)
    console.log(str.length); 
}
```