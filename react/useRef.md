# useRef

useEffect는 useState와 마찬가지로 상태를 관리해서 비슷하다. 그래서 익숙하지만 useRef를 어디에 어떻게 써야 할지 감이 잘 안 온다. 이론적으로는 실제 프로젝트에서 꽤 자주 사용하며, 성능 최적화나 DOM 직접 접근 등 다양한 상황에서 유용하게 쓰인다거 하는데 상세 내용을 정리해보자.

## useRef란

앞서 말한것 처럼 useRef는 일반적인 상태 관리 훅인 useState와 마찬가지로 상태를 관리하며 하며 .current 프로퍼티를 통해 값을 읽거나 쓸 수 있다. 하지만 가장 큰 차이점은 useRef로 만든 값은 변경되어도 ***컴포넌트가 리렌더링되지 않는다.***

```typescript
import { useRef } from 'react';

const MyComponent = () => {
  const myRef = useRef(initialValue);
};
```

## useRef 주요 사용 예제

### 1. DOM 요소 직접 참조하기

가장 많이 쓰이는 경우 중 하나로 focus, scroll 등 DOM을 컨트롤해야하는 작업에서 유리하다. 지금 프로잭트에서는 다이어로그를 show,. hide할때도 사용하고 있다.
하지만 useRef로 직접 DOM을 제어하는 방법은 권장되지 않으며 필요할 때만 최소한으로 쓰는게 좋다.
