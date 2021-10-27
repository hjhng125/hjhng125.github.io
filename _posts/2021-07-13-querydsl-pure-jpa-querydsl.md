---
title:  "querydsl 순수 JPA와 querydsl"
excerpt: "querydsl 순수 JPA와 querydsl"

categories:
  - querydsl
tags:
  - querydsl
  - Spring boot
comments: true
last_modified_at: 2021-07-13
---

## 순수 JPA 리포지토리
* spring-data-jpa를 사용하면 아래의 기능이 전부 지원되기 때문에 필요없다.

~~~
@Repository
@RequiredArgsConstructor
@Transactional(readOnly = true)
public class MemberJpaRepository {
    private final EntityManager em;
    private final JPAQueryFactory jpaQueryFactory;

    @Transactional
    public void save(Member member) {
        em.persist(member);
    }

    public Optional<Member> findById(Long id) {
        Member findByMember = em.find(Member.class, id);
        return Optional.of(findByMember);
    }

    public List<Member> findAll() {
        return em.createQuery("select m from Member m", Member.class)
            .getResultList();
    }

    public List<Member> findByUsername(String username) {
        return em.createQuery("select m from Member m where m.username = :username", Member.class)
            .setParameter("username", username)
            .getResultList();
    }
}
~~~

## querydsl 사용 리포지토리
~~~
@Repository
@RequiredArgsConstructor
@Transactional(readOnly = true)
public class MemberJpaRepository {

    public List<Member> findAllQuerydsl() {
        return jpaQueryFactory
            .selectFrom(member)
            .fetch();
    }

    public List<Member> findByUsernameQuerydsl(String username) {
        return jpaQueryFactory
            .selectFrom(member)
            .where(usernameEq(username))
            .fetch();
    }

    private BooleanExpression usernameEq(String username) {
        return member.username.eq(username);
    }
}
~~~



<hr>

코드는 [github](https://github.com/hjhng125/querydsl)에 있습니다.

참고: [실전! Querydsl - 순수 JPA 리포지토리와 Querydsl](https://www.inflearn.com/course/Querydsl-%EC%8B%A4%EC%A0%84/lecture/30144?tab=curriculum)
