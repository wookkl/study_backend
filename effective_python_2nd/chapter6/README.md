# 6. 메타클래스와 애트리뷰트

메타클래스는 어렴풋이 이 개념이 클래스를 넘어서는 것임을 암시한다. 메타클래스를 사용하면 파이썬의 `class` 문을 가로채서 클래스가 정의될 때마다 특별한 동작을 제공할 수 있다. 메타 클래스와 애트리뷰트를 잘 사용하기 위해서는 **최소 놀람의 법칙** 을 따르고 잘 정해진 관용어로만 사용해야 한다.

> 최소 놀람의 법칙이란? 함수나 클래스는 다른 프로그래머가 당연하게 여길 만한 동작과 기능을 제공해야 한다는 뜻이다.

## 44. 세터와 게터 메서드 대신 평법한 애트리뷰트를 사용하라.

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

그러나 파이썬 답지 않고 파이썬에서는 명시적인 세터나 게터 메서드를 구현할 필요가 전혀 없다. 애트리뷰트가 설정될 떄 특별한 기능을 수행해야 한다면 `@property` 데코레이터와 대응하는 `setter` 애트리뷰트로 옮겨갈 수 있다.
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
이 애트리뷰트에 대한 `Grade` 인스턴스가 단 한 번만 생성된다. `Exam` 인스턴스가 생성될 때 마다 생성되지 않는다.

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

## 47. 지연 계산 애트리뷰트가 필요하면 `__getattr__`, `__getattribute__`, `__setattr__`을 사용하라

어떤 클래스 안에 `__getattr__` 메서드 정의가 있으면, 이 객체의 인스턴스 딕셔너리에서 찾을 수 없는 애트리뷰트에 접근할 떄마다 `__getattr__` 이 호출된다.

```python
class LazyRecord:
    def __init__(self):
        self.exists = 5

    def __getattr__(self, name):
        value = f"{name}를 위한 값"
        setattr(self, name, value)
        return value


data = LazyRecord()
print(data.__dict__) # {'exists': 5}
print(data.foo) # foo를 위한 값
print(data.__dict__) # {'exists': 5, 'foo': 'foo를 위한 값'}
```

exists 애트리뷰트가 인스턴스 딕셔너리에 있으므로 `__getattr__`이 호출되지 않는다. 반면에 `foo` 애트리뷰트는 처음에 인스턴스 딕셔너리에 없으므로 맨 처음 `foo`에 접근하면 `__getattr__`이 호출된다. 하지만 `__getattr__`이 호출되고, 안에서 `setattr을` 수행해 인스턴스 딕셔너리안에 `foo`라는 애트리뷰트를 추가한다.

고급 사용법을 제공하기 위해 파이썬은 `__getattribute__`라는 다른 `object` 훅을 제공한다. 이 특별메서드는 객체의 애트리뷰트에 접근할 때마다 이 훅이 호출된다. 이를 사용하면 프로퍼티에 접근할 때마다 항상 전역 트랜잭션 상태를 검사하는 작업을 수행할 수 있다.

```python
class ValidatingRecord:
    def __init__(self):
        self.exists = 5

    def __getattribute__(self, name):
        try:
            value = super().__getattribute__(name)
            return value
        except AttributeError:
            value = f"{name}을 위한 값"
            setattr(self, name, value)
            return value
```

임의의 애트리뷰트에 값을 설정할 때마다 호출되는 `object` 훅인 `__setattr__`을 사용하면, 이런 기능을 비슷하게 구현할 수 있다. `__getattr__`나 `__getattribute__`로 값을 읽을 떄와 달리 메서드가 두 개 있을 필요가 없다. `__setattr__`은 인스턴스 애트리뷰트에 대입이 이뤄질 때마다 항상 호출된다.

```python
class SavingRecord:
    def __setattr__(self, name, value):
        print("호출: __setattr__")
        super().__setattr__(name, value)


data = SavingRecord()
print("이전: ", data.__dict__)
data.foo = 5
print("이후: ", data.__dict__)
data.foo = 7
print("최후: ", data.__dict__)
"""
이전:  {}
호출: __setattr__
이후:  {'foo': 5}
호출: __setattr__
최후:  {'foo': 7}
"""
```

