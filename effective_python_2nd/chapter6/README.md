# 6. 메타클래스와 애트리뷰트

메타클래스는 어렴풋잉 이 개념이 클래스를 넘어서는 것임을 암시한다. 메타클래스를 사용하면 파이썬의 class 문을 가로채서 클래스가 정의될 때마다 특별한 동작을 제공할 수 있다. 메타 클래스와 애트리뷰트를 잘 사용하기 위해서는 **최소 놀람의 법칙** 을 따르고 잘 정해진 관용어로만 사용해야 한다.

> 최소 놀람의 법칙이란? 함수나 클래스는 다른 프로그래머가 당연하게 여길 만한 동작과 기능을 제공해야 한다는 뜻이다.

## 44. 세터와 게터 메섣 대신 평법한 애트리뷰트를 사용하라.

클래스에 게터나 세터 메서드를 명시적으로 정의하곤한다.

```python
class OldResister:
    def __init__(self, ohms):
        self._ohms = ohms

    def get_ohms(self):
        return self._ohms

    def set_ohms(self, ohms):
        self._ohms = ohms
```

그러나 파이썬 답지 않고 파이썬에서는 명시적인 세터나 게터 메서드를 구현할 필요가 전혀 없다. 애트리뷰트가 설정될 떄 특별한 기능을 수행해야 한다면 `@property` 데코레이터와 대응하는 setter 애트리뷰트로 옮겨갈 수 있다.
코드가 정상적으로 작동하려면 세터와 게터의 이름이 프로퍼티 이름과 일치해야 한다.

```python
class Resistor:
    def __init__(self, ohms):
        self.ohms = ohms
        self.voltage = 0
        self.current = 0


class VoltageResistance(Resistor):
    def __init__(self, ohms):
        super().__init__(ohms)
        self._voltage = 0

    @property
    def voltage(self):
        return self._voltage

    @voltage.setter
    def voltage(self, voltage):
        self._voltage = voltage
        self.current = self._voltage / self.ohms
```

정의된 애트리뷰트를 불변으로 만들 수도 있다.

```python
class FixedResistance(Resistor):
    def __init__(self, ohms):
        super().__init__(ohms)

    @property
    def ohms(self):
        return self._ohms

    @ohms.setter
    def ohms(self, ohms):
        if hasattr(self, '_ohms'):
            raise AttributeError("Ohms는 불변 객체입니다")
        self._ohms = ohms
```

게터 프로퍼티 메서드 안에서 다른 애트리뷰트를 설정하면 안 된다. 게터나 세터를 정의할 떄 가장 좋은 정책은 관련이 있는 객체 상태를 `@property.setter` 메서드 안에서만 변경하는 것이다.

`@property`의 가장 큰 단점은 애트리뷰트를 처리하는 메서드가 하위 클래스사이에서만 공유될 수 있다는 것이다. 서로 관련이 없는 클래스 사이에 같은 구현을 공유할 수는 없다. 대신 재사용 가능한 프로퍼티 로직을 구현할 떄는 **디스크립터**를 제공한다.

### 정리

- 간단한 공개 애트리뷰트에서 시작하고, 세터나 게터의 메서드를 가급적 사용하지 않는다.
- 애트리뷰트에 접근할 때 특별한 동작이 필요하면 `@property`로 이를 구현할 수 있다.
- `@property` 메서드를 만들 때는 최소 놀람의 법칙을 따른다.
- `@property` 메서드가 빠르게 실행되도록 유지한다. 느리거나 복잡한 작업의 경우에 프로퍼티 대신 일반적인 메서드를 사용한다.

## 45. 애트리뷰트를 리팩터링하는 대신 @property를 사용하라

`@property`를 사용하면 데이터 모델을 점진적으로 개선할 수 있다.

```python
class Buket:
    def __init__(self, period):
        self.period_delta = timedelta(seconds=period)
        self.reset_time = datetime.now()
        self.quata = 0

def fill(bucket, amount):
    now = datetime.now()
    if (now - ubcket.reset_time) > bucket.period_delta:
        bucket_quota = 0
        bucket.reset_time = now
    bucket.quota += amount

def deduct(bucket, amount):
    now = datetime.now()
    if (now - bucket.reset_time) > bucket.period_delta:
        return False # 새 주기가 시작됐는데 아직 버킷 할당량이 설정되지 않았다.
    if bucket.quota - amount < 0:
        return False # 버킷 가용 룡량이 충분하지 않다.
    else:
        bucket.quota -= amount
        return True # 필요한 분량을 사용한다.

```

이 구현의 문제점은 버킷이 시작할 떄 가용 용량이 얼마인지 알 수 없다는 것이다. 그리고 가용 용량이 0이 되면, 버킷에 있는 가용 용량이 0이 될 떄까지 감소할 것이다. 가용 용량이 0이 되면, 버킷에 새로운 가용 용량을 할당하기 전까지 deduct는 항상 False를 반환한다. deduct를 호출하는 쪽에서 자신이 차단된 이유를 모른다. 문제를 해결하기 위해 이번 주기에 재설정된 가용 용량인 max_quota와 이번 주기에 버킷에서 소비한 용량의 합계인 quota_consumed를 추적한다.

