---
title:  "querydsl 집합"

categories:
  - querydsl
tags:
  - querydsl
  - Spring boot
last_modified_at: 2021-06-25
---

## 집합 함수
  * JPQL이 지원하는 집합 함수를 제공
  * count() : 조회 데이터의 수
  * sum() : 조회 데이터 컬럼의 합
  * avg() : 조회 데이터 컬럼의 평균
  * max() : 조회 데이터 컬럼의 최대값
  * min() : 조회 데이터 컬럼의 최소값

  ~~~
  List<Tuple> result = queryFactory
              .select(
                  member.count(),
                  member.age.sum(),
                  member.age.avg(),
                  member.age.max(),
                  member.age.min())
              .from(member)
              .fetch();

          Tuple tuple = result.get(0);
          assertThat(tuple.get(member.count())).isEqualTo(4);
          assertThat(tuple.get(member.age.sum())).isEqualTo(100);
          assertThat(tuple.get(member.age.avg())).isEqualTo(25);
          assertThat(tuple.get(member.age.max())).isEqualTo(40);
          assertThat(tuple.get(member.age.min())).isEqualTo(10);
  ~~~

* select를 위와 같이 선언할 경우 querydsl core가 지원하는 `Tuple`로 리턴된다.
* tuple.get()의 파라미터로 select문에 명시한 값(`Expression<T>`)을 넣으면 값을 얻을 수 있다.

## group by
  * groupBy() : group by 쿼리를 발생시킨다.

  ~~~
  List<Tuple> result = queryFactory
            .select(team.name, member.age.avg())
            .from(member)
            .join(member.team, team)
            .groupBy(team.name)
            // .having()
            .fetch();
  ~~~
  
  * groupBy()를 통해 그룹화된 결과는 `having()`을 통해 제한할 수 있다.

<hr>

참고: [실전! Querydsl - 집합](https://www.inflearn.com/course/Querydsl-%EC%8B%A4%EC%A0%84/lecture/30128?tab=note)