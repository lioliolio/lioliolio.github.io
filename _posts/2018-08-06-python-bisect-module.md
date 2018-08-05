---
layout: post
title: "Python bisect 라이브러리"
date: 2018-08-06
---
[**bisect**](https://docs.python.org/3.7/library/bisect.html)는 파이썬에서 제공하는 표준 라이브러리다. 이진 검색 알고리즘을 이용해서 시퀀스를 검색하고, 시퀀스에 항목을 삽입할수 있는 함수를 제공한다.

### bisect()
 
`bisect.bisect(a, x, lo=0, hi=len(a))`
 
[**bisect()**](https://docs.python.org/3.7/library/bisect.html#bisect.bisect)는 `a`라는 오름차순으로 정렬된 시퀀스에 `x`값이 들어갈 위치를 리턴한다.
 
 ```
 import bisect
 
 sequence = [1, 3, 4, 5]
 
 print(bisect.bisect(sequence, 2))
 ---------------------------------
 1
 ``` 
 
 `lo`와 `hi`를 통해 검색하는 시퀀스의 영역으로 늘이거나 줄일 수 있다. 기본적으로 시퀀스 전체를 범위로 한다.
 
[**bisect()**](https://docs.python.org/3.7/library/bisect.html#bisect.bisect)는 [**bisect_right()**](https://docs.python.org/3.7/library/bisect.html#bisect.bisect_right)와 동일하다. 비슷한 함수로 [**bisect_left()**](https://docs.python.org/3.7/library/bisect.html#bisect.bisect_left)가 있다.

두 함수는 차이는 `x`와 동일한 값이 시퀀스 `a`에 존재할 때 다른 값을 반환다는 것이다.

[**bisect_right()**](https://docs.python.org/3.7/library/bisect.html#bisect.bisect_right)는 `x`와 동일한 값이 시퀀스 `a`에 존재할 때 `x`와 동일한 값 바로 뒤 위치를 리턴한다.

반면에 [**bisect_left()**](https://docs.python.org/3.7/library/bisect.html#bisect.bisect_left)는 `x`와 동일한 값 위치를 리턴한다.

```
import bisect
 
sequence = [1, 3, 4, 5]
 
print(bisect.bisect_right(sequence, 3))
print(bisect.bisect_left(sequence, 3))
----------------------------------------
2
1
```

### insort()
`bisect.insort(a, x, lo=0, hi=len(a))`

[**insort()**](https://docs.python.org/3.7/library/bisect.html#bisect.insort)는 `a`라는 오름차순으로 정렬된 시퀀스에 `x`값을 삽입한다.

```
import bisect

sequence = [1, 3, 4, 5]

bisect.insort(sequence, 2)
print(sequence)
---------------------------
[1, 2, 3, 4, 5]
```

[**bisect()**](https://docs.python.org/3.7/library/bisect.html#bisect.bisect)와 마찬가지로 `lo`와 `hi`를 통해 검색하는 시퀀스의 영역으로 늘이거나 줄일 수 있다. 기본적으로 시퀀스 전체를 범위로 한다. 

[**insort()**](https://docs.python.org/3.7/library/bisect.html#bisect.insort)는 [**insort_right()**](https://docs.python.org/3.7/library/bisect.html#bisect.insort_right)와 같고 [**insort_left()**](https://docs.python.org/3.7/library/bisect.html#bisect.insort_left)가 존재한다.

[**insort_right()**](https://docs.python.org/3.7/library/bisect.html#bisect.insort_right)는  `x`와 동일한 값이 시퀀스 `a`에 존재할 때 `x`와 동일한 값 바로 뒤 위치에 `x`를 삽입한다.

[**insort_left()**](https://docs.python.org/3.7/library/bisect.html#bisect.insort_left)는 `x`와 동일한 값이 시퀀스 `a`에 존재할 때 `x`와 동일한 값 위치에 `x`를 삽입한다.

### 참고
- 전문가를 위한 파이썬
- [8.6. bisect — Array bisection algorithm](https://docs.python.org/3.7/library/bisect.html)
- [SortedCollection (Python recipe) ](https://code.activestate.com/recipes/577197-sortedcollection/)
