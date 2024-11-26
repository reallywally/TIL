# TypeScript in React - COMPLETE Tutorial

* 유튜브: <https://www.youtube.com/watch?v=TPACABQTHvM>

## 내용 정리

### Typing variables

* 타입을 지정할 수도 있고, 이미 지정된 타입아닌 다른 타입으로 나중에 저장 불가능

    ```typescript
    let url: string = "https://google.com" // : string은 생략 가능
    url = 100 // 이미 문자열로 저장되서 숫자 입력 불가능
    ```

### Typing Functions

* 파라미터랑 리턴값 지정 가능

    ```typescript
    function convertformation(score: number, name: string): String{
        // ...
    }

    convertformation(100, "kim")   // ok
    convertformation("100", "kim") // fail
    ```

### Typing React Components

* 함수 처럼 요소를 컴포넌트로 분리하여 재사용 가능

  ```typescript
  export default function MyButton(){
    return(
        <button className="....">
            Click!!
        </button>
    )
  }
  ```

### React.FC

* FunctionComponent 타입의 줄임말
* React + Typescript 조합으로 개발할 때 사용하는 타입
* 그런데 요즘엔 React.FC 사용을 선호하지 않으며 다른 방안을 사용
  * 선호하지 않는 상세 이유는 업데이트 예정

### Typing props

* 컴포넌트 간에 데이터를 전달하기 위해 사용되는 객체
* props가 객체라서 props.backgroundColor 이렇게도 사용할 수 있지만 일반적으로는 구조 분해 할당 많이 사용

    ```typescript
    export default function MyButton({
      backgroundColor,
      fontSize
    }: {
      backgroundColor: string,
      fontSize: number
    }){
      return(
          <button className="....">
              Click!!
          </button>
      )
    }

    export default function home(){
      return (
        <main>
          <MyButton backgroundColr="red" fontSize={12}>
          <MyButton backgroundColr="blue" fontSize={20}>
        </main>
      )
    }
    ```

### extracting type

* 가독성, 유지보수, 재사용 등 장점으로 타입 추출(extracing type)

```typescript
  type ButtonProps = {
    backgroundColor: string,
    fontSize: number
  }

  export default function MyButton({
    backgroundColor,
    fontSize
  }: ButtonProps){
    return(
        <button className="....">
            Click!!
        </button>
    )
  }
```

### Benefits of Typescript

* 타입, properties 체크 가능

### optional

* 상황에 따라 모든 props가 필요하진 않음
* 옵션 입력이 가능한 속성은 ?를 사용

```typescript
  type ButtonProps = {
    backgroundColor: string,
    fontSize?: number
  }
```

### any

* 어떤 타입이든 허용한다는 의미로 사용
* 타입을 명확하게 정하지 못하거나 등 임시적으로 사용하고 가능하면 사용을 피하는게 좋음

```typescript
  type ButtonProps = {
    backgroundColor: string,
    fontSize: any
  }
 ```

### union type

* 여러 갑, 타입 중에 하나를 의미하는 용도로 사용
* 예시는 enum 처럼 쓰는 방법만 나옴
* 자세한 union type 내용은 업데이트 예정

```typescript
  type Color = "red" | "black" | "blue"

  type ButtonProps = {
    backgroundColor: "red" | "black" | "blue",
    borderColor: Color, // 이렇게 타입 추출해서도 가능
    fontSize: number | string  // 타입을 union으로도 가능
  }
```

### Typing arrays

* array로 사용할 속성 타입 뒤에 [] 붙이기

```typescript
  type ButtonProps = {
    backgroundColor: "red" | "black" | "blue",
    fontSize: number | string,
    padding: number[],
  }
```
