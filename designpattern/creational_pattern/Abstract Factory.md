# Abstract Fatory

## 개념

추상 팩토리는 구체적인 클래스를 지정하지 않고 관련 객체군들을 생성할 수 있는 디자인 패턴이다.

## 문제 발견

가구점 시뮬레이터를 만들다고 가정해본다면 코드는 이렇게 구성될 것이다.

1. 관련 객체 군, 예: `Chair` + `Sofa` + `CoffeeTable`
2. 각 객체의 종류, 예: `Chair` + `Sofa` + `CoffeeTable` 는 `Modern`, `Victorian`, `ArtDeco`의 종류를 가진다.

![img](https://refactoring.guru/images/patterns/diagrams/abstract-factory/problem-en-2x.png)

각 종류마다 가구 세트을 생산할 방법이 필요하다. 만약 `Chair`와 `Sofa`는 `Modern` 타입인데 `CoffeeTable`만 `ArtDeco`이면 안된다.

또한 시뮬레이터에 기존의 코드를 수정하지 않고 가구 세트의 새로운 가구를 추가하는 것이 좋다.

## 해결 방법

먼저 추상 팩토리는 각각의 가구 종류마다 인터페이스를 정의하기를 제안한다. 그러면 모든 종류의 제품들을 각각 가구의 인터페이스를 따라 생성하게 된다.

![img](https://refactoring.guru/images/patterns/diagrams/abstract-factory/solution1-2x.png)

그 다음에는 모든 종류의 가구를 생산하는 메서드를 가지고 있는 `Abstract Factory` 인터페이스를 정의한다. 이 메서드는 꼭 **abstract** 한 클래스를 리턴해야한다. (ex, `Chair`, `Sofa`, `CoffeeTable`)

![img](https://refactoring.guru/images/patterns/diagrams/abstract-factory/solution2-2x.png)

추상 팩토리 클래스를 상속받아 각각의 가구 디자인의 종류를 나타내는 하위 클래스를 작성한다.
클라이언트 코드는 각각의 추상 인터페이스를 통해 팩토리 클래스와 가구 클래스에서 모두 동작해야한다. 클라이언트 코드를 손상시키지 않고 전달되는 팩토리 종류와 클라이언트가 받는 가구의 디자인 종류를 변경할 수 있다.
![img](https://refactoring.guru/images/patterns/content/abstract-factory/abstract-factory-comic-2-en-2x.png)

클라이언트는 팩토리가 의자를 생산하길 원한다고 가정해본다면, 클라이언트는 팩토리의 클래스와 생산할 의자의 다지안을 알 필요가 없다. 그저 `Chair`라는 인터페이스에 `sitOn`이라는 메서드를 가지고 있다는 것만 안다.

## 구조

![img](https://refactoring.guru/images/patterns/diagrams/abstract-factory/structure-indexed-2x.png)

## 코드

```python
from __future__ import annotations
from abc import ABC, abstractmethod


class AbstractFactory(ABC):
    @abstractmethod
    def create_product_a(self) -> AbstractProductA:
        return AbstractProductA()

    @abstractmethod
    def create_product_b(self) -> AbstractProductB:
        return AbstractProductB()


class ConcreteFactory1(AbstractFactory):
    def create_product_a(self) -> ConcreteProductA1:
        return ConcreteProductA1()

    def create_product_b(self) -> ConcreteProductB1:
        return ConcreteProductB1()


class ConcreteFactory2(AbstractFactory):
    def create_product_a(self) -> ConcreteProductA2:
        return ConcreteProductA2()

    def create_product_b(self) -> ConcreteProductB2:
        return ConcreteProductB2()


class AbstractProductA(ABC):
    def func_a(self):
        return f"< class {self.__class__.__name__} >"


class AbstractProductB(ABC):
    def func_b(self):
        return f"< class {self.__class__.__name__} >"


class ConcreteProductA1(AbstractProductA):
    pass


class ConcreteProductA2(AbstractProductA):
    pass


class ConcreteProductB1(AbstractProductB):
    pass


class ConcreteProductB2(AbstractProductB):
    pass


def client(factory: AbstractFactory):
    product_a = factory.create_product_a()
    product_b = factory.create_product_b()
    print(product_a.func_a())
    print(product_b.func_b())


if __name__ == '__main__':
    client(ConcreteFactory1())
    client(ConcreteFactory2())

```
