---
title:  "querydsl 정렬"

categories:
  - querydsl
tags:
  - querydsl
  - Spring boot
comments: true
last_modified_at: 2021-06-23
---

## 정렬
* orderBy() : order by 쿼리를 발생시킨다.
  * `asc()` : 오른차순
  * `desc()` : 내림차순
    * `nullsLast()` : 해당 필드 값이 null 이라면 조건의 `마지막`으로 가져온다.
    * `nullsFirst()` : 해당 필드 값이 null 이라면 조건의 `첫번째`로 가져온다.



<hr>

참고: [실전! Querydsl - 정렬](https://www.inflearn.com/course/Querydsl-%EC%8B%A4%EC%A0%84/lecture/30126?tab=curriculum)