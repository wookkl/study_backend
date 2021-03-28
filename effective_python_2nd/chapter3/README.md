# Chapter3. 함수

## 19. 함수가 여러 값을 반환하는 경우 절대로 네 값 이상을 언패킹하지 말라

악어 개체의 몸 길이가 전체 개체군의 몸 길이 평균에 비해 얼마나 큰지 계산하는 함수를 작성한다면,

```python
def get_avg_ratio(numbers):
    average = sum(sumbers) / len(numbers)
    scaled = [x / average for x in numbers]
    scaled.sort(reverse=True)
    return scaled

lengths = [63,75,85,64,34,74,53,88,53,86]
longest, * middle, shortest = get_avg_ratio(lengths)

print(f"최대 길이:  {longest:>4.0%}") # 최대 길이:  130%
print(f"최소 길이:  {shortest:>4.0%}") # 최소 길이:   50%
```

이때 몸길이의 평균, 중앙값, 악어 개체 수 까지 요구한다면,

```python
def get_stats(numbers):
    minimum = min(numbers)
    maximum = max(numbers)
    count = len(numbers)
    average = sum(numbers) / count

    sorted_numbers = sorted(numbers)
    middle = count // 2
    if count % 2 == 0:
        lower = sorted_numbers[middle - 1]
        upper = sorted_numbers[middle]
        median = (lower + upper) / 2
    else:
        median = sorted_numbers[middle]
    return minimum, maximum, average, median, count


lengths = [63, 75, 85, 64, 34, 74, 53, 88, 53, 86]
minimum, maximum, average, median, count = get_stats(lengths)
print(f"최소 길이: {minimum}, 최대 길이: {maximum}")
print(f"평균: {average}, 중앙 값: {median}, 개체 수: {count}")
```

이 코드에는 두 가지 문제가 있다. 모든 반환 값이 수이기 떄문에 순서를 혼동하기 쉽다. 또한 언패킹하는 부분이 길고 여러가지 방법으로 줄을 바꿀 수 있어서 가독성이 나빠진다. 따라서, 이러한 문제를 피하려면 함수가 여러 값을 반환하거나 언패킹할 때 값이나 변수를 네개이상 사용하면 안된다.

### 정리

- 함수가 반환한 여러 값을, 모든 값을 처리하는 별표식을 사용해 언패킹할 수도 있다.
- 언패킹 구문에 변수가 네 개 이상 나오면 실수하기 쉬우므로 네 개 이상 사용하면 안 된다. 클래스를 반환하거나 namedtuple 인스턴스를 반환한다.

## 20. None을 반환하기보다는 예외를 발생시켜라

None을 반환하면 False와 동등한 방환 값을 잘못 해석하는 경우는 None이 특별한 의미를 가지는 파이썬 코드에서 흔히 저지르는 실수이다.

실수를 줄이는 두 가지 방법 중

첫 번째 방법은 반환 값을 2-튜플로 분리하는 것이다.

```python
def careful_divide(a, b):
    try:
        return True, a / b
    except ZeroDivisionError:
        return False, None

success, result = careful_divide(x, y)
if not success:
    print("잘못된 입력")
```

이 방법의 문제점은 호출하는 쪽에서 튜플의 첫 번째 부분을 쉽게 무시할 수 있다. None을 반환한 경우와 마찬가지로 실수할 가능성이 높아진다.

두 번째 방법은 특별한 경우에 절대 None을 반환하지 않는 것이다.

```python
def careful_divide(a, b):
    try:
        return a / b
    except ZeroDivisionError:
        raise ValueError("잘못된 입력")


x, y = 5, 2
try:
    result = careful_divide(x, y)
except ValueError:
    print("잘못된 입력")
else:
    print(result)
```

이 접근 방법을 확장해서 타입 애너테이션을 사용하는 코드에도 적용할 수 있다.

```python
def careful_divide(a: float, b: float) -> float:
    """
    a를 b로 나눈다.
    Raises:
        ValueError: b가 0 이어서 나눗셈을 할 수 없을 떄
    """

    try:
        return a / b
    except ZeroDivisionError:
        raise ValueError("잘못된 입력")
```

### 정리

