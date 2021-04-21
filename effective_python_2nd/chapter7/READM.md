# 7. 동시성과 병렬성

**동시성**
컴퓨터가 같은 시간에 여러 다른 작업을 처리하는 것처럼 보이는 것을 말한다.
동시성은 어떤 특정 유형의 문제를 해결하기 위한 도구로 사용된다. 동시 프로그램은 문제를 해결하는 과정이 동시에 독립적으로 시행되는 것처럼 보이게 한다.

**병렬성**
같은 시간에 여러 다른 작업을 실쩨로 처리하는 것을 말한다.

병렬성과 동시성의 가장 핵심적인 차이는 속도 향상에 있다. 어떤 프로그램의 서로 다른 두 실행 경로가 병렬적으로 진행되면 걸리는 시간이 반으로 줄어든다. 그러나 동시성 프로그램은 겉으로 볼 때는 병렬적으로 실행되는 것처럼 보이나, 전체 작업에 걸리는 시간은 빨라지지 않는다.

파이썬을 사용하면 다양한 스타일로 동시성 프로그램을 쉽게 작성할 수 있다. 스레드는 상대적으로 적은 양의 동시성을 제공하지만, 코루틴은 수많은 동시성 함수를 사용할 수 있게 해준다.

## 52. 자식 프로세스를 관리하기 위해 `subprocess`를 사용하라

파이썬이 시작한 자식 프로세스는 서로 병렬적으로 실행되기 떄문에 파이썬이 컴퓨터의 모든 CPU 코어를 사용할 수 있고, 그에 따라 프로그램의 스루풋을 최데로 높일 수 있다. 파이썬 자체는 한 CPU에 묶여 있지만, 파이썬을 사용해 CPU를 많이 상ㅇ하는 여러 부하를 조작하면서 서로 협력하게 조정하기는 쉽다.

```python
import subprocess

result = subprocess.run(["echo", "자식 프로세스가 보내는 인사!"],
                        capture_output=True,
                        encoding="utf-8")
result.check_returncode()
print(result)
print(result.stdout)
```

자식 프로세스는 부모 프로세스인 파이썬 인터프리터와 독립적으로 실행된다. run 함수 대신 Popen 클래스를 사용해 하위 프로세스를 만들면, 파이썬이 다른 일을 하면서 주기적으로 자식 프로세스의 상태르 검사할 수 있다.

```python
proc = subprocess.Popen(["sleep", "1"])
while proc.poll() is None:
    print("작업 중...")
    # 시간이 걸리는 작업
    ...

print("종료 상태", proc.poll())
```

자식 프로세스와 부모를 분리하면 부모 프로세스가 원하는 개수많큼 많은 자식 프로세스를 병렬로 실행할 수 있다.

```python
import time

start = time.time()
sleep_procs = []

for _ in range(10):
    proc = subprocess.Popen(["sleep", "1"])
    sleep_procs.append(proc)
for proc in sleep_procs:
    proc.communicate()
end = time.time()
delta = end - start
print(f"{delta:.3} 초 만에 끝남")
```

파이썬 프로그램의 데이터를 파이프를 사용해 하위 프로세스를 보내거나, 하위 프로세스의 출력을 받을 수 있다.
예를 들어 openssl 명렬줄 도구를 사용해 데이터를 암호화한다면 명렬줄 인자를 사용해 자식 프로세스를 시작하고 자식 프로세스와 I/O 파이프를 연결하는 것은 쉽다.

```python
import os


def run_encrypt(data):
    env = os.environ.copy()
    env["password"] = "0&dU%2ccu}9}pa/x|U%20H:dLrXTNR"
    proc = subprocess.Popen(
        ["openssl", "enc", "-des3", "-pass", "env:password"],
        env=env,
        stdin=subprocess.PIPE,
        stdout=subprocess.PIPE)
    proc.stdin.write(data)
    proc.stdin.flush()
    return proc

```

다음은 난수 바이트 문자열을 암호화 함수에 연결하지만, 실전에서는 파이프를 통해 사용자 입력, 파일 핸들, 네트워크 소켓 등에서 받은 데이터를 암호화 함수에 보내게 될 것이다.

```python
procs = []
for _ in range(3):
    data = os.urandom(10)
    proc = run_encrypt(data)
    procs.append(proc)

```

유닉스 파이프파인처럼 한 자식 프로세스의 출력을 다음 프로세스의 입력으로 연결시켜 여러 병렬 프로세스를 연쇄적으로 연결할 수도 있다.

`openssl` 명렬줄 도구를 하위 프로세스로 만들어서 입력 스트림의 월풀 해시를 계산한다.

```python
def run_hash(input_stdin):
    # dgst: 주어진 파일의 해쉬값 추출
    #
    return subprocess.Popen(
        ["openssl", "dgst", "-whirlpool", "-binary"],
        stdin=input_stdin,
        stdout=subprocess.PIPE)

encrypt_procs = []
hash_procs = []
for _ in range(3):
    data = os.urandom(100)

    encrypt_proc = run_encrypt(data)
    encrypt_procs.append(encrypt_proc)
    hash_proc = run_hash(encrypt_proc.stdout)
    hash_procs.append(hash_proc)

    # 자식이 입력 스트림에 들어오는 데이터를 소비하고 communicate() 메서드가
    # 불필요하게 자식으로부터 오는 입력을 훔쳐가지 못하게 한다.
    # 또 다운스트림 플세스가 죽으면 SIGPIPE를 업스르팀 프로세스에 전달한다.
    encrypt_proc.stdout.close()
    encrypt_proc.stdout = None

for proc in encrypt_procs:
    proc.communicate()
    assert proc.returncode == 0

for proc in hash_procs:
    out, _ = proc.communicate()
    print(out[-10:])
    assert proc.returncode == 0
```

자식 프로세스들이 시작되면 프로세스간 I/O가 자동으로 일어난다. 각 프로세스가 끝날 떄까지 기다려서 최종 결과를 얻고 출력한다.

