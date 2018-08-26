---
layout: post
title: "튜플(tuple)을 딕셔너리 키로 사용할 수 있을까?"
date: 2018-08-26
---
**"리스트와 튜플을 딕셔너리 키로 사용할 수 있나요?"**

면접 질문 중에 이런 질문이 있었다는 얘기를 들었다. ~~내가 면접서 받은 질문은 아니기 떄문에 정확하지 않을 수 있다~~

위 질문에 답하기 전에 딕셔너리 키로 사용할 수 있는 게 무엇인지 알아보자.

### Hashable Object
 
**[4.10. Mapping Types — dict](https://docs.python.org/3/library/stdtypes.html#mapping-types-dict)** 문서에는 

**A mapping object maps hashable values to arbitrary objects.**

라고 설명하고 있다. 매핑 타입인 딕셔너리는 키로 **해시 가능한 값(hashable value)**을 사용한다. 그러면 **해시 가능한(Hashable)**은 무슨 뜻일까?

**[Python Glossary](https://docs.python.org/3/glossary.html)** 의 **해시 가능한(hashable)**을 찾아보면 다음과 같이 설명하고 있다.

**An object is hashable if it has a hash value which never changes during its lifetime (it needs a __hash__() method), and can be compared to other objects (it needs an __eq__() method). Hashable objects which compare equal must have the same hash value.**

객체의 생명주기동안 변하지 않는 해시 값을 갖고, 다른 객체와 비교 가능할 때 **그 객체는 해시 가능 하다** 라고 한다. 또, 파이썬의 모든 불변 내장 객체는 해시 가능하다고 하니 불변 객체인 튜플도 딕셔너리 키로 사용할 수 있을 것 같다.

### 튜플을 딕셔너리 키로 사용할 수 있을까?

튜플은 int, str와 같은 불변 객체와는 차이점이 있다. 튜플은 **컨테이너(container)** 다. 튜플이 컨테이너이기 때문에 미묘하게 동작하는 부분이 있었다. **[파이썬 튜플의 불변성](https://lioliolio.github.io/immutability-of-tuple-in-python/)** 에서 관련 내용을 포스팅 한적 있다.

튜플이 가변 객체의 참조를 담고 있는 경우 그 값이 바뀔 수 있다. 이 때도 해시가능 할까? 간단히 테스트 해보자.

해시 값을 리턴하는 **[Hash Function](https://en.wikipedia.org/wiki/Hash_function)**은 파이썬에서 **[hash()](https://docs.python.org/3/library/functions.html#hash)** 함수로 구현되어 있다.

```python
>>> a = (1, 2, 3)
>>> b = ([1], 2, 3)
>>> hash(a)
2528502973977326415
>>> hash(b)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: unhashable type: 'list'
``` 

불변 객체의 참조만 담고 있는 튜플 a의 경우 문제없이 해시 값을 받을 수 있었다. 반면에 가변 객체의 참조를 담고 있는 튜플 b는 해시 값을 받지 못하고 에러가 났다. 튜플 b에서는 해시 불가능한 객체를 참조하고 있었기 때문이다.

지금까지 알아본 내용을 바탕으로 처음 면접 질문에 이렇게 답할 수 있을 듯 하다.

**"가변 객체인 리스트는 해시 가능하지 않기 때문에 딕셔너리 키로 사용할 수 없고, 튜플은 불변 객체의 참조만 담고 있는 경우에 딕셔너리 키로 사용가능합니다."**
### 참고
- 전문가를 위한 파이썬
- [8.6. bisect — Array bisection algorithm](https://docs.python.org/3.7/library/bisect.html)
- [SortedCollection (Python recipe) ](https://code.activestate.com/recipes/577197-sortedcollection/)