- 특별한 의미를 표시하는 None을 반환하는 함수를 사용하면 None과 다른 값이 조건문에서 False로 평가될 수 있기 떄문에 실수하기 쉽다.
- None을 반환하는 대신 예외를 발생시킨다.
- 절대 None을 반환하지 않는다는 사실을 타입 애너테이션으로 명시할 수 있다.

## 21. 변수 영역과 클로저의 상호 작용 방식을 이해하라.

숫자로 이뤄진 list를 정렬하되, 정렬한 리스트의 앞쪽에는 우선순위를 부여한 몇몇 숫자를 위치시켜야 한다고 가정한다.
각 원소를 정렬할 때 도우미 함수가 반환하는 값을 기준으로 사용한다.

```python
def sort_priority(values, group):
    def helper(x):
        if x in group:
            return (0, x)
        else:
            return (1, x)
    values.sort(key=helper)


numbers = [9, 4, 2, 7, 5, 3]
group = [3, 7]
sort_priority(numbers, group)
print(numbers) # [3, 7, 2, 4, 5, 9]
```

이 함수가 잘 작동하는 이유는

- 파이썬이 클로저를 지원: 클로저로 인해 도우미 함수가 sort_priority 함수의 group 인자에 접근할 수 있다.
- 함수가 **일급 시민**: sort 메서드는 클로저 함수를 key인자로 받을 수 있다.
- 파이썬에는 시퀀스를 비교하는 구체적인 규칙이 있음: 순서댈 원소를 비교해 두 값이 같으면 그다음 원소로 넘어가는 작업을 시퀀스의 모든 원소를 비교하거나 결과가 다 정해질 때까지 계속한다. 이 성질로 helper 클로저가 반환하는 튜플이 서로 다른 두 그룹을 정렬하는 기준 역할을 할 수 있다.

> 클로저란? 자신이 정의된 영역 밖의 변수를 참조하는 함수
> 일급 시민이란? 이를 직접 가리킬 수 있고, 변수에 대입하거나 다른 함수에 인자로 전달할 수 있다. if 문에서 함수를 비교하는 등이 가능하다는 거슬 의미

식 안에서 변수를 참조할 때 파이썬 인터프리터는 참조를 해결하기 위해 다음 순서로 영역을 뒤진다.

1. 현재 함수 영역
2. 현재 함수를 둘러싼 영역
3. 현재 코드가 들어 있는 모듈의 영역(전역)
4. 내장 영역(len, str등 내장 함수가 들어 있는 영역)

위의 네 가지 영역에 없으면 NameError 예외가 발생한다. 변수가 현재 영역에 정의돼 있지 않다면 파이썬은 변수 대입을 변수 정의로 취급한다.
이 문제는 **영역 지정 버그**라고 부르기도 한다.
함수에서 사용한 지역 변수가 그 함수를 포함하고 이는 모듈 영역을 더럽히지 못하게 막는 것이다. 파이썬에는 클로저 밖으로 데이터를 끌어내는 특별한 구문이 있다. `nonlocal` 구문이 지정된 변수는 앞에서 설명한 영역 결정 규칙에 따라 대입될 변수의 영역이 결정된다.
nonlocal은 모듈 수준의 영역까지 변수 이름을 찾아 올라가지 않는다.

```python
class Sorter:
    def __init__(self, group):
        self.group = group
        self.found = False
    def __call__(self, x):
        if x in self.group:
            self.found = True
            return (0, x)
        return (1, x)
sorter = Sorter(group)
numbers.sort(sorter)
```

### 정리

- 기본적으로 클로저 내부에 사용한 대입문은 클로저를 감싸는 영역에 영향을 끼칠 수 없다.
- 클로저가 자신을 감싸는 영역의 변수를 변경한다는 사실을 표시할 떄는 nonlocal 문을 사용한다.
- 간단한 함수가 아닌 경우에는 nonlocal 문을 사용하지 않는다.

## 22. 변수 위치 인자를 사용해 시각적인 잡음을 줄여라

예를 들어, 디버깅 정보를 로그에 남기고 싶다고 한다. 인자 수가 고정돼 있으면 메시지와 값의 list를 받는 함수가 필요하다.

```python
def log(message, values):
    if not values:
        print(message)
    else:
        values_str = ', '.join(str(x) for x in values)
        print(f"{message}: {values_str}")

log('내 숫자는', [1, 2])
log('안녕', [])
```

스타 인자를 사용하게 되면 잡음이 줄어든다.

