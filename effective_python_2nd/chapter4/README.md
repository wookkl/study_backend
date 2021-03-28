# 4. 컴프리헨션과 제너레이터

컴프리헨션 코딩 스타일은 **제너레이터** 이라는 특별한 구문을 사용해 이런(리스트, 딕셔너리, 집합 등) 타입을 간결하게 이터레이션하면서 원소로부터 파생되는 데이터 구조를 생성할 수 있다.  
제너레이터는 함수가 전짐적으로 반환하는 값으로 이뤄지는 스트림을 만들어준다. 이터레이터를 사용할 수 있는 곳이라면 어디에서나 제너레이터 함수를 호출한 결과를 사용할 수 있다.

## 27. map과 filter 대신 컴프리헨션을 사용하라

다른 시퀀스나 이터러블에서 새 리스트를 만들어내는 간결한 구문을 **리스트 컴프리헨션** 이라고 한다.
리스트 컴프리헨션을 사용해 루프로 처리할 대상인 입력 시퀀스의 원소에게 적용할 변환식을 지정함으로써 같은 결과를 얻을 수 있다.

```python
squares = [x**2 for x in range(1,11)]
print(squares)
```

딕셔너리 컴프리헨션과 집합 컴프리헨션도 있다.

```python
even_squares_dict = {x: x**2 for x in a if x % 2 == 0}
threes_cubed_set = {x**3 for x in a if x % 3 == 0}
print(even_squares_dict)
print(threes_cubed_set)
```

### 정리

- 리스트 컴프리헨션은 lambda 식을 사용하지 않기 때문에 같은 일을 하는 map과 filter 내장 함수를 사용하는 것보다 더 명확하다.
- 리스트 컴프리헨션을 사용하면 쉽게 입력 리스트의 원소를 건너뛸 수 있다.
- 딕셔너리와 집합도 컴프리헨션이 가능하다.

## 28. 컴프리헨션 내부에 제어 하위 식을 세 개 이상 사용하지 말라

```python
matrix = [[[1, 2, 3,], [4, 5, 6], [7, 8, 9]],
         [[10, 11, 12,], [13, 14, 15], [16, 17, 18]]]
flat = [x for sublist1 in my_lists
        for sublist2 in sublist1
        for x in sublist2]
```

이렇게 3 차원이상 되면 다중 컴프리헨션이 다른 대안에 비해 더 길어진다

```python
flat = []
for sublist1 in my_lists:
    for sublist2 in sublist1:
        flat.extend(sublist2)
```

리스트 컴프리헨션보다 이 코드의 루프가 더 명확해 보인다.

### 정리

- 컴프리헨션은 여러 수준의 루프를 지원하며 각 수준마다 여러 조건을 지원한다.
- 제어 하위 식이 세 개 이상인 컴프리헨션은 이해하기 매우 어려우므로 가능하면 피해야한다.

## 29. 대입식을 사용해 컴프리헨션 안에서 반복 작업을 피하라

예들 들어 회사에서 주문을 관리하기 위한 프로그램을 작성한다고 한다. 고객이 새로운 주문을 보내면 주문을 처리할 만한 재고가 있는 지 알려줘야 한다. 그러려면 고객의 요청이 재고 수량을 넘지 않고, 배송에 필요한 최소수량을 만족하는지 확인해야 한다.

```python
stock {
    '못': 125,
    '나사못': 35,
    '나비너트': 8,
    '와셔': 24
}
order = ['나사못', '나비너트' ,'와셔']

def get_batches(count, size):
    return coutn // size

found = {name: get_batches(stock.get(name, 0), 8)
        for name in order
        if get_batches(stock.get(name, 0), 8)}
```

get_batches가 반복되는 단점이 있다. 두 식을 항상 똑같이 변경해야 하므로 실수할 가능성도 높아진다.
이러한 문제에 대한 쉬운 해법은 왈러스 연산자를 사용하는 것이다.

```python
found = {name: batches for name in order
         if (batches := get_batches(stock.get(name, 0), 8))}
```

