---
title:  "@WebMvcTest"
excerpt: "@WebMvcTest에 대하여 공부한 내용을 기술합니다."

categories:
  - Spring
tags:
  - test
last_modified_at: 2021-03-31
---

* WebMvcTest
    * MVC만를 테스트하기 위한 어노테이션으로 컨트롤러를 테스트하는데 사용된다.
    * full auto-configuration을 하지 않으며 Controller, ControllerAdvice, JsonComponent, Converter, GenericConverter, Filter, WebMvcConfigurer, HandlerMethodArgumentResolver 관련 Bean만 생성한다.
    * Service, Repository, Component로 선언된 bean은 생성하지 않는다.

* DataJpaTest
    * Jpa 테스트를 위한 어노테이션
    * full auto-configuration을 하지 않으며 Jpa를 사용하여 데이터의 CRUD의 테스트가 가능하다.
    * 내장형 데이터베이스를 사용하여 실제 데이터베이스를 사용하지 않고 테스트할 수 있도록 지원한다.