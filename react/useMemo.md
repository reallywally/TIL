# useMemo

useMemo는 비용이 큰 연산에 대한 결과를 저장해두는 훅으로 렌더링 중 계산된 값을 캐싱하여 불필요한 재계산 방지한다. 리액트에서 최적화를 떠올릴때 가장 먼저 언급되는 훅이다.

## 기본 사용법

useMemo의 기본 문법은 다음과 같다.

```typescript
const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);
```

useMemo는 두 가지 인자를 받는다.

1. 계산을 수행하는 함수 => computeExpensiveValue(a, b)
2. 의존성 배열 => [a, b]

그리고 의존성 배열이 변경될 때만 함수가 재실행된다. 상세 예시는 아래 코드를 돌려보자.

```javascript
import React, { useState, useMemo } from 'react';

function ExpensiveCalculation({ number }) {
  // 복잡한 계산을 시뮬레이션
  const calculateFactorial = (num) => {
    console.log('계산 중...');
    if (num <= 1) return 1;
    
    let result = 1;
    for (let i = 2; i <= num; i++) {
      result *= i;
    }
    return result;
  };

  // useMemo를 사용하여 number가 변경될 때만 계산 실행
  const factorial = useMemo(() => calculateFactorial(number), [number]);

  return (
    <div>
      {number}의 팩토리얼: {factorial}
    </div>
  );
}

function App() {
  const [number, setNumber] = useState(5);
  const [count, setCount] = useState(0);

  return (
    <div>
      <input 
        type="number" 
        value={number} 
        onChange={(e) => setNumber(parseInt(e.target.value))} 
      />
      <ExpensiveCalculation number={number} />
      
      <button onClick={() => setCount(count + 1)}>
        카운트 증가 (현재: {count})
      </button>
    </div>
  );
}
```

## 언제 useMemo를 사용해야 할까?

가장 복잡한 수학적 계산, 대량의 데이터 필터링 또는 변환, 불필요한 렌더링을 방지할때가 대표적인 경우다. 데이터 필터링을 예제로 코드를 확인해 보자. 아래 예제에서 darkMode 상태가 변경되어도 상품 필터링은 다시 실행되지 않습니다.

```javascript
import React, { useState, useMemo } from 'react';

function ProductList({ products, category }) {
  // 카테고리에 따라 상품 필터링
  const filteredProducts = useMemo(() => {
    console.log('상품 필터링 중...');
    return products.filter(product => product.category === category);
  }, [products, category]);

  return (
    <div>
      <h2>{category} 상품 목록</h2>
      <ul>
        {filteredProducts.map(product => (
          <li key={product.id}>{product.name} - ₩{product.price}</li>
        ))}
      </ul>
    </div>
  );
}

function App() {
  const [products] = useState([
    { id: 1, name: '노트북', price: 1200000, category: '전자제품' },
    { id: 2, name: '스마트폰', price: 800000, category: '전자제품' },
    { id: 3, name: '청바지', price: 50000, category: '의류' },
    { id: 4, name: '티셔츠', price: 30000, category: '의류' },
    // ... 더 많은 상품
  ]);
  
  const [category, setCategory] = useState('전자제품');
  const [darkMode, setDarkMode] = useState(false);

  return (
    <div className={darkMode ? 'dark-mode' : ''}>
      <button onClick={() => setDarkMode(!darkMode)}>
        {darkMode ? '라이트 모드' : '다크 모드'}
      </button>
      
      <select value={category} onChange={(e) => setCategory(e.target.value)}>
        <option value="전자제품">전자제품</option>
        <option value="의류">의류</option>
      </select>
      
      <ProductList products={products} category={category} />
    </div>
  );
}
```

## useMemo를 많이 사용하면 성능이 좋아질까?

useMemo를 사용할 때는 실제로 성능 향상이 있는지 측정하는 것이 중요다. 모든 계산을 메모이제이션하는 것은 오히려 성능을 저하시킬 수 있으니 과유불급을 잊지 말자.
