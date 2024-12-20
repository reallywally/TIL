# API Mocking을 사용해보자

## 왜 쓰는 걸까

프론트 작업을 하는데 서버가 개발이 안된 경우 화면에 표시할 데이터가 없다.  
이때 가짜 데이터가 필요한데 수동으로 관리하면 번거로운 부분이 많아 API Mocking 라이브러리를 이용하자.

## 무엇을 사용할까

여러가지가 있는데 아래 3가지를 많이 쓰는것 같다.

1. Mock Service Worker(MSW)
2. JSON Server
3. Axios Mock Adapter

간단하게 사용법을 리서칭해본 결과 JSON Server가 우리 개발 환경에 잘 맞았다.
일시적, 부분적으로 mocking 사용할때가 많은데 MSW는 전체 설정으로 mocking이 적용되어 내가 원하는 사용법은 아니었다.

## 설치

```shell
    npm install -g json-server
```

## 설정

src/mocks/handlers.ts 생성

``` typescript

```

## 메모

* MSW는 1.x에서 2.x로 바뀌면서 많은 부분이 변경된거 같다. 잘 찾아보고 하자
  * <https://mswjs.io/docs/migrations/1.x-to-2.x/>