### 정리

- `__getattr__`과 `__setattr__`을 사용해 객체의 애트리뷰트를 지연해 가져오거나 저장할 수 있다.
- `__getattr__`은 애트리뷰트가 존재하지 않을 때만 호출되지만, `__getattribute__`는 애트리뷰트를 읽을 때마다 항상 호출된다.
- `__getattribute__`와 `__setattr__`에서 무한 재귀를 피하려면 `super()`에 있는 메서드를 사용해 인스턴스 애트리뷰트에 접근한다.

## 48. `__init_subclass__`를 사용해 하위 클래스를 검증하라

어떤 클래스 타입의 객체가 실행 시점에 생성될 때 클래스 검증 코드를 `__init__`메서드 안에서 실행하는 경우도 종종 있다. 검증에 메타클래스를 사용하면, 프로그램 시작 시 클래스가 정의된 모듈을 처음 임포트할 떄와 같은 시점에 검증이 이뤄지기 떄문에 예외가 훨씬 더 빨리 발생할 수 있다.

메타클래스느 `type`을 상속해 정의된다. 기본적인 경우 메타클래스는 `__new__`메서드를 통해 자신과 연관된 클래스의 내용을 받는다.

```python
class Meta(type):
    def __new__(meta, name, bases, class_dict):
        print(f"실행 {name}의 메타 {meta}.__new__")
        print("기반 클래스들: ", bases)
        print(class_dict)
        return type.__new__(meta, name, bases, class_dict)


class MyClass(metaclass=Meta):
    stuff = 123

    def foo(self):
        pass


class MySublass(MyClass):
    other = 1321

    def boo(self):
        pass

"""
실행 MyClass의 메타 <class '__main__.Meta'>.__new__
기반 클래스들:  ()
{'__module__': '__main__', '__qualname__': 'MyClass', 'stuff': 123, 'foo': <function MyClass.foo at 0x10dee8a60>}
실행 MySublass의 메타 <class '__main__.Meta'>.__new__
기반 클래스들:  (<class '__main__.MyClass'>,)
{'__module__': '__main__', '__qualname__': 'MySublass', 'other': 1321, 'boo': <function MySublass.boo at 0x10df59dc0>}
"""
```

메타클래스는 클래스 이름, 클래스가 속하는 부모 클래스들(`bases`), `class`의 본문에 정의된 모든 클래스 애트리뷰트에 접근할 수 있다. 모든 클래스는 `object`를 상속하기 때문에 메타클래스가 받는 부모 클래스의 튜플 안에는 `object`가 명시적으로 들어 있지 않다.

연관된 클래스가 정의되기 전에 이 클래스의 모든 파라미터를 검증하려면 Meta.`__new__`에 기능을 추가해야 한다.

```python
class ValidatePolygon(type):
    def __new__(meta, name, bases, class_dict):
        if bases:
            if class_dict["sides"] < 3:
                raise ValueError("다각형 변은 3개 이상")
        return type.__new__(meta, name, bases, class_dict)


class Polygon(metaclass=ValidatePolygon):
    sides = None

    @classmethod
    def interior_angles(cls):
        return (cls.sides - 2) * 180


class Triangle(Polygon):
    sides = 3


class Rectangle(Polygon):
    sides = 4


class Nonagon(Polygon):
    sides = 9


assert Triangle.interior_angles() == 180
assert Rectangle.interior_angles() == 360
assert Nonagon.interior_angles() == 1260

# 여기까지 통과


class Line(Polygon):
    sides = 2

# 에러
# raise ValueError("다각형 변은 3개 이상")
# ValueError: 다각형 변은 3개 이상
```

파이썬 3.6부터는 이런 메타클래스를 정의하지 않고 같은 동작을 구현할 수 있다.

