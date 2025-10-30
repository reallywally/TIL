# Vector Store vs Retriever

Langchain을 공부하다보면 검색을 할때 vector store로 검색하거나 retriever를 이용한 검색이있다. 그렇다면 둘 다 똑같이 검색하는건데 차이점은 무엇일까?

```python
# vector store를 이용한 검색
results = vector_store.similarity_search(
    "How many distribution centers does Nike have in the US?"
)

# retriever를 이용한 검색
retriever = vector_store.as_retriever(
    search_type="similarity",
    search_kwargs={"k": 1},
)

retriever.batch([
    "How many distribution centers does Nike have in the US?",
])
```

## Runable을 먼저 이해하자

상세한 차이점을 보기 전에 Runable이라는 개념을 간략하게 이해하자. 쉽게 정리하면 Runnable은 LangChain에서 실행 가능하고 호출 가능한 표준 인터페이스이다. 그래서 Runable이 아니면 LangChain 파이프라인에 연결될 수 없고 Runnable이면 LangChain 파이프라인에 연결되어 LangChain의 관리를 받을 수 있다.

## Vector Store의 역할

Vetctor Store는 단순 저장소이다. 그래서 Runaable을 상속하지 않았으며 chain이 아니다. 즉 파이프라인에 직접 연결할 수 가 없다.

## Retriever의 역할

Retriever는 검색을 표준화한 인터페이스이다. 구조는 Vector Store를 Retriever로 래핑하여 Runable이 되고 파이프라인에 연결이 가능해 진다.

## 정리

앞서 설명한 내용을 테이블로 정리하면 다음과 같다.

|구분|vector_store.similarity_search()|vector_store.as_retriever()|
|---|---|---|
|레벨|Low-level|High-level|
|리턴 값|List[Document]|Retriever 객체|
|사용 목적|단순 검색, 테스트|RAG 파이프라인 연결|
|Batch 검색|❌ 불가능|✅ 가능|
|LangChain 체인 연결성|❌ 직접 연결 불가|✅ RetrievalQA 등과 통합 용이|

Vector Store 조회와 Retriever 조회는 모두 조회 기능을 제공하기 때문에 상황에 맞게 사용하는게 중요하다. 단순히 벡터 검색만 하고 싶다면 similarity_search(), LangChain 체인(RetrievalQA, ConversationalRetrievalChain)에 연결하고 싶다면 as_retriever()를 이용하자.
