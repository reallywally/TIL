# useState

리액트 훅에서 가장 먼저 떠올리는것은 아마 useState일것이다. useState는 함수 컴포넌트 내부에 상태를 관리하는 훅이다.

## useState를 안쓴다면?

아래 코드를 실제로 돌려보면 count는 로그에는 증가된 값으로 찍히지만 화면에는 증가되지 않는것 처럼 보인다.

```typescript
const UseStateTest = () => {
  let count = 0;

  function increment() {
    count++;
    console.log(count);
  }

  return (
    <div>
      <h1>Count Component</h1>
      {count}
      <button onClick={increment}>Increment</button>
    </div>
  );
};
```

이유는 리렌더링 조건에 만족하지 않기 때문이다. 그렇다면 렌더링 조건만 충족하면 count는 올바르게 표시가 될까?

```typescript
import { useState } from "react";

const UseStateTest = () => {
  const [, triggerRender] = useState();
  let count = 0;

  function increment() {
    count++;
    triggerRender();
    console.log(count);
  }

  return (
    <div>
      <h1>Count Component</h1>
      {count}
      <button onClick={increment}>Increment</button>
    </div>
  );
};

export default UseStateTest;
```

안타깝게도 여전히 count는 증가하지 않았다. 이유는 렌더링을 하더라도 ```let count = 0;```때문에 count는 다시 0으로 초기화가 되기 때문이다.

## useState를 사용해보자

그래서 의도한 대로 동작할려면 상태를 기억하도록 useState를 이용해서 count의 상태를 관리하면 된다.

```typescript
import { useState } from "react";

const UseStateTest = () => {
  const [count, setCount] = useState(0);

  function increment() {
    setCount((count) => count + 1);
    console.log(count);
  }

  return (
    <div>
      <h1>Count Component</h1>
      {count}
      <button onClick={increment}>Increment</button>
    </div>
  );
};

export default UseStateTest;
```

정상적으로 보이는거 같다. 그런데 로그를 보면 찍히는 값이 상태 증가되기 이전값으로 찍힌다.
이유는 **useState가 비동기적으로 동작하는 훅이기 때문이다.**
즉, 상태 업데이트가 즉시 반영되는 것이 아니라, 다음 렌더링 시점에서 새로운 값이 적용된다.

## 왜 비동기적으로 작동하지? 해결 방법은?

만약 즉각적인 상태 변경이 발생하면 렌더링이 그만큼 많이 발생하고, 성능 이슈가 발생한다. 그래서 성능을 최적화하기 위해 React가 여러 개의 상태 업데이트를 하나로 배치(batch)로 처리한다.
만약 동기적인 방법으로 할려면 크게 두가지 방법이다. 첫번째는 useEffect를 사용하면 되는데 이건 useEffect.md 파일을 참조하자. 두번째는 set 함수의 콜백으로 처리하는거다. 위에 increment 함수는 다음과 같이 수정할 수도 있다.

```typescript
function increment() {
  setCount((prev) => {
    const newCount = prev + 1;

    if (newCount > 5) {
      alert("count is greater than 5");
    }

    return newCount;
  });
}
```

둘중에 어떤 방법이 더 좋다고 말하기 어려우니 취향과 상황에 맞게 편한걸로 사용하면 될것 같다.

## 게으른 초기화(lazy initialization)

보통 useState로 상태 초기화를 할때 특정 값이나 객체를 넣을것이다. 그런데 함수는 넣는 경우도 있고 이를 게으른 초기화라고 한다.

```typescript
// 바로 값 넣기
const [count, setCount] = useState(
  window.localStorage.getItem("count")
);

// 게으른 초기화
const [count, setCount] = useState(() =>
  window.localStorage.getItem("count")
);
```

게으른 초기화는 오직 state가 처음 만들어질때만 사용되며 이후 리렌더링될때는 함수 실행을 무시한다.
오래된 리액트 공식 문서(<https://legacy.reactjs.org/docs/hooks-reference.html#lazy-initial-state>)에서는 localStorage, sessionStorage, map, filter, find 등 state의 초기값이 무거운 연산을 할때 사용하기를 권장한다.