컴프리헨션이 값 부분에서 왈러스 연잔자를 사용할 떄 그 값에 대한 조건 부분이 없다면 루프 밖 영역으로 루프 변수가 누출된다.

```python
half = [(last := count // 2) for count in stock.values()]
print(f'{half}의 마지막 원소는 {last}') # 누출
half = [ count // 2 for count in stock.values()]
print(count) # 오류
```

루프 변수를 누출하지 않는 편이 낫다.

### 정리

- 대입식을 통해 컴프리헨션이나 제너레이터 식의 조건 부분에서 사용한 값을 같은 컴프리헨션이나 제너레이터의 다른 위치에서 재사용할 수 있다. 이를 통해 가독성과 성능을 향상시킬 수 있다.
- 조건이 아닌 부분에도 대입식을 사용할 수 있지만, 사용하지 않는 편이 좋다.

## 30. 리스트를 반환하기 보다는 제너레이터를 사용하라

```python
def index_words(text):
    result = []
    if text:
        result.append(0)
    for idx, letter in enumerate(text):
        if letter == ' ':
            result.append(idx + 1)
    return result
address = '조건이 아닌 부분에도 대입식을 사용할 수 있지만, 사용하지 않는 편이 좋다.'
result = index_words(address)
print(result) # [0, 4, 7, 12, 17, 21, 23, 28, 33, 36, 39]
```

index_words 함수에는 두 가지 문제점이 있다.

### 1. 문제점: 코드에 잡음이 많고 핵심을 알아보기 어렵다.

새로운 결과를 찾을 떄 마다 append 메섣그를 호출한다. 메서드 호출이 너무 크기 때문에(result.append) 리스트에 추가될 값(index + 1)의 중요성을 희석해버린다.

이 함수를 개선하는 방법은 **제너레이터** 를 사용하는 것이다.

```python
def index_words_iter(text):
    result = []
    if text:
        yield 0
    for idx, letter in enumerate(text):
        if letter == ' ':
            yield idx + 1
```

이 함수가 호출 되면 제너레이터 함수가 실제로 실행되지 않고 즉시 이터레이터를 반환한다.
제너레이터 함수가 실제로 실행되지 않고 즉시 이터레이터를 반환한다. 이터레이터가 next 내장 함수를 호출할 때마다 이터레이터는 제너레이터 함수를 다음 yield 식 까지 진행시킨다.

```python
it = index_words_iter(address)
print(next(it))
print(next(it))
```

반환하는 리스트와 상호작용하는 코드가 사라졌으므로 index_words_iter 함수가 훨씬 읽기 쉽다. 대신 결과는 yield 식에 의해 전달된다.

### 2. 문제점: 반환하기 전에 리스트의 모든 결과를 다 저장해야한다.

입력이 매우 크면 프로그램이 메모리를 소진해서 종단될 수 있다.
반면 같은 함수를 제너레이터 버전으로 만들면 사용하는 메모리 크기를 어느정도 제한할 수 있으므로 아무리 길어도 쉽게 처리할 수 있다.

제너레이터를 실행하면 다음 결과를 얻을 수 있다.

```python
def index_file(handle):
    offset = 0
    for line in handle:
        if line:
            yield offset
        for letter in line:
            offset += 1
            if letter = ' ':
                yield offset


with open('address.txt', 'r', encoding='utf-8') as f:
    it = index_file(f)
    result = itertools.islice(it, 0, 10)
```

제너레이터를 정의할 때는 한 가지 알아야 할 점이 있다. 제너레이터가 반환하는 이터레이터에 상태가 있기 때문에 호출하는 쪽에서 재사용이 불가능하다.

### 정리

- 제너레이터를 사용하면 결과를 리스트에 합쳐서 반환하는 것보다 더 깔끔하다.
- 제너레이터가 반환하는 이터레이터는 제너레이터 함수의 본문에서 yield가 반환하는 값들로 이뤄진 집합을 만들어진다.
- 제너레이터를 사ㅛㅇ하면 작업 메모리에 모든 입출력을 저장할 필요가 없으므로 입력이 아주 커도 출력 시퀀스를 만들 수 있다.

