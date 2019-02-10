---
layout: post
title: "파이썬 객체의 값(value)"
date: 2019-02-10
---
[파이썬 변수에 대하여](https://lioliolio.github.io/about-variable-in-python/) 라는 글에서 `is` 연산자에 대해 언급을 했습니다. 하지만 비교 연산자인 `==`에 대해서는 언급을 하지 않았습니다. 그 이유는 비교 연산자에 대해 공부를 하던 중 단순히 **값을 비교한다** 라고 설명하기에는 더 많은 내용이 있었기 때문입니다. 이번에는 파이썬에서 **객체의 값**에 대해 정리를 해보고자 합니다.

### 값(Value)이란 무엇인가?

[6.10.1. Value comparison](https://docs.python.org/3/reference/expressions.html?highlight=comparison#value-comparisons) 문서에서 **값(Value)**에 대해 다음과 같이 설명하고 있습니다.

**The value of an object is a rather abstract notion in Python: For example, there is no canonical access method for an object’s value. Also, there is no requirement that the value of an object should be constructed in a particular way, e.g. comprised of all its data attributes. Comparison operators implement a particular notion of what the value of an object is. One can think of them as defining the value of an object indirectly, by means of their comparison implementation.**

파이썬에서 객체의 **값**이란 개념은 추상적인 개념이며, 비교 연산자를 통해서 간접적으로 객체의 **값**이 무엇인지 정의하는 것으로 생각할 수 있다고 설명하고 있습니다.

밑에 예제 코드가 있습니다.

```
class Location:
    
    def __init__(self, x, y):
        self.x = x
        self.y = y
        

location_a = Location(3, 4)
location_b = Location(3, 4)

location_a == location_b
----------------------------
False
```

기본적으로 유저가 만든 클래스의 경우 값이 동등한지 비교하는 연산자인 `==` 는 `is`와 마찬가지로 객체의 identity(Cpython의 경우 객체의 메모리 주소)를 비교합니다. 이는 클래스를 만들 때 object를 상속받는데 이때 object의 기본 동작이 위와 같기 때문입니다.

여기에서 Location의 x, y의 값이 같은지 비교하기를 원한다면 비교 메서드인 `__eq__(self, other)`메서드를 구현하면 됩니다.

```
class Location:
    
    def __init__(self, x, y):
        self.x = x
        self.y = y
    
    def __eq__(self, other):
        if self.x == other.x and self.y == other.y:
            return True
        return False


location_a = Location(3, 4)
location_b = Location(3, 4)

location_a == location_b
----------------------------
True    
```

`__eq__(self, other)` 메서드를 통해 Location 인스턴스의 x, y 값이 같은지 비교 하도록 했습니다.

위와 같이 비교 메서드를 어떻게 구현하느냐에 따라 값이 다르게 정의되는 걸 확인할 수 있습니다.

### built-in 타입들의 비교
이미 파이썬에서 구현한 타입들(int, float, str 등등)은 각 타입에 맞춰서 비교 동작이 커스터마이징 되어있습니다. 따라서 저희는 `==` 연산자와 `is` 연산자를 구분하여 사용하고 있었던 것입니다.

각 타입이 값을 어떻게 비교하고 있는지는 다음 문서 [6.10.1. Value comparisons](https://docs.python.org/3/reference/expressions.html?highlight=comparison#value-comparisons)에 자세히 정리되어 있습니다.


### 참고
[6.10.1. Value comparison](https://docs.python.org/3/reference/expressions.html?highlight=comparison#value-comparisons)