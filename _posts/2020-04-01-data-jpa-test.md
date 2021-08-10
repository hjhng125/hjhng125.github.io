---
title:  "@DataJpaTest"
excerpt: "@DataJpaTest에 대하여 공부한 내용을 기술합니다."

categories:
  - spring
tags:
  - test
  - JUnit
last_modified_at: 2021-04-01
---

* DataJpaTest
    * Jpa 테스트를 위한 어노테이션
    * full auto-configuration을 하지 않으며 Jpa를 사용하여 데이터의 CRUD의 테스트가 가능하다.
    * 내장형 데이터베이스를 사용하여 실제 데이터베이스를 사용하지 않고 테스트할 수 있도록 지원한다.
    * @Entity 어노테이션이 선언된 클래스를 스캔하여 스프링 데이터 JPA 저장소를 구성한다.
    * @Transactional 어노테이션을 포함하여 각 메소드의 transaction을 지정하지 않아도된다.
    * transaction을 사용하지 않으려면 @Transactional(propagation = Propagation.NOT_SUPPORTED)를 선언한다.
    * 실제 DB를 테스트하고 싶은 경우 @AutoConfigureTestDatabase 어노테이션을 사용한다.
    * 아래와 같이 Replace.None으로 설정하면 실제 DB를 사용할 수 있으며 @ActiveProfiles 어노테이션으로 환경별 설정도 가능하다.

![1](/assets/images/BaseDataJpaTest.png)

    