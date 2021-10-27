---
title:  "querydsl JPAQueryFactory와 스프링 빈"

categories:
  - querydsl
tags:
  - querydsl
  - Spring boot
comments: true
last_modified_at: 2021-07-13
---

### QuerydslConfig.java
~~~
@Configuration
public class QuerydslConfig {

    @PersistenceContext
    private EntityManager entityManager;

    @Bean
    public JPAQueryFactory jpaQueryFactory() {
        return new JPAQueryFactory(entityManager);
    }
}
~~~
* JPAQueryFactory를 빈으로 등록하여 싱글톤으로 관리할 시 동시성 문제가 발생할것 같지만 문제가 없다.
* JPAQueryFactory는 EntityManager에 의존한다.
* EntityManager는 스프링과 함께 쓸 경우 동시성 문제와 관계없이 트랜잭션 단위로 분리되어 동작한다.
* 스프링에서 주입해주는 EntityManager는 사실 실제 영속성 컨텍스트가 아닌 프록시다.
* 이 프록시 객체는 요청이 트랜잭션 단위로 실제 엔터티 매니저를 할당해준다.

<hr>
