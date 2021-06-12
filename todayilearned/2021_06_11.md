# Design patterns

https://refactoring.guru/design-patterns 의 디자인 패턴을 해석한 글입니다.

## Factory Method

팩토리 메서드 패턴은 인터페이스를 제공하는 생성 디자인 패턴이다. 인터페이스는 슈퍼 클래스에서 객체를 생성하고 서브 클래스가 생성 될 객체의 유형을 변경한다.

## 문제 도출

물류 관리 애플리케이션을 만든하고 상상해보자. 만들 앱의 첫 버전은 오직 트럭으로만 운송을 할 수 있디고 한다면, 코드는 대부분 `Truck` 클래스에 있다.

![img](https://refactoring.guru/images/patterns/diagrams/factory-method/problem1-en-2x.png?id=9a4959d9dde4edadf809)

만약에 애플리케이션이 잘된다고 가정해보자. 하루에 해양 물류를 앱에 통합하라는 수십개의 요청이 해양 운송 업체에서 들어온다면 어떨까? 좋은 소식이긴 하지만 현재 코드는 Truck 클래스에 묶여있다. `Ships`이라는 클래스를 앱에 추가하려면 아마 전체의 코드 베이스를 바꿔야 한다. 또 나중에 다른 운송 수단을 통합하려면 전체 코드를 다시 바꿔야 한다.

결과적으로 이러한 방식은 코드를 아주 더럽게 만든다. 유지 보수도 힘들뿐더러 앱은 운송수단 객체의 클래스에 의존적이게 된다.

## 해결 방안

팩토리 메서드 패턴은 직접적으로 객체를 생산하지 않고(`new` operator) 특별한 _factory_ 메서드를 통해 객체를 불러온다. `new` operator는 팩토리 메서드에서 호출한다.

![img](https://refactoring.guru/images/patterns/diagrams/factory-method/solution1-2x.png?id=c482b3cd7a4d8dd73b4c)

이제 서브 클래스는 팩토리 메서드에서 리턴되는 객체의 클래스를 변경할 수 있다.
그러나 제한사항아 존재한다. 서브 클래스는 오직 공통적으로 베이스 클래스 또는 인터페이스를 가진 경우에만 다른 유형의 객체를 반환할 수 있다.

![img](https://refactoring.guru/images/patterns/diagrams/factory-method/solution2-en-2x.png?id=1209a3156e450b9d7c43)

예를들어 `Transport` 인터페이스에 `deliver`이라는 메서드가 정의되어 있다면 `Truck`과 `Ship` 클래스에도 구성해야한다. `RoadLogistics`클래스에 있는 팩토리 메서드는 트럭 객체를 반환하고, `SeaLogistics`클래스에 있는 팩토리 메서드는 배 객체를 반환한다.

![img](https://refactoring.guru/images/patterns/diagrams/factory-method/solution3-en-2x.png?id=542c0ba89e91ac11ea79)

팩토리 메서드(또는 클라이언트)에서 사용되는 코드는 반환하는 객체가 어떤 서브 클래스인지 알 수 없다. 팩토리 메서드는 `Transport`라는 추상 객체를 다룬다. 왜냐하면 어떤 운송 수단인지 팩토리 메서드는 알 필요가 없기 때문이다.

## 구조

![img](https://refactoring.guru/images/patterns/diagrams/factory-method/structure-indexed-2x.png?id=c794e4f2d05013fb1764)

## 코드

```python
from __future__ import annotations
from abc import ABC, abstractmethod


class Creator(ABC):
    @abstractmethod
    def factory_method(self):
        pass

    def some_operation(self):
        product = self.factory_method()
        result = product.operation()

        return result


class ConcreteCreator1(Creator):
    def factory_method(self):
        return ConcreteProduct1()


class ConcreteCreator2(Creator):
    def factory_method(self):
        return ConcreteProduct2()


class Product(ABC):
    @abstractmethod
    def operation(self):
        pass


class ConcreteProduct1:
    def operation(self):
        return f"Return of the {self.__class__.__name__}"


class ConcreteProduct2:
    def operation(self):
        return f"Return of the {self.__class__.__name__}"


if __name__ == '__main__':
    print(ConcreteCreator1().some_operation())
    print(ConcreteCreator2().some_operation())
```
