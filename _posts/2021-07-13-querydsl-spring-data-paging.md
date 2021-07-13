---
title:  "querydsl Spring Data Paging 활용"
excerpt: "querydsl Spring Data Paging 활용 정리"

categories:
  - querydsl
tags:
  - querydsl
  - Spring boot
last_modified_at: 2021-07-13
---

## 스프링 데이터 Page, Pageable 활용
### MemberTeamDTO
[querydsl-동적쿼리](/querydsl/querydsl-dynamic-query)에 있습니다.
### 전체 카운트를 한번에 조회
~~~
public Page<MemberTeamDTO> searchPageSimple(MemberSearchCondition condition, Pageable pageable) {
    QueryResults<MemberTeamDTO> results = jpaQueryFactory
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
        .orderBy(member.username.desc()) 
        .offset(pageable.getOffset()) 
        .limit(pageable.getPageSize())
        .fetchResults();

    return new PageImpl<>(results.getResults(), pageable, results.getTotal());

}
~~~
* offset : 몇번째까지 스킵할 것인가
* limit : 몇개를 가져올 것인가
* fetchResult : contents를 가져오는 쿼리와 count를 가져오는 쿼리 두번 날린다.
* fetchResult에서 발생한 count query는 만들어질때 orderBy 쿼리가 필요없기 때문에 제거한다.

### 데이터와 카운트 쿼리를 분리하여 조회
~~~
public Page<MemberTeamDTO> searchPageComplex(MemberSearchCondition condition, Pageable pageable) {
    List<MemberTeamDTO> contents = getMemberTeamDTOS(condition, pageable);

    long total = getTotalQuery(condition).fetchCount();

    return new PageImpl<>(contents, pageable, total);
}

private List<MemberTeamDTO> getMemberTeamDTOS(MemberSearchCondition condition, Pageable pageable) {
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
        .offset(pageable.getOffset())
        .limit(pageable.getPageSize())
        .fetch();
}

private JPAQuery<Member> getTotalQuery(MemberSearchCondition condition) {
    return jpaQueryFactory
        .select(member)
        .from(member)
        .leftJoin(member.team, team)
        .where(
            usernameEquals(condition.getUsername()),
            teamNameEquals(condition.getTeamName()),
            ageGoe(condition.getAgeGoe()),
            ageLoe(condition.getAgeLoe())
        );
}
~~~
* 쿼리 두개를 분리
* 카운트 쿼리는 더 간단한 쿼리로 만들어지는 경우가 있다.
    * 예를 들어 join이 필요없을 수 있다. 
    * 또한 카운트 쿼리를 먼저 실행하고 값이 0인 경우 컨텐트 쿼리를 날리지 않을 수 있다.
* 이런 경우 쿼리를 분리하면 최적화가 된다.

### 테스트코드
~~~
@DataJpaTest
@Import(QuerydslConfig.class)
class MemberRepositoryTest {

    @Autowired
    MemberRepository memberRepository;

    @PersistenceContext
    EntityManager em;

    @BeforeEach
    public void beforeEach() {
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
    void searchPageSimple() {
        //given
        MemberSearchCondition memberSearchCondition = MemberSearchCondition.builder()
            .build();
        PageRequest pageRequest = PageRequest.of(0, 3);

        //when
        Page<MemberTeamDTO> memberTeamDTOPage = memberRepository.searchPageSimple(memberSearchCondition, pageRequest);

        //then
        assertThat(memberTeamDTOPage.getSize()).isEqualTo(3);
        assertThat(memberTeamDTOPage.getContent())
            .extracting("username")
            .containsExactly("member1", "member2", "member3");
    }

    @Test
    void searchPageComplex() {
        //given
        MemberSearchCondition memberSearchCondition = MemberSearchCondition.builder()
            .build();
        PageRequest pageRequest = PageRequest.of(0, 3);

        //when
        Page<MemberTeamDTO> memberTeamDTOPage = memberRepository.searchPageComplex(memberSearchCondition, pageRequest);

        //then
        assertThat(memberTeamDTOPage.getSize()).isEqualTo(3);
        assertThat(memberTeamDTOPage.getContent())
            .extracting("username")
            .containsExactly("member1", "member2", "member3");

    }
}
~~~

## CountQuery 최적화
### PageableExecutionUtils.getPage()
~~~
public Page<MemberTeamDTO> searchPageNoCountQuery(MemberSearchCondition condition, Pageable pageable) {
    List<MemberTeamDTO> contents = getMemberTeamDTOS(condition, pageable);

    JPAQuery<Member> countQuery = getTotalQuery(condition);

    return PageableExecutionUtils.getPage(contents, pageable, countQuery::fetchCount);
}
~~~
* 스프링 데이터가 제공
* count query가 필요없는 경우 생략한다.
     * 시작 페이지면서 총 컨텐츠 사이즈가 페이지 사이즈보다 작을 때
     * 마지막 페이지일 때 (offset + 컨텐츠 사이즈로 전체 사이즈를 구한다.)
* PageableExecutionUtils.getPage() 함수는 마지막 인자로 카운트 쿼리 함수를 받기에 내부적으로 위처럼 카운트를 생략할 수 있는 경우 함수를 실행하지 않는다.
![1](/assets/images/PageableExecutionUtils.png)

### 테스트코드
~~~
@DataJpaTest
@Import(QuerydslConfig.class)
class MemberRepositoryTest {

    @Autowired
    MemberRepository memberRepository;

    @PersistenceContext
    EntityManager em;

    @BeforeEach
    public void beforeEach() {
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
    void searchPageNoCount() {
        //given
        MemberSearchCondition memberSearchCondition = MemberSearchCondition.builder()
            .build();
        PageRequest pageRequest = PageRequest.of(0, 5);

        //when
        Page<MemberTeamDTO> memberTeamDTOPage = memberRepository.searchPageNoCountQuery(memberSearchCondition, pageRequest);

        //then
        assertThat(memberTeamDTOPage.getSize()).isEqualTo(5);
        assertThat(memberTeamDTOPage.getContent())
            .extracting("username")
            .containsExactly("member1", "member2", "member3", "member4");
    }
}
~~~

## 스프링 데이터 Sort
* Spring Data가 제공하는 Pageable의 Sort는 querydsl에서 사용하기 위해서 OrderSpecifier를 사용해야한다.
* 하지만 OrderSpecifier는 조건이 복잡해지거나, join을 해야하는 경우 제대로 동작하지 않기에 실무에서 사용하기 어렵다.
* 위의 케이스에서 Sort를 정상 작동하게 하는 방법은 다음 포스트에서 다룰 예정이다.

<hr>

참고: [실전! Querydsl - 스프링 데이터 페이징 활용1 - Querydsl 페이징 연동](https://www.inflearn.com/course/Querydsl-%EC%8B%A4%EC%A0%84/lecture/30151?tab=note)

참고: [실전! Querydsl - 스프링 데이터 페이징 활용2 - CountQuery 최적화](https://www.inflearn.com/course/Querydsl-%EC%8B%A4%EC%A0%84/lecture/30150?tab=note)

참고: [실전! Querydsl - 스프링 데이터 페이징 활용3 - 컨트롤러 개발](https://www.inflearn.com/course/Querydsl-%EC%8B%A4%EC%A0%84/lecture/30153?tab=community)
