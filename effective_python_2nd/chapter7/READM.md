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
