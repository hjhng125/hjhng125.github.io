---
title:  "querydsl 검색 조건 쿼리"

categories:
  - querydsl
tags:
  - querydsl
  - Spring boot
last_modified_at: 2021-06-21
---

## 기본 검색 쿼리
  * 검색 조건은 .and()와 .or()를 메서드 체인으로 연결할 수 있다.
  * select(entity).from(entity)은 selectFrom()으로 합칠 수 있다.
  * JPQL이 제공하는 모든 검색 조건을 제공한다. - querydsl 자체가 JPQL의 builder이기 때문
    * eq() : equals(==)
    * ne() : not equals(!=)
    * eq().not() : not equals(!=)
    * isNotNull()
    * in()
    * notIn() 
    * between(from, to) : from <= x <= to
    * goe() : greater or equals than a
    * gt() : greater than 
    * loe() : lower or equals than 
    * lt() : lower than 
    * like()
    * contains(a) : `like '%a%'`
    * startsWith(a) : `like '%a'`

## AND 조건을 파라미터로 처리
  * where()에 파라미터로 검색조건을 추가하면 AND 조건이 추가됨
    * where() 메서드는 파라미터를 Predicate... 로 받기 때문

    ~~~

    public Q where(Predicate o) {
        return queryMixin.where(o);
    }

    public Q where(Predicate... o) {
        return queryMixin.where(o);
    }

    ~~~
  * 이러한 경우 null은 무시되어 메서드 추출 방식을 사용하여 동적 쿼리를 깔끔하게 만들 수 있음
<hr>

참고: [실전! Querydsl - 검색 조건 쿼리](https://www.inflearn.com/course/Querydsl-%EC%8B%A4%EC%A0%84/lecture/30124?tab=curriculum)