---
title:  "JPA3 - Value 타입 맵핑"
excerpt: "JPA Value 타입 맵핑에 대하여 기술합니다."

categories:
  - spring-data-jpa
tags:
  - JPA
  - Spring
last_modified_at: 2021-06-01
---
 
### Entity 타입과 Value 타입의 구분
* 식별자가 있는가 (`@Id`) 
* 독립적으로 존재해야 하는가

### Value 타입 종류 : properties (Entity type에 종속적이다.)
* 기본 타입(String, Date, Boolean)
* Composite Value 타입
* Collection Value 타입
  * 기본 타입의 collection
  * Composite 타입의 collection

### Composite Value 타입 맵핑
* @Embaddable
* @Embadded
* @AttributeOverrides
* @AttributeOverride

![1](/assets/images/Embaddable.png)
![1](/assets/images/Embadded.png)


  참고: [https://www.inflearn.com/course/스프링-데이터-jpa](https://www.inflearn.com/course/스프링-데이터-jpa)