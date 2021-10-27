---
title:  "JPA2 - 엔터티 타입 맵핑"
excerpt: "JPA 엔터티 타입 맵핑에 대하여 기술합니다."

categories:
  - spring-data-jpa
tags:
  - JPA
  - Spring
comments: true
last_modified_at: 2021-05-30
---
 
### Entity 생성
* @Entity: 
  1. Relation과 맵핑할 도메인 선언 
  2. Entity의 이름은 보통 클래스 이름과 같게 하지만 값을 변경하면 Hibernate 내부에선 Entity의 이름이 변경된다.
  3. 즉 Jpa 안에서, 객체의 세상에서의 이름이 변경된 것이다.
* @Table:
  1. 테이블과 관련된 설정을 작성하기 위한 어노테이션
  2. @Table의 Default는 Entity의 name과 같다. relation 세상에서의 이름
  3. Entity의 name을 지정하지 않으면 기본적으로 클래스 이름을 사용하기 때문에 클래스의 이름이 곧 Table의 이름이 된다. 이러면 생략 가능
* @Id: 
  1. DB 주키에 맵핑
  2. primary key mapping 자바의 primitive, wrapper 타입과 String, Date, BigDecimal, BigInteger 타입 가능
  3. 필자는 새로 만든 객체와 0인 id를 갖는 객체를 구분하기 위해 wrapper타입을 씀.
* @GeneratedValue: 
  * 값 자동 생성. 
  * Default는 DB에 따라 생성 전략이 다르다.(GenerationType.AUTO).
  * mysql은 `IDENTITY`, postgresql은 `SEQUENCE`
  * `IDENTITY`: 기본 키 생성을 DB에 위임. db의 auto_increment 동작이 수행됨.
    * 해당 값으로 선언한 경우, 트랜잭션 내의 쓰기 지연 기능을 사용할 수 없다.
    * 그 이유는 Persistence Context가 관리하는 상태로 만들려면 식별자가 필요하기 때문이며,
    * 이 전략은 DB에 키 생성을 위임했으므로 엔터티를 테이블에 저장해야 식별자를 구할 수 있다.
    * 따라서 EntityManager의 persist()를 호출하면 바로 insert 쿼리가 DB에 전달된다.
  * `TABLE`: 키 생성 테이블을 사용. 키 생성 전용 테이블을 만들고, 이름과 값으로 사용할 컬럼을 만들어 DB 시퀀스처럼 동작하게 하는 전략
  * `SEQUENCE`: 
    1. DB 시퀀스를 사용해 기본 키 할당. DB Sequence object 사용.
    2. db 시퀀스는 유일한 값을 순서대로 생성하는 db object
    3. @SequenceGenerator에 sequenceName 속성을 추가해야 한다.
  * `AUTO`: 데이터베이스 벤더에 의존하지 않고, 벤더에 따라 타입을 지정 ex) oracle은 SEQUENCE.
* @Column: 
  1. relation의 컬럼과 매핑
  2. 생략할 경우 Entity의 property의 이름으로 맵핑함.
  3. unique: 중복 데이터 방지
  4. nullable: null 값 지원
  5. length: 길이 값 제한
  6. columnDefinition: sql 문법을 통한 맵핑 지원

* @Temporal
  1. 날짜 데이터를 맵핑을 지원
  2. Jpa2.1까지는 Calendar, Date 타입만 지원했다.(Java8 이전에 Jpa2 스펙이 나옴)
  3. 그 이후부터 Java8에서 지원하는 LocalDate, LocalDatetime을 사용할 때는 생략가능

* @Transient: relation 컬럼과 매핑하지 않음.

* @Lob
  1. 가변 길이의 큰 데이터를 저장할 때 사용
  2. 일반적인 데이터베이스에 저장하는 길이인 255개 이상의 문자를 저장할 때 사용
  3. CLOB: 문자기반 데이터
  4. BLOB: 바이너리 데이터

  참고: [https://www.inflearn.com/course/스프링-데이터-jpa](https://www.inflearn.com/course/스프링-데이터-jpa)