### 정리

- `subprocess` 모듈을 사용해 자식 프로세스를 실행하고 입력과 출력 스트림을 관리할 수 있다.
- 자식 프로세서는 파이썬 인터프리터와 병렬로 실행되므로 CPU 코어를 최대로 쓸 수 있다.
- 간단하게 자식 프로세스를 실행하고 싶은 경우에는 `run` 편의 함수를 사용한다. 유닉스 스타일의 파이프라인이 필요하면 `Popen` 클레스를 사용한다.
- 자식 프로세스가 멈추는 경우나 교착 상태를 방지하려면 `communicate` 메서드에 대해 `timeout` 파라미터를 사용한다.

## 53. 블로킹 I/O의 경우 스레드를 사용하고 병렬성을 피하라

파이썬의 표준 구현을 CPython이라고 한다. CPython은 두 단계를 거쳐 파이썬 프로그램을 실행한다. 첫 번째 단계는 소스 코드 구문 분석해서 **바이트코드** 로 변환한다. 그 후 CPython은 바이트코드를 스택 기반 인터프리터를 통해 실행한다. CPython은 **전역 인터프러터 락(Global Interpreter Lock, GIL)** 이라는 방법을 사용해 일관서을 강제로 유지한다.

GIL은 상호 배제 락이며, CPython이 선점형 멀티스레드로 인해 영향을 받는 것을 방지한다. 선점형 멀티스레드에서는 한 스레드가 다른 스레드의 실행을 중간에 인터럽트시키고 제어를 가져올 수 있다. GIL은 CPython 자체와 CPython이 사용하는 C 확장 모듈이 실행되면서 인터럽트가 함부로 발생하는 것을 방지해, 인터프리터 상태가 제대로 유지되게 한다.

CPython에서도 다중 코어를 활용할 수 있는 방법이 있다. 하지만 표준 Thread 클래스를 사용하지 않으며, 코딩하는 데 상당한 노력이 필요하다. 이런 한계에도 불구하고 파이썬이 스레드를 지원하는 이유는

1. 다중 스레드를 사용하면 프로그램이 도시에 여러 일을 하는 것처럼 보이게 만들기 쉽다. 스레드를 사용하면 개발자가 작성한 함수를 파이썬으로 동시에 실행시킬 수 있다. 파이썬 GIL로 인해 스레드 중 하나만 앞으로 진행할 수 있음에도 불구하고, CPython이 어느 정도 균일하게 각 스레드를 실행시켜주므로 다중 스레드를 통해 여러 함수를 동시에 실행할 수 있다.

2. 블로킹 I/O를 다루기 위해서다. 블로킹I/O는 파이썬이 특정 시스템 콜을 사용할 때 일어난다. 파이썬 프로그램은 시스템 콜을 사용해 컴퓨터 운영체제가 자기 대신 외부 환경과 상호작용하도록 의뢰한다. 스레드를 사용함ㄴ 운영체제가 시스템 콜 요청에 응답하는데 걸리는 시간 동안 파이썬 프로그램이 다른일을 할 수 있다.

```python
import socket
import select


def slow_systemcall():
    select.select([socket.socket()], [], [], 0.1)

threads = []
for _ in range(5):
    thread = Thread(target=slow_systemcall)
    thread.start()
    threads.append(thread)

for thread in threads:
    thread.join()

```

병렬화한 버전은 순차적으로 실행한 경우보다 시간이 1/5로 줄어든다.

### 정리

- 파이썬 스레드는 GIL(전역 인터프리터 락)로 인해 다중 CPU 코어에서 병렬로 실행될 수 없다.
- GIL이 있음에도 불구하고 파이썬 스레드는 여전히 유용하다. 스레드를 사용하면 여러 일을 동시에 진행하는 프로그램을 쉽게 기술할 수 있다.
- 파이썬 스레드를 사용해 여러 시스템 콜을 병렬로 할 수 있다.

## 54. 스레드에서 데이터 경합을 피하기 위해 Lock을 사용하라

GIL이 동시 접근을 보장해주는 락 역할을 하는 것처럼 뵤여도 실제로는 전혀 그렇지 않다. 파이썬 스레드는 한번에 단 하나만 실행될 수 있지만, 파이썬 인터프리터에서 어떤 스레드가 데이터 구조에 대해 수행하는 연산은 연속된 두 바이트코드 사이에서 언제든지 인터럽트될 수 있다. 여러 스레드가 같은 데이터 구조에 동시에 접근하면 위험하다.

```python
class Counter:
    def __init__(self):
        self.count = 0

    def increment(self, offset):
        self.count += offset

def worker(sensor_index, how_many, counter):
    for _ in range(how_many):
        counter.increment(1)

how_many = 10**5
counter = Counter()

threads = []
for i in range(5):
    thread = Thread(target=worker,
                    args=(i, how_many, counter))
    threads.append(thread)
    thread.start()
for thread in threads:
    thread.join()

exptected = how_many * 5
found = counter.counter
print(found)
```

실행한 결과는 예상과 전혀 다르다. 파이썬 인터프리터는 실행되는 모든 스레드를 강제로 공평하게 취급해서 각 스레드의 실행 시간을 거의 비슷하게 만든다. 파이썬은 실행 중인 스레드를 일시 중지시키고 다른 스레드를 실행시키는 일을 반복한다.

객체 애트리뷰트에 대한 += 연산자는 실제로 세 가지 연산으로 이뤄진다.

```python
value = getattr(count, "count")
result = value + 1
setattr(counter, "count", result)
```

카운터를 증가시키는 파이썬 스레드는 세 연산 사이에서 일시 중단될 수 있다. 따라서 연산 순서가 뒤섞이면서 value의 이전 값을 카운터에 대입하는 일이 발생할 수 있다.
락을 사용하면 Counter 클래스가 여러 스레드를 동시 접근으로부터 자신의 현재 값을 보호할 수 있다.

