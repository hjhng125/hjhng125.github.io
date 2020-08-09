---
title:  "Python Generator 와 코루틴"
excerpt: "Python Generator 와 코루틴에 대하여"

categories:
  - Python
tags:
  - Python
  - Coroutine
last_modified_at: 2020-03-05
---

# Generator
* 제네레이터는 유한 혹은 무한한 횟수만큼 어떤 값을 만들어 내놓는 객체 (iterable 한 값을 return 하는 객체 )
* 제네레이터 함수는 이러한 제네레이터를 생성하는 함수이다. 
* 내부에 `yield` 를 사용하여, return 과 달리 값을 내놓은 후에 종료되지 않고 중단된다.

* 제네레이터 함수를 호출하면 제네레이터 객체가 생성됨.
* 이 객체는 생성됨과 동시에 실행되지 않는다. 대신 외부로부터 동작을 요청받으면 그때부터 실행되기 시작한다.
* 제네레이터 객체에 값을 요청하는 함수는 next() 이고, 이 함수로 넘겨진 제네레이터는 yield 문을 만날때 까지 실행된 후 중단한다.

# Coroutine
* 보통 함수는 입력으로 주어진 인수에 대하여 한 번 실행되며 이는 서브루틴이라고 불린다.
* Generator 를 서브루틴과 같이 동작할 수 있게 하는 것이 코루틴이다.
* 코루틴은 제네레이터를 소비하는 코드에서 send 함수를 사용하여 역으로 제네레이터 함수의 각 yield 표현식에 값을 보낼 수 있게 하는 방법으로 동작
* 제네레이터가 첫 번째 yield 표현식으로 전진하여 첫 번째 send 의 값을 받을 수 있게 하려면, 먼저 next() 를 호출해야 한다.
* yield 와 send 의 조합은 제네레이터가 외부 입력에 반응하여 다음에 다른 값을 얻게 하는 표준 방법이다.
* line = (yield) 는 값을 넘겨받아 line 변수에 할당하는 것
* yield value 는 value값을 호출한 함수에 돌려주며 현재 상태를 기억
```
def function():
  while True:
    line = (yield)
    print(line)
ans = function()
ans.next()
ans.send('hello world')
```