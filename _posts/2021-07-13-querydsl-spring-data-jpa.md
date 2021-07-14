---
title:  "querydsl &  Spring Data JPA"

categories:
  - querydsl
tags:
  - querydsl
  - Spring boot
last_modified_at: 2021-07-13
---

## Spring Data JPA 사용하기
* Repository interface를 생성하고 `JpaRepository<Entity, ID>`를 상속받는다.
* Spring data JPA는 이전의 포스트에서 손수 구현한 기능들을 기본적으로 제공한다.
    * [순수JPA와 querydsl](/querydsl/querydsl-pure-jpa-querydsl)
* 아래와 같은 기능은 Spring data JPA가 기본으로 제공하진 않는다. 하지만 메서드 이름을 통해 JPQL을 발생시키는 전략을 통해 쿼리를 자동으로 만들어준다.
    * [Spring Data JPA 쿼리만들기](/스프링-데이터-jpa/Common4-repository-query)

~~~
public interface MemberRepository extends JpaRepository<Member, Long> {

    List<Member> findByUsername(String username);
}
~~~

* Spring Data JPA는 JpaRepository를 상속받는 인터페이스를 만들면 기본적인 CRUD와 간단한 정적쿼리를 간편학게 사용할 수 있게 해준다.
    * 인터페이스를 자동으로 빈으로 등록해준다.
* 하지만 동적쿼리는 querydsl을 사용해야만 하기 때문에 Spring Data JPA와 querydsl을 함께 사용하는 방법을 알아볼 것이다.

## querydsl과 Spring Data JPA 함께 사용하는 방법
* 복잡한 구현이나 커스텀한 구현이 필요한 경우 사용
* querydsl을 쓰려면 결국 구현 코드를 만들어야 하는데, Spring Data JPA는 인터페이스로 동작하기 때문에 원하는 구현 코드를 넣기 위해서 조금 복잡한 방법으로 사용자 정의 레파지토리를 선언해야 한다.
    1. 사용자 정의 인터페이스 정의
    2. 사용자 졍의 인터페이스 구현
    3. Spring Data JPA 리포지토리에 사용자 정의 인터페이스 상속

### 사용자 정의 리포지토리
~~~
public interface MemberRepositoryCustom {

    List<MemberTeamDTO> search(MemberSearchCondition condition);
}
~~~

### 사용자 정의 리포지토리 구현체
~~~
@RequiredArgsConstructor
public class MemberRepositoryCustomImpl implements MemberRepositoryCustom {

    private final JPAQueryFactory jpaQueryFactory;

    @Override
    public List<MemberTeamDTO> search(MemberSearchCondition condition) {
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
}
~~~
* naming은 postfix가 기본적으로 Impl 이어야 Spring data JPA가 인식할 수 있으며 EnableJpaRepository 어노테이션의 옵션으로 수정 가능
* naming 규칙이 맞지 않은 경우 Spring Data JPA가 클래스 파일을 찾지 못하여 에러가 발생한다.
* EnableJpaRepository javadoc
![1](/assets/images/EnableJpaRepositories.png)

### 기존 리포지토리
~~~
public interface MemberRepository extends JpaRepository<Member, Long>, MemberRepositoryCustom {

    List<Member> findByUsername(String username);
}
~~~

### 테스트 코드
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
    void searchByCustomRepository() {
        //given
        MemberSearchCondition memberSearchCondition = MemberSearchCondition.builder()
            .teamName("teamB")
            .ageGoe(20)
            .ageLoe(40)
            .build();

        //when
        List<MemberTeamDTO> memberTeamDTOS = memberRepository.search(memberSearchCondition);

        //then
        assertThat(memberTeamDTOS).extracting("username")
            .containsExactly("member3", "member4");
    }
}
~~~

## 커스텀 리포지토리를 만들 때 고민 사항
* 만약 구현한 조회 쿼리가 특정 화면 혹은 API에서만 단독으로 사용되며,
* 특정 entity를 가져오는 기능이 아니고 특정 조건에 특화된 경우(특정 화면에 맞춰진 API인 경우) 커스텀 리포지토리로 구현하지 않고 조회용 리포지토리를 구현해보는 것을 생각해보자
* ex) MemberQueryRepository
<hr>

참고: [실전! Querydsl - 스프링 데이터 JPA 리포지토리로 변경](https://www.inflearn.com/course/Querydsl-%EC%8B%A4%EC%A0%84/lecture/30149?tab=curriculum)

참고: [실전! Querydsl - 사용자 정의 리포지토리](https://www.inflearn.com/course/Querydsl-%EC%8B%A4%EC%A0%84/lecture/30150?tab=note)
