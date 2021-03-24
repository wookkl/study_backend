# Chapter2. 리스트와 딕셔너리

- 리스트를 자연스럽게 보완 할 수 있는 타입이 딕셔너리 타입이다.
- 딕셔너리 타입은 검색에 사용할 키와 키에 연관된 값을 저장한다.(해시테이블, 연관 배열)
- 동적인 정보를 관리하는 데는 딕셔너리가 가장 이상적이다.

## 11. 시퀀스를 슬라이싱하는 방법을 익혀라

- 시퀀스를 여러 조각으로 나누는 슬라이싱 구문이 있다.
- 최소한의 노력으로 시퀀스에 들어 있는 아이템의 부분 집합에 쉽게 접근할 수 있다.
- 어떤 파이썬 클래스에도 슬라이싱을 추가할 수 있다. `__getitem__`과 `__setitem` 특별 메서드를 구현하면 된다.
- 리스트의 맨 앞부터 슬라이싱할 때는 시각적 잡음을 없애기 위해 0을 생략한다.

리스트의 끝에서부터 원소를 찾고 싶을 때는 음수 인덱스를 사용하면된다.

```python
a = list(range(10))
a[:] # [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
a[:5] # [0, 1, 2, 3, 4]
a[:-1] # [0, 1, 2, 3, 4, 5, 6, 7, 8]
a[4:] # [4, 5, 6, 7, 8, 9]
a[-3:] # [7, 8, 9]
a[2:5] # [2, 3, 4]
a[2:-1] # [2, 3, 4, 5, 6, 7, 8]
a[-3:-1] # [7, 8]
```

리스트를 슬라이싱한 결과는 완전히 새로운 리스트이다.

```python
a = list(range(10))
b = a[3:]
print('이전', b)
b[1] = 99
print('이후', b)
print('변화 없음', a)
"""
결과:
이전 [3, 4, 5, 6, 7, 8, 9]
이후 [3, 99, 5, 6, 7, 8, 9]
변화 없음 [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
"""
```

슬라이스 대입에서는 슬라이스와 대입되는 리스트의 길이가 같을 필요가 없다.

```python
a = list(range(10))
print('이전', a)
a[2:7] = list(range(10, 12))
print('이후', a)
"""
결과:
이전 [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
이후 [0, 1, 10, 11, 7, 8, 9]
"""
```

슬라이싱에서 시작과 끝 인덱스를 모두 생략하면 원래 리스트를 복사한 새 리스트를 얻는다.

```
a = list(range(10))
b = a[:]
assert a == b and a is not b
```

### 정리

- 슬라이싱은 범위가 넘어가는 시작 인덱스나 끝 인덱스도 허용한다. 따라서 시퀀스의 시작이나 끝에서 길이를 제한하는 슬라이스를 쉽게 표현할 수 있다.
- 리스트 슬라이스에 대입하면 원래 시퀀스에서 슬라이스가 가리키는 부분을 대입 연산자 오른쪽에 있는 시퀀스로 대치한다.
- 슬라이스와 대치되는 시퀀스의 길이가 달라도 된다.

## 12. 스트라이드와 슬라이스를 한 식에 함께 사용하지 말라

파이썬은 일정한 간격을 두고 슬라이싱 할 수 특별한 구문을 제공한다. 이를 스트라이드라고 한다.

```python
a = list(range(2, 10))
print(a[::2])
print(a[1::2])
"""
결과:
[2, 4, 6, 8]
[3, 5, 7, 9]
"""
```

스트라이드를 사용하는 구문은 종종 예기치 못한 동작이 일어나서 버그를 야기할 수 있는 단점이 있다.

바이트 문자열도 역으로 뒤집을 수 있다.

```python
x = b'mongoose'
y = x[::-1]
print(y)
"""
결과:
b'esoognom
"""
```

### 정리

- 슬라이스에 시작, 끝, 증가값을 함께 지정하면 코드의 의미를 혼동하기 쉽다.
- 가급적 음수 증가값은 피한다.
- 한 슬라이스 안에서 시작, 끝, 증가값을 함께 사용하지 않는다. 세 파라미터를 사용해야 하는 경우 두 번 대입을 사용하거나 itertool의 `islice`를 사용한다.

## 13. 슬라이싱 보다는 나머지를 모두 잡아내는 언패킹을 사용하라

기본 언패킹의 한 가지 한계점은 언패킹할 시퀀스의 길이를 미리 알고 있어야 한다는 것이다.

이를 다룰 수 있기 위해 **별표** 식을 사용하여 언패킹 할 수 있게 지원한다.
이 구문을 사용하면 다른 부분에 들어가지 못하는 모든 값을 별이 붙은 부분에 다 담을 수 있다.

