---
title:  "Spring Data JPA가 제공하는 Querydsl 기능"

categories:
  - querydsl
tags:
  - querydsl
  - Spring boot
last_modified_at: 2021-07-03
---

Spring Data JPA는 Querydsl에 몇가지 간편한 기능을 제공한다.<br>
이 기능들은 코드 몇줄을 통하여 편하게 사용할 수 있지만 하나의 테이블을 이용한 쿼리에서 사용해야 정상동작을 기대할 수 있다는 제약이 있다.<br>
여러 테이블을 조인해야 하는 경우 기능이 정상 동작하지 않을 수 있기에 복잡한 실무환경에선 적합하지 않지만 Spring Data JPA가 이런 기능도 지원하고 있음을 기억하기 위해 기록으로 남기도록 하겠다.

## QuerydslPredicateExecutor
* [공식 문서](https://docs.spring.io/spring-data/jpa/docs/2.2.3.RELEASE/reference/html/#core.extensions.querydsl)
* Spring Data JPA가 지원하는 querydsl interface
* Where절의 조건에 해당하는 내용(Predicate)를 파라미터로 받는 함수를 제공한다.
  * findOne
  * findAll
  * count
  * exists 

  ~~~
  public interface MemberRepository extends JpaRepository<Member, Long>, MemberRepositoryCustom, QuerydslPredicateExecutor<Member> {

    List<Member> findByUsername(String username);
  }
  ~~~

  ~~~
  Iterable<Member> result = memberRepository.findAll(
            member.age.between(20, 40) // QMember.member
                .and(member.username.eq("member2")));
  ~~~

* 묵시적 join만 가능하기에 outer join이 불가능하다.
* 클라이언트가 Querydsl 코드에 의존적이다.
  * 서비스 코드가 querydsl 기술에 의존하게 되어 기술이 바뀔 경우 서비스 게층 또한 변화가 크다


## Querydsl - web 지원
* [공식 문서](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#core.web.type-safe)
* @QuerydslPredicate(root = Entity.class)를 선언하면 parameter로 넘어온 값을 Predicate로 바인딩 해준다.
* 단순한 조건만 가능하다. (eq(), contains(), in())
* 복잡한 조건을 위한 커스텀하기 어렵다. (`QuerydslBinderCustomizer<T>`로 커스텀할 수 있으나 복잡하고 명시적이지 않기에 추천하지 않음)
* 컨트롤러의 파라미터로 @QuerydslPredicate 어노테이션을 사용하여 Predicate타입으로 바인딩하기 때문에 컨트롤러 계층이 querydsl에 의존하게 된다.

## QuerydslRepositorySupport
* Spring Data JPA와 querydsl을 함께 사용하기 위해 Spring Data JPA가 지원하는 추상 클래스
* QuerydslRepositorySupport를 사용하면 JPAQueryFactory와 다르게 from()으로 시작한다.

### 장점
* Spring Data가 지원하는 페이징을 편리하게 변환 가능하다.
* getQuerydsl()이라는 메소드를 제공하는데, 이는 spring data jpa가 제공하는 Querydsl라는 헬퍼 클래스를 반환한다.
* Querydsl 객체는 applyPagination() 메소드를 제공하는데 이는 Pageable을 인자로 받아 내부에서 페이징한다.
* 위에서 메소드 체인으로 적용했던 offset(), limit() 코드가 줄어드는 장점이 있다.

### 단점
* Sort는 적용할 시 오류가 발생한다.
* 또한 from()으로 시작하기에 JPAQueryFactory 보다 명시적이지 않다.

## Custom QuerydslRepositorySupport
* QuerydslRepositorySupport의 단점을 보완
* [Custom QuerydslRepositorySupport](https://github.com/hjhng125/querydsl/blob/master/src/main/java/me/hjhng125/querydsl/repository/Querydsl4RepositorySupport.java)

### 장점
* spring data가 제공하는 페이징을 편리하게 해준다.
* 페이징과 카운트 쿼리를 분리할 수 있다. 
* spring data가 제공하는 Sort를 지원한다.
* select(), selectFrom()으로 시작할 수 있다. 
* EntityManager, QueryFactory를 제공한다.

<hr>

참고: [실전! Querydsl - 인터페이스 지원 - QuerydslPredicateExecutor](https://www.inflearn.com/course/Querydsl-%EC%8B%A4%EC%A0%84/lecture/30155?tab=note)

참고: [실전! Querydsl - Querydsl Web 지원](https://www.inflearn.com/course/Querydsl-%EC%8B%A4%EC%A0%84/lecture/30156?tab=note&mm=null)

참고: [실전! Querydsl - 리포지토리 지원 - QuerydslRepositorySupport](https://www.inflearn.com/course/Querydsl-%EC%8B%A4%EC%A0%84/lecture/30157?tab=note&mm=null)

참고: [실전! Querydsl - Querydsl 지원 클래스 직접 만들기](https://www.inflearn.com/course/Querydsl-%EC%8B%A4%EC%A0%84/lecture/30158?tab=note&mm=null)