---
title:  "Spring Data Common - Repository Interface 정의"

categories:
  - 스프링-데이터-jpa
tags:
  - JPA
  - Spring
last_modified_at: 2021-06-10
---

JpaRepository에 정의된 메소드를 전부 가져오는 것이 아닌 사용할 기능을 직접 정의하고 싶을때는 아래와 같이 할 수 있다.

![1](/assets/images/RepositoryDefinition.png)

* 메서드의 기능을 Spring Data Jpa가 구현할 수 있는 것이라면 구현하여 제공해준다.
* RepositoryDefinition 어노테이션을 통해 domainClass를 위와 같이 정의
* 아래의 메서드는 spring data가 구현하여 제공해준다.

![1](/assets/images/RepositoryDefinitionMethod.png)

만약 위 처럼 새로 정의한 여러 Repository가 중복된 기능을 제공한다면 공통 인터페이스를 정의할 수 있다.



참고: [https://www.inflearn.com/course/스프링-데이터-jpa](https://www.inflearn.com/course/스프링-데이터-jpa)