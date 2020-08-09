---
title:  "Django DB 최적화 #2"
excerpt: "Django DB 최적화 에 대하여"

categories:
  - Django
tags:
  - Django
  - QuerySet 
  - ORM
  - 최적화
last_modified_at: 2020-03-07
---

지난 포스트에 이은 Django DB 최적화 2편입니다.

## Basic
Django에서 진행할 최적화를 하기 전, DB의 index나, field를 잘 정의하는 DB 최적화가 우선적으로 수행되어야 한다.

## QuerySet의 특징
이전 포스트에서 언급했듯, QuerySet은 Lazy한 특성을 갖고 있다.

* 코드 상에서 QuerySet을 생성하는 작업은 DB에 아무런 영향을 끼치지 않는다.
* DB의 query는 QuerySet이 계산될 때까지 실제로 query를 생성하지 않는다.
* 실제로 계산되는 시점
    * 반복 (Iteration)
    * 슬라이싱 (Slicing)
    * 피클링/캐싱 (Pickling/Caching)
        * 피클링 : 캐싱의 타이밍을 임의로 정의하는 기법
    * repr(), len(), list(), bool() etc.

## DB에서 작업 실행
