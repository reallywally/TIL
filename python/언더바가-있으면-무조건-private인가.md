# 언더바가 있으면 무조건 private 인가

## _asdict()?

GPT씨랑 협업하던 도중 페이징한 결과를 딕셔너리로 받기 위해 ```dict_results = [row._asdict() for row in pagination.items]``` 이런 코드를 생성해주었다. 그런데 파이썬은 언더바(_)가 있으면 private으로 간주하고 코딩하는데 잘못해준게 아닌가 하고 리서칭하게 되었다.

## 결론은

```_asdict()```는 언더바가 붙었지만 **공식적으로 문서화된 메서드**이며, 내부 구현에 종속된 비공개 API가 아니다.

## 그럼 왜 언더바(_)가 붙었을까?

asdict, replace, make 등과 같은 이름이 필드명으로 사용될 수 있기 때문에, 메서드 이름이 필드명과 충돌하지 않도록 하기 위한 선택이다.

## 참고 링크

- 공식 문서 링크: <https://docs.python.org/3/library/collections.html#collections.somenamedtuple._asdict>
- 네임드 튜플 설명1: <https://miguendes.me/everything-you-need-to-know-about-pythons-namedtuples>
- 네임드 튜플 설명2: <https://towardsdatascience.com/python-tuple-named-tuples-edaee4d1b191/>
