# Item 29: 딕셔너리, 리스트, 튜플을 깊게 중첩하지 말고 클래스로 구성하라

딕셔너리는 객체의 동적 내부 상태(미리 알 수 없는 식별자 집합)를 관리하기 좋다. 하지만 너무 쉽고 편해서, 자기도 모르게 취약한 코드를 만들게 되는 위험이 있다.

## 1단계: 단순한 시작

이름을 미리 모르는 학생들의 성적을 기록한다고 가정하자. 딕셔너리로 시작하면 단순하다.

```python
class SimpleGradebook:
    def __init__(self):
        self._grades = {}

    def add_student(self, name):
        self._grades[name] = []

    def report_grade(self, name, score):
        self._grades[name].append(score)

    def average_grade(self, name):
        grades = self._grades[name]
        return sum(grades) / len(grades)
```

## 2단계: 과목별로 — 딕셔너리 안에 딕셔너리

"과목별로도 성적을 나누고 싶다"는 요구가 생기면, _grades를 이름 → (과목 → 점수 리스트) 2단계 딕셔너리로 바꾸게 된다.

```python
from collections import defaultdict

class BySubjectGradebook:
    def __init__(self):
        self._grades = {}

    def add_student(self, name):
        self._grades[name] = defaultdict(list)

    def report_grade(self, name, subject, grade):
        by_subject = self._grades[name]
        by_subject[subject].append(grade)

    def average_grade(self, name):
        by_subject = self._grades[name]
        total, count = 0, 0
        for grades in by_subject.values():
            total += sum(grades)
            count += len(grades)
        return total / count
```

아직은 관리할 만해 보인다.

## 3단계: 가중치 추가 — 무너지기 시작

"중간/기말은 쪽지시험보다 비중이 커야 한다"며 점수마다 가중치를 추가해야하는 요구사항이 생겼다. 가장 안쪽 값을 (점수, 가중치) 튜플로 바꾼다. report_grade는 간단히 바뀌지만, average_grade는 이중 루프가 되어 읽기 어려워진다.

```python
class WeightedGradebook:
    def __init__(self):
        self._grades = {}

    def add_student(self, name):
        self._grades[name] = defaultdict(list)

    def report_grade(self, name, subject, score, weight):
        by_subject = self._grades[name]
        by_subject[subject].append((score, weight))

    def average_grade(self, name):
        by_subject = self._grades[name]
        score_sum, score_count = 0, 0
        for scores in by_subject.values():       # 바깥 루프
            subject_avg, total_weight = 0, 0
            for score, weight in scores:         # 안쪽 루프
                subject_avg += score * weight
                total_weight += weight
            score_sum += subject_avg / total_weight
            score_count += 1
        return score_sum / score_count
```

쓰는 쪽도 끔찍해진다. 위치 인자 숫자들이 무슨 의미인지 알 수 없다.

```python
book.report_grade("Albert Einstein", "Math", 75, 0.05)   # 75? 0.05? 뭐가 뭐지?
```

이 정도 복잡함이 보이면, 내장 타입에서 클래스 계층으로 도약할 때이다. 핵심 원칙은 중첩은 한 단계까지만. 딕셔너리 안의 딕셔너리는 유지보수 악몽으로 가는 길이다.

## 클래스로 리팩터링하기

의존 트리의 바닥(점수 하나)부터 리팩토링해보자. 먼저 튜플로 시작할 수 있지만, 튜플은 위치 기반이라 정보가 늘어날 때마다(예: 교사 코멘트 추가) 모든 사용처를 고쳐야 하는 문제가 있다. 가벼운 불변 데이터 컨테이너엔 dataclasses가 딱이다.

```python
from dataclasses import dataclass

@dataclass(frozen=True)
class Grade:
    score: int
    weight: float
```

다음으로 한 과목을 표현하는 클래이다.

```python
class Subject:
    def __init__(self):
        self._grades = []

    def report_grade(self, score, weight):
        self._grades.append(Grade(score, weight))

    def average_grade(self):
        total, total_weight = 0, 0
        for grade in self._grades:
            total += grade.score * grade.weight
            total_weight += grade.weight
        return total / total_weight
```

그다음 한 학생(과목들의 모음)

```python
class Student:
    def __init__(self):
        self._subjects = defaultdict(Subject)

    def get_subject(self, name):
        return self._subjects[name]

    def average_grade(self):
        total, count = 0, 0
        for subject in self._subjects.values():
            total += subject.average_grade()
            count += 1
        return total / count
```

마지막으로 전체 학생 컨테이너

```python
class Gradebook:
    def __init__(self):
        self._students = defaultdict(Student)

    def get_student(self, name):
        return self._students[name]
```

코드 줄 수는 거의 두 배가 됐지만, 훨씬 읽기 쉽고 사용 코드도 명확하며 확장하기 좋다.

```python
book = Gradebook()
albert = book.get_student("Albert Einstein")
math = albert.get_subject("Math")
math.report_grade(75, 0.05)   # 이제 의미가 메서드 호출 흐름으로 드러남
math.report_grade(65, 0.15)
print(albert.average_grade())
```

## 한줄 요약

내부 상태 딕셔너리가 복잡해지면 코드를 여러 클래스로 분리하자