```python
one, two, *others = list(range(10))
"""
결과:
0
1
[2, 3, 4, 5, 6, 7, 8, 9]
"""

# 다른 위치에서도 쓸 수 있다.
one, *others, ten = list(range(10))
"""
결과:
0
[1, 2, 3, 4, 5, 6, 7, 8]
9
"""
```

하지만 별표 식이 포함된 언패킹 대입을 처리하려면 필수인 부분이 적어도 하나는 있어야 한다.
또한, 한 수준의 언패킹 패턴에 별표 식을 두 개 이상 쓸 수도 없다.

```python
*others = list(range(10)) # 에러
first, *first_middle, *second_middle, last = list(range(10)) # 에러
```

별표식은 항상 리스트 인스턴스가 된다. 시퀀스에 남은 원소가 없으면 별표 식 부분은 빈 리스트가 된다.

주의할 점은 별표 식은 항상 리스트를 만들어내기 때문에 이터레이터를 별표 식으로 언패킹하면 컴퓨터 메모리르 모두 다 사용해서 프로그램이 멈출 수 있다. 따라서 결과 데이터가 모두 메모리에 들어갈 수 있다고 확실할 때만 나머지를 모두 잡아내는 언패킹을 사용해야 한다.

### 정리

- 언패킹 대입에 별표 식을 사용하면 언패킹 패턴에서 대입되지 않는 모든 부분을 리스트에 잡아낼 수 있다.
- 별표 식은 언패킹 패턴의 어떤 위치에든 놓을 수 있다.
- 별표 식에 대입된 결과는 항상 리스트이다.

## 14. 복잡한 기준을 사용해 정렬할 때는 key 파라미터를 사용하라

sort 메서드는 자연스럽게 순서를 정할 수 있는 거의 대부분의 내장 타입에 대해 잘 작동한다. 그러나 객체는 어떻게 처리할까?

```python
class Tool:
    def __init__(self, name, weight):
        self.name = name
        self.weight = weight

    def __repr__(self):
        return f'Tool({self.name}, {self.weight})'
tools = [
    Tool('수준계', 3.5),
    Tool('해머', 1.25),
    Tool('스크류드라이버', 0.5),
    Tool('끌', 0.25),
]
tools.sort() # 에러
```

sort 메서드가 호출하는 객체 비교 특별 메서드가 정의돼 있징 않으므로 정렬할 수 없다.

이런 상황을 지원하기 위해 key 파라미터가 있다. key는 함수여야 한다.

```python
tools.sort(key=lambda x: x.name)
print(tools)
"""
결과:
[Tool(끌, 0.25), Tool(수준계, 3.5), Tool(스크류드라이버, 0.5), Tool(해머, 1.25)]
"""
```

예를 들어 weight로 먼저 정렬한 다음에 name으로 정렬하고 싶다면 어떻게 할까?
가장 쉬운 해법은 tuple 타입을 쓰는 것이다. 튜플은 값을 넣을 수 있는 불변 값이다. 비교 가능하며 자연스러운 순서가 정해져 있다.
이는 sort에 필요한 `**lt** 정의가 들어 있다는 뜻이다. 이 특별 메서드는 튜플의 각 위치를 이터레이션하면서 각 인덱스에 해당하는 원소를 한 번에 하나씩 비교하는 방식으로 구현돼 있다.

정렬에 사용할 두 애트리뷰트를 우선순위에 따라 튜플에 넣어 변환하는 key 함수를 정의한다.

```python
tools.sort(key=lambda x: (x.weight, x.name))
print(tools)
"""
결과:
[Tool(끌, 0.25), Tool(스크류드라이버, 0.5), Tool(해머, 1.25), Tool(수준계, 3.5)]
"""
```

파이썬은 안정적인 정렬 알고리즘을 제공한다. 리스트 타입의 sort 메서드는 key 함수가 반환하는 값이 서로 값은 경우 리스트에 들어 있던 원래 순서를 그대로 유지해준다. 이는 같은 리스트에 대해 서로 다른 기준으로 sort를 여러 번 호출해도 된다는 뜻이다.

### 정리

- 리스트 타입에 들어 있는 sort 메서드를 사용하면 원소 타입이 문자열, 정수, 튜플 등과 같은 내장 타입인 경우 자연스러운 순서로 정렬할 수 있다.
- 원소 타입에 특별 메서드를 통해 자연스러운 순서가 정의돼 있지 안흥면 sort 메서드를 쓸 수 없다.
- sort 메서드의 key 파라미터를 사용하면 리스트의 각 원소 대신 비교에 사용할 객체를 반환하는 도우미 함수를 제공할 수 있다.
- key 함수에서 튜플을 반환하면 여러 정렬 기준을 하나로 엮을 수 있다. 부호 반전 연산자를 사용하면 부호를 바꿀 수 있는 타입이 정렬 기준인 경우 정렬 순서를 반대로 바꿀 수 있다.

## 15. 딕셔너리 삽입 순서에 의존할 떄는 조심하라

파이썬 3.5 이전에는 딕셔너리에 대해 이터레이션을 수행하면 키를 임의의 순서로 돌려줬다.  
그 이유는 내장 hash 함수와 파이썬 인터프리터가 시작할 때 초기화되는 난수 씨앗값을 사용하느 해시 테이블 알고리즘으로 만들어졌기 때문이다.
파이썬 3.6부터는 딕셔너리가 삽입 순서를 보존하도록 동작이 개선됐고, 파이썬 3.7 부터는 아예 언어 명세에 이 내용이 포함됐다.

list, dict 등 표준 프로토콜을 흉내 내는 커스텀 컨테이너 타입을 쉽게 정의할 수 있다.
파이썬은 엄격한 클래스 계층보단 객체의 실질적인 타입을 결정하는 **덕 타이핑** 에 의존한다.

> 덕타이핑이란, '어떤 존재가 오리처럼 꽥꽥 소리르 내고 오리처럼 보인다면 그것은 오리다'라는 말로 겍체가 실행 시점에 어떻게 행동하는지를 기준으로 객체의 타입을 판단하는 타입 지정 방식이다.

예들들어 가장 귀여운 아기 동물을 뽑는 콘테스트의 결과를 보여주는 프로그램을 만든다고 한다면

```python
votes = {
    'otter': 1281,
    'polar bear': 587,
    'fox': 852,
}