```python

class NewBucket:
    def __init__(self, period):
        self.period_delta = timedelta(seconds=period)
        self.reset_time = datetime.now()
        self.max_quata = 0
        self.quota_consumed = 0

    @property
    def quota(self):
        return self.max_quota - self.quota_consumed

    @quota.setter
    def quota(self, amount):
        delta = self.max_quota - amount
        if amount == 0:
            # 새로운 주기가 되고 가용 용량을 재설정하는 경우
        elif delta < 0:
            # 새로운 주기가 되고 가용 용량을 추가하는 경우
            assert self.quota_consumed == 0
            self.max_quota = amount
        else:
            # 어떤 주기 안에서 가용 용량을 소비하는 경우
            assert self.max_quota >= self.quota_consumed
            self.quota_consumed += delta
```

가장 좋은 점은 `Bucket.quota`를 사용하는 코드를 변경할 필요가 없고 클래스의 구현이 변경됐음을 알 필요도 없다는 것이다. 추가로 `max_quota`와 `quota_consumed에도` 직접 접근할 수 있다. `@property`를 사용하면 데이터 모델을 점진적으로 개선할 수 있다.

### 정리

- `@property`를 사용해 기존 인스턴스 애트리뷰트에 새로운 기능을 제공할 수 있다.
- `@property`를 사용해 데이터 모델을 점진적으로 개선한다.
- `@property` 메서드를 너무 과하게 쓰고 있다면, 클래스와 클래스를 사용하느 모든 코드를 리팩터링하는 것을 고려한다.

## 46. 재사용 가능한 `@property` 메서드를 만들려면 디스크립터를 사용하라

`@property` 내장 기능의 가장 큰 문제점은 재사용성이다. `@property`가 데코레이션하는 메서드를 같은 클래스에 속하는 여러 애트리뷰트로 사용할 수는 없다. 그리고 서로 무관한 클래스 사이에서 `@property` 데코레이터를 적용한 메서드를 사용할 수도 없다. 이를 대처하는 방법은 **디스크립터** 를 사용하는 것이다. **디스크립터 프로토콜** 은 파이썬 언어에서 애트리뷰트 접근을 해석하는 방법을 정의한다. 디스크립터의 클래스는 `__get__`과 `__set__` 메서드를 제공한다. 이 메서드를 사용하면 원하는 검증을 재사용할 수 있다.

```python
class Grade:
    def __get__(self, instance, instance_type):
        ...

    def __set(self, instance, value):
        ...


class Exam:
    math_grade = Grade()
    writing_grade = Grade()
    science_grade = Grade()

```

Exam 인스턴스에 잇는 이런 디스크립터 애트리뷰트에 대한 접근을 파이썬이 어떻게 처리하는지 이해하는 것이 중요하다.

```python
exam = Exam()

exam.writing_grade = 40
# 다음과 같이 해석된다.
Exam.__dict__['writing_grade'].__set__(exam, 40)

exam.writing_grade
# 프로퍼티를 읽으면
Exam.__dict__['writing_grade'].__get__(exam, Exam)
```

`object`의 `__getattribute__` 메서드 떄문에 이러한 동작이 이루어진다.

```python
class Grade:
    def __init__(self):
        self._value = 0

    def __get__(self, instance, instance_type):
        return self._value

    def __set__(self, instance, value):
        if not (0 <= value <= 100):
            raise ValueError("점수는 0과 100 사이입니다")
        self._value = value

```

이 구현은 틀렸고 잘못 동작한다. 여러 `Exam` 인스턴스 객체에 애트리뷰트 접근을 시도하면 예기치 못한 동작을 볼 수 있다.
이 애트리뷰트에 대한 `Grade` 인스턴스가 단 한 번만 생성된다. `Exam` 인스턴스가 생성될 떄 마다 생성되지 않는다.

`Grade` 클래스가 각각의 유일한 `Exam` 인스턴스에 대해 따로 값을 추척하고 메모리 누수르 방지하기 위해 `weakref` 내장 모듈의 `WeakKeyDictionary` 라는 클래스를 사용한다. 딕셔너리 객체를 저장할 떄 일반적인 강한 참조 대신에 약한 참조를 사용한다.
약한 참조로만 참조되는 객체가 사용 중인 메모리를 언제든지 재활용할 수 있다.

```python
from weakref import WeakKeyDictionary


class Grade:
    def __init__(self):
        self._values = WeakKeyDictionary()
    ...
```

### 정리

- `@property` 메서드의 동작과 검증 기능을 재사용학 싶다면 디스크립터 클래스를 만든다.
- 디스크립터 클래스를 만들 때는 메모리 누수를 방지하기 위해 `WeakKeyDictionary`를 사용한다.
- `__getattribute__`가 디스크립터 프로토콜을 사용해 애트리뷰트 값을 읽거나 설정하는 방식을 정확히 이해한다.
