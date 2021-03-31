# 5. 클래스와 인터페이스

파이썬은 상속, 다형성, 캡슐화등과 같은 모든 기능을 제공한다.

## 37. 내장 타입을 여러 단계로 내포시키기보다는 클래스를 합성하라

파이썬의 내장 딕셔너리 타입을 사용하면 객체의 생명 주기 동안 동적인 내부 상태를 유지할 수 있다. **동적** 은 어떤 값이 들어올지 미리 알 수 없는 식별자들을 유지해야 한다는 뜻이다.

```python
class SimpleGradeBook:
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

이 클래스는 쉽게 사용할 수 있다. 그러나 너무 쉬우므로 과하게 확장하면서 깨지기 쉬운 코드를 작성할 위험성이 있다. 만약 이 클래스를 확장해서 과목별 성적을 리스트로 저장한다고 한다면

```python
class BySubjectGradeBook:
    def __init__(self):
        self._grades = {}

    def add_student(self, name):
        self._grades[name] = defaultdict(list)

    def report_grade(self, name, subject, score):
        by_subject = self._grades[name]
        grade_list =  by_subject[subject]
        grade_list.append(score)

    def average_grade(self, name, subject):
        by_subject = self._grades[name]
        total = count = 0
        for grades in by_subject.values():
            total += sum(grades)
            coutn += len(grades)
        return total / count
```

여전해 쉽게 쓸 수 있다. 그러나 각 점수의 가중치를 함께 저장해서 중간고사와 기말고사가 다른 쪽지 시험보다 성적에 더 큰 영향을 주고 싶다면,
과목(키)을 성적 리스트(값)로 매핑하던 것을 (성적, 가중치) 튜플의 리스트로 매핑한다.

```python
class WeightedGradeBook:
    def __init__(self):
        self._grades = {}

    def add_student(self, name):
        self._grades[name] = defaultdict(list)

    def report_grade(self, name, subject, score, weight):
        by_subject = self._grades[name]
        grade_list =  by_subject[subject]
        grade_list.append((score, weight))

    def average_grade(self, name, subject):
        by_subject = self._grades[name]
        score_sum = score_count = 0
        for subject, score in by_subject.items():
            subject_avg = total_weight = 0
            for score , weight in subject:
                subject_avg += score * weight
                total_weight += weight
            score_sum += subject_avg / total_weight
            score_count += 1
        return score_sum / score_count
```

클래스도 쓰기 어려지고 읽기가 어려워진다. 이렇게 복잡도가 심해진다면 더 이상 내장 타입을 사용하지 말고 클래스 계층 구조를 사용해야 한다.

### 클래스를 활용해 리팩터링하기

의존 관계 트리의 맨 밑바닥(즉, 내포된 자료구조의 맨 안쪽)을 표현하는 클래스로 옮길 수 있으나 너무 많은 비용이 든다. 점수는 불편 값이기 때문에 적당하다.

```python
grades = []
grades.append((95. 0.45))
grades.append((85. 0.55))
total = sum(score * weight for score, weight in grades)
total_weight = sum(weight for _, weight in grades)
```

`total_weigh`t를 계산할 때는 \*를 사용해 각 점수 튜플의 첫 번쨰 원소를 무시한다. 이 코드의 문제점은 튜플에 저정된 내부 원소에 위치를 사용해 접근한다는 것이다. `namedtuple`을 사용하면 작은 불편 데이터 클래스를 쉽게 정의할 수 있다.

```python
from collections import namedtuple

Grade = namedtuple('Grade', ('score', 'weight'))

```

> namedtuple 한계
>
> - namedtuple 클래스에는 디폴트 인자 값을 지정할 수 없다. 따라서 선택적인 프로퍼티가 많은 데이터에 namedtuple을 사용하기는 어렵다.
> - namedtuple 인스턴스의 애트리뷰트 값을 숫자 인덱스를 사용해 접근할 수 있다. 따라서 나중에 namedtuple을 실제 클래스로 변경하기 어려울 수 있다. 모든 부분을 제어할 수 있는 상황이 아니라면 명시적으로 새로운 클래스를 정의하는 편이 더 낫다.

```python
class Subject:
    def __init__(self):
        self._grades = []

    def report_grade(self, score, weight):
        self._grades.append(Grade(score, weight))

    def average_grade(self):
        total = total_weight = 0
        for grade in self._grades: # grade: namedtuple
            total += grade.score * grade.weight
            total_weight += grade.weight
        return total / total_weight

