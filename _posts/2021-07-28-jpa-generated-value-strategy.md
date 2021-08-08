---
title:  "JPA key 생성전략 변경 사항"

categories:
  - JPA
tags:
  - Spring

comments: true

last_modified_at: 2021-07-28
---

최근 JPA에 대하여 공부를 할 때 DB 벤더를 MySQL로 설정할 경우 기본 키 생성 전략이 IDENTITY, Postgresql은 SEQUENCE라고 학습하였다.

학습할때는 Postgresql만 사용하여 그런가보다 하고 넘어갔었는데 실제 업무를 진행할때 MySQL로 확인해보니 Table 전략이 사용됨을 볼 수 있었다.

혹시 Spring Data JPA 버전이 올라가며 수정사항이 생긴건가 싶어 알아보았고 그 내용을 정리해볼것이다.

## JPA 키 생성전략
* Spring Data JPA에서는 엔터티 선언 시 키 생성 전략을 정의할 수 있는 `@GeneratedValue` 어노테이션을 지원한다.
* 키 생성 전략은 [여기](/스프링-데이터-jpa/JPA2-entity_type_mapping/)를 참고.

## GenerationType.AUTO?
* 이 enum 값은 데이터베이스 벤더에 따라 자동으로 키 생성 전략을 선택해주는 옵션이다.

## 왜 TABLE 전략이 사용되었는가?

### Spring Data JPA 버전 2.0 변경
* Spring Data JPA 2.0 버전부터 `spring.jpa.use-new-id-generator-mappings`의 옵션 기본값이 true로 변경되었다. [spring migration guide](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.0-Migration-Guide#id-generator)
  * 해당 옵션은 Hibernate의 새로운 key 생성 전략을 따라갈지 말지에 대한 옵션이다.
  * true인 경우 Hibernate 5 버전의 새로운 전략을 따르고
  * false인 경우 과거의 전략을 따른다.
* Spring Data JPA 2.5
![1](/assets/images/hibernateUseNewIdGeneratorMappings.png)

### Hibernate의 키 생성 전략 변경
* Hibernate 5.0 버전 이전엔 GenerationType이 `AUTO`인경우 mysql 기준 `IDENTITY` 옵션을 사용했으며
* Hibernate 5.0 버전부터는 `TABLE`을 사용한다.

### 결론
위의 내용을 토대로 결론을 내리자면 아래와 같다.
* Spring boot 2.5.1 버전을 사용했기 때문에 spring.jpa.use-new-id-generator-mappings 옵션이 true로 설정되어 Hibernate 5 버전의 키 생성전략을 따라갔다.
* Hibernate 5.4 버전을 사용하였고 해당 버전의 AUTO 값은 mysql의 키 생성 전략으로 TABLE을 사용한다.

* 만약 DB에 키 생성을 위임하고 싶은 경우 아래의 방법 중 하나를 선택하면 된다.
  * spring.jpa.use-new-id-generator-mappings를 false로 준다.
  * GeneratedValue(GenerationType.IDENETY)를 명시한다.
