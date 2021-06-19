# Singleton

## 개념

싱글톤은 클래스에 오직 하나의 인스턴스만 존재하도록 하며 인스턴스에 대헤 전역으로 액세스할 수 있도록 제공하는 디자인 패턴이다.

## 문제

싱글톤 패턴은 'Single Responsibility Principle'을 위배하여 동시에 두 개의 문제를 해결해야한다.

> Single Responsibility Principle: 모든 클래스는 하나의 책임을 가지며, 클래스는 그 책임을 완전히 캡술화해야 함을 일컫는다. 클래스가 제공하는 모든 기능은 이 책임과 주의 깊게 부합해야 한다.(https://en.wikipedia.org/wiki/Single-responsibility_principle)

1. **클래스가 오직 하나의 인스턴스만 있어야 한다.** 대부분에 데이터베이스같은 공유되는 리소스를 액세스 하기 위해 인스턴스 생성을 제어한다.
   - 공유 자원에 접근하는 것을 제어하기 위해 하나의 인스턴스만 존재하도록 한다.
   - 인스턴스를 생성한 후, 다시 생성하려고 한다면 새로운 인스턴스를 반환하는 것이 아니라 이미 존재하는 인스턴스를 반환한다.
   - 생성자 호출은 새 객체를 반환하므로 일반적인 생성자 호출로는 구현할 수 없다.

![img](https://refactoring.guru/images/patterns/content/singleton/singleton-comic-1-en-2x.png)

2. **전역으로 인스턴스에 접근하도록 제공한다.** 이는 편리하지만, 모든 코드에서 이 인스턴스를 애트리뷰트들을 변경할 수 있고 이로 인해 다운될 수도 있다.
   - 전역 변수같지만, 싱글톤 패턴은 프로그램에 모든 곳에서 접근할 수 있지만, 다른 코드로 부터 변경되는 것을 막는다.

## 해결 방법

모든 싱글톤 페턴 구현은 일반적으로 두 단계를 거친다.

1. 다른 코드가 `new`호출을 막을 수 있도록, 생성자를 private 하게 만든다.
2. 대신 생성자 역할을 하는 static method를 구현한다. 이 메서드 call 안에서 생성자로 인스턴스를 생성하고 static 변수에 저장한다.

만약 인스턴스를 이미 생성했을 경우 static method를 호출하게 되면, static 변수(cached)에 있는 인스턴스를 반환한디.

## 구조

![img](https://refactoring.guru/images/patterns/diagrams/singleton/structure-en-2x.png)

## 코드

```python
class SingletonMeta(type):
    _instances = {}

    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            cls._instances[cls] = super().__call__(*args, **kwargs)
        return cls._instances[cls]


class Singleton(metaclass=SingletonMeta):
    def foo(self):
        print('foo')


if __name__ == '__main__':
    singleton_1 = Singleton()
    singleton_2 = Singleton()

    assert id(singleton_1) == id(singleton_2)
```
