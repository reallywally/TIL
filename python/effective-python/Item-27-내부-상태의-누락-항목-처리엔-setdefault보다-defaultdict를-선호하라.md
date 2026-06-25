# Item 27 내부 상태의 누락 항목 처리엔 setdefault보다 defaultdict를 선호하라

핵심 갈림길은 **"그 딕셔너리를 내가 만드는가, 아니면 남이 만든 걸 받는가"**이다.

## 남이 만든 딕셔너리를 받을 때

내가 생성을 통제하지 않는 딕셔너리라면, Item 26에서 봤듯 get이 보통 낫다. 다만 어떤 경우엔 setdefault가 가장 짧기도 하다. 방문한 나라별 도시를 셋(set)으로 모으는 예를 보면 다음과 같다.

```python
visits = {
    "Mexico": {"Tulum", "Puerto Vallarta"},
    "Japan": {"Hakone"},
}

# 짧음
visits.setdefault("France", set()).add("Arles")

# 길음 (get + 대입 표현식)
if (japan := visits.get("Japan")) is None:
    visits["Japan"] = japan = set()
japan.add("Kyoto")
```

나라가 있든 없든 도시를 추가할 수 있고, get 버전보다 확실히 짧다.

## 내가 딕셔너리 생성을 통제할 때 (내부 상태)

객체의 내부 상태를 딕셔너리로 관리하는 경우가 여기 해당합니다. 위 로직을 클래스로 감싸 보면 다음과 같다.

```python
class Visits:
    def __init__(self):
        self.data = {}

    def add(self, country, city):
        city_set = self.data.setdefault(country, set())
        city_set.add(city)
```

setdefault 호출의 복잡함을 감추고 깔끔한 인터페이스를 제공한다.

```python
visits = Visits()
visits.add("Russia", "Yekaterinburg")
visits.add("Tanzania", "Zanzibar")
```

하지만 이 add 구현은 이상적이지 않다. 두 가지 문제가 있다.

1. setdefault라는 이름이 여전히 혼란스러워 코드를 처음 보는 사람이 의도를 파악하기 어렵다.
2. 호출할 때마다 새 set을 만든다. 나라가 이미 있든 없든 매번 셋을 할당하니 비효율적이다.(add를 많이 호출하면 비용이 커짐)

## 해결책: defaultdict

collections.defaultdict는 키가 없을 때 기본값을 자동으로 만들어 저장해준다. 키가 없을 때마다 호출해 기본값을 만들 함수 하나만 넘기면 된다. 예시에서는 set을 사용한다.

```python
from collections import defaultdict

class Visits:
    def __init__(self):
        self.data = defaultdict(set)

    def add(self, country, city):
        self.data[country].add(city)   # 키가 없으면 자동으로 빈 set 생성
pythonvisits = Visits()
visits.add("England", "Bath")
visits.add("England", "London")
print(visits.data)
# defaultdict(<class 'set'>, {'England': {'Bath', 'London'}})
```

이제 add가 짧고 단순하다. 어떤 키에 접근하든 항상 set이 존재한다고 가정할 수 있고, 불필요한 set 할당도 없다.

## 한줄 요약

임의의 키 집합을 관리하는 딕셔너리를 직접 만든다면 defaultdict를 선호하라