def populate_ranks(votes, ranks):
    names = list(votes.keys())
    names.sort(key=votes.get, reverse=True)
    for i, name in enumerate(names, 1):
        ranks[name] = i


def get_winner(ranks):
    return next(iter(ranks))


ranks = {}
populate_ranks(votes, ranks)
winner = get_winner(ranks)
print(ranks)
print(winner)
"""
결과:
{'otter': 1, 'fox': 2, 'polar bear': 3}
otter
"""
```

만약 결과를 보여줄 때 등수가 아니라 알파벳 순으로 표시해야 한다면 알파벳 순서대로 이터레이션 해주는 클래스를 정의할 수 있다.

```python
from collections.abc import MutableMapping

class SortedDict(MutableMapping):
    def __init__(self):
        self.data = {}

    def __getitem__(self, key):
        return self.data[key]

    def __setitem__(self, key, value):
        self.data[key] = value

    def __delitem__(self, key);
        del self.data[key]

    def __iter__(self):
        keys = list(self.data.keys())
        keys.sort()
        for key in keys:
            yield key

    def __len__(self):
        return len(self.data)
```

SortedDict는 표준 딕셔너리의 프로토콜을 지키기 때문에, 앞에 정의한 함수를 호출하면서 표준 dict 위치에 사용해도 상관없다.
그러나 실행 결과는 요구 사항에 맞지 않는다.
문제는 get_winner의 구현이 populate_ranks의 삽입 순서에 의존한다는 부분이다. 문제점을 해결하기 위해 세가지가 있다.

첫번째는 get_winner함수를 특성 순서로 이터레이션이된다고 가정하지 않고 구현한다.

```python
def get_winner(ranks):
    for name, rank in ranks.items():
        if rank == 1:
            return rank
```

두번쨰는 맨 앞에 원하는 타입인지 검사하는 코드를 추가한다 보수적인 접근 방법보다 실행 성능이 더 좋을 것이다.

```python
def get_winner(ranks):
    if not isinstance(ranks, dict):
        raise TypeError('dict 인스턴스가 필요합니다')
    return next(iter(ranks))
```

세번째는 타입 애너테이션을 사용해서 MutableMapping 인스턴스가 아니라 dict 인스턴스가 되도록 강제하는 것이다.

```python
from typing import Dict, MutableMapping


def polulate_ranks(votes: Dict[str, int],
                   ranks: Dict[str, int]) -> None:
    names = list(votes.keys())
    names.sort(key=votes.get, reverse=True)
    for i, name in enumerate(names, 1):
        ranks[name] = i

def get_winner(ranks: Dict[str, int]) -> str:
    return next(iter(ranks))