```python
from threading import Lock

class LockingCounter:
    def __init__(self):
        self.lock = Lock()
        self.count = 0

    def increment(self, offset):
        with self.lock:
            self.count += offset
```

### 정리

- 파이썬에서는 GIL이 있지만, 파이썬 프로그램 코드는 여전히 여러 스레드 사이에서 일어나는 데이터 경합으로부터 자신을 보호해야한다.
- 코드에서 여러 스레드가 상호 배제 락 없이 같은 객체를 변경하도록 허용하면 코드가 데이터 구조를 오염시킨다.
- 여러 스레드 사이에서 프로그램의 불변 조건을 유지하려면 Lock클래스를 사용한다.

## 55. Queue를 사용해 스레드 사이에서 작업을 조율하라

동시성 작업을 처리할 때 가장 유용한 방식은 함수 파이프라인이다.

파이프라인은 순차적으로 실행해야 하는 여러 단계가 있고, 각 단계마다 실행할 구체적인 함수가 정해진다.작업은 매 단계 함수가 완료될 때마다 다음 단계로 전달되며, 더 이상 실행할 단계가 없을 때 끝난다.

`queue` 내장 모듈에 있는 `Queue`는 새로운 데이터가 나타날 때까지 `get` 메서드가 블록되게 만들어서 작업자의 바쁜 대기 문제를 해결한다.

큐에 입력 데이터가 들어오기를 기다리는 스레드를 하나 시작한다.

```python
from queue import Queue

my_queue = Queue()


def consumer():
    print('소비자 대기')
    my_queue.get()  # 다음에 보여줄 put()이 실행된 다음에 실행된다.
    print('소비자 완료')


thread = Thread(target=consumer)
thread.start()

print('생산자 데이터 추가')
my_queue.put(object())  # get()이 실행되기전에 실행된다.
print('생산자 완료')
thread.join()

```

파이프라인 중간에 막히는 경우를 해결하기 위해 `Queue` 클래스에서는 두 단계 사이에 허용할 수 있는 미완성 작업의 최대 개수를 지정할 수 있다. 버퍼의 크기를 정하면 큐가 이미 가득 찬 경우 `put`이 블록된다.

```python
from queue import Queue
from threading import Thread
import time

my_queue = Queue(1)


def consumer():
    time.sleep(0.1)  # 대기
    my_queue.get()  # 두 번째로 실행
    print('소비자 1')
    my_queue.get()  # 네 번째로 실행
    print('소비자 2')
    print('소비자 완료')


thread = Thread(target=consumer)
thread.start()

my_queue.put(object())  # 첫 번째로 실행
print('생산자 1')
my_queue.put(object())  # 세 번쨰로 실행
print('생산자 2')
print('생산자 완료')
thread.join()
```

`Queue` 클래스의 `task_done` 메서드를 통해 작업의 진행을 추적할 수 있다. 이 메서드를 사용하면 어떤 단게의 입력 큐가 다 소진될 때까지 기다릴 수 있고, 파이프라인의 마지막 단계를 폴링할 필요도 없어진다.

```python
in_queue = Queue()
def consumer():
    print('소비자 대기')
    work = in_queue.get() # 두 번째로 실행됨
    print('소비자 작업 중')
    ...
    print('소비자 완료')
    in_queue.task_done() # 세 번째로 실행됨

thread = Thread(target=consumer)
thread.start()
"""
생산자 코드가 소비자 스레드를 조인하거나 폴링할 필요가 없다. 생산자는 Queue 인스턴스의 join 메서드를 호출함으로써 in_queue가 끝나기를 기다릴 수 있다. in_queue가 비어 있더라도 지금까지 이 큐에 들어간 모든 원소에 대해 task_done이 호출되기 전까지는 join이 끝나지 않는다.
"""
print('생산자 데이터 추가')
in_queue.put(object()) # 첫 번째로 실행됨
print('생산자 대기')
in_queue.join() # 네 번째로 실행됨
print('생산자 완료')
thread.join()

```

이 모든 동작을 `Queue` 하위 클래스에 넣고, 처리가 끝났음을 작업자 스레드에 알리는 기능을 추가할 수 있다. 큐에 더 이상 다른 입력이 없음을 표시하는 특별한 **센티넬** 원소를 추가하는 `close` 메서드를 정의한다.

```python
class ClosableQueue(Queue):
    SENTINEL = object()


    def close(self):
        self.put(self.SENTINEL)

    def __iter__(self):
        while True:
            item = self.get()
            try:
                if item is self.SENTINEL:
                    return
                yield item
            finally:
                self.task_done()
```

작업자 스레드가 `ClosableQueue` 클래스의 동작을 활용하게 할 수 있다.

```python
class StoppableWorker(Thread):
    def __init__(self, func, in_queue, out_queue):
        self.func = func
        self.in_queue = in_queue
        self.out_queue = out_queue

    def run(self):
        for item in self.in_queue:
            result = self.func(item)
            self.out_queue.put(result)
```

새 작업자 클래스를 사용해 작업자 스레드를 새로 정의한다.

```python
download_queue = ClosableQueue()
resize_queue = ClosableQueue()
upload_queue = ClosableQueue()
done_queue = ClosableQueue()
thread = [
    StoppableWorker(download, download_queue, resize_queue),
    StoppableWorker(resize, resize_queue, upload_queue),
    StoppableWorker(uplaod, upload_queue, done_queue),
]

for thread in threads:
    thread.start()
for _ in range(1000):
    download_queue.put(object())

download_queue.close()
download_queue.join()
resize_queue.close()
resize_queue.join()
upload_queue.close()
upload_queue.join()
print(done_queue.qsize())

for thread in threads:
    thread.join()
```

### 정리

