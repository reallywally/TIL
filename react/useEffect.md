# useEffect

useState만큼 자주 쓰이는 훅이다. 그리고 자주 쓰지만 생각보다 사용하기 쉬운 훅이 아니다.

## useEffect 기본 형태

useEffect는 다음과 같이 첫 번째 인수는 콜백, 두 번째 인수는 의존성을 배열을 받는 형태를 가진다.
그리고 두번째 인수로 들어가는 의존성 배열의 값이 변경될때 마다 첫 번째 인수로 들어간 콜백 함수가 실행된다.

```typescript
function Component(){
  ...

  useEffect(() =>{
    console.log("useEffect!")     
  }, [count]) 
  ...
}
```

그런데 useEffect는 아땋게 의존성 배열의 값이 변경되는 것을 알고 실행할까?
useEffect는 자바스크립트의 proxy나 데이터 바인딩, 옵저버 등 특별한 기능으로 값의 변화를 관찰하지 아니라 **렌더링할때** 의존성 배열에 있는 값이 변경됬는지 확인하고 변경된게 있으면 부수 효과(콜백)을 실행하는 평범한 함수다.

## 클린업

useEffect 내부에서 반환 되는 함수는 클린업 함수라한다. 

```typescript
function Counter(){
  const [count, setCount] = useState(0)

  useEffect(() => {
    console.log("useEffect!");
    console.log(count);

    return () => {
      console.log("Cleanup!");
      console.log(count);
    };
  }, [count]);
}
```

