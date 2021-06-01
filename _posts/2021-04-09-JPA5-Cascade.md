---
title:  "JPA5 - Cascade"
excerpt: "JPA Cascade에 대하여 공부한 내용을 기술합니다."

categories:
  - 스프링-데이터-jpa
tags:
  - JPA
  - Spring
last_modified_at: 2021-06-02
---

# Cascade
  * OneToMany, ManyToOne 어노테이션에서 설정 가능
  * 관계가 매핑되어 있는 엔터티에 엔터티의 상태 변화를 전파시키는 옵션
  * default는 {} 이다.
  * 상태란?
    1. Transient
      - JPA가 모르는 상태
      - 계속 사용하지 않게되면 결국 GC의 대상이 될 것이다.
    2. Persistent
      - JPA가 관리중인 상태 (1차 캐시, Dirty Checking, Write Behind)
      - 영속성 컨텍스트 내부에는 엔터티를 보관하는 장소가 있는데 이를 1차 캐시라 한다.
      - Persistent 상태가 된다고 (save() 실행) 바로 쿼리가 실행되는 것이 아닌 commit이 될 때 쿼리가 실행된다. 이것이 Write Behind이다.
      - Persistent 상태인 객체는 조회 시 select 문을 통해 값을 가져오는 것이 아닌 1차 캐시에서 바로 가져온다.
      - Persistent 상태인 객체는 setter를 통한 수정 시 1차 캐시 내부의 엔터티가 수정된 것을 감지하여 update에 관한 메소드를 작성하지 않아도 commit 시 update 쿼리가 실행되는데 이를 Dirty Checking이라 한다. (set을 수십번해도 실제 entity의 값이 변경되지 않으면 update 쿼리는 실행되지 않음)
    3. Detached
      - JPA가 더이상 관리하지 않는 상태
      - Transaction이 종료되고 난 후에도 Detached 상태가 된다.
      - 이 상태에선 Persistent 상태에서 JPA가 제공하던 기능을 사용할 수 없다.
    4. Removed
      - JPA가 관리하긴 하지만 삭제하기로 한 상태
      - 이 상태에서 commit되면 그때 delete 쿼리 실행됨.
  * ALL
    - 모든 CASCADE를 적용
  * PERSIST
    - 연관된 엔터티도 함께 Persistent 상태로 만든다.
  * MERGE
    - 연관된 엔터티도 함께 다시 Persistent 상태로 바꾼다. Detach -> Persistent
  * REMOVE
    - 연관된 엔터티도 함께 Remove 상태로 만든다.
  * REFRESH
    - 연관된 엔터티도 함께 다시 읽어온다.
  * DETACH
    - 연관된 엔터티도 함께 Detach 상태로 만든다.



  참고: [https://www.inflearn.com/course/스프링-데이터-jpa](https://www.inflearn.com/course/스프링-데이터-jpa)