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
    * MockMvc Bean을 주입받아 테스트하기 어려운 Controller layer를 쉽게 테스트할 수 있게 해준다.
    * 요청과 응답을 Mock 기반으로 테스트하기 때문에 상용 환경에서는 제대로 동작하지 않을 수도 있다.
    * Junit5부터 ExtendWith(SpringExtension.class) 어노테이션을 포함하고 있다. - 테스트 확장을 위한 어노테이션

![1](/assets/images/@WebMvcTest.png)

* perform()
  - 요청을 전송하는 역할. ResultActions 객체를 리턴한다.
  - ResultActions객체는 리턴값을 검증하고, 확인하는 andExpect() 메소드를 제공한다.

* 요청 메소드
  - get()
  - post()
  - put()
  - delete() 
  - patch()
  - 추가 하위 메소드
    - param() - key, value 쌍의 파라미터를 전달할 수 있게 해줌
    - params() - MultiValueMap 타입으로 여러 파라미터를 전달
    - content() - 요청 바디를 objectMapper를 통해 string으로 변환하여 전달.
    - contentType()
    - accept()
    - header()
    - cookie()
    - session()
    - principal() 등이 있음
  

* andExpect() - 응답을 검증
  * jsonPath() - json 형식의 responseBody의 subset 검증 - ("$.blah").value()
  * status() - 상태코드 검증
  * view() - 뷰 검증
  * header() - 응답 헤더 검증
  * content() - 응답 바디 검증
  * cookie() - 응답 쿠키 검증
  * redirect() - 리다이렉트 주소 검증
  * model() - 모델 검증

* andDo(print()) - 요청 및 응답의 메세지 확인