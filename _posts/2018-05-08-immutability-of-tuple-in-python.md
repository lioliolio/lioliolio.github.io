---
layout: post
title: "파이썬 튜플의 불변성"
date: 2018-05-07
---

### 튜플(tuple)은 정말 값이 변하지 않는가?
튜플(tuple)은 변하지 않는 불변 객체로 알고 있다. 튜플 값이 변하지 않을까?

```
a = (1, 2, [1, 2, 3])

print(a)
id_1 = a[-1]

a[-1].append(1000)

print(a)
id_2 = a[-1]

id_1 == id_2
----------------------
(1, 2, [1, 2, 3])
(1, 2, [1, 2, 3, 1000])
True
```

튜플은 값이 변하지 않는다고 했는데, 값이 변경됐다. 이상하다. 왜 값이 변경됐을까? 이걸 알기 위해서는 가변 객체(mutable object), 불변 객체(immutable object), 컨테이너(container)에 대한 이해가 필요하다.


### 가변 객체(mutable object)와 불변 객체(immutable object)
파이썬 객체는 값을 변경할 수 있는 객체와 변경할 수 없는 객체가 있다. 전자는 가변 객체(mutable object), 후자는 불변 객체(immutable object)이다.

가변 객체에는 리스트(list), 딕셔너리(dictionary), 집합(set) 등이 있고, 불변 객체에는 정수형(integer), 문자열(string), 튜플(tuple) 등이 있다.

### 컨테이너(container)
[Objects, values and types](https://docs.python.org/3/reference/datamodel.html#objects-values-and-types) 문서에서 컨테이너(container)에 대해 다음과 같이 설명하고 있다.

**Some objects contain references to other objects; these are called containers.**

다른 객체의 참조(reference)를 담고있는 객체를 컨테이너(container)라고 부른다.

컨테이너에는 리스트(list), 딕셔너리(dictionary), 튜플(tuple), 집합(set) 등이 있다.

다른 객체의 참조를 포함한다는 것 어떤 의미일까?

```
a = [1, "hello", [1, 2, 3]]
``` 

위의 예시처럼 어떤 타입의 데이터도 포함할 수 있다. 다만, 몇몇 컨테이너(집합 등)의 경우 해시 가능한(hashable) 데이터 타입만 포함할 수 있다.

### 튜플의 불변성
다시 한번 맨 처음 코드를 보자.

```
a = (1, 2, [1, 2, 3])

print(a)
id_1 = a[-1]

a[-1].append(1000)

print(a)
id_2 = a[-1]

print(id_1 == id_2)
----------------------
(1, 2, [1, 2, 3])
(1, 2, [1, 2, 3, 1000])
True
```
튜플은 불변 객체이기 때문에 값이 변경되지 않는다고 했는데 위의 코드에서 보듯이 값이 변경됐다. 정확히 말하면 위 튜플에서 참조하고 있는 리스트이 값이 변경됐다.

여기서 생각해봐야 할 것은 **'튜플은 어떤 값을 포함하고 있는가'** 이다.

튜플은 불변 객체이면서 컨테이너이다. 다시 말해서, 튜플은 **다른 객체의 참조**를 담고 있다. 튜플이 불변 객체라 하는 것은 이 참조들이 변경되지 않는다는 의미이다. 

튜플이 담고 있는 참조가 불변이라고 해서 참조된 객체의 값 역시 불변이라는 것을 의미하지 않는다. 만약, 리스트나 딕셔너리처럼 가변 객체일 경우에는 위의 코드처럼 참조된 객체의 값이 변경될 수 있다.

### 참고
- 전문가를 위한 파이썬
- [Python 3.1. Objects, values and types](https://docs.python.org/3/reference/datamodel.html#objects-values-and-types)