```python
class BetterPolygon:
    sides = None

    def __init_subclass__(cls):
        super().__init_subclass__()
        if cls.sides < 3:
            raise ValueError("다각형 변은 3개 이상")

    @classmethod
    def interior_angles(cls):
        return (cls.sides - 2) * 180

class Hexagon(BetterPolygon):
    sides = 6
```

`__init_subclass__` 특별 클래스 메서드를 사용하면 합성성에 대한 문제도 해결할 수 있다.
`super` 내장 함수를 사용해 부모나 형제자매 클래스의 `__init_subclass__`를 호출해주는 한, 여러 단계로 이뤄진 `__init_subclass__`를 활용하는 클래스 계층 구조를 쉽게 정의할 수 있다. 심지어 다중 상속과도 잘 어우러진다.

```python
class Filled:
    color = None

    def __init_subclass__(cla):
        super().__init_subclass__()
        if cls.color not in ("RED", "GREEN", "BLUE"):
            raise ValueError("지원하지 않는 color 값")


class RedTriangle(Filled, Polygon):
    color = "RED"
    sides = 3


ruddy = RedTriangle()
assert isinstance(ruddy, Filled)
assert isinstance(ruddy, Polygon)
```

### 정리

- 메타클래스의 `__new__` 메서드는 `class` 문의 모든 본문이 처리된 직후에 호출된다.
- 메타클래스를 사용해 클래스가 정의된 직후이면서 클래스가 생성되기 직전인 시점에 클래스 정의를 변경할 수 있다. 하지만 메타클래스는 원하는 목적을 달성하기에 너무 복잡해지는 경우가 많다.
- `__init_subclass__`를 사용해 하위 클래스가 정의된 직후, 하위 클래스 타입이 만들어지기 직전에 해당 클래스가 원하는 요건을 잘 갖췄는지 확인한다.
- `__init_subclass__`정의 안에서 `super().__init_subclass__`를 호출해 여러 계층에 걸쳐 클래스를 검증하고 다중 상속을 제대로 처리한다.

## `__init_subclass__`를 사용해 클래스 확장을 등록하라

메타클래스의 다른 용례로 프로그램이 자동으로 타입을 등록하는 것이 있다.
간단한 식별자를 이용해 그에 해당하는 클래스를 찾는 역검색을 하고 싶을 때 이런 등록 기능이 유용하다.

파이썬 `object`를 JSON으로 직렬화하는 직렬화 표현 방식을 구현한다고 한다면

```python
import json

class Serializable:
    def __init__(self, *args):
        self.args = args

    def serialize(self):
        return json.dumps({'args': self.args})

class Point2D(Serializable):
    def __init__(self, x, y):
        super().__init__(x, y)
        self.x = x
        self.y = y

point = Point2D(5, 3)
print(point.serialize())

```

역직렬화해서 문자열이 표현하는 Point2D 객체를 구현한다면

```python
class Deserializable(Serializable):
    @classmethod
    def deserialize(cls, json_data):
        params = json.loads(json_data)
        return cls(*params['args'])

class BetterPoint2D(Deserializable):
    def __init__(self, x, y):
        super().__init__(x, y)
        self.x = x
        self.y = y

before = BetterPoint2D(5, 3)
data = before.serialize()
print(data)
after = BetterPoint2D.deserialize(data)
print(after)
```

이 접근 방식은 직렬화할 데이터의 타입을 미리 알고 있는 경우에만 사용할 수 있다는 문제가 있다. JSON으로 직렬화할 클래스가 아주 많더라도 `object`로 역직렬화하는 함수는 공통으로 하나만 있는 것이 이상적이다.

```python
class BetterSerializable:
    def __init__(self, *args):
        self.args = args

    def serialize(self):
        return json.dumps({
            "class": self.__class__.__name__,
            "args": self.args,
        })
```

클래스 이름을 객체 생성자로 다시 연결해주는 매핑을 유지할 수 있다. 매핑을 사용해 구현한 일반적인 `deserialize` 함수는 `register_class`를 등록된 모든 클래스에 대해 잘 작동한다.