```python
def log(message, *values):
    if not values:
        print(message)
    else:
        values_str = ', '.join(str(x) for x in values)
        print(f"{message}: {values_str}")

log('내 숫자는', 1, 2)
log('안녕')
```

가변적인 위치 인자를 받는 데는 두 가지 문제점이 있다.

첫 번째 문제점은 항상 튜플로 변환된다는 것이다. 이렇게 만들어진 튜플은 제너레이터가 만들어낸 모든 값을 포함하며, 이로 인해 메모리를 아주 많이 소비하거나 프로그램이 중단돼버릴 수 있다.

두 번쨰 문제점은 함수에 새로운 위치 인자를 추가하면 해당 함수를 호출하는 모든 코드를 변경해야 한다.
예외가 발생하지 않고 코드가 작동할 수 있기 때문에 이런 버그는 추적하기 어렵다.
\*args를 받아들이는 함수를 확장할 떄는 키워드 기반의 인자만 사용해야 한다. 더 방어적으로 프로그래밍하려면 타입 애너테이션을 사용한다.

### 정리

- def 문에서 \*args를 사용하면 가변 위치 기반 인자를 받을 수 있다.
- \*연산자를 사용하면 가변 인자를 받는 함수에게 시퀀스 내의 원소들을 전달할 수 있다.
- 제너레이터에 \* 연산자를 사용하면 메모리를 모두 소진하고 중단될 수 있다.
- \*args를 받는 함수에 새로운 위치 기반 인자를 넣으면 감지하기 힘들 버그가 생길 수 있다.

## 23. 키워드 인자로 선택적인 기능을 제공하라.

키워드 인자를 넘기는 순서는 관계없다. 위치 기반 인자를 지정하려면 키워드 인자보다 앞에 지정해야 한다. 키워드 인자가 제공하는 유연성을 활용하면 세 가지 큰 이점이 있다.

1. 키워드 인자를 사용하면 코드를 처음 보는 사람들에게 함수 호출의 의미를 정확히 알려줄 수 있다.
2. 함수 정의에서 디폴트 값을 지정할 수 있다.
3. 기존 호출자에게는 하위 호환성을 제공하면서 함수 파라미터를 확장할 수 있는 방법을 제공한다.

별도로 마이그레이션 하지 않아도 기능을 추가할 수 있다. 모든 기존 호출 코드는 동작이 바뀌지 않는다. 이 접근 방법의 문제점은 선택적인 키워드 인자를 여전히 위치 인자로 지정할 수 있다는 것이다.

### 정리

- 함수 인자를 위치에 따라 지정할 수도 있고, 키워드를 사용해 지정할 수도 있다.
- 키워드를 사용하면 위치 인자만 사용할 때는 혼돌할 수 있는 여러 인자의 목적을 명확히 할 수 있다.
- 키워드 인자와 디플트 값을 함께 사용하면 기본 호출 코드를 마이그레이션하지 않아도 쉽게 추가할 수 있다.
- 선택적 키워드 인자는 항상 위치가 아니라 키워드를 사용해 전달돼야 한다.

## 24. None과 독스트링을 사용해 동적인 디폴트 인자를 지정하라

함수가 정의되는 시점에 키워드 인자의 디폴트 값은 단 한 번만 호출된다. 디폴트 인자값은 모듈이 로드될 때 단 한번만 평가된다. 이 경우에 원하는 동작을 달성하는 방법은 디폴트 값으로 None으로 지정하는 것이다.

```python
def log(message, when=None)
    """메시지와 타임 스탬프를 로그에 남긴다
    Args:
        message: 출력할 메시지.
        when: 메시지가 발생한 시각(datetime)
            디폴트 깂은 현재 시간이다.
    """

    if when is None
        whne = datetime.now()
    print(f"{when}: {message}")
```

타입 애너테이션을 사용해도 잘 작동한다.

```python
from typing import Optional


def log(message: str,
        when: Optional[datetime]=None) -> None:
    """메시지와 타임 스탬프를 로그에 남긴다
    Args:
        message: 출력할 메시지.
        when: 메시지가 발생한 시각(datetime)
            디폴트 깂은 현재 시간이다.
    """

    if when is None
        whne = datetime.now()
    print(f"{when}: {message}")
```

### 정리

