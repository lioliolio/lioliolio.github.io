---
layout: post
title: "파이썬 변수에 대해서"
date: 2017-04-08
---

### 파이썬 변수는 상자가 아니다.
처음 프로그래밍을 공부할 때 변수는 보통 값을 저장하는 상자라고 배운다. 하지만 파이썬에서는 변수를 상자라고 했을 때 이해할 수 없는 동작이 있다.
```
a = [1, 2, 3]
b = a
b.append(4)
print(a)
-------------------
[1, 2, 3, 4]
```
파이썬을 사용하면서 리스트를 복사하려다 하게 되는 실수(?)인데 b 리스트에 4라는 값을 추가했음에도 a 리스트의 값도 같이 변경되었다.

### 파이썬 변수는 상자가 아니라 포스트잇
전문가를 위한 파이썬(Fluent Python)에서는 파이썬 변수를 상자가 아닌 포스트잇에 비유한다. 위의 코드에서 [1,2,3] 이라는 리스트 객체에 a라는 이름을 붙이는 것이다.

포스트잇으로 비유한다면 `[1, 2, 3]`이라는 하나의 리스트 객체에 a와 b라는 이름의 포스트잇 붙인 것이다. 따라서 b에서 4를 추가하면 a 역시 b와 같은 객체이기 때문에 a에서도 같이 값이 변경된 것이다.

![variable_is_label]({{site.baseurl}}/assets/img/var-boxes-x-labels.png)

위와 같이 동작하기 때문에 파이썬에서는 하나의 객체에서 여러 변수들을 할당할 수 있다(aliasing).

### `is` 연산자
여러 변수를 하나의 객체에 할당을 할수 있다고 했다. 그러면 변수들이 같은 객체를 참조하고 있다는 것을 어떻게 확인할 수 있을까?

이때 사용하는 것이 바로 `is` 연산자이다. `is` 연산자는 객체의 **identity**를 비교한다.

```
a = [1, 2, 3]
b = a
a is b
True
```

### **identity**란?
`is` 연산자가 비교하는 **identity** 란 무엇일까?

[python data model](https://docs.python.org/3/reference/datamodel.html?highlight=data%20model) 문서를 보면 identity에 대해 다음과 같이 설명하고 있다.

**Every object has an identity, a type and a value. An object’s identity never changes once it has been created; you may think of it as the object’s address in memory. The ‘is’ operator compares the identity of two objects; the id() function returns an integer representing its identity. CPython implementation detail: For CPython, id(x) is the memory address where x is stored.**

파이썬 객체는 identity, type, value를 가지고 있고 identity는 객체가 만들어진 후에는 객체가 사라지기 전까지 변하지 않는 값이다. CPython에서는 identity는 해당 객체의 메모리 주소이다.

CPython 에서 `is` 연산자가 비교하는 것은 **객체의 메모리 주소**인 것이다.

### 참고
- 전문가를 위한 파이썬
- [Python 3.Data model](https://docs.python.org/3/reference/datamodel.html)