```python
registry = {}

def register_class(target_class):
    registry[target_class.__name__] = target_class

def deserialize(data):
    params = json.loads(data)
    name = params["class"]
    target_class = registry[name]
    return target_class(*param["args"])
```

deserialize가 항상 제대로 작동하려면 역직렬화할 클래스에 `register_class`를 호출해야 한다. 그러나 `register_class` 호출을 잊어버릴 수 있다. 이를 방지하기 위해서
클래스의 `__init_subclass__` 특별 메서드를 사용하면 된다.

```python
class BetterRegisterSerializable(BetterSerializable):
    def __init_subclass__(cls):
        super().__init_subclass__()
        register_class(cls)
```

### 정리

- 클래스 등록은 파이썬 프로그램을 모듈화할 때 유용한 패턴이다.
- 메타클래스를 사용하면, 프로그램 안에서 기반 클래스를 상속한 하위 클래스가 정의될 때마다 등록 코드를 자동으로 실행할 수 있다.
- 메타클래스를 클래스 등록에 사용하면 클래스 등록 함수를 호출하지 않아서 생기는 오류를 피할 수 있다.
- 표준적인 메타클래스 방식보다는 `__init_subclass__`가 더 낫다.

## 50. `__set_name__`으로 클래스 애트리뷰트를 표시하라

메타클래스를 통해 사용할 수 있는 유용한 기능이 한 가지 더 있다. 클래스가 정의된 후 클래스가 실제로 사용되기 이전인 시점에 프로퍼티를 변경하거나 표시할 수 있는 기능이다.

고객 데이터베이스의 로우를 표현하는 새 클래스를 정의한다고 한다.

```python
class Field:
    def __init__(self, name):
        self.name = name
        self.internal_name = "_" + self.name

    def __get__(self, instance, instance_type):
        if instance is None:
            return self
        return getattr(instance, self.internal_name, "")

    def __set__(self, instance, value):
        setattr(instance, self.internal_name, value)
```

컬럼 이름을 `Field` 디스크립터에 저장하고 나면, `setattr` 내장 함수를 사용해 인스턴스별 상태를 직접 인스턴스 딕셔너리에 저장할 수 있고, 나중에 `getattr로` 인스턴스의 상태를 읽을 수 있다.

```python
class Customer:
    first_name = Field("first_name")
    last_name = Field("last_name")
    prefix = Field("prefix")
    suffix = Field("suffix")

cust = Customer()

cust.first_name = "유클리드"
```

하지만 이 클래스의 정의느 중복이 많다. 클래스 안에서 왼쪽에 필드 이름을 이미 정의했는데 굳이 같은 정보가 들어 있는 문자열을 `Field` 디스크립터에게 다시 전달해야 할 이유가 없다. 이문제를 해결하는 방법은 디스크립터에 `__set_name__`특별 메서드를 사용하는 것이다. 클래스가 정의될 때마다 파이썬은 해당 클래스 안에 들어있는 디스크립터 인스턴스의 `__set_name__`을 호출한다. 디스크립터 인스턴스를 소유 중인 클래스와 인스턴스가 대입될 애트리뷰트 이름을 인자로 받는다.

```python
class Field:
    def __init__(self, name):
        self.name = None
        self.internal_name = None

    def __get__(self, instance, instance_type):
        if instance is None:
            return self
        return getattr(instance, self.internal_name, "")

    def __set__(self, instance, value):
        setattr(instance, self.internal_name, value)

    def __set_name__(self, owner, name):
        self.name = name
        self.internal_name = "_" + name
```

### 정리

- 메타클래스를 사용하면 어떤 클래스가 완전히 정의되기 전에 클래스의 애트리뷰트를 변경할 수 있다.
- 디스크립터와 메타클래스를 조합하면 강력한 실행 시점 코드 검사와 선언적인 동작을 만들 수 있다.
- `__set_name__`특별 메서드를 디스크립터 클래스에 정의하면 디스크립터가 포함된 클래스의 프로퍼티 이름을 처리할 수 있다.
- 디스크립터가 변경한 클래스의 인스턴스 딕셔너리에 데이터를 저장하게 만들면 메모리 누수를 피할 수 있고, `weakref` 내장 메서드를 사용하지 않아도 된다.

