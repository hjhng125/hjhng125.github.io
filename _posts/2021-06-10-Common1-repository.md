---
title:  "Spring Data Common - Repository"

categories:
  - spring-data-jpa
tags:
  - JPA
  - Spring
last_modified_at: 2021-06-10
---


### JpaRepository
* `JpaRepositor`y는 `PagingAndSortingRepository에서` JPA와 관련된 기능을 확장한 interface로, spring data jpa가 제공해준다.
* `PagingAndSortingRepository`부터 그 상위의 `CrudRepository`, `Repository`는 Spring data common에서 제공해준다.

* PagingAndSortingRepository : paging과 sorting을 지원
* CrudRepository : 기본적인 Create, Read, Update, Delete를 지원
* Repository : 마커 인터페이스로 실질적인 기능을 하진 않지만 Repository을 알려주는 마커 용도


![1](/assets/images/CommonRepository.png)
* @NoRepositoryBean는 이 인터페이스가 레파지토리의 용도로 사용되는 것이 아닌 Repository의 메서드를 정의하는 인터페이스라는 정보를 부여하며 
* 실제로 해당 인터페이스의 빈이 생성되는 것을 방지한다. 
* @NoRepositoryBean는 `Repository` 인터페이스 이하 중간 인터페이스에 모두 등록되어있다.



참고: [https://www.inflearn.com/course/스프링-데이터-jpa](https://www.inflearn.com/course/스프링-데이터-jpa)