class Student:
    def __init__(self):
        self._subjects = defaultdict(Subject)

    def get_subject(self, name):
        return self._subjects[name]

    def average_grade(self):
        total = count = 0
        for subject in self._subjects.values():
            total += subject.average_grade()
            count += 1
        return total / count

class GradeBook:
    def __init__(self):
        self._students = defaultdict(Student)

    def get_student(self, name):
        return self._students[name]
```

코드를 길어졌지만 더 읽기 쉽고 확장성이 좋아졌다. 또한, 하위 호환성을 제공하는 메서드를 제공해서 예전 스타일의 API를 사용 중인 코드를 새로운 객체 계층을 사용하는 코드로 쉽게 마이그레이션할 수 있다.

### 정리

- 내장 타입이 복잡하게 내포된 데이터 값으로 사용하는 딕셔너리를 만들지 않는다.
- 완전한 클래스가 제공하는 유연성이 필요하지 않고 가벼운 불변 데이터 컨테이너가 필요하다면 `namedtuple`을 사용한다.
- 내부 상태를 표현하는 딕셔너리가 복잡해지면 여러 클래스를 나눠서 작성한다.

## 38. 간단한 인터페이스의 경우 클래스 대신 함수를 받아라

내부 API중 상당수는 함수를 전달해서 동작을 원하는 대로 바꿀 수 있게 해준다. 이런 함수를 **훅** 이라고 부른다.

```python
names = ['소크라테스', '아르키메데스', '플라톤', '아리스토텔레스']
names.sort(key=len) # key: Hook
```

파이썬은 **일급 시민 객체** 로 취급하기 떄문에 함수를 훅으로 사용할 수 있다.

> 일급 시민 객체란?
>
> > 언어 안에서 아무런 제약 없이 사용할 수 있는 데이터 값을 뜻한다. 일반적으로 함수에 인자로 넘길 수 있고, 변수나 데이터 구조에 저장할 수 있으며, 함수에서 반환할 수 있고, 동등성을 검사할 수 있는 값을 '일급 시민 값'이라고 한다.

defaultdict에는 딕셔너리 안에 없는 키에 접근할 경우 호출되는 인자가 없는 함수를 전달할 수 있다.

```python
def log_missing():
    print('키 추가')
    return 0

current = {'초록': 12, '파랑': 3}
increments = [
    ('빨강', 5),
    ('파랑', 17),
    ('주황', 9),
]
result = defaultdict(log_missing, current)
print('이전:', dict(result))
for key, amount in increments:
    result[key] += amount
print('이후:', dict(result))
"""
결과:
이전: {'초록': 12, '파랑': 3}
키 추가
키 추가
이후: {'초록': 12, '파랑': 20, '빨강': 5, '주황': 9}
"""
```

디폴트 값 훅이 존재하지 않는 키에 접근한 총 횟수를 세고 싶다면, 클로저 개념을 사용한다.

```python
def increment_with_report(current, increments):
    added_count = 0
    def missing():
        nonlocal added_count
        added_count += 1
        return 0

    result = defaultdict(missing, current)

    for key, amount in increments:
        result[key] += amount
    return added_count
```

하지만 상태를 다루기 위한 훅으로 클로저를 사용하면 상태가 없는 함수에 비해 읽고 이해하기가 어렵다. 다른 방법은 추적하고 싶은 상태를 저장하는 작은 클래스를 정의하는 것이다.

```python
class CountMissing:
    def __init__(self):
        self.added = 0

    def missing(self):
        self.added +=1
        return 0

counter = CountMissing()
result = defaultdict(counter.missing, current)
for key, amount in increments:
    result[key] += amount