## 31. 인자에 대해 이터레이션할 때는 방어적이 돼라

이터레이터가 결과를 단 한번만 만들어내기 때문에 주의해야 한다.

```python
def read_visits(data_path):
    with open(data_path) as f:
        for line in f:
            yield int(line)
it = read_visits('my_numbers.txt')
print(list(it))
print(list(it)) # 이미 모든 원소를 다 소진했다.
```

이미 소진된 이터레이터에 대해 이터레이션을 수행해도 오류가 발생되지 않는다. 이 문제를 해결하기 위해 이터레이터를 명시적으로 소진시키고 이터레이터의 전체 내용을 리스트에 넣을 수 있다. 그 후 원하는 만큼 이터레이션을 수핸한다. 그러나 문제점이 있다. 복사하면 메모리를 엄청나게 많이 사용할 수 있다. 이 부분을 해결하기 위해 호출될 떄마다 새로 이터레이터를 반환하면 된다.

```python
def normalize_func(get_iter):
    total = sum(get_iter()) # 새 이터레이터
    result = []
    for value in get_iter(): 새 이터레이터
        ...
```

함수에 이터레이터를 넘기기 위해 **이터레이터 프로토콜** 을 구현한 새로운 컨테이너 클래스를 제공한다.

```python
class ReadVisit:
    def __init__(self, data_path):
        self.data_path = data_path

    def __iter__(self):
        with open(self.data_path) as f:
            for line in f:
                yield int(line)

visits = ReadVisits(path)

print(list(visits))
print(list(visits)) # 둘 다 출력이 잘 된다.
```

이 접근 방법의 유일한 단점은 입력 데이터를 여려 번 읽는다는 것이다.

### 정리

- 입력 인자를 여러 번 이터레이션하는 함수나 메서드를 조심한다. 입력받은 인자가 이터레이터면 함수가 이상하게 작동하거나 결과가 없을 수 있다.
- 파이썬의 이터레이터 프로토콜은 컨테이나와 이터레이터가 iter, next 내장 함수나 for 루프 등의 관련 식과 상호작용하는 절차를 정의한다.
- \_\_iter\_\_ 메서드를 제너레이터로 정의하면 쉽게 이터러블 컨테이너 타입을 정의할 수 있다.
- 어떤 값이 이터레이터인지 컨테이너인지 감지하려면 iter 내장 함수에 넘겨서 반환하는 값이 원래 값과 같은지 확인한다.

## 32. 긴 리스트 컴프리헨션보다는 제너레이터 식을 사용하라

리스트 컴프리헨션의 문제점은 입력이 커지면 메모리를 상당히 많이 사용하고 그로 인해 프로그램이 중단될 수 있다.
따라서, 입력이 큰 경우에는 **제너레이터 식** 을 사용한다. 제너레이터 식은 리스트 컴프리헨션과 제너레이터를 일반화한 것이다.
제너레이터 식은 실행해도 출력 시퀀스 전체가 실체화되지 않는다.

```python
it = (len(x) for x in open('my_file.txt'))
```

다만 제너레이터가 반환하는 이터레이터에는 상태가 있기 때문에 이터레이터를 한 번만 사용해야 한다.

### 정리

- 입력이 크면 메모리를 너무 많이 사용하기 때문에 리스트 컴프리헨션은 문제를 일으킬 수 있다.
- 제너레이터 식은 이터레이터 처럼 한 번에 원소를 하나씩 출력하기 때문에 메모리 메모리 문제를 피할 수 있다.
- 서로 연결된 제너레이터 식은 매우 빠르게 실행되며 메모리도 효율적으로 사용한다.

## 33. yield from을 사용해 여러 제너레이터를 합성하라

애니메이션을 만들려면 move와 pause를 합성해서 변위 시퀀스를 하나만 만들어야 한다.
각 이터레이션에서 나오는 변위를 순서대로 내보내는 방식으로 다음과 같이 시퀀스를 만든다.

```python
def move(period, speed):
    for _ in range(period):
        yield speed


def pause(delay):
    for _ in range(delay):
        yield 0


def animate():
    for delta in move(4, 5.0):
        yield delta
    for delta in pause(3):
        yield delta
    for delta in move(2, 3.0):
        yield delta
```

