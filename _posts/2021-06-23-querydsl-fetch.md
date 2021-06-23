---
title:  "querydsl 결과 조회"

categories:
  - querydsl
tags:
  - querydsl
  - Spring boot
last_modified_at: 2021-06-23
---

## 결과 조회
  * fetch() : 리스트 조회, 결과값 없는 경우 빈 리스트를 리턴.
  * fetchOne() : 단 건 조회
    * 결과 없는 경우 : `null`
    * 결과가 2개 이상인 경우 : `NonUniqueResultException`
  * fetchFirst() : `limit(1).fetchOne()`
  * fetchResult() : 페이징 정보 포함, total count 쿼리 추가 실행 - 실제 쿼리 2번 발생
    * 복잡한 쿼리의 경우 results를 가져오는 쿼리와 다르게 count를 조회하는 쿼리에서 최적화를 위해 다른 쿼리가 발생할 수 있기에 유의해야 한다.
    * total
    * results
    * limit
    * offset
  * fetchCount() : count쿼리로 변경하여 count 조회



<hr>

참고: [https://www.inflearn.com/course/Querydsl-실전/dashboard](https://www.inflearn.com/course/Querydsl-%EC%8B%A4%EC%A0%84/dashboard)