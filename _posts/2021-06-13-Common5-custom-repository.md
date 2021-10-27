---
title:  "Spring Data Common - Custom Repository"

categories:
  - spring-data-jpa
tags:
  - JPA
  - Spring
comments: true
last_modified_at: 2021-07-15
---

### 쿼리 메소드(CREATE, USE_DECLARED_QUERY)로 해결할 수 없는 경우 직접 코딩으로써 구현
* custom 리포지토리 인터페이스 정의 
* custom 리포지토리 인터페이스에 새로운 기능 추가
* 만약 spring data기 지원하는 기능의 성능이 마음에 들지 않은 경우 재정의 가능
  * Spring Data JPA는 항상 유저가 재정의한 메서드에 가장 높은 우선순위를 제공한다.
  * JpaRepository의 기본 구현체인 SimpleJpaRepository 클래스의 delete(), deleteAll() 메서드
  * Entity의 컬렉션을 한번에 처리하는 것이 아닌 delete() 메서드를 순회하며 처리한다.
  * delete() 메서드는 영속성 컨텍스트 내에 해당 엔터티가 있는 경우 바로 em.remove()를 통해 removed 상태로 만들고,
  * 없으면 detached 상태로 판단하여 em.merge()를 실행하여 영속화한 후 removed 상태로 만든다.
  * 엔터티 매니저의 remove()를 사용하는 이유는 cascade를 위함이다. - 연관된 엔티티의 데이터를 같이 지우기 위해
* 인터페이스 구현 클래스 만들기 (기본 접미어는 Impl이며, @EnableJpaRepositories의 repositoryImplementationPostfix 옵션으로 수정 가능)
* 엔터티 리포지토리에 커스텀 리포지토리 인터페이스 추가

### Spring Data JPA가 지원하는 delete() 메서드
~~~
public void delete(T entity) {
    Assert.notNull(entity, "Entity must not be null!");
    if (!this.entityInformation.isNew(entity)) {
        Class<?> type = ProxyUtils.getUserClass(entity);
        T existing = this.em.find(type, this.entityInformation.getId(entity));
        
        if (existing != null) {
            this.em.remove(this.em.contains(entity) ? entity : this.em.merge(entity));
        }
    }
}
~~~

### delete 새로 구현학기
~~~
public interface PostCustomRepository<T> {
  
    void delete(T entity);
}
~~~
~~~
@Transactional(readOnly = true)
public class PostCustomRepositoryImpl implements PostCustomRepository<Post>{

    private final EntityManager entityManager;

    public PostCustomRepositoryImpl(EntityManager entityManager) {
        this.entityManager = entityManager;
    }

    @Override
    @Transactional
    public void delete(Post post) {
        System.out.println("custom delete");
        entityManager.remove(post);
    }
}
~~~
~~~
public interface PostJpaRepository extends JpaRepository<Post, Long>, PostCustomRepository<Post> {
}
~~~

참고: [https://www.inflearn.com/course/스프링-데이터-jpa](https://www.inflearn.com/course/스프링-데이터-jpa)