이 코드의 문제점은 animate가 너무 반복적이라는 것이다. 이 문제의 해법은 yield from을 사용하는 것이다.

```python
def animate()
    yield from move(4, 5.0)
    yield from pause(3)
    yield from move(2, 3.0)

```

yield from은 근복적으로 파이썬 인터프리터가 for 루프를 내포시키고 yield 식을 처리하도록 만든다. 제너테이터를 합성한다면 가급적 yield from을 사용하는 것이 좋다.

### 정리

- yield from 식을 사용하면 여러 내장 제너레이터를 모아서 제너레이터 하나로 합성할 수 있다.
- 직접 내포된 제너레이터를 이터레이션하면서 각 제너레이터의 출력을 내보내는 것보다 yield from을 사용ㅎ는 것이 성능 면에서 더 좋다.

## 34. send로 제너레이터에 데이터를 주입하지 말라

yield 식을 사용해서 제너레이터 함수가 이터레이션이 가능한 출력 값을 만든다. 그러나, 이렇게 만들아네는 (데이터) 채널은 단방향이다. 제너레이터가 데이터를 내보내면서 다른 데이터를 받아들이려면 send 메서드를 사용하면 된다. 일반적으로 yield가 반환하는 값은 None 이다.

```python
def my_gen():
    recieved = yield 1
    print(recieved) # None

it = iter(my_gen())
print(next(it)) # 1

try:
    next(it)
except StopIteration:
    pass
```

하지만 for 루프나 next 내장 함수로 제너레이터를 이터레이션하지 않고 send 메서드를 호출하면, 제너레이터가 재개될 떄 yield가 send에 절달된 파라미터 값을 반환한다. 하지만, 방금 시작한 제너레이터는 아직 yield 식에 도달했지 못했기 때문에 최초로 send를 호출할 때 전달할 수 있는 값은 None뿐이다.

```python
it = iter(my_gen())
output = it.send(None)
print(output)

try:
    it.send("Hi")
except StopIteration:
    pass
```

이런 동작을 활용해 입력 시그널을 바탕으로 사인 파의 진폭을 변조할 수 있다.

```python
def wave_modulating(steps):
    step_size = 2 * math.pi / steps
    amplitude = yield # 초기 진폭을 받는다.
    for step in range(steps):
        radians = step * step_size
        fraction = math.sin(radians)
        output = amplitude * fraction
        amplitude = yield count # 다음 진폭을 받는다.

def run_modulating(it):
    amplitude = [None, 7, 7, 7, 2, 2, 2, 2, 10, 10, 10, 10, 10]
    for amplitude in aplitudes:
        output = it.send(amplitude)
        transmin(output)
```

첫 번째 식에 도달할 때까지는 amplitude 값을 받지 못한므로 None이다.
문제점은 코드를 처음봤을 때 이해하기 어렵다. wave 함수에 이터레이터를 전달하면 해결할 수 있다.

```python
def wave_cascading(amplitude_it, steps):
    step_size = 2 * math.pi / steps
    for step in range(steps):
        radians = step * step_size
        fraction = math.sin(radians)
        amplitude = next(amplitude_it)
        output = amplitude * fraction
        yield output


def complex_wave_cascading(amplitude_it):
    yield from wave_cascading(amplitude_it, 3)
    yield from wave_cascading(amplitude_it, 4)
    yield from wave_cascading(amplitude_it, 5)


def run_cascading():
    amplitudes = [7, 7, 7, 2, 2, 2, 2, 10, 10, 10, 10, 10]
    it = complex_wave_cascading(iter(amplitudes))
    for amplitude in amplitudes:
        output = next(it)
        tansmit(output)


run_cascading()
```

이 접근 방법에서 가장 멋진 부분은 아무 데서나 이터레이터를 가져올 수 있고, 이터레이터가 완전히 동적인 경우에도 잘 작동한다는 점이다. 다만 입력 제너레이터가 완전히 스레드 안전하다고 가정한다는 단점이 있다. 하지만 제너레이터가 항상 스레드 안전하지 않다. 스레드 경계를 넘나들면서 제너레이터를 사용해야 한다면 async 함수가 더 나은 해법일 수도 있다.

