---
title:  "Spring Data Common - 기본 리포지토리 커스텀"

categories:
  - spring-data-jpa
tags:
  - JPA
  - Spring
last_modified_at: 2021-07-15
---

### 쿼리 메소드(CREATE, USE_DECLARED_QUERY)로 해결할 수 없는 경우 직접 코딩으로써 구현
* custom 리포지토리 인터페이스 정의 
* custom repsitory interface에 기능 추가
* 만약 spring data기 지원하는 기능의 성능이 마음에 들지 않은 경우 재정의 가능
* 인터페이스 구현 클래스 만들기 (기본 접미어는 Impl이며, @EnableJpaRepositories의 repositoryImplementationPostfix 옵션으로 수정 가능)
* 엔터티 리포지토리에 커스텀 리포지토리 인터페이스 추가

 

참고: [https://www.inflearn.com/course/스프링-데이터-jpa](https://www.inflearn.com/course/스프링-데이터-jpa)