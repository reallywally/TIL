# pyspark 메모리 얼만큼 설정할까

## 어려운 메모리 설정

현제 개발 서버 메모리를 512GB로 아주 넉넉하다.
그래서 pyspark 설정에 메모리를 256GB로 주면 좋을것 같았는데 GPT랑 클로드씨는 64GB를 추천하였다.

## 주요 이유들

1. JVM GC

    JVM은 메모리가 커질수록 GC 시간이 선형 이상으로 늘어난다.
    클로드씨에 의하면 64G는 GC 시간이 1-2초인 반면 256GB는 10-30초로 매우 오래 걸린다.
    그래서 GC pause가 길어지는 문제를 피하기 위해 Spark는 executor 1개에 너무 많은 메모리를 주지 말고, 여러 executor로 나누라고 권장한다.

2. 메모리 분산 처리의 장점

    Spark 자체가 "분산 처리" 구조이므로, executor를 여러 개 띄워 각자 자원을 쓰도록 하는 게 설계 취지에 부합하다.

3. Heap 말고도 OS 레벨 메모리도 필요
    spark.executor.memory는 JVM heap 메모리만 의미하지만 실제롤 heap 이외에도 JVM off-heap, Spark shuffle spill, 등등 다양한 부분에서 메모리가 필요하기 때문에 전체 보다 여유있게 할당

### 참고문서

- 스파크 메모리 관리 관련 공식 문서: <https://spark.apache.org/docs/latest/tuning.html#memory-management-overview>
