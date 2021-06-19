# 데코레이터

## 개념

**데코레이터** 는 구조적 디자인 패턴이다. 특정 컴포넌트를 특수한 래퍼 객체 안에 배치하여 새로운 동작을 할 수 있도록 한다.
![img](https://refactoring.guru/images/patterns/content/decorator/decorator-2x.png)

## 문제 발견

중요한 이벤트를 사용자에게 알려주는 알림 라이브러리를 개발한다고 하자.

라이브러리의 첫 버전은 `Notifier` 클래스를 기반으로 한다. 메서드는 클라이언트로부터 메세지를 주고 받을 수 있다. 클라이언트 역할을 하는 서드-파티 애플리케이션은 `Notitier` 객체를 한번 생성하고 구성해야한다. 그리고 중요한 이벤트가 생겼을 때 각 시간에 사용하도록 되어있다.
![img](https://refactoring.guru/images/patterns/diagrams/decorator/problem1-en-2x.png)

시간이 지나 라이브러리 유저들은 이메일 알림보다 다른 종류의 알림을 원한다. 많은 사람들이 중요햔 이슈들을 SMS로 받길 원하며 그외 사람들은 페이스북으로 알림을 받고싶어한다. 또한, 기업사용자들은 슬랙으로 알림을 받고싶어한다.

![img](https://refactoring.guru/images/patterns/diagrams/decorator/problem2-2x.png)

`Notifier` 클레스를 확장하고 추가적으로 메서드를 서브클래스에 추가해야 할 것이다. 클라이언트는 원하는 알림을 받기 위해 해당 클래스를 인스턴스화 하여 사용해야 한다.
만약 여러개의 알림을 한 번에 받고싶다면 어떻게 해야할까?
지금의 구조라면 하나의 클래스를 만들어서 알람을 보내는 메서드를 모두 정의해야할 것이다.그러나 이렇게 정의해버리면 중복되는 코드가 많기 떄문에 코드의 양이 상당히 커진다.
![img](https://refactoring.guru/images/patterns/diagrams/decorator/problem3-2x.png)

## 해결 방법

객체들을 각각 다르게 행위를 할 수 있도록 첫 번쨰로 떠오르는 것은 클래스를 확장하는 것이다. 그러나, 상속은 주의해야 할 점이 있다.

- 상속은 정적이다. 런타임 중에 객체의 행동을 마음대로 바꿀 수 없다는 것이다. 또 다른 클래스에 상속 받은 후애 원하는 행동을 정의해야 한다.

- 서브 클래스는 오직 하나의 부모 클래스에게 상속 받아야 한다. 대부분의 프로그래밍 언어는 하나의 클래스에 두 개이상 클래스를 상속 받을 수 없다.

## 이런 위험요소들을 해결하는 방법중 하나는 상속 대신에 _Aggregation_ 또는 _Composition_ 을 사용하는 것이다. 둘 다 비슷한 방식이다.

> _Aggregation_ : 전체와 부분의 연관 관계를 맺지만, 동일한 생명 주기를 갖지 않는다. ('Person'과 'Address')
> _Composition_ : 전체와 부분이 강력한 관계를 맺고 전체와 부분이 같은 생명 주기를 갖는다. ('Car'와 'Engine')

하나의 객체는 다른 객체에 대한 참조는 하여 작업을 위임하는 반면에, 상속은 객체 자체가 해당 작업을 수행할 수 있다.

"helper"라는 객체를 통해 다른 객체와 연결하여 런타임중에 쉽게 객체의 동작을 바꿀 수 있다. 해당 객체는 복수 개의 객체를 참조하여 다양한 클래스의 행동들을 할 수 있다. Aggregation과 composition은 많은 디자인 패턴(Decorator)에 속해 있다.

![img](https://refactoring.guru/images/patterns/diagrams/decorator/solution1-en-2x.png)

데코레이터 패턴은 "Wrapper"로 다르게 부를 수 있다. 이 _wrapper_ 는 _타겟_ 객체를 연결해주는 객체이다. 래퍼 클래스는 타겟 객체의 메서드들을 포함한다. 그리고 모든 요청에 대한 응답을 받는다. 그러나, 래퍼 클래스는 해당 객체의 요청 전후로 어떤 작업을 통해 결과물을 바꿀 수 있다.

래퍼는 래핑된 객체와 같은 인터페이스를 구성한다. 왜냐하면 클라이언트의 관점에서는 그 객체들은 모두 같아야 하기 때문이다.

이제 알림 관련 예제에서, 간단한 이메일 알림을 `Notofier`클래스에 넣고. 모든 알림 메서드드들을 데코레이터로 구성한다.

![img](https://refactoring.guru/images/patterns/diagrams/decorator/solution2-2x.png?id=a7855fa704955b432240)

클라이언트는 원하는 대로 `Notifier` 객체를 래핑해서 사용한다. `SMS`, `FaceBook`, `Slack을` 모두 래핑한다면, 스택과 같은 구조가 될 것이다.

![img](https://refactoring.guru/images/patterns/diagrams/decorator/solution3-en-2x.png?id=9a4ef2b4267685a83d02)

## 구조

![img](https://refactoring.guru/images/patterns/diagrams/decorator/structure-indexed-2x.png?id=2733e7d0e322bfb2f150)

## 코드

```python
class Component:
    """
    베이스 컴포넌트 인터페이스이다.
    메서드는 데코레이터에 의해 변경된다.
    """

    def operation(self):
        pass


class ConcreteComponent(Component):
    def operation(self):
        return self.__class__.__name__


class Decorator(Component):
    _component: Component = None

    def __init__(self, component: Component):
        self._component = component

    @property
    def component(self):
        return self._component

    def operation(self):
        return self._component.operation()


class ConcreteDecoratorA(Decorator):
    def operation(self):
        return f"{self.__class__.__name__} <- {super().operation()}"


class ConcreteDecoratorB(Decorator):
    def operation(self):
        return f"{self.__class__.__name__} <- {super().operation()}"


def client_code(component: Component):
    print(f"Result: {component.operation()}")


if __name__ == '__main__':
    component = ConcreteDecoratorA(ConcreteComponent())
    component = ConcreteDecoratorB(component)
    client_code(component)
```
