# 우아한 타입스크립트 with 리액트

* 책: <https://www.yes24.com/Product/Goods/123049083>

## 내용 정리

### 1장

#### 1.1 웹 개발의 역사

* 생략

#### 1.2 자바스크립틔의 한계

* 동적 타입 언어
* 변수 age가 받는 타입이 number인지 string인지는 동작할때 35인지 "35"인지에 따라 달라짐
* 그러다 보니 에러가 발생해야하는 상황에서도 일단 동작하는 경우가 생김
* 이런 한계 극복하기 위해 JSDoc, propTypes, 다트 같은 해결 방안이 나왔지만 문제를 해결하지 못함
* 그래서 MS가 자바스크립트의 슈퍼셋으로 타입스크립트를 공개

### 2장 타입

#### 2.1 타입이란

* 여러 종류의 데이터를 식별하는 분류 체계
* 정적 타입은 컴파일 시점에 타입이 결정되어 안정적
* 강타입은 다른 타입끼리 연상을 시도하면 에러가 발생하고 약타입은 암묵적 타입 변환으로 연산 수행

#### 2.2 타입스크립트의 타입 시스템

* 타입 애너테이션: 변수, 상수 혹은 함수의 인자와 반환 값에 타입을 명시하는 문법
* 구조적 타이핑: int age과 같이 이름으로 타입을 구분하는게 아니고 구조로 타입을 구분

    ```typescript
    interface Developer {
        value: number
    }

    let developer: Developer = { value: 35 };
    ```

* 구조적 서브타이핑: 속성을 바탕으로 타입을 구분

    ```typescript
    type stringOrNumber = string |number
    ```

* 타입스크립트가 구조적 타이핑을 채택한 이유는 자바스크립트를 모델링 하였기 때문이고, 자바스크립트는 덕 타이핑을 기반으로 한다.
  * 덕 타이핑: 어떤 타입에 부합하는 변수와 메서드를 가질 경우 해당 타입으로 간주하는 방식

#### 2.3 원시 타입

* boolean, undefined, null, number, bitint, string, symbol,

#### 2.4 객체 타입

* object, {}, array, type, interface, function
* object는 any와 유사하게 모든 타입 값을 유동적으로 할당할 수 있어 사용하지 않도록 권장
* type과 interface를 어떻게 구분해서 사용할까?
  * 배달이팀: 대부분 interface를 쓰고 간단한건 type
  * 냥이팀: 정적인 내용은 type, 확장될수 있으면 interface
  * 왕팀: 주로 type
  * 공식 문서: 전역적으로 사용하면 interface, 작은 범위 내에서 사용하면 type.
    * 이 내용은 책에서 언급한 내용이고 실제 공식 문서에서 찾아봤는데 완벽한 하게 동일한 내용은 못찾음
    * 링크: [Type aliases and interfaces are very similar, and in many cases you can choose between them freely](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#differences-between-type-aliases-and-interfaces)
  * 내 생각:
    * 한 회사에서도 팀바팀으로 사용 기준이 다르기 때문에 우리도 정하기 나름인거 같음
    * 공식문서도 자유롭게 선택하라는것으로 보면 너무 강압적으로 정할 필요는 없는듯

### 3장 고급 타입

#### 3.1 타입스크립트만의 독자적 타입 시스템

* any:
  * 자바스크립트의 모든 값을 오류 없이 받을 수 있음.
  * 타입이 중요한 타입스크립트에서는 any 사용을 지양해다 좋은 패턴
  * 하지만 어쩔 수 없이 사용해야하는 상황에는 사용
    * 개발 단계에서 임시로 값을 지정할때
    * 어떤 값을 받을지 정할 수 없을때
    * 값을 예측할 수 없을때
* unkowwn:
  * any와 유사하게 모든 타입에 할당 가능
  * 차이점은 unknown 타입은 any 타입 이외에 다른 타입으로 할당 불가

    ``` typescript
    let unknownValue: unknown;
    unknownValue = 111; // (O)
    unknownValue = "일일일"; // (O)
    
    let anyValue: any = unknownValue;  // (O)
    let numberValue: number = unknownValue;  //(X)
    let stringValue: string = unknownValue;  //(X)
    ```

  * any 타입과 비슷한데 추가된 이유는 뭘까?
    * any 타입은 임시로적을 사용하다가 특정 타입으로 수정해야 하는 것을 누락할 수 있지만 unknown은 실행할때 에러가 발생하여 any 타입 보다 안전하다.

    ```typescript
    // 할당은 에러 않남
    const unknownFunction: unknown = () => console.log("!!!");

    // 실행은 에러 발생
    unknownFunction()
    ```

  * any타입 사용을 자제하는건 좋으나 아예 사용안하는것 또한 어렵다.

* void 타입