- 순차적인 작업을 동시에 여러 파이썬 스레드에서 실행되도록 조직하고 싶을 때, 특히 I/O 위주의 프로그램인 경우라면 파이프라인이 매우 유용하다.
- 동시성 파이프라인을 만들 때 발생할 수 있는 여러 가지 문제를 잘 알아준다.
- `Queue` 클래스는 튼튼한 파이프라인을 구축할 때 필요한 기능인 블로킹 연산, 버퍼 크기 지정, `join`을 통한 완료 대기를 모두 제공한다.

## 56. 언제 동시성이 필요할지 인식하는 방법을 알아두라

프로그램이 다루는 영역이 커짐에 따라 복잡도도 증가한다. 프로그램의 명확성, 테스트 가능성, 효율성을 유지하면서 늘어나는 요구 조건을 만족시키는 것은 프로그래밍에서 가장 어려운 부분이다. 가장 처리하기 어려운 일은 단일 스레드 프로그래밍을 동시 실행되는 여러 흐름으로 이뤄진 프로그램으로 바꾸는 경우일 것이다.

만약 기존의 프로그램에서 요구 사항이 바뀌어서 I/O(소켓 통신 등)가 필요하다면, I/O를 병렬로 수행해서 해결할 수 있다.
각 작업 단위에 대해 동시 실행되는 여러 실행 흐름을 만들어내는 과정을 **팬아웃** 이라고 한다. 전체를 조율하는 프로세스 안에서 다음 단계로 진행하기 전에 동시 작업 단위의 작업이 모두 끝날 때까지 기다리는 과정을 **팬인** 이라고 한다.

파이썬은 팬아웃과 팬인을 지원하는 여러 가지 내장 도구를 제공한다.

### 정리

- 프로그램이 커지면서 범위와 복잡도가 증가함에 따라 동시에 실행되는 여러 실행 흐름이 필요해지는 경우가 많다.
- 동시성을 조율하는 가장 일반적인 방법으로는 팬아웃(새로운 동시성 단위들을 만들어냄)과 팬인(기존 동시성 단위들의 실행이 끝나기를 기다림)이 있다.
- 파이썬은 팬아웃과 팬인을 구현하는 다양한 방법을 제공한다.

## 57. 요구에 따라 팬아웃을 진행하려면 새로운 스레드를 생성하지 말라.

병렬 I/O를 실행하고 싶을 때는 자연스레 스레드를 가장 먼저 고려하게 된다. 그러나, 여러 동시 실행흐름을 만들어내는 팬아웃을 수행하고자 스레드를 사용할 경우 중요한 단점과 마주하게 된다.

여기서는 `game_logic` 안에서 I/O를 수행함으로써 생기는 지연시간을 해결한다.

```python
from threading import Lock


ALIVE = '*'
EMPTY = '-'


class Grid:
    ...


class LockingGrid(Grid):
    def __init__(self, height, width):
        super().__init__(height, width)
        self.lock = Lock()

    def __str__(self):
        with self.lock:
            return super().__str__()

    def get(self, y, x):
        with self.lock:
            return super().__get__(y, x)

    def set(self, y, x, state):
        with self.lock:
            return super().set(y, x, state)
```

각 `step_cell` 호출마다 스레드를 정의해 **팬아웃** 하도록 `simulate` 함수를 정의한다. 스레드를 병렬로 실행되며, 다른 I/O가 끝날 때까지 기다리지 않아도 된다. 다음 세대로 진행하기 전에 모든 스레드가 작업을 마칠 때까지 기다리므로 **팬인** 할 수 있다.

```python
from threading import Thread


def count_neighbors(y, x, get):
    ...

def game_logic(state, neighbors):
    ...
    # Blocking I/O
    data = my_socket.recv(100)
    ...

def step_cell(y, x, get, set):
    state = get(y, x)
    neighbors = count_neighbors(y, x, get)
    next_state = game_logic(state, neighbors)
    set(y, x, next_state)

def simulate_threaded(grid):
    next_grid = LockingGrid(grid.height, grid.width)

    threads = []
    for y in range(grid.height):
        for x in range(grid.width):
            args = (y, x, grid.get, next_grid.set)
            thread  = Thread(target=step_cell), args=args)
            thread.start()
            threads.append(thread)

    for thread in threads:
        thread.join()

    return next_grad


```

스레드 사이에 I/O가 병렬화됐다. 하지만 이 코드에는 세 가지 문제점이 있다.

1. `Thread` 인스턴스를 서로 안전하게 조율하려면 특별한 도구가 필요하다. 이로 인해 순차적인 단일 스레드보다 스레드를 사용하는 코드가 읽기 어렵다. 복잡도 때문에 시간이 지남에 따라 스레드를 사용한 코드를 확장하고 유지 보수하기도 더 어렵다.
2. 스레드는 메모리를 많이 사용하며, 스레드 하나당 약 8MB가 더 필요하다.
3. 스레드를 시작하는 비용이 비싸며, 컨텍스트 전환에 비용이 들기 때문에 스레드는 성능에 부정적인 영향을 미친다.

지속적으로 새로운 동시성 함수를 시작하고 끝내야 하는 경우 스레드는 적절한 해법이 아니다. 파이썬은 이런 경우에 더 적합한 다른 해법을 제공한다.

### 정리

- 스레드는 많은 단점이 있다. 스레드를 시작하고 실행하는 데 비용이 들기 때문에 스레드가 많이 필요하면 상당히 많은 메모리를 사용한다. 그리고 스레드 사이를 조율하기 위해 `Lock` 과 같은 특별한 도구를 사용해야 한다.
- 스레드를 시작하거나 스레드가 종료하기를 기다리는 코드에게 스레드 실행중에 발행한 예외를 돌려주는 파이썬 내장 기능은 없다. 이로 인해 스레드 디버깅이 어렵다.

## 58. 동시성과 `Queue`를 사용하기 위해 코드를 어떻게 리팩터링 하는지 이해하라.

