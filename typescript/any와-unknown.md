# any와 unknown

## any란?

any 타입은 단어 처럼 **어느 타입이든** 입력이 가능한 타입이다.

```typescript
let anyVar: any = "10";
// 어느 타입이든 할당 가능
anyVar = "hello";
anyVar = true;
anyVar = {};
```

그리고 어느 타입의 변수이건 any 타입을 할당할 수 있다.

```typescript
let anyVar: any = "10";
// num 타입 변수에 any 할당도 가능
let num: number = 10
num = anyVar
```

## any의 문제점

아래 2개 얘시 코드는 정상적으로 동작 할까?

```typescript
// 예시1
let anyVar: any = "10";
anyVar = "hello";
anyVar = {};
anyVar.toUpperCase();

// 예시2
let num: number = 10
num = anyVar
anyVar.toFixed();
```

결론은 두 예시 모드 정상 작동을 하지 않는다. 예시1은 마지막 값으로 {}가 들어가서 toUpperCase()가 실행될 수 없고, 예시2는 숫자로 타입을 지정했지만 마지막에 문자값이 들어가서 toFixed()를 실행할 수 없었다.  
예시 처럼 any는 **타입 검사를 하지 않는다**. 그래서 예시와 같이 어느 순간의 코드로 예상치 못한 버그를 발생시킬 가능성이 매우 높으며, 타입스크립트의 장점을 모두 포기하는것과 마찬가지이다.  
그래서 정말 어쩔 수 없는 경우를 제외하고는 any 타입을 사용하지 않는것을 강력히 권장한다.

## 정말 어쩔수 없는 경우가 언제인데?

명확한 케이스를 찾기는 어려웠지만 배민의 우아한 타입스크립트 책을 기준으로 보면 다음과 같은 상황을 예시로 들었으니 참고해보자

1. 개발 단계에서 임시로 값을 지정할때
2. 어떤 값을 받을지 정할 수 없을때
3. 값을 예측할 수 없을때

## unknown?

unknown 타입은 any와 마찬가지로 모든 값을 넣을 수 있다.

```typescript
let unknownVar: unknown = "10";
unknownVar = "hello";
unknownVar = true
unknownVar = {};
```

## 그럼 any와 다른점은?

any와 비교했을때 두 가지 차이점이 있는데 첫 번째는 unknown 타입의 값은 어떤 타입의 변수에도 저장할 없다.

```typescript
let unknownVar: unknown = "10";
let num: number = 10

num = unknownVar // 에러!
```

그리고 두 번째는 타입을 보장하는 연산을 해야한다. 보통은 아래 코드와 같이 typeof로 타입을 체크하고 실행되도록한다.

```typescript
let unknownVar: unknown = 10

if (typeof unknownVar === "number") {
  console.log(unknownVar * 2);
}
```

## 한줄요약

만약에 정말로 어쩔수없이 any를 사용해야하는 경우가 아니라면 unknown을 사용하자.
