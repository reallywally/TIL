# Reranker를 알아보자

## Reranker란

Retriever가 찾아낸 문서들의 순위를 더 정교하게 분석하여 순위를 재조정한다.

## Reranker 사용 이유

Reranker가 더 정교하고 정확하게 찾을 수 있는데 처음부터 Reranker를 쓰면 되지 왜 Retriever까지 써서 2번 조회하나 라는 생각이 든다. 이유는 Reranker는 연산이 복잡하여 계산 비용과 처리 시간이 오래 걸려서 대규모 데이터셋에 적용하기 어렵다. 그래서 연산이 빠른 Retriever로 먼저 필요한 데이터만 가져오고 다시 Reranker로 유사도 분석을 한다.  

## Reranker 종류

Reranker는 사용법이 어렵지 않고 많이 사용하는 모델이 거의 정해져 있어서 이번 글에서는 Reranker의 종류별 특징을 간단하게 살펴본다.

### Cohere Rerank

- 강력한 상용 모델로, API 기반으로 쉽게 통합 가능하며 한국어 등 다국어 지원이 우수함.
- Rerank 3 Nimble 등 속도 최적화 버전도 제공.
- 실시간 서비스와 다국어 서비스에 적합.

### Voyage Rerank

- 최고 정확도를 추구하는 상용 모델로, 다양한 벤치마크에서 우수한 성능을 보임.
- voyage-rerank-2, voyage-rerank-2-lite로 세분화되어 있으며, 정확도와 속도의 균형 모델도 존재.

### BGE-Reranker

- 오픈소스 모델로, 성능이 상용 모델(Cohere, Voyage)에 거의 근접함.
- 비용 부담이 적고 실제 프로젝트에서 널리 활용됨.

### FlashRank

- 실시간성과 빠른 응답 속도가 중요한 경우에 적합한 모델.
- CPU 환경에도 최적화되어 있음.

### ColBERT

- 대용량 문서 처리에 강점이 있으며, 빠른 검색과 확장성이 뛰어남.

### MixedBread (Large)

- 벤치마크 상위권 성능, 긴 문서(8K~32K 토큰) 처리에 강점.
