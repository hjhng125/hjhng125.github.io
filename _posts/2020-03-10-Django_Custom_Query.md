---
title:  "Django Model Manager과 QuerySet"
excerpt: "Django Model Manager과 QuerySet"

categories:
  - Django
tags:
  - Django
  - ORM
  - Model Manager
  - QuerySet
last_modified_at: 2020-03-07T021:13:00
---
# Django - QuerySet
쿼리셋은 전달받은 모델의 객체 목록.
데이터베이스로부터 데이터를 읽고, 필터를 하거나 정렬할 수 있다.
 
# Django - Model Manager
모델은 데이터 정보를 정의한 객체다.
일반적으로 모델은 데이터베이스 테이블과 매핑된다.
ORM과 QuerySet의 목적은 Django를 데이터베이스에 연결하여 데이터를 저장하는 것이다.

### Default Model Manager - objects
* 모델 매니저는 데이터베이스 쿼리와 연동되는 인터페이스이다.
* 각 모델은 애플리케이션에서 최소 하나의 매니저를 가진다. 
* default 모델 매니저의 이름은 `objects`이다.

### Custom Model Manager
* 커스텀 모델 매니저를 구현하는 방법은 크게 두 가지가 있다.
    * 모델 매니저를 매번 추

