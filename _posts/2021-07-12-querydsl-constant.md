---
title:  "querydsl 상수"

categories:
  - querydsl
tags:
  - querydsl
  - Spring boot
last_modified_at: 2021-07-12
---

## 상수 사용
* `Expressions.constant()` 사용 

~~~
List<Tuple> result = queryFactory
    .select(member.username, Expressions.constant("A"))
    .from(member)
    .fetch();
~~~

* 위와 같이 최적화가 가능한 경우 결과에서만 상수를 받고 JPQL, sql에서는 상수를 가져오는 쿼리가 발생하지 않는다.

~~~
List<String> result = queryFactory
    .select(member.username.concat("_").concat(member.age.stringValue()))
    .from(member)
    .where(member.username.eq("member1"))
    .fetch();
~~~

* 상수에 연산이 필요한 경우와 같이 최적화가 힘든 경우 JPQL, sql에도 쿼리가 발생한다.
* stringValue()는 문자가 아닌 다른 타입들을 문자로 캐스팅해준다. Enum을 처리할 때 유용하다.


<hr>

참고: [실전! Querydsl - 상수, 문자 더하기](https://www.inflearn.com/course/Querydsl-%EC%8B%A4%EC%A0%84/lecture/30134?tab=curriculum)
