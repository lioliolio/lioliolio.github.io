---
layout: post
title: "파이썬으로 파일을 읽고 쓰기 할 때 주의할 점"
date: 2019-02-10
---
파일을 다루면서 실수했던 것들을 정리해보았습니다.

### 1. encoding
모든 OS의 기본 인코딩이 똑같다면 신경쓰지 않아도 되는 문제입니다. 하지만 현실에서는 OS마다 기본 인코딩이 다릅니다(**Unix/Linux**는 **UTF-8**, **Windows**는 **CP-1252**). 따라서 인코딩, 디코딩 에러를 사전에 방지하기 위해서는 파일을 읽고 쓸 때는 항상 인코딩을 지정해야 합니다.

```
# 전문가를 위한 파이썬 예제
# 기본 인코딩이 CP-1252 인 경우
open('cafe.txt', 'w', encoding='utf_8').write('Café')
open('cafe.txt').read()
-----------------------
'Cafأ©'
``` 
### 2. close()
파일을 `open()` 함수를 연 경우에는 `close()` 함수를 사용하여 명시적으로 파일을 닫는 것이 좋습니다. 파일의 크기가 작다면 큰 문제가 없지만 데이터가 큰 파일을 다루는 경우, `close()` 함수를 사용하여 닫지 않는다면 파일의 참조 객체가 다른 파일에 재할당 되기 전까지 메모리에 계속 있게 됩니다.

```
fp = open('cafe.txt', encoding='utf_8').read()
fp.close()
```

다른 방법으로는 `with` 문이 있습니다. 
`with` 문이 끝날 때 자동으로 파일을 닫습니다.(`with`문에 대해서는 다음에 정리를 하도록 하겠습니다.)

```
with open('cafe.txt', encoding='utf_8') as fp:
    fp.read()
```    

### 3. 파일 경로
파일 경로의 경우 **절대 경로**와 **상대 경로**를 사용할 수 있습니다. **상대 경로**를 사용하기 위해서는 **current working directory**를 알아야 합니다. 

**상대 경로** 를 사용하여 파일을 읽는데 **FileNotFoundError** 에러가 난다면 `os.getcwd()` 메서드를 사용하여 **current working directory**를 찾으면 됩니다.

### 참고
- 전문가를 위한 파이썬
- [python open](https://docs.python.org/3/library/functions.html?highlight=open#open)