`queue` 내장 모듈의 `Queue` 클래스를 사용해 파이프라인을 스레드로 실행하게 구현하는 것이다. 일반적인 접근 방법은 필요한 병렬 I/O의 숫자에 맞춰 미리 정해진 작업자 스레드를 만든다. 프로그램은 이를 통해 자원 사용을 제어하고, 새로운 스레드를 자주 시작하면서 생기는 부가 비용을 덜 수 있다.

```python
from queue import Queue

class ClosableQueue(Queue):
    ...

in_queue = ClosableQueue()
out_queue = ClosableQueue()
```

`in_queue`에서 원소를 소비하는 스레드를 여러 개 시작할 수 있다. 각 스레드는 `game_logic`을 호출해 원소를 처리한 다음 `out_queue` 에 결과를 넣는다. 각 스레드는 동시에 실행되며 병력적으로 I/O를 수행하므로, 필요한 지연 시간이 줄어든다.

```python
from threading import Thread


class StoppableWorker(Thread):
    ...

def game_logic(state, neighbors):
    ...
    # Blocking I/O
    data = my_socket.recv(100)
    ...

def game_logic_thread(item):
    y, x, state, neighbors = item
    try:
        next_state = game_logic(state, neighbors)
    except Exception as e:
        next_state = e
    return (y, x, next_state)

# 스레드를 미리 시작한다.

threads = []

for _ in range(5):
    thread = StoppableWorker(
        game_logic_thread, in_queue, out_queue)
    thread.start()
    threads.append(thread)
```

이제 큐와 상호작용하면서 상태 전이 정보를 요청하고 응답을 받도록 `simulate` 함수를 재정의할 수 있다. 원소를 `in_queue`에 추가하는 과정은 **팬아웃** 이고, `out_queue` 가 빈 큐가 될 때까지 원소를 소비하는 과정은 **팬인** 이다.

```python
ALIVE = '*'
EMPTY = '-'


class SimulationError(Exception):
    pass


class Grid:
    ...

def count_neighbors(y, x, get):
    ..

def simulate_pipeline(grid, in_queue, out_queue):
    for y in range(grid.height):
        for x in range(grid.width):
            state = grid.get(y, x)
            neighbors = count_neighbors(y, x, grid.get)
            in_queue.put((y, x, state, neighbors))
    in_queue.join()
    out_queue.close()
    next_grid = Grid(grid.height, grid.width)
    for item in out_queue:
        y, x, next_state = item
        if isinstance(next_state, Exception):
            raise SimulationError(y, x) from next_state
        next_grid.set(y, x, next_state)

    return next_grid
```

`Grid.get`과 `Grid.set` 호출은 모두 새로운 `simulate_pipline` 함수 안에서만 일어난다. 이는 동기화를 위해 Lock인스턴스를 사용하는 `Grid` 구현을 새로 만드는 대신에 기존의 단일 스레드 구현을 쓸 수 있다는 뜻이다.

메모리를 폭팔적으로 사용하는 문제, (스레드) 시작 비용, 스레드를 디버깅하는 문제 등은 해결했지만, 여전히 많은 문제가 남아있다.

- `simulate_pipeline` 함수가 따라가기 어렵다.
- 코드의 가독성을 개선하려면 `ClosableQueue`와 `StoppableWorker` 라는 추가 지원 클래스가 필요하며, 복잡도가 늘어난다.
- 병렬성을 활용해 필요에 따라 자동으로 시스템 규모가 확장되지 않는다. 미리 부하는 예측해서 잠재적인 병렬성 수준을 미리 지정해야 한다.
- 디버깅을 활성화하려면 발생한 예외를 작업 스레드에서 수동으로 잡아 `Queue` 를 통해 전달함으로써 주 스레드에서 다시 발생사켜줘야 한다.

하지만 가장 큰 문제점은 요구 사항이 변경될 떄 드러난다. `count_neighbors` 함수에서도 I/O를 수행해야 한다고 가정한다면,

```python
def count_neighbors(y, x, get):
    ...
    #Blocking I/O
    data = my_socket.recv(100)
    ...

```

이 코드를 병렬화하려면 `count_neighbors`를 별도의 스레드에서 실행하는 단계를 파이프라인에 추가해야 한다. 작업자 스레드 사이의 동기화를 위해 `Grid` 클래스에 대해 `Lock`을 사용해야 한다.

```python
def count_neighbors_thread(item):
    y, x, sate, get = item
    try:
        neighbors = count_neighbors(y, x, get)
    except Exception as e:
        neighbors = e
    return (y, x, state, neighbors)


def game_logic_thread(item):
    y, x, state, neighbors = item
    if isinstance(neighbors, Exception):
        next_state = neighbors
    else:
        try:
            next_state = game_logic(state, neighbors)
        except Exception as e:
            next_state = e
    return (y, x, next_state)


class LockingGrid(Grid):
    ...
```

`count_neighbors_thread` 작업자와 그에 해당하는 `Thread` 인스턴스를 위해 또 다른 `Queue` 인스턴스 집합을 만들어야 한다.

```python
in_queue = ClosableQueue()
logic_queue = ClosableQueue()
out_queue = ClosableQueue()

threads = []
for _ in range(5):
    thread = StoppableWorker(
        count_neighbors_thread, in_queue, logic_queue)
    thread.start()
    threads.append(thread)


for _ in range(5):
    thread = StoppableWorker(
        game_logic_thread, logic_queue, out_queue)
    thread.start()
    threads.append(thread)

def simulate_phased_pipeline(
    grid, in_queue, logic_queue, out_queue):
    for y in range(grid.height):
        for x in range(grid.width):
            state = grid.get(y, x)
            item = (y, x, state, grid.get)
            in_queue.put(item) # 팬 아웃

    in_queue.join()
    logic_queue.join()
    out_queue.close()

    next_grid = LockingGrid(grid.height, grid.width)
    for item in out_queue:
        y, x, next_state = item
        if isinstance(next_state, Exception):
            raise SimulationError(y, x) from next_state
        next_grid.set(y, x, next_state)

    return next_grid
```

