---
title:  "JPA 컬렉션 패치 조인의 문제점"

categories:
  - jpa
tags:
  - Spring
comments: true
last_modified_at: 2021-07-12
---

> JPA에서 컬렉션 패치 조인을 하는 경우 리스트가 중복되어 나오는 문제가 있다. <br>아래의 예시를 보자.

## JPQL distinct

~~~
@Test
void fetchJoinByTeam() {
    //given

    //when
    List<Team> result = queryFactory
        .selectFrom(team)
        .join(team.members, member).fetchJoin()
        .fetch();

    //then
    for (Team team : result) {
        System.out.println("team = " + team);
        for (Member member : team.getMembers()) {
            System.out.println("member = " + member);
        }
    }
}
~~~
![1](/assets/images/fetchjoinduplicate.png)
* 1:N의 관계를 조회하는 경우 row수는 당연히 증가되어 있기 때문에 이 문제는 어찌보면 당연한 결과이다.
* teamA와 teamB에 각각 두명의 member가 맵핑되어 있기 때문에 총 4개의 row가 리턴되었다.
* JPQL은 총 4개의 결과물을 team 엔티티 기준으로 저장할 것이기에 중복된 결과가 만들어지는 것이다.
* 따라서 1:N 패치조인을 할 경우 distinct를 필수적으로 해주어야 한다.

~~~
@Test
    void fetchJoinByTeamDistinct() {
        //given

        //when
        List<Team> result = queryFactory
            .selectFrom(team).distinct()
            .join(team.members, member).fetchJoin()
            .fetch();

        //then
        for (Team team : result) {
            System.out.println("team = " + team);
            for (Member member : team.getMembers()) {
                System.out.println("member = " + member);
            }
        }
    }
~~~
![1](/assets/images/fetchjoindistinct.png)