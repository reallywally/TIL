# useCallback

useCallback은 리액트 16.8에서 도입된 훅(Hook)으로, 메모이제이션된 콜백 함수를 반환한다. 간단히 말해, 동일한 함수를 리렌더링될 때마다 재생성하지 않고 재사용할 수 있게 해주는 도구이다.

## 기본 문법

```typescript
const memoizedCallback = useCallback(
  () => {
    doSomething(a, b);
  },
  [a, b],
);
```

기본 문법은 위와 같으며 두 가지 인자를 받는다.  

1. 메모이제이션할 함수 () => { doSomething(a, b);}
2. 의존성 배열 [a, b]

코드를 해석하면 의존성 배열 a와 b가 변경되지 않는 한, 리렌더링이 발생해도 함수를 새로 생성하지 않고 이전에 생성된 함수를 재사용한다.

## 상세 예시

아래 코드를 돌려보자

```javascript
import React, { useState, useCallback } from 'react';

function Counter() {
  const [count, setCount] = useState(0);
  
  // count가 변경될 때만 함수가 재생성됨
  const incrementCount = useCallback(() => {
    setCount(prevCount => prevCount + 1);
  }, []);
  
  // 의존성 배열이 빈 배열이므로 컴포넌트 최초 렌더링 시 한 번만 생성됨
  const resetCount = useCallback(() => {
    setCount(0);
  }, []);
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={incrementCount}>증가</button>
      <button onClick={resetCount}>리셋</button>
    </div>
  );
}
```
