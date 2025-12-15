# Map과 HashMap의 차이점

## 핵심 개념: 인터페이스 vs 구현 클래스

가장 먼저 알아야 할 것은 **Map은 인터페이스**이고, **HashMap은 Map 인터페이스를 구현한 구현체 클래스**라는 점이다.

```java
// Map: 인터페이스 (설계도)
public interface Map<K, V> {
    V put(K key, V value);
    V get(Object key);
    // ... 기타 메서드들
}

// HashMap: 구현 클래스 (실제 동작)
public class HashMap<K, V> implements Map<K, V> {
    // 실제 구현 코드
}
```

## 왜 Map으로 선언할까?

보통 아래와 같이 **인터페이스 타입으로 선언**하는 것이 권장된다.

```java
// 권장하는 방식 (인터페이스로 선언)
Map<String, Integer> userScores = new HashMap<>();

// 피해야 할 방식 (구현 클래스로 선언)
HashMap<String, Integer> userScores = new HashMap<>();
```

### 인터페이스 선언을 선호하는 이유

#### 1. 유연성 확보

나중에 혹시라도 요구사항 변경으로 코드를 수정하더라도 수정할 코드양이 최소화 된다.

```java
// 처음에는 HashMap 사용
Map<String, Integer> cache = new HashMap<>();

// 나중에 순서 유지가 필요하면 쉽게 변경
cache = new LinkedHashMap<>();

// 정렬이 필요하면
cache = new TreeMap<>();
```

#### 2. 의존성 감소

인터페이스에 의존하면 특정 구현체에 종속되지 않아 테스트와 유지보수가 용이하다.

```java
public class UserService {
    // HashMap의 특정 메서드에 의존하지 않음
    private Map<String, User> userCache;

    public UserService(Map<String, User> cache) {
        this.userCache = cache;
    }
}
```

#### 3. 다형성 활용

앞서 언급한 1. 유연성 확보와 비슷한 내용인데 인터페이스를 사용했기에 다양한 구현체를 처리할 수 있다.

```java
public void processData(Map<String, Object> data) {
    // HashMap, TreeMap, LinkedHashMap 모두 받을 수 있음
}
```

## Map 인터페이스의 주요 구현체들

각 구현체는 특성이 다르므로 상황에 맞게 선택하자.

### HashMap

- **특징**: 가장 빠른 성능 (O(1))
- **순서**: 보장하지 않음
- **null**: key와 value 모두 허용
- **용도**: 순서가 중요하지 않고 빠른 조회가 필요할 때

```java
Map<String, Integer> scores = new HashMap<>();
scores.put("Alice", 95);
scores.put("Bob", 87);
// 출력 순서는 입력 순서와 다를 수 있음
```

### LinkedHashMap

- **특징**: 입력 순서 유지
- **성능**: HashMap과 유사하지만 약간 느림
- **용도**: 순서가 중요한 캐시 구현, LRU 캐시 등

```java
Map<String, Integer> orderedScores = new LinkedHashMap<>();
orderedScores.put("Alice", 95);
orderedScores.put("Bob", 87);
// 입력한 순서대로 출력됨
```

### TreeMap

- **특징**: key 기준 자동 정렬
- **성능**: O(log n)
- **null**: key에 null 불가
- **용도**: 정렬된 데이터가 필요할 때

```java
Map<String, Integer> sortedScores = new TreeMap<>();
sortedScores.put("Charlie", 92);
sortedScores.put("Alice", 95);
// 알파벳 순서로 정렬됨: Alice, Charlie
```

### ConcurrentHashMap

- **특징**: 스레드 안전
- **성능**: 멀티스레드 환경에서 우수
- **용도**: 동시성 제어가 필요한 환경

```java
Map<String, Integer> threadSafeMap = new ConcurrentHashMap<>();
```

## 성능 비교표

| 구현체 | 조회 | 삽입 | 삭제 | 순서 유지 | 정렬 | Thread-Safe |
|--------|------|------|------|-----------|------|-------------|
| HashMap | O(1) | O(1) | O(1) | ❌ | ❌ | ❌ |
| LinkedHashMap | O(1) | O(1) | O(1) | ✅ | ❌ | ❌ |
| TreeMap | O(log n) | O(log n) | O(log n) | ✅ | ✅ | ❌ |
| ConcurrentHashMap | O(1) | O(1) | O(1) | ❌ | ❌ | ✅ |

## 기타

만약 면접에서 이 질문을 받았다면, 단순히 "하나는 인터페이스고 하나는 클래스"라고 답하는 것을 넘어서, 왜 인터페이스로 선언하는 것이 좋은지, 각 구현체는 언제 사용하는지까지 설명해보자
