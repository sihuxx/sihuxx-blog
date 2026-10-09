#typescript 

= 값이 없음
**null**: 값이 의도적으로 비어있음, 데이터가 명시적으로 비어있는 상태
**undefined**: 변수가 선언되었지만 값이 아직 할당되지 않았거나 정의되지 않은 상태

```typescript
let nullValue: null = null; 
let undefinedValue: undefined = undefined;
// null, undefined 타입에는 각각 null, undefined만 들어올 수 있기 때문에
// 실무에서 이런 변수는 만들지 않는다.

let stringValue: string = null; // => 오류
let numberValue: number = undefined; // => 오류
// null이나 undefined는 다른 타입의 값에 할당하지 못한다.
```

### 01 유니온 타입
자바스크립트의 OR 연산자`(||)` 와 같이 `'A'이거나 'B'이다`라는 의미의 타입이다.
>변수명: 타입 1 | 타입 2

변수의 값으로 타입 1이나 타입 2의 값이 들어올 수 있음을 의미
=> 값이 있을 수도, 없을 수도 있는 데이터는 이처럼 유니온 데이터를 사용하여
null 또는 undefined를 타입에 포함시킬 수 있음