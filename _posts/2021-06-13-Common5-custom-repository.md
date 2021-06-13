---
title:  "Spring Data Common - Custom Repository"

categories:
  - 스프링-데이터-jpa
tags:
  - JPA
  - Spring
last_modified_at: 2021-06-13
---

### 쿼리 메소드(CREATE, USE_DECLARED_QUERY)로 해결할 수 없는 경우
* repsitory interface에 기능 추가
* 만약 기존의 기능의 성능이 마음에 들지 않은 경우 재정의 가능
  * custom 리포지토리 인터페이스 정의
  * 인터페이스 구현 클래스 만들기 (기본 접미어는 Impl이며, @EnableJpaRepositories의 repositoryImplementationPostfix 옵션으로 수정 가능)
  * 엔터티 리포지토리에 커스텀 리포지토리 인터페이스 추가

 

참고: [https://www.inflearn.com/course/스프링-데이터-jpa](https://www.inflearn.com/course/스프링-데이터-jpa)