- 디폴트 인자 값은 그 인자가 포함된 함수 정의가 속한 묘듈로 로드되는 시점에 단 한 번만 평가된다.
- 동적인 값을 가질 수 있는 키워드 인자의 디폴트 값을 표현할 떄는 None을 사용한다. 그리고 문서화 해둔다.
- 타입 애너테이션을 사용할 때도 None을 사용해서 적용할 수 있다.

## 25. 위치로만 인자를 지정하게 하거나 키워드로만 인자를 지정하게 해서 함수 호출을 명확하게 만들라.

키워드만 사용하는 인자는 키워드를 반드시 사용해 지정해야 하고 절대 위치를 기반으로 지정할 수 없다.

```python
def safe_deivision_c(number, divisor, *
                     ignore_overflow=False,
                     ignore_zero_division=False):
    ...
```

\* 기호는 위치 인자의 마지막과 키워드만 사용하는 인자의 시작은 구분해준다.
그러나, 아직 문제가 존재한다. 호출하는 쪽에서 이 함수의 맨 앞에 있는 두 필수 인자를 호출하면서 위치와 키워드를 혼용할 수 있다.
파이썬 3.8에는 이 문제에 대한 해법이 있다 이를 **위치로만 지정하는 인자**라고 부른다.

```python
def safe_deivision_c(number, divisor, /, *,
                     ignore_overflow=False,
                     ignore_zero_division=False):
    ...
```

/와 \* 사이에는 모든 파라미터의 위치와 키워드를 사용해 전달할 수 있다.

### 정리

- 키워드로만 지정해야 하는 인자는 인자 목록에서 \* 다음에 위치한다.
- 위치로만 지정해야 하는 인자는 인자 목록에서 / 앞에 위치한다.
- /와 \* 사이에 있는 파라미터는 키워드와 위치를 기반으로 전달할 수 있다/

## 26. functools.wrap을 사용해 함수 데코레이터를 정의하라

데코레이터를 정의하는 특별한 구문을 제공한다. 데코레이터는 자신이 감싸고 있는 함수가 호출되기 전과 후에 코드를 추가로 실행해준다.
함수의 의미를 강화하거나 디버깅을 하거나 함수를 등록하는 등의 일에 이 기능을 유용하게 쓸 수 있다.

```python
def trace(func):
    def wrapper(*args, **kwargs):
        result = func(*args, **kwargs):
        print(f"{func.__name__} {args!r}, {kwargs!r}) "
              f"-> {result!r}")
        return result
    return wrapper


@trace
def fibonacci(n):
    if n in (0, 1):
        return n
    return (fibonacci(n - 2)+ fibonacci(n - 1))

print(fibonacci(10))
```

데코레이터가 반환하는 함수의 이름이 fibonacci가 아니게 된다.

```python
print(finobacci) # <function trace.<locals>.wrapper at 0x109d55a60>
```

trace함수는 자신의 본문에 정의된 wrapper 함수를 반환한다. 데코레이터로 인해 wrapper 함수가 모듈에 fibonacci라는 이름으로 등록된다.
fibonacci 함수의 맨 앞부분에 있는 독스트링이 출력 돼야 하지만, 실제로는 그렇지 않다.

```python
print(help(finobacci))
"""
Help on function wrapper in module __main__:

wrapper(*args, **kwargs)
"""
```

데코레이터가 감싸고 있는 원래 함수의 위치를 찾을 수 없기 떄문에 객체 직렬화도 꺠진다.

wraps 도우미 함수를 사용하면 된다. 이 함수는 데코레이터 작성을 돕는 데코레이터다.

```python
from functools import wraps


def trace(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        result = func(*args, **kwargs)
        print(f"{func.__name__} {args!r}, {kwargs!r})"
              f"-> {result!r}")
        return result
    return wrapper


@trace
def fibonacci(n):
    if n in (0, 1):
        return n
    return (fibonacci(n - 2) + fibonacci(n - 1))


print(fibonacci) # <function fibonacci at 0x106980a60>
```

### 정리

- 파이썬 데코레이터는 실행 시점에 함수가 다른 함수를 변경할 수 있게 해주는 구문이다.
- 데코레이터를 사용하면 디버거 등 인트로스펙션을 사용하는 도구가 잘못 작동할 수 있다.
- functools의 내장 모듈의 wraps 데코레이터를 사용한다.