### 정리

- send메서드를 사용해 데이터를 제너레이터에 주입할 수 있다. 제너레이터는 send로 주입된 값을 yield 식이 반환하는 값을 통해 받으면, 이 값을 변수에 저장해 활용할 수 있다.
- send와 yield from 식을 함께 사용하면 제너레이터 출력에 None이 불뿍 나타나는 의외의 결과를 얻을 수 있다.
- 합성 제너레이터들의 입력으로 이터레이터를 전달하는 방식이 더 낫다.

## 35. 제너레이터 안에서 throw 상태를 변화시키지 말라

제너레이터의 고급 기능으로 yield from 식과 send 메서드 외에, 제너레이터 안에서 Exception을 다시 던질 수 있는 throw 메서드가 있다. throw가 작동하는 방식 간단하다. 어떤 제너레이터에 대해 throw가 호출되면 이 제너레이터는 값을 내놓은 yield로부터 평소처럼 제너레이터 실행을 계속하는 대신 throw가 제공한 Exception을 다시 던진다.

이 기능은 제너레이터와 제너레이터를 호출하는 쪽 사이에 양방향 통신 수단을 제공한다.

만약 재설정할 수 있는 타이머가 필요하다면

```python
class Reset(Exception):
    pass

def timer(period):
    current = period
    while current:
        current -= 1
        try
            yield current
        except Reset:
            current = period
```

Reset 예외가 발생할 때마다 카운터가 period가 재설정된다.

이터러블 컨테이너 객체를 사용해 상태가 있는 클로저를 정의할 수 있다.

```python
class Timer:
    def __init__(self, period):
        self.current = period
        self.period = period

    def reset(self):
        self.current = self.period

    def __iter__:
        while self.current:
            self.current -= 1
            yield self.current
```

따라서 예외적인 경우를 처리해야 한다면 throw를 전혀 사용하지 말고 이터러블 클래스를 사용하는 것이 좋다.

### 정리

- throw 메서드를 사용하면 제너레이터가 마지막으로 실행한 yield 식의 위치에서 예외를 다시 발생시킬 수 있다.
- throw를 사용하면 가독성이 나빠진다. 예외를 잡아내고 다시 발생시키는 데 준비 코드가 필요하며 내포 단계가 깊어지기 때문이다.
- 제너레이터에서 예외적인 동작을 제공하는 더 나은 방법은 \_\_iter\_\_ 메서드를 구현하는 클래스를 사용하는 것이다.

## 36. 이터레이터나 제너레이터를 다룰 때는 itertools를 사용하라

복잡한 이터레이션 코드를 작성하고 있다는 사실을 깨달을 때마다 혹시 쓸만한 기능이 없는지 itertools 문서를 다시 살펴본다.

```python
import itertools
```

### 여러 이터레이터 연결하기

**chain**
여러 이터레이터를 하나의 순차적인 이터레이터로 합치고 싶을 때 chain을 사용한다.

```python
it = itertools.chain([1, 2, 3], [4, 5, 6])
```

**repeat**
한 값을 반복해 내놓고 싶을 때 repeat을 사용한다. 두 번째 인자로 최대 횟수를 지정할 수 있다.

```python
it = itertools.repeat('안녕', 3)
```

**cycle**

어떤 이터레이터가 내놓은 원소들은 반복하고 앂을 때는 cycle을 사용한다.

```python
it = itertools.cycle([1, 2])
result = [next(it) for _ in range(10)] # [1, 2, 1, 2 ...]
```

**tee**
한 이터레이터를 병렬적으로 두 번째 인자로 지정딘 개수의 이터레이터로 만들고 싶을 때 tee를 사용한다.

```python
it1, it2, it3 = itertools.tee(['하나', '둘'], 3)
```

**zip_longest**
zip_longset는 zip 내장 함수의 변종으로, 여러 이터레이터 중 짧은 쪽 원소를 다 사용한 경우 fillvalue를 지정한 값을 채워 넣어준다. 없으면 None을 넣는다.