## 51. 합성 가능한 클래스의 확장이 필요하면 메타클래스보다는 클래스 데코레이터를 사용하라

메타클래스를 사용하면 클래스 생성을 다양한 방법으로 커스텀화할 수 있지만, 여전히 메타클래스로 처리할 수 없는 경우가 있다.

어떤 클래스의 모든 메서드를 감싸서 메서드에 전달되는 인자, 반환 값, 발생한 예외를 모두 출력하고 싶다면,

```python
from functools import wraps

def trace_func(func):
    if hasattr(func, "tracing"): # 단 한 번만 데코레이터를 적용한다.
        return func
    @wraps(func)
    def wrapper(*args, **kwargs):
        result = None
        try:
            result = func(*args, **kwargs)
            return result
        except Exception as e:
            result = e
            raise
        finally:
            print(f"{func.__name__}({args!r}, {kwargs!r}) ->",
                  f"{result!r}")
    wrapper.tracing = True
    return wrapper

class TraceDict(dict):
    @trace_func
    def __init__(self, *args, **kwargs)
        super().__init__(*args, **kwargs)

    @trace_func
    def __setitem__(self, *args, **kwargs)
        super().__setitem__(*args, **kwargs)

    @trace_func
    def __getitem__(self, *args, **kwargs)
        super().__getitem__(*args, **kwargs)
```

이 코드의 문제점은 모든 메서드를 데코레이터를 써서 재정의 해야 한다는 것이다. 문제를 해결하는 방법은 메타클래스를 사용해 클래스에 속한 모든 메서드를 자동으로 감싸는 것이다.

```python
class TraceMeta(type):
    def __new__(meta, name, bases, class_dict):
        klass = super().__new__(meta, name, bases, class_dict)
        for key in dir(klass):
            value = getattr(klass, key)
            if isinstance(value, trace_types):
                wrapped = trace_func(value)
                setattr(klass, key, wrapped)
        return klass


class TraceDict(dict, metaclass=TraceMeta):
    pass


trace_dict = TraceDict([("안녕", 1)])
trace_dict["거기"] = 2
trace_dict["안녕"]
```

메타클래스를 사용하는 접근 방식은 적용 대상 클래스에 대한 제약이 너무 많다. 이런 문제를 해결하고자 파이썬은 **클래스 데코레이터**를 지원한다.
클래스 데코레이터는 함수 데코레이터처럼 사용할 수 있다. 클래스 선언 앞에 @ 기호와 데코레이터 함수를 적으면 된다.

```python
def my_class_decorator(klass):
    klass.extra_param = "안녕"
    return klass

@my_class_decorator
class MyClass:
    pass

print(MyClass)
print(MyClass.extra_param)
```

`TraceMeta.new` 메서드의 핵심 부분을 별도의 함수로 옮겨서 클래스 데코레이터를 만들 수 있다.

```python
def trace(klass):
    for key in dir(klass):
        value = getattr(klass, key)
        if isinstance(value, trace_types):
            wrapped = trace_func(value)
            setattr(klass, key, wrapped)

@trace
class TraceDict(dict):
    pass
```

### 정리

- 클래스 데코레이터는 `class` 인스턴스를 파라미터로 받아서 이 클래스를 변경한 클래스나 새로운 클래스를 반환해주는 간단한 함수다.
- 준비 코드를 최소화하면서 클래스 내부의 모든 메서드나 애트리뷰트틑 변경하고 싶을 때 클래스 데코레이터가 유용하다.
- 메타클래스는 서로 쉽게 합성할 수 없지만, 여러 클래스 데코레이터를 충돌 없이 사용해 똑같은 클래스를 확장할 수 있다.