```

클래스를 놓고 보면 `CountMissing` 클래스의 목적이 분명히 알기 어렵다.
파이썬에서는 클래스에 `__call__` 특별 메서드를 정의할 수 있다. `__call__` 사용하면 객체를 함수처럼 호출할 수 있다. 그리고, `__call__` 정의된 클래스의 인스턴스에 대해 `callable` 내장함수를 호출하면, 다른 일반 함수나 메서드와 마찬가지로 `True`가 반환된다. 이런 방식으로 정의돼서 호출될 수 있는 모든 객체를 **호출 가능 객체** 라고 부른다.

```python
class BetterCountMissing:
    def __init__(self):
        self.added = 0
    def __call__(self):
        self.added += 1
        return 0
counter = BetterCountMissing()
result = defaultdict(counter, current) # __call__에 의존
```

### 정리

- 파이썬의 여러 컴포넌트 사이에 간단한 인터페이스가 필요할 때는 클래스를 정의하고 인스턴스화하는 대신 간단한 함수를 사용할 수 있다.
- 파이썬의 함수나 메서드는 일급 시민이다. 따라서 함수나 함수 참조를 식에 사용할 수 있다.
- `__call__`특별 메서드를 사용하면 클래스의 인스턴스 객체인 일반 파이썬 함수 처럼 호출이 가능하다.
- 상태를 유지하기 위한 함수가 필요한 경우에는 상태가 있는 클로저를 정의하는 대신 `__call__` 메서드가 있는 클래스를 정의할지 고려한다.

## 39. 객체를 제내릭하게 구성하려면 @classmethod를 통한 다형성을 활용하라

파이썬은 객체 뿐아니라 클래스도 다형성을 지원한다. 다형성을 사용하면 계층을 이루는 여러 클래스가 자신에 맞는 유일한 메서드 버전을 구현할 수 있다. 인터페이스를 만족하거나 같은 추상 기반 클래스를 공유하는 많은 클래스가 서로 다른 기능을 제공할 수 있다는 뜻이다.

예들 들어 맵리듀스 구현을 작성하고 있는데, 입력 데이터를 표현할 수 있는 공통 클래스가 필요하다면

```python
class InputData:
    def read(self):
        raise NotImplementedError

class PathInputData(InputData):
    def __init__(self, path):
        super().__init__()
        self.path = path

    def read(self):
        with open(self.path) as f:
            return f.read()
```

입력 데이터를 소비하는 공통 방법을 제공하는 맵리듀스 작업자로 쓸 수 있는 추상 인터페이스를 정의한다면

```python
class Worker:
    def __init__(self, input_data):
        self.input_data = input_data
        self.result = None

    def map(self):
        raise NotImplementedError

    def reduce(self, other):
        raise NotImplementedError

class LineCountWorker(Worker):
    def map(self):
        data = self.input_data.read()
        self.result = data.count('\n')

    def reduce(self, other):
        self.result += other.result

```

각 부분을 어떻게 연결해야 할까? 객체를 생성해 활용해야만 이 모든 클래스가 쓸모 있게 된다. 각 객체를 만들고 맵리듀스를 조화롭게 실행하는 책임은 누가 져야 할까?
도우미 함수를 활용해 객체를 직접 만들고 연결한다.

```python
import os

def generate_inputs(data_dir):
    for name in os.listdir(data_dir):
        yield PathInputData(os.path.join(data_dir, name))

# 다음으로 방급 generate_inputs를 통해 만든 InputData 인스턴스들을 사용하는 LineCountWorker 인스턴스를 만든다.

def create_workers(input_list):
    workers = []
    for input_data in input_list:
        workers.append(LineCountWorker(input_data))
    return workers

# 이 Worker 인스턴스의 map 단계를 여러 스레드에 공급해서 실행할 수 있다.
from threading import Thread

def execute(workers):
    threads = [Thread(target=w.map) for w in workers]
    for thread in threads:
        thread.start()
    for thread in threads:
        thread.join()
    first, *rest = workers
    for worker in rest:
        first.reduce(worker)
    return first.result

## 마지막으로 지금까지 만든 모든 조각을 한 함수 안에 합쳐서 각 단계를 실행한다.

def mapreduce(data_dir):
    inputs = generate_inputs(data_dir)
    workers = create_workers(inputs)
    return execute(workers)
