---
layout: post
title: "얕은 복사(swallow copy)와 깊은 복사(deep copy)"
date: 2018-05-27
---
전에 쓴 글인 [파이썬 변수에 대해서](https://lioliolio.github.io/about-variable-in-python/)의 예제를 보자.

```
a = [1, 2, 3]
b = a
b.append(4)
print(a)
-------------------
[1, 2, 3, 4]
```

`b = a`는 리스트를 카피하는 것이 아니다. a가 가리키는 리스트 객체에 b라는 이름을 하나 더 붙여주는것 뿐이다.

그러면 리스트를 복사하기 위해서는 어떤 방법을 써야할까?

### 생성자, 슬라이싱 또는 copy 모듈의 `copy()` 사용
리스트를 복사하는 방법은 생성자(`list()`) 또는 슬라이싱(`[:]`)을 사용하는 방법이 있다. 또 다른 방법으로는 copy 모듈의 `copy()`를 사용하는 방법도 있다.

```
import copy

a = [1, 2, 3]
b = list(a)
c = a[:]
d = copy.copy(a)
b.append(4)
c.append(5)
d.append(6)
print(a)
print(b)
print(c)
print(d)
print(a is b)
print(a is c)
print(a is d)
--------------------
[1, 2, 3]
[1, 2, 3, 4]
[1, 2, 3, 5]
[1, 2, 3, 6]
False
False
False
```

생성자 또는 슬라이싱, copy 모듈의 `copy()`함수를 사용하면 리스트를 복사할 수 있다. 단, 위의 방법으로 복사할 경우 얕은 복사(swallow copy)가 된다. 얕은 복사란 무엇일까?

### 얕은 복사(swallow copy)
[8.10. copy — Shallow and deep copy operations](https://docs.python.org/3.6/library/copy.html)에서 얕은 복사를 이렇게 정의하고 있다.

**A shallow copy constructs a new compound object and then (to the extent possible) inserts references into it to the objects found in the original.**

얕은 복사란 최상위 컨테이너는 복사하지만 카피한 컨테이너 내부의 참조는 기존 컨테이너와 동일한 참조로 채워져 있는 것을 얘기한다. 

컨테이너 내부의 참조가 전부 불변 객체(immutable object)라면 문제 되지 않는다. 값을 수정하게 되면 참조 하고는 있는 객체 값이 변하는 게 아니라 새로운 객체의 참조로 바뀌기 때문이다. 

문제는 가변 객체(mutable object)로 구성되어 있을 때 문제가 된다.

```
a = [1, 2, 3, [1, 2, 3]]
b = list(a)

b[3].append(4)
print(a)
print(b)
print(a is b)
print(a[3] is b[3])
-------------------------
[1, 2, 3, [1, 2, 3, 4]]
[1, 2, 3, [1, 2, 3, 4]]
False
True
```

분명히 `a is b`가 `False`인 것을 보면 리스트를 제대로 복사한 것은 맞다. 하지만 `a[3] is a[3]`가 `True`인 것은 위에서 얘기한 것처럼 최상위 리스트를 복사했지만 내부 참조들은 동일하다는 것을 뜻한다. 따라서 a를 복사한 b 리스트 내부의 리스트를 변경하니 a 리스트 내부의 리스트 역시 변경됐다.

그러면 컨테이너 내부의 객체에 대한 참조까지 복사하려면 어떻게 해야할까?

### 깊은 복사(deep copy)
[8.10. copy — Shallow and deep copy operations](https://docs.python.org/3.6/library/copy.html)에서 깊은 복사를 이렇게 정의하고 있다.

**A deep copy constructs a new compound object and then, recursively, inserts copies into it of the objects found in the original.**

깊은 복사는 최상위 컨테이너를 복사하고 내부의 참조 역시 원본 컨테이너의 참조를 복사한다. 깊은 복사는 copy 모듈의 `deepcopy()` 함수를 사용하면 된다.

```
import copy

a = [1, 2, 3, [1, 2, 3]]
b = copy.deepcopy(a)

b[3].append(4)
print(a)
print(b)
print(a is b)
print(a[3] is b[3])
-------------------------
[1, 2, 3, [1, 2, 3]]
[1, 2, 3, [1, 2, 3, 4]]
False
False
```

얕은 복사와는 달리 리스트 내부의 참조들까지 복사가 됐기 때문에 a 리스트는 변경되지 않았다.
### 참고
- 전문가를 위한 파이썬
- [8.10. copy — Shallow and deep copy operations](https://docs.python.org/3.6/library/copy.html)
