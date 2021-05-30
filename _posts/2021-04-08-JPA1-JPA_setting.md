---
title:  "JPA - 초기 설정"
excerpt: "JPA 초기 설정한 것에 대하여 기술합니다."

categories:
  - 스프링-데이터-jpa
tags:
  - JPA
  - Spring
last_modified_at: 2021-05-30
---
 
spring-boot-starter-data-jpa 의존성이 추가되면 디비 커넥션 풀 설정을 해줌 
 - 커넥션 풀을 생성하기 위해 어떤 디비에 붙을 지 커넥션 정보를 알려주어야함.

### JDBC 설정
* spring.datasource.url 
* spring.datasource.username
* spring.datasource.password

### Hibernate를 이용한 DB 초기화  
* spring.jpa.hibernate.ddl-auto
  - @Entity가 명시된 클래스를 scan하여 어플리케이션 로드 시점에 ddl문을 생성하여 DB에 적용해준다.
  - none: 아무것도 하지않음.
  - update: 스키마를 새로 생성하지 않기에 데이터가 지워지지 않으며, 매핑할때 추가할 컬럼이 있으면 반영함. 하지만 컬럼을 여러번 수정되면 테이블이 지저분해질 우려가 있다. domain의 매핑 정보를 변경해도 적용되지 않았다면, 이미 컬럼이 존재하고 있기 때문이다. 컬럼이 이미 존재하면 반영되지 않는다.

  - validate: 이미 스키마가 만들어져 있다고 가정하고, 매핑해야할 정보가 데이터베이스 잘 매핑이 되는지 검증하는 옵션. 매핑이 불가능하다고 판단되면 구동 시 에러를 뱉음. 상용 환경에 적합
  - create: 매 실행마다 스키마를 새로 생성
  - create-drop: 매 실행마다 스키마를 생성하며 sessionFactory 종료 시 drop 함.

  - embeded db를 사용할 경우 기본값은 create-drop이기에 h2와 같은 db를 사용할 경우 sessionFactory가 재구동되면 데이터는 사라진다.
  - 이 외의 db를 사용하는 경우 기본값은 none이다.
* Hibernate는 classpath의 루트에 `import.sql` 파일이 존재하면 어플리케이션 구동 시 자동으로 실행한다.

### Spring JDBC를 이용한 DB 초기화  
* Spring JDBC는 datasource 초기화 특성을 갖고 있다.
* Spring Boot는 기본적으로 classpath의 루트에 `schema.sql`, `data.sql` 파일이 존재한다면, 어플리케이션 구동 시 자동으로 실행한다.
* 또한 `schema-${platform}.sql`, `data-${platform}.sql` 파일이 존재한다면, 어플리케이션 구동 시 자동으로 데이터베이스 플랫폼에 맞춘 스크립트를 실행하며, `platform` 값은 `spring.datasource.platform` 값을 따른다.

### Entity 생성
* @Entity: 
  1. Relation과 맵핑할 도메인 선언 
  2. Entity의 이름은 보통 클래스 이름과 같게 하지만 값을 변경하면 Hibernate 내부에선 Entity의 이름이 변경된다.
  3. 즉 Jpa 안에서, 객체의 세상에서의 이름이 변경된 것이다.
* @Table:
  1. @Table의 Default는 Entity의 name과 같다. relation 세상에서의 이름
  2. Entity의 name을 지정하지 않으면 기본적으로 클래스 이름을 사용하기 때문에 클래스의 이름이 곧 Table의 이름이 된다. 이러면 생략 가능
* @Id: 
  1. DB 주키에 맵핑
  2. primary key mapping 자바의 primitive, wrapper 타입과 String, Date, BigDecimal, BigInteger 타입 가능
* @GeneratedValue: 
  * 값 자동 생성. 
  * Default는 DB에 따라 생성 전략이 다르다.(GenerationType.AUTO).
  * `IDENTITY`: 기본 키 생성을 DB에 위임. db의 auto_increment 동작이 수행됨.
  * `TABLE`: 키 생성 테이블을 사용. 키 생성 전용 테이블을 만들고, 이름과 값으로 사용할 컬럼을 만들어 DB 시퀀스처럼 동작하게 하는 전략
  * `SEQUENCE`: 
    1. DB 시퀀스를 사용해 기본 키 할당. DB Sequence object 사용.
    2. db 시퀀스는 유일한 값을 순서대로 생성하는 db object
    3. @SequenceGenerator에 sequenceName 속성을 추가해야 한다.
  * `AUTO`: 데이터베이스 벤더에 의존하지 않고, 벤더에 따라 타입을 지정 ex) oracle은 SEQUENCE.
* @Column: 
  1. relation의 컬럼과 매핑
  2. 생략할 경우 Entity의 property의 이름으로 맵핑함.


### 자동 설정 : HibernateJpaAutoConfiguration
* 컨테이너가 관리하는 EntityManager (프록시) 빈 설정
* PlatformTransactionManager 빈 설정

### @PersistenceContext
* Jpa의 이 어노테이션을 사용하여 EntityManager 빈을 주입받을 수 있음.
* EntityManager는 Jpa의 핵심 클래스
* Entity를 영속화할 수 있게 해줌.
![1](/assets/images/jpa_persist.png)
* Jpa는 Hibernate를 사용하기 때문에 Hibernate API를 사용할 수 있다. 
* Hibernate의 핵심: Session
![1](/assets/images/hibernate_api.png)

### @Transactional
* EntityManager와 관련된 모든 오퍼레이션들은 한 트랜잭션 안에서 일어나야한다.
* 스프링이 제공해주는 트랜잭션 매니저 기능을 사용하기 위해선 
@Transactional 어노테이션을 사용하면된다.
* 이 어노테이션을 클래스에 붙이면 해당 클래스의 모든 메소드에 적용되면 개별적으로 메소드에 붙이면 해당 메소드에만 적용할 수 있다.


  참고: [https://www.inflearn.com/course/스프링-데이터-jpa](https://www.inflearn.com/course/스프링-데이터-jpa)