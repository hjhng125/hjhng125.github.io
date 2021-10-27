---
title:  "querydsl SQL function"

categories:
  - querydsl
tags:
  - querydsl
  - Spring boot
comments: true
last_modified_at: 2021-07-13
---

SQL function은 Dialect에 등록된 function만 호출할 수 있다.

* Expressions.~Template() 사용

## 예제1 - replace
~~~
List<String> result = queryFactory
    .select(
        Expressions.stringTemplate(
            "function('replace', {0}, {1}, {2})",
            member.username,
            "member",
            "M"))
    .from(member)
    .fetch();
~~~

## 예제2 - lower
~~~
List<String> result = queryFactory
    .select(member.username)
    .from(member)
    .where(member.username.eq(
        Expressions.stringTemplate(
            "function('lower', {0})",
            member.username)))
    .fetch();
~~~
* lower와 같이 모든 DB에서 공통적으로 지원하는 ansi 표준 함수들은 querydsl에서 기본적으로 지원한다.

~~~
List<String> result2 = queryFactory
    .select(member.username)
    .from(member)
    .where(member.username.eq(member.username.lower()))
    .fetch();
~~~

<hr>

참고: [실전! Querydsl - SQL function 호출하기](https://www.inflearn.com/course/Querydsl-%EC%8B%A4%EC%A0%84/lecture/30142?tab=community)
