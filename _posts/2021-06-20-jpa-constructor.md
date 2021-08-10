---
title:  "JPA와 생성자"
excerpt: "JPA와 생성자가 필요한 이유"

categories:
  - jpa
tags:
  - Spring

last_modified_at: 2021-06-20
---

> Entity Class에 테스트 중 필요한 생성자를 만들어 사용하던 중 에러를 발견하게 되었다. <br>
이 에러에 대해 알아본 내용을 아래 기술할 것이다.

## Entity는 기본 생성자가 필수이다.
* 조회 시 DB값을 필드에 매핑할 때 `Reflection`을 사용

### Java Reflection API
* 구체적인 class의 타입을 알지 못해도 해당 클래스의 정보(메소드, 타입, 변수 등)에 접근할 수 있도록 도와주는 API

## 기본 생성자는 protected를 사용하자.
* JPA는 매핑된 Entity 를 조회할 때 `Eager`, `Lazy` 두가지 옵션이 있다.
* FetchMode가 `Lazy(지연로딩)`일 경우 지금 당장 필요로 하지 않는 연관관계에 `Proxy 객체`를 만들어 넣게 된다.
* Entity의 프록시는 `상속`을 통해 만들어 지기 때문에 접근 제한자는 `public, protected`여야만 한다.
* 안정성 측면에서 더 작은 scope인 `protected`를 사용하는 것이 권장된다.