```

잘 작동하나 문제가 있다. 앞에서 정의한 `mapreduce` 함수의 가장 큰 문제점은 함수가 전혀 제네릭하지 않다는 것이다. 다른 `InputData`나 `Worker` 하위 클래스를 사용하고 싶다면 각 하위 클래스에 맞게 도우미 함수들을 재작성해야 한다.
이 문제를 해결하는 가장 좋은 방법은 **클래스 메서드** 다형성을 사용하는 것이다. 이 방식은 `InputData.read`에서 사용했던 인스턴스 메서드의 다형성과 같은데, 개별 객체에 적용되지 않고 클래스 전체에 적용이 된다.

```python
class GenericInputData:
    def read(self):
        raise NotImplementedError

    @classmethod
    def generate_inputs(cls, config):
        raise NotImplementedError


class PathInputData(GenericInputData):
    def __init__(self, path):
        super().__init__()
        self.path = path

    def read(self):
        with open(self.path) as f:
            return f.read()

    @classmethod
    def generate_inputs(cls, config):
        data_dir = config['data_dir']
        for name in os.listdir(data_dir):
            yield cls(os.path,join(data_join, name))

class GenericWorker:
    def __init__(self, input_data):
        self.input_data = input_data
        self.result = None

    def map(self):
        raise NotImplementedError

    def reduce(self, other):
        raise NotImplementedError

    @classmethod
    def create_workers(cls, input_class, config)
        workers = []
        for input_data in input_class.generate_inputs(config):
            workers.append(cls(input_data))
        return workers

class LineCountWorker(GenericWorker):
    def map(self):
        data = self.input_data.read()
        self.result = data.count('\n')

    def reduce(self, other):
        self.result += other.result

def mapreduce(worker_class, input_class, config):
    workers = worker_class.create_workers(input_class, config)
    return execute(workers)
```

이제는 각 하위 클래스의 인스턴스 객체를 결합하는 코드를 변경하지 않아도 된다.

### 정리

- 파이썬 클래스에는 생성자가 `__init__`메서드 뿐이다.
- `@classmethod`를 사용하면 클래스에 다른 생성자를 정의할 수 있다.
- 클래스 메서드 다형성을 활용하면 여러 구체적인 하위 클래스의 객체를 만들고 연결하는 제네릭한 방법을 제공한다.

## 40. super로 부모 클래스를 초기화하라

자식 클래스에서 부모 클래스의 `__init__`메서드를 직접 호출할 수 있지만, 잘못될 수도 있다. 어떤 클래스가 다중 상속에 의해 영향을 받은 경우, 상위 클래스의 `__init__`메서드를 직접 호출하면 프로그램이 예측할 수 없는 방식으로 작동할 수 있다.

그리고 다중 상속의 경우 모든 하위 클래스에서 `__init__`호출의 순서가 정해져 있지 않다는 것이다. 부모 클래스의 호출 순서에 따라 결과 값이 달라질 수도 있다.

다이아몬드 상속으로 인해 다른 문제가 발생할 수도 있다.

> 다이아몬드 상속이란
>
> > 어떤 클래스가 두 가지 서로 다른 클래스를 상속하는데, 두 상위 클래스의 상속 계층을 올라가다 보면 같은 조상 클래스가 존재하는 경우를 말한다.

다이아몬드 상속이 이루어지면 공통 조상 클래스의 `__init__`메서드가 여러 번 호출될 수 있기 때문에 문제가 발생할 수 있다.

`super`라는 내장 함수와 표준 메서드 결성 순서(MRO)가 있다. `super`을 사용하면 다이아몬드 계층의 공통 상위 클래스를 단 한번만 호출하도록 보장한다. MRO는 상위 클래스를 초기화하는 순서를 정의한다. 각 초기화 메서드는 각 클래스의 `__init__`이 호출된 순서의 역순으로 작업하게 된다.

`super`함수에 두 가지 파라미터를 넘길 수 있다.
첫 번째 파라미터는 접근하고 싶은 MRO 뷰를 제공할 부모 타입이고,
두 번째 파라미터는 첫 번째 파라미터로 지정한 타입의 MRO 뷰에 접근할 떄 사용할 인스턴스이다.

클래스 정의 안에서 아무 인자도 지정하지 않고 `super`을 호출하면 올바른 파라미터(`__class__`와 `self`)를 넣어준다.

### 정리

- 파이썬은 표준 메서드 결정 순서를 활용해 상위 클래스 초기화 순서와 다이아몬드 상속 문제를 해결한다.
- 부모 클래스를 초기화할 때는 super 내장 함수를 아무 인자 없이 호출한다. 아무 인자 없이 호출하면 올바른 파라미터를 넣어준다.

## 41. 기능을 합설할 떄는 믹스인 클래스를 사용하라.

믹스인 클래스는 메서드 몇 개만 정의하는 클래스다. 자체 애트리뷰트 정의가 없으므로 `__init__`메서드를 호출할 필요도 앖다.

믹스인의 가장 큰 장점은 제너릭 기능을 쉽게 연결할 수 있고 필요할 때 기존 기능을 다른 기능으로 오버라이드해 변경할 수 있다는 것이다.

```python
class ToDictMixin:
    def to_dict(self):
        return self._traverse_dict(self.__dict__)

    def _traverse_dict(self, instance_dict):
        output = {}
        for key, value in instance_dict.items():
            output[key] = self._traverse(key, value)
        return output

    def _traverse(self, key, value):
        if isinstance(value, ToDictMixin):
            return value.to_dict(key, value)
        elif isinstance(value, dict):
            return self._traverse_dict(value)
        elif isinstance(value, list):
            return [self._traverse(key, i) for i in value]
        elif hasattr(value, '__dict__'):
            return self._traverse_dict(value.__dict__)
        else:
            return value
