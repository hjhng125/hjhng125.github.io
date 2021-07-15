---
title:  "Spring Data JPA가 지원하는 querydsl"
excerpt: Spring Data JPA가 지원하는 querydsl 정리

categories:
  - querydsl
tags:
  - querydsl
  - Spring boot
last_modified_at: 2021-07-15
---

이번 포스트는 Spring Data Jpa가 querydsl을 어떻게 지원하는지 기록한다. 
아쉽게도 Spring Data Jpa가 지원하는 방법은 여러 제약사항으로 인해 복잡한 실무환경에서의 활용은 쉽지 않다.

어떤 부분이 실무에 사용하기 어려운지 알아보고 추가로 이 부분을 개선하는 방법을 기록할 것이다.

## 인터페이스 지원 - QuerydslPredicateExecutor
* [공식 문서](https://docs.spring.io/spring-data/jpa/docs/2.2.3.RELEASE/reference/html/#core.extensions.querydsl)
* 파라미터로 where문의 조건으로 사용할 Predicate로 받아 쿼리를 만들어주는 interface를 지원한다.
* 지원 메소드
    * findOne()
    * findAll()
    * count()
    * exists()
* Spring Data Sort, Pageable을 지원한다.

### MemberRepository
~~~
public interface MemberRepository extends JpaRepository<Member, Long>, MemberRepositoryCustom, QuerydslPredicateExecutor<Member> {

    List<Member> findByUsername(String username);
}
~~~

### 테스트코드
~~~
@Test
void querydslPredicateExecutorTest() {
    Iterable<Member> result = memberRepository
        .findAll(member.age.between(20, 40).and(member.username.eq("member2"))); // QMember.member

    result.forEach(System.out::println);
    assertThat(result).extracting("username")
        .containsExactly("member2");
}
~~~

### 한계
* 조인이 필요한 쿼리에서 사용 불가 (묵시적 조인은 가능하나 외부 조인 불가능)
* 서비스 코드에 querydsl 기술이 노출되어 기술 의존성이 생긴다.
* 이러한 이유로 복잡한 실무 환경에서의 사용은 고려되어야 한다.

## querydsl Web 지원 - @QuerydslPredicate(root = Entity.class)
* [공식문서](https://docs.spring.io/spring-data/jpa/docs/2.2.3.RELEASE/reference/html/#core.web.type-safe)
* 쿼리스트링으로 넘어온 파라미터를 Predicate 조회 조건으로 바인딩 해준다.
* 바인딩된 결과를 QuerydslPredicateExecutor가 지원하는 메소드의 파라미터로 넘기면 where문 조회 조건으로 간편하게 사용할 수 있다.

~~~
@GetMapping("/v6/members")
public List<Member> searchMemberV6(@QuerydslPredicate(root = Member.class) Predicate predicate) {
    System.out.println("predicate = " + predicate);
    Iterable<Member> all = memberRepository.findAll(predicate);
    List<Member> listAll = new ArrayList<>();
    all.forEach(listAll::add);
    return listAll;
}
~~~

### 한계
* 단순한 조회조건만 가능하다.
    * eq()
    * contains()
    * in()
* 조건을 커스텀하기 복잡하고, 방법이 명시적이지 않다.
    * repository에 QuerydslBinderCustomizer<`QType`>를 상속받아 구현해야한다.
* 컨트롤러 계층이 querydsl에 의존적이다.
* 이러한 이유로 복잡한 환경에서 사용하기 어렵다.

## 레파지토리 지원 - QuerydslRepositorySupport
* 커스텀 리포지토리에 QuerydslRepositorySupport를 상속받는다.
* QuerydslRepositorySupport를 사용하면 JPAQueryFactory와 다르게 from()으로 시작한다.
* 또한 getQuerydsl()이라는 메소드를 제공하는데, 이는 spring data jpa가 제공하는 Querydsl라는 헬퍼 클래스를 반환한다.
* Querydsl 객체는 applyPagination() 메소드를 제공하는데 이는 Pageable을 인자로 받아 내부에서 페이징한다.
* 위에서 메소드 체인으로 적용했던 offset(), limit() 코드가 줄어드는 장점이 있다.
~~~
public class MemberRepositoryCustomImpl extends QuerydslRepositorySupport implements MemberRepositoryCustom {
    
    public MemberRepositoryCustomImpl(JPAQueryFactory jpaQueryFactory) {
        super(Member.class);
    }

    @Override
    public Page<MemberTeamDTO> searchPageSimpleV2(MemberSearchCondition condition, Pageable pageable) {
        JPQLQuery<MemberTeamDTO> query = from(member)
            .leftJoin(member.team, team)
            .where(
                usernameEquals(condition.getUsername()),
                teamNameEquals(condition.getTeamName()),
                ageGoe(condition.getAgeGoe()),
                ageLoe(condition.getAgeLoe())
            )
            .select(new QMemberTeamDTO(
                member.id,
                member.username,
                member.age,
                team.id,
                team.name
            ))
            .offset(pageable.getOffset())
            .limit(pageable.getPageSize());

        JPQLQuery<MemberTeamDTO> memberTeamDTOJPQLQuery = Objects.requireNonNull(getQuerydsl()).applyPagination(pageable, query);

        return new PageImpl<>(memberTeamDTOJPQLQuery.fetch(), pageable, memberTeamDTOJPQLQuery.fetchCount());

    }
}
~~~

### 단점
* 하지만 Sort는 적용할 시 오류가 발생한다.
* queryFactory를 지원하지 않으며, from()으로 시작하기에 JPAQueryFactory 보다 명시적이지 않다.

<hr>

참고: [실전! Querydsl - 인터페이스 지원 - QuerydslPredicateExecutor](https://www.inflearn.com/course/Querydsl-%EC%8B%A4%EC%A0%84/lecture/30155?tab=curriculum)

참고: [실전! Querydsl - Querydsl Web 지원](https://www.inflearn.com/course/Querydsl-%EC%8B%A4%EC%A0%84/lecture/30156?tab=curriculum)

참고: [실전! Querydsl - 리포지토리 지원 - QuerydslRepositorySupport](https://www.inflearn.com/course/Querydsl-%EC%8B%A4%EC%A0%84/lecture/30157?tab=community)
