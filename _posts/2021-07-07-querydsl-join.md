---
title:  "querydsl join"

categories:
  - querydsl
tags:
  - querydsl
  - Spring boot
last_modified_at: 2021-07-10
---

## 기본 조인
* Join의 기본 문법은 join(조인 대상, 조인에 사용할 Q타입의 별칭(alias)) 이다.
* 
~~~ 
List<Member> teamA = queryFactory
            .selectFrom(member)
            .join(member.team, team)
            .where(team.name.eq("teamA"))
            .fetch();
~~~
* join(), innerJoin() : 내부 조인(inner join)
* leftJoin() : left 외부 조인(left outer join)
* rightJoin() : right 외부 조인(right outer join)

## 세타 조인 (theta join)
* 연관 관계가 없는 필드로 조인하는 것
* 세타 조인은 from절에 테이블을 나열한다. (연관관계로의 조인이 아니기에)
* 조인하고자 하는 테이블의 모든 행을 결합하여, 두 테이블의 행 갯수를 곱한 만큼의 결과를 반환한다. (`Cartesian Product: 곱집합`)
* 실제 sql은 `cross join`이 발생한다.
* 외부 조인은 불가능 하나, JPQL의 `on`절을 통해 외부 조인이 가능하다.
* 
~~~
List<Member> eqName = queryFactory
            .selectFrom(member)
            .from(member, team) 
            .where(member.username.eq(team.name))
            .fetch();
~~~

## on절
* JPA 2.1부터 지원한다.
* 조인 대상을 필터링
* 연관관계가 없는 엔터티의 외부 조인을 가능케함(내부 조인도 가능).

* 연관관계가 있는 경우 sql문을 확일할 시, id가 조인 조건에 들어감
* `JPQL: SELECT member, team FROM Member member LEFT JOIN member.team AS team WITH team.name = ?1`
* 
~~~
List<Tuple> result = queryFactory
            .select(member, team)
            .from(member)
            .leftJoin(member.team, team) 
            .on(member.team.name.eq("teamA"))
            .fetch();
~~~

* 연관관계가 없는 경우 일반 조인과 다르게 join절에 엔터티 하나만 들어간다.
* `JPQL: SELECT member, team FROM Member member LEFT JOIN Team team WITH member.username = team.name`
~~~
List<Tuple> result = queryFactory
            .select(member, team)
            .from(member)
            .leftJoin(team).on(member.username.eq(team.name)) 
            .fetch();
~~~

* inner join 의 경우 on에서 필터링을 하는것과 where에서 필터링하는 것은 같다.
* left join 의 경우 where절 에서 필터링 할 경우 필요한 left값이 필터링 될 수 있기에 on절에서 풀어야 한다.

## fetch join
* sql에서 지원하는 기능은 아니다.
* sql 조인을 활용하여 연관된 엔터티를 쿼리 한번으로 모두 가져오는 방법 (FetchType.EAGER와 같음)
* 주로 JPQL 성능 최적화에 사용됨.
* 페치 조인 미적용

~~~
Member findMember = queryFactory
            .selectFrom(member)
            .where(member.username.eq("member1"))
            .fetchOne();

boolean isLoaded = emf.getPersistenceUnitUtil().isLoaded(findMember.getTeam());

assertThat(loaded).as("패치 조인 미적용").isFalse();
~~~
* isLoaded() 메서드는 해당 엔터티가 컨텍스트에 로딩되었는지 아닌지 확인할 수 있도록 해줌.
* `JPQL: SELECT member FROM Member member WHERE member.username = ?1`
* 위의 엔터티의 연관관계 맵핑 시 fetchType을 Lazy로 주었기 때문에 isLoaded()의 반환값은 false로 테스트는 성공한다.

~~~
Member findMember = queryFactory
            .selectFrom(member)
            .join(member.team, team)
            .where(member.username.eq("member1"))
            .fetchOne();

boolean isLoaded = emf.getPersistenceUnitUtil().isLoaded(findMember.getTeam());

assertThat(loaded).as("패치 조인 미적용").isFalse();
~~~
* `JPQL: SELECT member FROM Member member INNER JOIN member.team AS team WHERE member.username = ?1`
* 조인은 발생하지만 team과 관련된 데이터를 가져오진 않는다.

* 패치 조인 적용

~~~
Member findMember = queryFactory
            .selectFrom(member)
            .join(member.team, team).fetchJoin()
            .where(member.username.eq("member1"))
            .fetchOne(); 

boolean loaded = emf.getPersistenceUnitUtil()
    .isLoaded(findMember.getTeam());

assertThat(loaded).as("패치 조인 적용").isTrue();
~~~
* join() 뒤에 .fetchJoin()을 추가한다.
* `JPQL: SELECT member FROM Member membeer INNER JOIN member.team AS team WHERE member.username = ?1`
* 발생한 sql을 확인해보면 즉시 로딩 전략으로 인해 team 데이터도 함께 select한다.

### FetchType.EAGER vs fetchJoin()
* 연관관계를 맵핑할 때 FetchType.EAGER 옵션을 주어버리면, 해당 연관관계에서 발생하는 모든 쿼리에서 연관관계 데이터도 가져오게 된다.
* 특정 상황에서만 한번에 데이터를 가져오고 싶은 경우 fetchJoin()을 사용하면 될 것 같다.

<hr>

참고: [실전! Querydsl - 조인 - 기본 조인](https://www.inflearn.com/course/Querydsl-%EC%8B%A4%EC%A0%84/lecture/30129?tab=curriculum&mm=close)

참고: [실전! Querydsl - 조인 - on절](https://www.inflearn.com/course/Querydsl-%EC%8B%A4%EC%A0%84/lecture/30130?tab=note&mm=close)

참고: [실전! Querydsl - 조인 - fetch join](https://www.inflearn.com/course/Querydsl-%EC%8B%A4%EC%A0%84/lecture/30131?tab=note&mm=close)