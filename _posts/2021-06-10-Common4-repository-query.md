---
title:  "Spring Data Common - 쿼리만들기"

categories:
  - spring-data-jpa
tags:
  - JPA
  - Spring
comments: true
last_modified_at: 2021-06-11
---

### 스프링 데이터 저장소의 쿼리 만드는 방법
* 메소드의 이름을 분석하여 쿼리를 생성(CREATE)
![1](/assets/images/create.png)
* 미리 정의해 둔 쿼리 찾아 사용(USE_DECLARED_QUERY)
  * JPA의 구현체마다 구현이나 동작하는 방식이 다르지만 spring-data-jpa가 기본적으로 Hibernate를 사용하기 때문에 HQL을 기본으로 받아들임
  * sql을 사용하고 싶다면 nativeQuery옵션을 true로 줘야한다.
  * @Query, @Procedure, @NamedQuery의 우선순위를 가짐(셋 다 선언되어 있을 경우) - 이렇게 코딩할 일 없겠지만..
![1](/assets/images/use_declared_query.png)
* 미리 정의한 쿼리 찾아보고 없으면 만듬(CREATE_IF_NOT_FOUND)
* EnableJpaRepositories 어노테이션의 queryLookupStrategy로 세팅가능하며 default는 CREATE_IF_NOT_FOUND이다.

### 쿼리 만드는 방법
```
리턴타입 {접두어}{도입부}By{프로퍼티 표현식}(조건식)[(And|Or){프로퍼티 표현식}(조건식)]{정렬 조건} (매개변수)
```

* 접두어: Find, Get, Query, Count, ...
* 도입부: Distinct, First(N), Top(N)
* 프로퍼티 표현식: Person.Address.ZipCode => find(Person)ByAddress_ZipCode(...)
* 조건식: IgnoreCase, Between, LessThan, GreaterThan, Like, Contains, ...
* 정렬 조건: OrderBy{프로퍼티}Asc/Desc
* 리턴 타입: `E`, `Optional<E>`, `List<E>`, `Page<E>`, `Slice<E>`, `Stream<E>`
* 매개변수: Pageable, Sort

### 쿼리 찾는 방법
* 메소드 이름으로 쿼리를 표현하기 힘든 경우에 사용.
* 저장소 기술에 따라 다름.
  * JPA: @Query @NamedQuery

 

참고: [https://www.inflearn.com/course/스프링-데이터-jpa](https://www.inflearn.com/course/스프링-데이터-jpa)