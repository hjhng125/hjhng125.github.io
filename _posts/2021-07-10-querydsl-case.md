---
title:  "querydsl case문"

categories:
  - querydsl
tags:
  - querydsl
  - Spring boot
comments: true
last_modified_at: 2021-07-10
---

## Case
* 조건에 따른 값을 지정해주는 문법
* select, where, order by 절에서 사용 가능

### simple case
~~~
List<String> result = queryFactory
    .select(member.age
        .when(10).then("열살")
        .when(20).then("스무살")
        .otherwise("기타"))
    .from(member)
    .fetch();
~~~

### complex case
~~~
 List<String> result = queryFactory
    .select(
        new CaseBuilder()
            .when(member.age.between(0, 20)).then("0 ~ 20살")
            .when(member.age.between(21, 30)).then("21 ~ 30살")
            .otherwise("기타")
    )
    .from(member)
    .fetch();
~~~

### With orderBy
~~~
NumberExpression<Integer> ageCase = new CaseBuilder()
    .when(member.age.between(0, 20)).then(0)
    .when(member.age.between(21, 30)).then(1)
    .otherwise(2);

 List<Tuple> fetch = queryFactory
    .select(member.username, member.age, ageCase)
    .from(member)
    .orderBy(ageCase.desc())
    .fetch();
~~~
* 위와 같이 복잡한 케이스를 변수로 선언하여 select, orderBy절에서 사용할 수 있다.

<hr>

참고: [실전! Querydsl - Case문](https://www.inflearn.com/course/Querydsl-%EC%8B%A4%EC%A0%84/lecture/30133?tab=curriculum)

