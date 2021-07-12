---
title:  "querydsl 동적 쿼리"

categories:
  - querydsl
tags:
  - querydsl
  - Spring boot
last_modified_at: 2021-07-13
---

> 동적 쿼리를 해결하는 방식은 두가지가 있다.
1. BooleanBuilder
2. Where 다중 파라미터 사용 

## MemberTeamDTO
~~~
@Getter
@ToString
public class MemberTeamDTO {

    private final Long memberId;
    private final String username;
    private final int age;

    private final Long teamId;
    private final String teamName;

    @QueryProjection
    public MemberTeamDTO(Long memberId, String username, int age, Long teamId, String teamName) {
        this.memberId = memberId;
        this.username = username;
        this.age = age;
        this.teamId = teamId;
        this.teamName = teamName;
    }
}
~~~

## BooleanBuilder
* 특정 조건이 필수여야 한다면 BooleanBuilder 생성 시 인자로 조건을 넣어주어야 한다.

~~~
 public List<MemberTeamDTO> searchByBuilder(MemberSearchCondition condition) {
    return jpaQueryFactory
        .select(new QMemberTeamDTO(
            member.id,
            member.username,
            member.age,
            team.id,
            team.name
        ))
        .from(member)
        .leftJoin(member.team, team)
        .where(condition(condition))
        .fetch();
}

private BooleanBuilder condition(MemberSearchCondition condition) {
    BooleanBuilder builder = new BooleanBuilder();

    if (hasText(condition.getUsername())) {
        builder.and(member.username.eq(condition.getUsername()));
    }

    if (hasText(condition.getTeamName())) {
        builder.and(team.name.eq(condition.getTeamName()));
    }

    if (condition.getAgeGoe() != null) {
        builder.and(member.age.goe(condition.getAgeGoe()));
    }

    if (condition.getAgeLoe() != null) {
        builder.and(member.age.loe(condition.getAgeLoe()));
    }

    return builder;
}
~~~

## Where 다중 파라미터 사용 
* where()의 조건으로 null을 받을 경우 조건에서 무시된다.

~~~
public List<MemberTeamDTO> searchByWhereParam(MemberSearchCondition condition) {
    return jpaQueryFactory
        .select(new QMemberTeamDTO(
            member.id,
            member.username,
            member.age,
            team.id,
            team.name
        ))
        .from(member)
        .leftJoin(member.team, team)
        .where(
            usernameEquals(condition.getUsername()),
            teamNameEquals(condition.getTeamName()),
            ageGoe(condition.getAgeGoe()),
            ageLoe(condition.getAgeLoe())
            )
        .fetch();
}

private BooleanExpression usernameEquals(String username) {
    return hasText(username) ? member.username.eq(username) : null;
}

private BooleanExpression teamNameEquals(String teamName) {
    return hasText(teamName) ? team.name.eq(teamName) : null;
}

private BooleanExpression ageGoe(Integer ageGoe) {
    return ageGoe != null ? member.age.goe(ageGoe) : null;
}

private BooleanExpression ageLoe(Integer ageLoe) {
    return ageLoe != null ? member.age.loe(ageLoe) : null;
}
~~~
* 특정 조건(BooleanExpression)을 묶어 이름을 부여할 수도 있다.

~~~
private BooleanExpression betweenAge(Integer ageGoe, Integer ageLoe) {
    return ageGoe(ageGoe).and(ageLoe(ageLoe));
}

private BooleanExpression ageGoe(Integer ageGoe) {
    return ageGoe != null ? member.age.goe(ageGoe) : null;
}

private BooleanExpression ageLoe(Integer ageLoe) {
    return ageLoe != null ? member.age.loe(ageLoe) : null;
}
~~~

## 결론
* BooleanBuilder를 사용하는 방식과 BooleanExpression을 사용하는 방식 중 좋고 나쁘다를 따지긴 애매모호하다.
* 하지만 BooleanExpression을 사용한 방식은 각 조건을 따로 나누었기에 코드 재사용성은 더 좋을 수 있다고 생각한다.
* 또한 각 조건에 이름을 부여할 수 있어 코드가 더 직관적이게 되는 장점이 있다.

<hr>

참고: [실전! Querydsl - 동적 쿼리 - BooleanBuilder 사용](https://www.inflearn.com/course/Querydsl-%EC%8B%A4%EC%A0%84/lecture/30139?tab=curriculum)

참고: [실전! Querydsl - 동적 쿼리 - Where 다중 파라미터 사용](https://www.inflearn.com/course/Querydsl-%EC%8B%A4%EC%A0%84/lecture/30140?tab=curriculum)
