* 유튜브: https://www.youtube.com/watch?v=TPACABQTHvM

### 1. Typing variables
* 타입을 지정할 수도 있고, 이미 지정된 타입아닌 다른 타입으로 나중에 저장 불가능
    ```javascript
    let url: string = "https://google.com" // : string은 생략 가능
    url = 100 // 이미 문자열로 저장되서 숫자 입력 불가능
    ```

### 2. Typing Functions
* 파라미터랑 리턴값 지정 가능
    ``` javascript
    function convertformation(score: number, name: string): String{
        // ...
    }

    convertformation(100, "kim")   // ok
    convertformation("100", "kim") // fail
    ```
