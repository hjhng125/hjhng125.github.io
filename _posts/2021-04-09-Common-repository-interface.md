---
title:  "Spring Data Common - Repository Interface 정의"

categories:
  - 스프링-데이터-jpa
tags:
  - JPA
  - Spring
last_modified_at: 2021-06-03
---

JpaRepository에 정의된 메소드를 전부 가져오는 것이 아닌 사용할 기능을 직접 정의하고 싶을때는 아래와 같이 할 수 있다.

![1](/assets/images/RepositoryDefinition.png)

* 메서드의 기능을 Spring Data Jpa가 구현할 수 있는 것이라면 구현하여 제공해준다.
* 아래의 메서드는 Jpa가 구현하여 제공해준다.

![1](/assets/images/RepositoryDefinitionMethod.png)

만약 위 처럼 새로 정의한 여러 Repository가 중복된 기능을 제공한다면 공통 인터페이스를 정의할 수 있다.

![1](/assets/images/CommonRepository.png)

* @NoRepositoryBean는 이 인터페이스가 레파지토리의 용도로 사용되는 것이 아닌 Repository의 메서드를 정의하는 인터페이스라는 정보를 부여하며 
* 실제로 해당 인터페이스의 빈이 생성되는 것을 방지한다. 
* @NoRepositoryBean는 `Repository` 인터페이스 이하 중간 인터페이스에 모두 등록되어있다.

* 상속받은 `Repository` 인터페이스는 기능을 제공하는 것이 아닌 마크 인터페이스이다. 단지 Repository 용도로 사용될 것이라고 알릴뿐이다.

참고: [https://www.inflearn.com/course/스프링-데이터-jpa](https://www.inflearn.com/course/스프링-데이터-jpa)