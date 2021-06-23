---
title:  "querydsl paging"

categories:
  - querydsl
tags:
  - querydsl
  - Spring boot
last_modified_at: 2021-06-23
---

## 페이징
* offset() 
  * 앞에 스킵할 row의 수
  * 0부터 시작한다. ex) 1이라면 1개를 스킵한 것.
* limit()
  * 최대로 가져올 row의 수
* 전체 조회수를 구하고 싶은 경우 
  * `fetchResults()`를 사용한다.
  * 만약 results를 가져오는 쿼리가 복잡한 경우 count쿼리와 select 쿼리를 분리한다.
    * count 쿼리는 results를 가져오는 쿼리에 비해 어떤 조건이 필요없거나, join이 필요 없을 수 있다.
    * count 쿼리는 더욱 단순하게 나올 수 있기 때문에 간단하고 단순한 쿼리가 아니라면 성능을 위해 분리하도록 하자.



<hr>

참고: [https://www.inflearn.com/course/Querydsl-실전/dashboard](https://www.inflearn.com/course/Querydsl-%EC%8B%A4%EC%A0%84/dashboard)