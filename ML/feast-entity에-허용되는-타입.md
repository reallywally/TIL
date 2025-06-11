# feast-entity에-허용되는-타입

엔티티를 등록하는 와중에 ```ValueError: Value type not supported for feast feature store: double_val: 3```이라는 에러가 발생하였고 문제를 해결하는 과정을 정리해 본다.

## 원인

결론적으로 제목 그대로 feast entity에는 허용되는 타입이 정해져 있는데 그에 맞지 않는 타입을 사용했기 때문이다. 우선 feast entity에서 허용 하는 타입을 파이썬 타입과 비교해서 정리하면 다음과 같다.

| Feast 타입 | Python 타입 예시 |
| -------- | ------------ |
| INT64    | int          |
| STRING   | str          |
| BYTES    | bytes        |

이처럼 feast entity에는 float 타입을 사용할 수 없는데 마침 내가 사용한 타입이 double(실수형)이라서 에러가 발생하였다.

## 해결 방법

실수형을 entity에서 지원하지 않으니 타입을 **실수형대신 int나 string**으로 변경하면 된다.

## 그럼 왜 실수형을 entity에 사용할 수 없을까

실수형은 부동소수점으로 오차가 발생할 수 있어 정확한 비교가 어렵고, 직렬화할때도 오차/불일치 발생 가능성 있다. 정리하면 데이터 정합성과 기술적 이슈 때문에 지원하지 않는다.