```

믹스인의 가장 큰 장점은 제네릭 기능을 쉽게 연결할 수 있고 필요할 때 기존 기능을 다른 기능으로 오버라이딩해 변경할 수 있다.

믹스인을 서로 합성할 수도 있다.

```python
import json

class JsonMixin:
    @classmethod
    def from_json(cls, data):
        kwargs = json.loads(data):
        return cls(**kwargs)

    def to_json(self):
        return json.dumps(self.to_dict())
"""
JsonMixin 클래스 안에 인스턴스 메서드와 클래스 메서드가 함께 정의됐다. 믹스인을 사용하면 인스턴스의 동작이나 클래스의 동작 중 어느 것이든 하위 클래스에 추가할 수 있다. JsonMixin 하위클래스의 요구 사항은 to_dict 메서드를 제공해야하고 __init__ 메서드가 키워드 인자를 받아야 한다는 점이다.
"""
```

### 정리

- 믹스인을 사용해 구현할 수 있는 기능을 인스턴스 애트리뷰트와 `__init__`을 사용하는 다중 상속을 통해 구현하지 않는다.
- 믹스인 클래스가 클래스별로 특화된 기능을 필요로 한다면 인스턴스 수준에서 끼워 넣을 수 있는 기능을 활용한다.
- 믹스인에는 필요에 따라 인스턴스 메서드는 물론 클래스 메서드도 포함될 수 있다.
- 믹스인을 합성하면 단순한 동작으로부터 더 복잡한 기능을 만들 수 있다.

## 42. 비공개 애트리뷰트보다는 공개 애트리뷰트를 사용하라

파이썬에서 클래스의 애트리뷰트에 대한 가시성은 공개와 비공개밖에 없다.

```python
class MyObject:
    def __init__(self):
        self.public_field = 5
        self.__private_field = 10

    def get_private_field(self):
        return self.__private_field

```

클래스 외부에서 비공개 필드에 접근하면 예외가 발생한다.

클래스 메서드는 자신을 둘러싸고 있는 `class` 블록 내부에 들어 있기 때문에 해당 클래스의 비공개 필드에 접근할 수 있다.

```python
class MyOtherObject:
    def __init__(self):
        self.__private_field = 71

    @classmethod
    def get_private_field_of_instance(cls, instance):
        return instance.__private_field
