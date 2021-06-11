---
title:  "Spring Data Common이란"

categories:
  - 스프링-데이터-jpa
tags:
  - JPA
  - Spring
last_modified_at: 2021-06-10
---

spring data는 하나의 프로젝트가 아니고 여러 SQL & NOSQL 저장소 지원 프로젝트의 묶음이다.
* spring data JPA
* spring data JDBC
* spring data KeyValue
* spring data MongoDB
* spring data Redis 
* etc..

spring data common은 각 저장소를 지원하는 프로젝트의 공통 기능을 제공한다.
리파지토리를 생성해서 빈으로 등록하는 방법, 쿼리 메소드를 만드는 기본 메커니즘이 들어있다.

spring data rest는 spring data 저장소들이 관리하는 데이터를 REST API로 웹으로 노출시켜주는 프로젝트로
하이퍼미디어 기반의 http resource로 데이터를 제공한다.

spring data jpa는 spring data common이 제공하는 기능에 JPA 관련 기능이 추가된 것이다.



참고: [https://www.inflearn.com/course/스프링-데이터-jpa](https://www.inflearn.com/course/스프링-데이터-jpa)