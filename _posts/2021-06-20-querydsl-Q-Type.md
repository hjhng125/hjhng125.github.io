---
title:  "querydsl Q-Type"

categories:
  - querydsl
tags:
  - querydsl
  - Spring boot
last_modified_at: 2021-06-20
---

## Q-Type 선언

~~~
QMember qMember = new QMember("qMember"); // 별칭(alias) 사용
QMemver qMember = QMember.member; // 기본 인스턴스 사용

import static me.hjhng125.querydsl.member.QMember.*; // 기본 인스턴스와 static import 사용 (권장됨)
~~~

* Q-Type의 생성자에 들어가는 String값 ("qMember")는 JPQL에서의 table alias 값이다.
* Querydsl은 JPQL 빌더이기 때문에 실제 JPQL이 어떻게 발생했는지 확인할 수 있다.
* 이 부분을 확인하기 위해선 아래 옵션 추가
~~~
spring:
  jpa:
    properties:
      hibernate:
        use_sql_comments: true
~~~
* 같은 테이블을 조인(self join)해야 하는 경우 alias가 겹치면 안되기 때문에 각각 기본 인스턴스와 alias를 사용하여 만들어야 한다.

<hr>

참고: [https://www.inflearn.com/course/Querydsl-실전/dashboard](https://www.inflearn.com/course/Querydsl-%EC%8B%A4%EC%A0%84/dashboard)