```

하위 클래스는 부모 클래스의 비공개 필드에 접근할 수 없다.

내부에 몰래 접근함으로써 생길 수 있는 피해를 줄이고자 파이썬 프로그래머는 스타일 가이드를 따른다. 필드 앞에 밑줄 하나만 있으면 `protected`를 뜻한다. 보호 필드는 클래스 외부에서 이 필드를 사용하는 경우 조심해야 한다는 뜻이다.

하지만 파이썬을 처음 사용하는 많은 프로그래머가 하위 클래스나 외부에서 사용하면 안되는 내부 API를 표현하기 위해 비공개 필드를 사용한다.
누군가는 이 클래스르 상속하면서 새로운 기능을 추가하거나, 기존 메서드의 단점을 해결하기 위해 새로운 동작을 추가하길 원할 수 있는데 비공개 애트리뷰트를 사용하면 이런 확장이나 오버라이드를 귀찮게 하고 깨지기 쉽게 만들 뿐이다.

일반적으로 상속을 허용하는 클래스 쪽에서 보호 애트리뷰트를 사용하고 오류는 내는 것이 좋다.

```python
class MyClass:
    def __init__(self, value):
        # 객체에게 사용자가 제공한 값을 저장한다.
        # 사용자가 제공하는 값은 문자열로 타입 변환이 가능해야하고
        # 일단 한번 객체 내부에 설정되고 나면
        # 불변 값으로 취급돼야 한다.
```

### 정리

- 파이썬은 비공개 애트리뷰트를 자식 클래스나 클래스 외부에서 사용하지 못하도록 엄격히 금지하지 않는다.
- 내부 API에 있는 클래스의 하위 클래스를 정의하는 사람들이 부모 클래스의 애트리뷰트를 사용하지 못하도록 막기보다는 애트리뷰트를 사용해 더 많은 일을 할 수 있게 허용하는 것이 좋다.
- 비공개 애트리뷰트로 접근을 막으려고 시도하기보다는 보호된 필드를 사용하면서 문서에 적절한 가이드를 넣는다.
- 코드 작성을 제어할 수 없는 하위 클래스에서 이름 충돌이 일어나는 경우를 막고 싶을 떄만 비공개 애트리뷰트를 사용하는 것이 좋다.

## 43. 커스텀 컨테이너 타입은 `collections.abc`를 상속하라

파이썬 프로그래밍의 상당 부분은 데이터를 포함하는 클래스를 정의하고 이런 클래스에 속하는 객체들이 서로 상호작용하는 방법을 기술하는 것으로 이뤄진다. 모든 파이썬 클래스는 함수와 애트리뷰트를 함께 캡술화하는 일종의 컨테이너라 할 수 있다.

파이썬을 사용할 떄 흔히 발생하는 어려움들을 덜어주기 위해 내장 `collections.abc` 모듈 안에는 컨테이너 타입에 정의해야 하는 전형적인 메서드를 모두 제공하는 추상 기반 클래스 정의가 여러 가지 들어 있다. 필요한 메서드 구현을 잊어버리면, 에러가 발생한다.

```python
class BinaryNode:
    def __init__(self, value, left=None, right=None)
        self.value = value
        self.left = left
        self.right = right


class IndexableNode(BinaryNode):
    def _traverse(self):
        if self.left is not None:
            yield from self.left._traverse()
        yield self
        if self.right is not None:
            yield from self.right._traverse()

    def __getitem__(self, index):
        for i, item in enumerate(self.traverse()): # 이터레이팅
            if i == index:
                return item.value
        raise IndexError(f'인덱스 범위 초과: {index}')


class SequenceNode(IndexableNode):
    def __len__(self):
        for count, _ in enumerate(self.traverse(), 1):
            pass
        return count


from collections.abc import Sequence


class BetterNode(SequenceNode, Sequence):
    pass
```

`SequenceNode`에서 한 것처럼 `collections.abc`에서 가져온 추상 기반 클래스가 요구하는 모든 메서드를 구현하면 index나 count와 같은 추가 메서드 구현을 거저 얻을 수 있다.

`Set` 이나 `MutableMapping`과 같이 파이썬의 관례에 맞춰 구현해야 하는 특별 메서드가 훨씬 많은 더 복잡한 컨테이너 타입을 구현할 때는 이런 추상 기반 클래스가 주는 이점이 커진다.

### 정리

- 커스텀 컨테이너를 제대로 구현하려면 수많은 메서드를 구현해야 한다.
- 커스텀 컨테이너 타입이 `collections.abc`에 정의된 인터페이스를 상속하면 정상적으로 작동하기 위해 필요한 인터페이스와 기능을 제대로 구현하도록 도와준다.
