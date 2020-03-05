---
title:  "Python Generator"
excerpt: "Python Generator에 대하여"

categories:
  - Python
tags:
  - Python
  - Coroutine
last_modified_at: 2020-03-05T020:42:00
---

# Generator
제네레이터는 유한 혹은 무한한 횟수만큼 어떤 값을 만들어 내놓는 객체
제네레이터 함수는 이러한 제네레이터를 생성하는 함수이다. 
내부에 `yield` 를 사용하여, return 과 달리 값을 내놓은 후에 종료되지 않고 중단된다.

제네레이터 함수를 호출하면 제네레이터 객체가 생성됨.
이 객체는 생성됨과 동시에 실행되지 않는다. 대신 외부로부터 동작을 요청받으면 그때부터 실행되기 시작한다.
제네레이터 객체에 값을 요청하는 함수는 next() 이고, 이 함수로 넘겨진 제네레이터는 yield 문을 만날때 까지 실행된 후 중단한다.