### 정리

- 작업자 스레드 수를 고정하고 `Queue`와 함께 사용하면 스레드를 사용할 때 팬인과 팬아웃의 규모 확장성을 개선할 수 있다.
- `Queue`를 사용하도록 기존 코드를 리팩터링하려면 상당히 많은 작업이 필요하다. 특히 다단계로 이뤄진 파이프라인이 필요하면 작업량이 더 늘어난다.
- 다른 파이썬 내장 기능이나 모듈이 제공하는 병렬 I/O를 가능하게 해주는 다른 기능과 비교하면, `Queue` 라는 프로그램이 활용할 수 있는 전체 I/O 병렬성의 정도를 제한한다는 단점이 있다.

## 59. 동시성을 위해 스레드가 필요한 경우에는 ThreadPoolExecuter를 사용하라

`Grid`의 각 셀에 대해 새 `Thread` 인스턴스를 시작하는 대신, 함수를 실행기(executor)에 제출함으로써 **팬아웃** 할 수 있다. 실행기는 제출받은 함수를 별도의 스레드에서 수행해준다.

실행기는 사용할 스레드를 미리 할당한다. 따라서 `simulate_pool` 을 실행할 때마다 스레드를 시작하는 데 필요한 비용이 들지 않는다. 또한, 스레드 풀에 사용할 스레드의 최대 개수를 지정할 수도 있다.

```python
from concurrent.futures import ThreadPoolExecutor


def simulate_pool(pool, grid):
    next_grid = LockingGrid(grid.height, grid.width)
    futures = []
    for y in range(grid.height):
        for x in range(grid.width):
            args = (y, x, grid.get, next_grid.set)
            future = pool.submit(step_cell, *args) # 팬아웃
            futures.append(future)

    for future in futures:
        future.result() # 팬인

    return next_grid

with ThreadPoolExecutor(max_workers=10) as pool:
    for i in range(5):
        grid = simulate_pool(pool, grid)
```

`ThreadPoolExecutor` 클래스에서 가장 좋은 점은 submit 메서드가 반환하는 `Future` 인스턴스에 대해 `result` 메서드를 호출하면 스레드를 실행하는 중에 발생한 예외를 자동으로 전파해준다. 하지만, `ThreadPoolExecutor` 가 제한된 수의 I/O 병렬성만 제공한다는 큰 문제점이 남아 있다. 비동기적인 해법이 존재하지 않는 상황을 처리할 때는 좋은 방법이다.

### 정리

- `ThreadPoolExecutor` 를 사용하면 한정된 리팩터링만으로 간단한 I/O 병렬성을 활성화할 수 있다. 동시성을 팬아웃해야 하는 경우에 발생하는 스레드 시작 비용을 쉽게 줄일 수 있다.
- `ThreadPoolExecutor` 를 사용하면 스레드를 직접 사용할 때 발생할 수 있는 잠재적인 메모리 낭비 문제를 없애주지만 `max_worker`의 개수를 미리 지정해야 하므로 I/O 병렬성을 제한한다.

## 60. I/O를 할 때는 코루틴을 사용해 동시성을 높여라

파이썬은 높은 I/O 동시성을 처리하기 위해 **코루틴** 을 사용한다. 코루틴을 사용하면 파이썬 프로그램 안에서 동시에 실행되는 것처럼 보이는 함수를 아주 많이 쓸 수 있다. 코루틴은 async 와 await 키워드를 사용해 구현되며, 제너레이터를 실행하기 위한 인프라를 사용한다.

코루틴을 시작하는 비용은 함수 호출뿐이다. 활성화된 코루틴은 종료될 떄까지 1KB 미만의 메모리를 사용한다. 스레드와 마찬가지로 코루틴도 환경으로부터 입력을 소비하고 결과를 출력할 수 있는 독립적인 함수다.
코루틴은 매 await 식에서 일시 중단되고 일시 중단된 **대기 가능성** 이 해결된 다음에 async 함수로부터 실행을 재개한다는 차이점이 있다. 여러 분리된 async 함수가 서로 장단을 맞춰 실행되면 마치 모든 async 함수가 동시에 실행되는 것처럼 보아며, 이를 통해 파이썬 스레드의 동시성 동작을 흉내낼 수 있다.
부가 비용, 시작 비용, 컨텍스트 전환 비용이 들지 않고 복잡한 락과 동기화 코드가 필요하지 않다.

코루틴을 가능하게 하는 매커니즘은 **이벤트 루프** 로 다수의 I/O를 효율적으로 동시에 실행할 수 있다.

코루틴을 사용해 생명 게임을 구현한다.

```python
ALIVE = '*'
EMPTY = '-'

class Grid:
    ...

def count_neighbors(y, x, get):
    ...

async def game_logic(state, neighbors):
    ...
    # 여기서 I/O를 수행
    data = await my_socket.read(50)
    ...

async def step_cell(y, x, get, set):
    state = get(y, x)
    neighbors = count_neighbors(y, x, get)
    next_state = await game_logic(state, neighbors)
    set(y, x, next_state)

import asyncio

async def simulate(grid):
    next_grid = Grid(grid.height, grid.width)

    tasks = []

    for y in range(grid.height):
        for x in range(grid.width):
            task = step_cell(
                y, x, grid.get, next_grid.set) # 한번에 처리하기 위해 await을 안씀
            tasks.append(task)

    await asyncio.gather(*task) # 여기서 사용

"""
- step_cell을 호출해도 이 함수가 즉시 호출하지 않고 await 식에서 사용될 수 있는 coroutine 인스턴스를 반환 (마치 yield를 사용하는 제너레이터 함수를 호출하면 즉시 실행되지 않고 제너레이터를 반환하는 것과 비슷) 이와같은 실행 연기 메커니즘이 팬아웃을 수행
- asyncio 내장 라이브러리가 제공하는 gather gkatnsms 팬인을 수행 gather에 대해 적용한 await식은 이벤트 루프가 step_cell 코루틴을 동시에 실행하면서 완료될 떄마다 simulate 코루틴 실행을 재개하라고 요청
- 모든 실행이 단일 스레드에서 이뤄지므로 Grid 인스턴스 락을 사용할 필요가 없다. I/O는 asyncio가 제공하는 이벤트 루프의 일부분으로 병렬화된다.
"""
```