```

이 해법은 정적 타입 안정성과 런타임 성능을 가장 잘 조합해준다.

### 정리

- 파이썬 3.7 부터는 dict 인스턴스에 들어 있는 내용을 이터레이션할 때 키를 삽입한 순서대로 돌려받는 사실에 의존할 수 있다.
- 파이썬은 딕셔너리와 비슷한 객체를 만들 수 있지만, 키 삽입 순서가 그대로 보존된다고 할 수 없다.

## 16. in을 사용하고 딕셔너리 키가 없을 때 KeyError를 처리하기보다는 get을 사용하라

카운터를 증가시키려면 먼저 키가 딕셔너리에 존제하는지 확인해야 한다. 키가 없으면 0을 딕셔너리레 넣고 그 값을 증가시킨다.

```python
counters = {
    '품퍼니켈': 2,
    '사워도우': 1,
}

key = '밀'
if key in counters:
    count = counters[key]
else:
    count = 0
counters[key] = count + 1

#또는

try:
    count = counter[key]
except KeyError:
    count = 0
counters[key] = count + 1
```

get을 사용하면 코드가 훨씬 짧다.

```python
count = counters.get(key, 0)
counters[key] = count + 1
```

더 복잡한 값이라면

```python
votes = {
    '바게트': ['철수', '순이'],
    '치아바타': ['하니', '유리'],
}
key = '브리오슈'
who = '민이'

if key in votes:
    names = votes[key]
else:
    votes[key] = names = []
names.append(who)

# 또는 wallus을 사용한다

if (names := votes.get(key)) is None:
    votes[key] = names = []

names.append(who)
```

setdefault은 가독성이 떨어진다. 또한, 키가 없으면 setdefault에 전달된 디폴트 값이 별도로 복사되지 않고 딕셔너리에 직접 대입되기 때문에 호출할 때 마다 리스트를 만들어야 하므로 성능이 저하될 수 있다.

### 정리

- 딕셔너리 키가 없는 경우 get 메서드를 사용하여 처리하는 것이 가장 좋다.
- 해결하려는 문제에 dict의 setdefault 메서드를 사용하는 방법이 가장 적합해 보인다면 defaultdict를 고려해본다.

## 17. 내부 상태에서 원소가 없는 경우를 처리할 떄는 setdefault보다 defaultdict를 사용하라

```python
from collections import defaultdict

class Visits:
    def __init__(self):
        self.data = defaultdict(set)
    def add(self, country, city):
        self.data[country].add(city)

visits = Visits()
visits.add('영국', '바스')
visits.add('영국', '런던')
```

딕셔너리에 있는 키에 접근하면 항상 기존 set 인스턴스가 반환된다고 가정한다. 그러나 불필요한 set 인스턴스가 만들어지는 경우는 앖다.

### 정리

- 키가 어떤 값이 들어올지 모르는 딕셔너리를 관리해야 할떄 collections 내장 모듈에 있는 defaultdict를 사용한다.

## 18. **\_\_missing\_\_** 을 사용해 키에 따라 다른 디폴트 값을 생성하는 방법을 알아두라

setdefault나 defaultdict 모두 사용하기가 적당하지 않은 경우가 있다.

```python
"""
SNS 프로필 사진을 관리하는 프로그램
"""
pictures = {}
path = 'profile_1234.jpg'

if (handle := pirctures.get(path)) is None:
    try:
        handle = open(path, 'a+b')
    except OSError:
        print(f'경로를 열 수 없습니다: {path}')
        raise
    else:
        pictures[path] = handle
handle.seek(0)
image_data = handle.read()
```

내부 상태를 관리하려 한다면 프로필 사진의 상태를 관리하기 위해 defaultdict를 쓸 수 있다고 생각하겠지만, 문제는 default 생성자에 전달한 함수는 인자를 받을 수 없다.

```python
from collections import defaultdict

def open_picture(profile_path):
    try:
        return open(profile_path, 'a+b')
    except OSError:
        print(f'경로를 열 수 없습니다: {path}')
        raise
    else:
pictures = defaultdict(open_picture) # 에러, 인자를 받을 수 없음
handle = pictures[path]
handle.seek(0)
```

**\_\_missing\_\_** 특별 메서드를 구현하면 키가 없는 경우를 처리하는 로직을 커스텀할 수 있다.

```python
class Pictures(dict):
    def __missin__(self, key):
        value = open_picture(key)
        self[key] = value
        return value
pictures = Pictures()
handle = pictures[key] # path가 없으면 __missing__ 메서드를 호출
handle.seek(0)
image_data = handle.read()
```

### 정리

- 디폴트 값을 만드는 계산 비용이 높거나 만드는 과정에서 예외를 발생할 수 있는 상황에서는 setdefault 메서드를 사용하지 않는다.
- defaultdict에 전달되는 함수는 인자를 받을 수 없다.
- 디폴트 키를 만들 때 어떤 키를 사용했는지 반드시 알아야 하는 상황이라면 직접 dict에 **missing** 메서드를 정의하면 된다.
