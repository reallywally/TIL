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

## 의존성 배열

의존성 배열은 크게 3가지 형태로 입력한다.  

1. []와 같은 빈 배열
2. 아무 값도 입력 하지 않음
3. 사용자가 원하는 값

빈 배열은 최초 렌더링 시에만 실행하고, 아무 값도 없으면 렌더링 할때마다 실행, 사용자 값을 입력하면 입력 값이 변경되었을때 실행된다.  
그렇다면 한 가지 의문점이 든다. 최초 렌더링시에만 실행하고 싶다면 아래와 같이 1번 처럼 작성해도 되지 않을까?

```typescript
// 1
function Component(){
  console.log('렌더링!')
}

// 2
function Component(){
  useEffect(() => {
    console.log("렌더링!");
  }, []);
}
```

두개의 차이점은 다음과 같다.

1. useEffect는 클라이언트 사이드에서 실행되는 것을 보장해 준다.
2. 2번은 **렌더링 완료 후 실행**되는 반면, 1번은 렌더링 도중 실행된다. 즉 무거운 작업이면 1번은 렌더링을 방해하여 성능에 영향을 끼친다.

## useEffect를 사용할때 주의할 점

1 eslint-disable-line react-hooks/exhaustive-deps는 사용을을 자제하자

```typescript
const Child1: React.FC<Child1Props> = ({ users }) => {
  useEffect(() => {
    console.log(users);
  }, []);
  ...
```

위 코드와 같이 의존선 배열에는 users가 없는데 useEffect 안에서 user를 사용하는 경우 eslint-disable-line react-hooks/exhaustive-deps 경고가 표시되는데 이 경고를 쉽게 생각하면 안된다.  
useEffect는 반드시 **의존성 배열로 전달한 값의 변경에 의해 실행**돼야하는 훅이다. 즉 위 코드는 관찰하는 값과 실행해야하는 값이 별개로 작동하는것을 의미하기 때문에 useEffect를 사용하는 의미가 없어진다.

2 useEffect의 첫번째 인수에 함수명을 작성하자

코드가 복잡하지 않다면 익명 함수를 사용해도 문제가 없으나 코드가 많아지고 복잡해지면 어떤 일을 하는 useEffect 인지 파악하기 어렵다. 이때 적절한 함수명을 사용하면 어떤 기능을 하는 함수인지 알 수 있어 useEffect의 목적을 파악하기 쉬워진다.

```typescript
  // 함수명 X
  useEffect(() => {
    console.log(userName);
  }, [userName]);

  // 함수명 O
  useEffect(() => {
    function logUserName() {
      console.log(userName);
    }
  }, [userName])
```

3 useEffect는 작게 만들자

하나의 의존성 배열에 많은 값이 들어가면 어떤 기능을 하는지 파악하기도 어렵고 성능에도 이슈가 발생할 수 있다. 그러니 useEffect를 여러개 쓰더라도 작은 단위로 쪼개는게 좋다.

4 불필요한 외부 함수를 만들지 말자

처음 useEffect를 쓸대 아래와 같이 코드를 작성한 적이 있다.

```typescript
const Component: React.FC<Child1Props> = ({ users }) => {
  const fetchUsers = () =>{
    // 사용자 조회 로직
    ... 

  }

  useEffect(() => {
    fetchUsers()
  }, [users]);
  ...
```

fetchUsers()의 코드가 길고 함수명으로 어떤 기능을 하는지 표시하니 가독성이 좋은거 같았다. 하지만 이건 오해였다. useEffect도 변경을 감지하고 특정 기능을 하는 함수와 비슷하다. 그러니 useEffect안에서 원하는 작업을 하는게 훨씬 좋다.
만약 정말로 함수를 쓰고 싶다면 2번에서 말한 방식을 사용하면 된다.

```typescript
const Component: React.FC<Child1Props> = ({ users }) => {

  useEffect(() => {
    // 사용자 조회 로직
    ... 

  }, [users]);
  ...
```
