# Vector Store vs Retriever

Langchain을 공부하다보면 검색을 할때 vector store로 검색하거나 retriever를 이용한 검색이있다. 그렇다면 둘 다 검색하는건데 무엇이 다를까?

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

## Vector Store의 역할

- 단순 저장소
- Runable을 상속하지 않음 -> chain이 아님 -> 파이프라인에 직접 연결 불가

## Retriever의 역할

- 검색을 표주화한 인터페이스
- vector store을 retriever로 래핑 -> Runable로 감싸짐 -> chain -> 파이프라인에 연결 가능

## 정리

|구분|vector_store.similarity_search()|vector_store.as_retriever()|
|---|---|---|
|레벨|Low-level|High-level|
|리턴 값|List[Document]|Retriever 객체|
|사용 목적|단순 검색, 테스트|RAG 파이프라인 연결|
|Batch 검색|❌ 불가능|✅ 가능|
|LangChain 체인 연결성|❌ 직접 연결 불가|✅ RetrievalQA 등과 통합 용이|

단순히 벡터 검색만 하고 싶다 -> similarity_search()  
LangChain 체인(RetrievalQA, ConversationalRetrievalChain)에 연결하고 싶다 -> as_retriever()
