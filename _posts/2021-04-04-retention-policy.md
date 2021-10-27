---
title:  "Retention Policy"
excerpt: "Retention Policy에 대하여 공부한 내용을 기술합니다."

categories:
  - spring
tags:
  - annotation
comments: true
last_modified_at: 2021-04-04
---

개발을 하며 Custom annotation을 만들때 @Retention, @Target 어노테이션을 자주 접하게 된다.
그 동안 사용하며 알고 있다고 생각했지만 막상 명확하게 설명하려니 정리가 잘 되지 않아 정리한다.


* @Retention
  * 이 어노테이션은 선언하고 있는 어노테이션을 어느 시점까지 유지할 지 알려주는 어노테이션이다.
  * RetentionPolicy.RUNTIME
    - 런타임시 까지 유지한다.
    - 어노테이션을 통해 특정 동작을 하고 싶다면 RUNTIME시점까지 유지해야한다.
  * RetentionPolicy.CLASS
    - 클래스 파일까지 유지한다.
    - 이 옵션이 Retention의 Default이다.
    - 런타임시 클래스를 메모리로 로드하면 사라진다.
  * RetentionPolicy.SOURCE
    - 소스코드에서만 유지한다.
    - 컴파일 시 사라지며 특정 로직없이 주석으로만 사용할 경우 사용한다.


@Target은 다음편에 계속
