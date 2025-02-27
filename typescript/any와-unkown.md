# any와 unknown

## any란?

any 타입은 단어 처럼 **어느 타입이든** 입력이 가능한 타입이다.
그리고 어느 타입의 변수이건 any 타입을 할당할 수 있다.

```typescript

let anyVar: any = 10;
// 어느 타입이든 할당 가능
anyVar = "hello";
anyVar = true;
anyVar = {};

// num 타입 변수에 any 할당도 가능
let num: number = 10
num = anyVar
```

## any의 문제점

아래 코드는 정상적으로 동작 할까?

```typescript
let anyVar: any = 10;
anyVar = "hello";

anyVar = true;
anyVar = {};

anyVar.toUpperCase();
anyVar.toFixed();

```

앞서 예시 처럼 any는 **타입 검사를 하지 않는다**. 그래서 예상치 못한 버그를 발생시킬 가능성이 매우 높으며, 타입스크립트의 장점을 모두 포기하는 형
아래와 같이 만약 any 타입 변수에 숫자를 넣고 문자