---
title:  "querydsl 수정, 삭제 벌크 연산"

categories:
  - querydsl
tags:
  - querydsl
  - Spring boot
last_modified_at: 2021-07-13
---

## Bulk 연산이란
> 쿼리 한번으로 대량의 데이터를 처리하는 방법

* JPA는 기본적으로 엔티티를 가져와서 값을 수정하면 트랜잭션을 커밋할 때 플러시가 발생하며 변경 감지(dirty checking)가 일어난다.
* 이로써 업데이트 쿼리가 발생하며 DB에 반영된다.
* 이 변경 감지는 개별 엔티티마다 발생하므로 상당한 수의 쿼리가 발생할 수 있다.
* 따라서 일괄적으로 데이터를 수정해야 하는 경우 bulk 연산이 필요하다.

### JPQL과 flush
* JPQL은 영속성 컨텍스트의 데이터에 관여하지 않고 실행 전 flush를 자동으로 실행시킨다.
* 영속성 컨텍스트의 데이터가 아직 DB와 동기화가 되지 않은 상태에서 JPQL이 반영되어 버리면 DB와 영속성 컨텍스트 일관성이 바로 깨져버리기 때문이다.
* 따라서 JPQL은 실행되기 전 flush를 실행시킴으로써 영속성 컨텍스트와 DB의 일관성을 먼저 확보한 후 실행되게 된다.

### update
~~~
@BeforeEach
void before() {
    memberSub = new QMember("memberSub");

    Team teamA = new Team("teamA");
    Team teamB = new Team("teamB");

    em.persist(teamA);
    em.persist(teamB);

    Member member1 = new Member("member1", 10, teamA);
    Member member2 = new Member("member2", 20, teamA);

    Member member3 = new Member("member3", 30, teamB);
    Member member4 = new Member("member4", 40, teamB);

    em.persist(member1);
    em.persist(member2);
    em.persist(member3);
    em.persist(member4);
}

@Test
void bulk_update() {
    long count = queryFactory
        .update(member)
        .set(member.username, "비회원")
        .where(member.age.lt(30))
        .execute();
    
    List<Member> result = queryFactory
            .selectFrom(member)
            .fetch();
 
    assertThat(result)
        .extracting("username")
        .doesNotContain("비회원");
}
~~~
위의 테스트는 실패한다. 그 이유를 알아보자.

* 위의 결과로 나이가 30 미만인 회원의 이름은 "비회원"으로 DB의 값이 변경된다.
* JPQL은 영속성 컨텍스트를 거치지 않고 쿼리를 바로 DB에 반영시키기 때문에 영속성 컨텍스트와 싱크가 깨져버린다.
* 그 다음 JPQL을 통해 데이터를 가져오면 영속성 컨텍스트에 저장을 시도한다.
* 이미 영속성 컨텍스트에 해당 데이터가 있기 때문에 DB에서 가져온 데이터를 버리고 영속성 컨텍스트에 있는 값을 반환한다.
* 따라서 위의 반환된 결과는 영속성 컨텍스트의 데이터이기 때문에 테스트가 실패하게 된다.
* 만약 영속성 컨텍스트에 데이터가 없었다면 DB에서 가져온 데이터를 영속성 컨텍스트에 저장 후 반환할 것이다.

위의 테스트를 성공시키기 위해 아래와 같이 해야한다.
~~~
@Test
void bulk_update() {

    long count = queryFactory
        .update(member)
        .set(member.username, "비화원")
        .where(member.age.lt(30))
        .execute();

    em.flush();
    em.clear();

    List<Member> result = queryFactory
        .selectFrom(member)
        .fetch();

    assertThat(result)
        .extracting("username")
        .doesNotContain("비회원");

}
~~~
* `flush()`를 강제로 실행시켜 영속성 컨텍스트의 쿼리 지연 저장소의 쿼리를 DB에 반영한다.
* 영속성 컨텍스트를 비워(`clear()`) DB에서 가져온 값이 영속성 컨텍스트에 저장될 수 있도록 한다.

### update - add
~~~
@Test
void bulk_add() {

    long count = queryFactory
        .update(member)
        .set(member.age, member.age.add(1))
        .execute();

    assertThat(count).isEqualTo(4);
}
~~~

### update - multiply
~~~
@Test
void bulk_multiply() {
    
    long count = queryFactory
        .update(member)
        .set(member.age, member.age.multiply(2))
        .execute();

    assertThat(count).isEqualTo(4);
}
~~~

### delete
~~~
@Test
void bulk_delete() {

    long count = queryFactory
        .delete(member)
        .where(member.age.goe(18))
        .execute();

    assertThat(count).isEqualTo(3);
}
~~~


<hr>

참고: [실전! Querydsl - 수정, 삭제 벌크 연산](https://www.inflearn.com/course/Querydsl-%EC%8B%A4%EC%A0%84/lecture/30141?tab=community)
