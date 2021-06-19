---
title:  "querydsl"

categories:
  - querydsl
tags:
  - querydsl
  - Spring boot
last_modified_at: 2021-06-19
---

Back-end 개발을 할 경우 Spring boot, JPA, Spring data jpa를 대체로 많이 사용한다.
하지만 이 기술 조합으로 해결하지 못하는 한계가 발생하기 마련인데, 이는 복잡한 쿼리와 동적 쿼리 문제다.
실무에서는 특히 복잡한 쿼리를 많이 사용하고, 검색 조건에 따른 동적쿼리를 사용할 일이 많다.
또한 필자가 개발하는 정산 도메인은 쿼리를 작성해야 하는 일이 대부분이다.

그 동안 위의 상황에서 JPA와 Querydsl의 학습이 제대로 되어 있지 않았기에 Sql Mapper인 MyBatis를 사용했지만 
이번 기회에 querydsl을 학습하여 실무에까지 적용해보려 한다.

### 장점
* querydsl은 자바 언어의 한계를 넘어 쿼리를 문자가 아닌 자바 코드로 작성할 수 있도록 해줌. 
* 또한 이로 인해 문법 오류를 컴파일 시점에 잡아주며 IDE의 자동완성 기능을 지원받을 수 있다.
* 자바 코드이지만 문법이 sql과 비슷하여 쉽게 학습할 수 있다.
* 복잡한 쿼리도 작성할 수 있다.
* 동적 쿼리 문제를 해결해 준다.

다음 포스트부터 최신 Back-end기술의 핵심인 Querydsl을 학습하며 어떤 기능이 있는지 알아볼 것이다.
 

참고: [https://www.inflearn.com/course/Querydsl-%EC%8B%A4%EC%A0%84/dashboard](https://www.inflearn.com/course/Querydsl-%EC%8B%A4%EC%A0%84/dashboard)