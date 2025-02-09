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
이유는 **useState가 비동기적으로 동작하는 훅이다**. 