새로운 코드는 asyncio.run 함수를 사용해 simulate 코루틴을 이벤트 루프상에서 실행하고 각 함수가 의존하는 I/O를 수행한다.

```python
for i in range(5):
    grid = asyincio.run(simulate(grid))

```

### 정리

- `async` 키워드로 정의한 함수를 코루틴이라고 부른다. 코루틴을 호출하는 호출자는 `await` 키워드를 사용해 자신이 의존하는 코루틴의 결과를 받을 수 있다.
- 코루틴은 수만 개의 함수가 동시에 실행되는 것처럼 보이게 만드는 효과적인 방법을 제공한다.
- I/O를 병렬화하면서 스레드로 I/O를 수행할 때 발생할 수 있는 문제를 극복하기 위해 팬인과 팬아웃에 코루틴을 사용할 수 있다.

## 61. 스레드를 사용한 I/O를 어떻게 `asyncio`로 포팅할 수 있는지 알아두라

### 정리

- 파이썬은 `for` 루프, `with` 문, 제너레이터, 컴프리헨션의 비동기 버전을 제공하고, 코루틴안에서 기존 라이브러리 도우미 함수를 대신해 즉시 사용할 수 있는 대안을 제공한다.
- `asyncio` 내장 모듈을 사용하면 스레드와 블로킹 I/O를 사용하는 기존 코드를 코루틴과 비동기 I/O를 사용하는 코드로 쉽게 포팅할 수 있다.

## 62. `asyncio`로 쉽게 옮겨갈 수 있도록 스레드와 코루틴을 함꼐 사용하라

코드베이스를 점진적으로 마이그레이션하면서 필요에 따라 테스트를 함께 갱신하며, 각 단계에서 모든 기능이 제대로 작동하는지 확인해야 한다. 이런 마이그레이션을 가능하려면, 코드베이스에서 블로킹 I/O에 스레드를 사용하는 부분과 비동기 I/O에 코루틴을 사용하는 부분이 서로 호환되면서 공존할 수 있어야한다.

스레드 기반 구현으로부터 코드를 점진적으로 `asyncio` 와 코루틴 기반으로 바꾸는 방법에는 하향식(Top-Down)과 상향식(Bottom-Up)이라는 두가지 방법이 있다.

하향식이란 `main` 진입점처럼 코드베이스에서 가장 높은 구성 요소로부터 시작해 점차 호출 계층의 잎 부분에 위치한 개별 함수와 클래스로 내려가면서 작업한다.

구체적인 단계는

1. 최상위 함수가 `def` 대신 `async` `def` 를 사용하게 변경한다.
2. 최상위 함수가 I/O를 호출하는 모든 부분을 `asyncio.run_in_executor` 로 감싸라
3. `run_in_executor` 호출이 사용하는 자원이나 콜백이 제대로 동기화 됐는지 확인하란다.
4. 호출 계층의 잎쪽으로 내려가면서 중간에 있는 함수와 메서드를 코루틴으로 변환하고 `get_event_loop` 와 `run_in_executor` 호출을 없애려고 시도한다.

상향식 접근 방법도 하향식 접근 방법과 비슷한 4단계로 이뤄지지만, 변환 과정에서 호출 계층을 반대 방향으로 옮겨간다.

구체적인 단계는

1. 프로그램에서 잎 부분에 있는, 포팅하려는 함수의 비동기 코루틴 버전을 새로 만든다.
2. 기존 동기 함수를 변경해서 코루틴 버전을 호출하고 실제 동작을 구현하는 대신 이벤트 루프를 실행하게 한다.
3. 호출 계층을 한 단계올려서 다른 코루틴 계층을 만들고, 기존에 동기적 함수를 호출하던 부분을 1단계에서 정의한 코루틴 호출로 바꾼다.
4. 이제 비동기 부분을 결합하기 위해 2단계에서 만든 동기적인 래퍼를 삭제한다.

### 정리

- `asyncio` 이벤트 루프의 `run_in_executor` 를 사용하면 코루틴이 `ThreadPoolExecutor` 스레드 풀을 사용해 동기적인 함수를 호출할 수 있다. 이 기능을 활용하면 코드를 하향식으로 `asyncio` 로 마이그레이션할 수 있다.
- `asyncio` 이벤트 루프의 `run_util_complete` 메서드를 사용하면 동기적인 코드가 코루틴을 호출하고 완료를 기다릴 수 있다. `asyncio.run_coroutine_threadsafe` 도 같은 기능을 제공하지만 스레드 경계에서도 안전하게 작동한다. 이 두 메서드를 활용하면 코드를 상향식으로 `asyncio` 로 마이그레이션할 때 도움이 된다.

## 63. 응답성을 최대로 높이려면 asyncio 이벤트 루프를 블록하지 말라

출력 파일 핸들에 대한 `open`, `close`, `write` 호출이 주 이벤트 루프에서 이루어지는 것은 큰 문제점이다. 프로그램을 실행하는 운영체제의 시스템 콜을 사용해야 하기 때문에 이벤트 루프를 상당히 오랫동안 블록할 수 있으므로 다른 코루틴이 진행하지 못하게 된다. 이로 인해 전체 응답성이 나빠지고, 특히 동시성이 아주 높은 서버에서는 응답 시간이 늘어날 수 있다.