```python
keys = ['하나', '둘', '셋']
values = [1, 2]

normal = list(zip(keys, values))
print(normal) # [('하나', 1), ('둘', 2)]

it = itertools.zip_longest(keys, values, fillvalue='없음')
logest = list(it) # [('하나', 1), ('둘', 2), ('셋', '없음')]
```

### 이터레이터에서 원소 거르기

itertools 내장 모듈애는 원소를 필터링할 때 쓸수 있는 여러 함수가 있다.

**islice**
이터레이터를 복사하지 않으면서 원소 인덱스를 사용해 슬라이싱하고 싶을 때 islice를 사용한다.

```python
values = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

first_five = itertools.islice(values, 5)
print(first_five) # [1, 2, 3, 4, 5]

middle_odds = itertools.islice(values, 2, 8, 2)
print(middle_odds) # [3, 5, 7]
```

**takewhile**
takewhile은 이터레이터에서 주어진 술어가 False를 반환하는 첫 원소가 나타날 때까지 원소를 돌려준다.(술어가 True를 반환하는 동안만 원소를 리턴한다.)

```python
values = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
less_than_seven = lambda x: x < 7
it = itertools.takewhile(less_than_seven, values)
print(list(it)) # [1, 2, 3, 4, 5, 6]
```

**dropwhile**
dropwhile은 takewhile의 반대다. 즉, 이터레이터에서 주어진 술어가 False를 반환하는 첫 번째 원소를 찾을 때까지 이터레이터의 원소를 건너뛴다(술어가 True를 반환하는동안 원소를 건너뛴다).

```python
values = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
more_than_seven = lambda x: x < 7
it = itertools.dropwhile(less_than_seven, values)
print(list(it)) # [7, 8, 9, 10]
```

**filterfalse**
filter 내장 함수의 반대이다. 주어진 이터레이터의 술어가 False를 반환하는 모든 원소를 돌려준다.

```python
values = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
more_than_seven = lambda x: x % 2 == 0

it = filter(evens, values)
print(list(it)) # [2, 4, 6, 8, 10]

it_false = itertools.filterfalse(evens, values)
print(list(it_false)) # [1, 3, 5, 7, 9]
```

### 이터레이터에서 원소의 조합 만들어내기

itertools 내장 모듈에는 이터레이터가 돌려주는 원소들의 조합을 만들어내는 몇 가지 함수가 들어있다.

**accumulate**
accumulate는 파라미터를 두 개 받는 함수(이항 함수)를 반복 적용하면서 원소의 값을 하나로 줄여준다.

```python
values = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
sum_reduce = itertools.accumulate(values)
print(list(sum_reduce)) # [1, 3, 6, 10, 15, 21, 28, 36, 45, 55]]


def sum_modulo_20(first, second):
    output = first + second
    return output % 20


modulo_reduce = itertools.accumulate(values, sum_modulo_20)
print(list(modulo_reduce)) # [1, 3, 6, 10, 15, 1, 8, 16, 5, 15]
```

**product**
하나 이상의 이터레이터에 들어있는 아이템들을 데카르트 곱을 반환한다. 리스트 컴프리헨션을 깊이 내포시키는 대신 이 함수를 사용하면 편리하다

```python

single = itertools.product([1, 2], repeat=2)
print(list(single))  # [(1, 1), (1, 2), (2, 1), (2, 2)]

multiple = itertools.product([1, 2], ['a', 'b'])
print(list(multiple))  # (1, 'a'), (1, 'b'), (2, 'a'), (2, 'b')]
```

### 정리

- 이터레이터나 제너레이터를 다루는 itertools 함수는 세 가지 범주로 나눌 수 있다.
  1. 여러 이터레이터를 연결한다.
  2. 이터레이터 원소를 걸러낸다.
  3. 원소의 조합을 만들어낸다.
- 파이썬 인터프리터에서 help(itertools)를 입력한 후 표시되는 문서를 살펴보면 더 많은 고급 함수와 추가 파라미터를 알 수 있다.