문제가 발생하는지 감시하고 싶으면 `debug=True` 라는 파라미터를 `asyncio.run` 함수에 넘기면 된다.

```python
import asyncio
import time


async def slow_coroutine():
    time.sleep(0.5)


asyncio.run(slow_coroutine(), debug=True)
"""
executing <Task finished name='Task-1' coro=<slow_coroutine()
done, defined at .../ex.py:5> result=None created
at .../asyncio/base_events.py:595> took 0.503 seconds
"""
```

응답성을 최대로 높이려면 이벤트 루프 안에서 시스템 콜이 이뤄질 잠재적인 가능성을 최소화해야 한다.

### 정리

- 시스템 콜(블로킹 I/O와 스레드 시작도 포함)을 코루틴으로 만들면 프로그램의 응답성이 좋아지고 사용자가 느끼는 지연 시간을 줄일 수 있다.
- `debug=True` 파라미터를 `asyncio.run` 에 넘기면 이벤트 루프가 빨리 반응하지 못하게 방해하는 코루틴을 식별할 수 있다.

## 64. 진정한 병렬성을 살리려면 concurrent.future 를 사용하라

파이썬 전역 인터프리터 락(GIL)으로 인해 파이썬 스레드는 진정한 병렬 실행이 불가능하다. 일반적인 조언은 성능에 가장 결정적인 영향을 미치는 부분을 C 언어를 사용한 확장 모듈로 작성하라는 것이다. 파이썬 보다 빠르개 실행되고, GIL에 대해 신경 쓰지 않고 여러 CPU 코어를 활용할 수 있도록 네이티브 스레드를 시작할 수도 있다. C 확장 API는 문서화가 잘돼 있고, 다중 코어를 활용해야 할 때 마지막 방법이 될 수 있다.

문제는 한 부분만 C로 바꾸면 되는 경우가 드물다는 점이다. 파이썬 프로그럄을 최적화할 때 속도 저하의 원인이 코드의 어느 한 곳에만 있는 경우는 드물다.

`concurrent.futures` 내장 모듈을 통해 쉽게 사용할 수 있는 `multiprocessing` 내장 모듈이 있다. 이 모듈을 사용하면 **자식 프로세서로 다른 파이썬 인터프리터를 실행함으로써** 파이썬에서 여러 CPU 코어를 활용할 수 있다. 자식 프로세스의 GIL이 주 인터프리터의 GIL과 분리된다. 각 자식 프록세스는 한 CPU 코어를 완전히 사용할 수 있다.

```python
from concurrent.futures import ProcessPoolExecutor


def gcd(pair):
    a, b = pair
    low = min(a, b)
    for i in range(low, 0, -1):
        if a % i == 0 and b % i == 0:
            return i
    assert False, '도달할 수 없음'


def main():
    pool = ProcessPoolExecutor(max_workers=2)
    results = list(pool.map(gcd, NUMBERS))


NUBMERS = [...]

if __name__ == '__main__':
    main()

```

### `ProcessPoolExecutor` 클래스가 (`multiprocessing` 모듈이 제공하는 저수준 요소를 활용해) 실제로 하는 일

1. (부모) 이 객체(`ProcessPoolExecutor` 인스턴스)는 입력 데이터로 들어온 `map` 메서드에 전달된 `NUMBERS`의 각 원소를 취한다.
2. (부모) 이 객체는 1번에서 얻은 원소를 `pickle` 모듈을 사용해 이진 데이터로 직렬화한다.
3. (부모, 자식) 이 객체는 로컬 소켓을 통해 주 인터프리터 프로세스로부터 자식 인터프리터 프로세스에게 2번에서 직렬화한 데이터를 복사한다.
4. (자식) 이 객체는 `pickle` 을 사용해 데이터를 파이썬 객체로 역직렬화한다.
5. (자식) 이 객체는 `gcd` 함수를 실행한다. 이때 다른 자식 인터프리터 프로세스와 병렬로 실행한다.
6. (자식) 이 캑체는 `gcd` 함수의 결과를 이진 데이터로 직렬화한다.
7. (부모, 자식) 이 객체는 로컬 소켓을 통해 자식 인터프리터 프로세스로부터 부모 인터프리터 프로세스에게 6번에서 직렬화한 결과 데이터를 돌려준다.
8. (부모) 이 객체는 데이터를 파이썬 객체로 역직렬화한다.
9. (부모) 여러 자식 프로세스가 돌려준 결과를 병합해서 한 `list`로 만든다.

부모와 자식 프로세스 사이에 데이터가 오고 갈 때마다 항상 직렬화와 역직렬화가 일어나야 하므로 추가 비용이 매우 크다.

서로 잘 격리되고 레버리지가 큰 유형의 작업에는 좋다.

> 격리란? 프로그램의 다른 부분과 상태를 공유할 필요가 없는 함수를 실행한다는 뜻
> 레버리지란? 부모와 자식 사이에 주고받아야 하는 데이터 크는 작지만, 이 데이터로 인해 자식 프로세스가 계산해야 하는 연산의 양을 말한다.

### 정리

- CPU 병복 지점을 C 확장 모듈로 옮기면 파이썬에 투자한 비용을 최대한 유지하면서 프로그램 성능을 개선에 효과적일 수 있다. 하지만 C 확장 모듈로 옮기려면 많은 비용이 들고 포팅하는 과정에서 버그가 생겨날 수도 있다.
- `multiprocessing` 모듈을 사용하면 특정 유형의 파이썬 게산을 최소의 노력으로 병렬화할 수 있다.
- `concurrent.futures` 내장 모듈이 재공하는 간단한 `ProcessPoolExecutor` 클래스를 호라용하면 `multiprocessing`의 능력을 최대한 활용할 수 있다.
- 사용할 수 있는 모든 방법을 다 써보기 전에는 `multiprocessing` 이 제공하는 (복잡한) 고급기능을 사용하지 않는 것이 좋다.
