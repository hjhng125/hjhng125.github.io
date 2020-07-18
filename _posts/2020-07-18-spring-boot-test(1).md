---
title:  "Spring Boot Test(1)"
excerpt: "Spring Boot Test에 대하여 공부한 내용을 기술합니다."

categories:
  - Spring
tags:
  - test
  - JUnit
  - Unit Tests
last_modified_at: 2020-07-18T0018:51:00
---

## JUnit
- Java에서 독립된 단위테스트(Unit Test)를 지원해주는 프레임워크.
- `단정(assert) 메서드`로 테스트 케이스 수행 결과를 판별한다.
(ex: assertEquals(예상값, 실제값))
- `어노테이션(@blahblah)`을 통해 간결하게 사용가능하다.

- `@Before` 어노테이션을 이용하면 테스트 클래스 안의 메소드들이 테스트 전에 실행할 코드를 정의할 수 있다.
- `@BeforeClass` 어노테이션을 이용하면 메소드들이 몇번 실행되던 간에 해당 클래스에서 `한번`만 실행하도록 할 수 있다.

- `@Test` 메서드가 호출될 때 마다 새로운 인스턴스를 생성하여 독립적인 테스트가 이루어지게 한다. 
  * `timeout`을 어노테이션 속성으로 선언하면 수행시간을 제한할 수 있다.
  * `expected`를 어노테이션 속성으로 선언하면 해당 메소드가 선언된 `exception`을 발생시켜야 테스트가 성공되도록 할 수 있다.

- `@After` 어노테이션은 테스트 클래스 안의 메소드들이 테스트 후 실행할 코드를 정의한다.
- `@AfterClass` 어노테이션은 메소드들이 몇번 실행되던 간에 해당 클래스에서 `한번`만 실행된다.

## Spring Boot Test Dependency
- 스프링부트는 애플리케이션 테스트를 위해 두가지 모듈을 제공한다.
  * spring-boot-test : 테스트를 위한 핵심 기능을 포함한다.
  * spring-boot-test-configuration : 테스트를 위한 AutoConfiguration을 제공
  * spring initailizer를 통하여 프로젝트를 생성하면 테스트를 위한 디펜던시가 추가되어 있을 것이다.
 ![1](/assets/images/spring-boot-test.png)

- spring-boot-starter-test 에는 다음 라이브러리들이 포함되어 있다.
  * JUnit5
  * Spring test 및 Spring boot test
  * AssertJ
  * Hamcrest
  * Mockito
  * JSONassert
  * JsonPath

- Spring boot Unit tests
  * 단위 테스트는 다른 디펜던시가 전혀 없는 상황에서 실행할 수 있는 테스트를 의미한다.
  * 스프링부트에서 제공하는 @SpringBootTest 어노테이션을 사용하는 것은 스프링을 실행시키기에 스프링 의존성이 생기게된다. 따라서 이를 사용하는 것은 `단위테스트`라기 보단 `통합테스트`에 가깝다.
  * Spring을 사용하지 않기때문에 스프링에서 제공하는 `@Autowired`는 사용할 수 없다. 따라서 `field injection`이 아닌 `constructor injection`을 사용한다.

- 테스트 코드 작성
  * 테스트 클래스는 반드시 publicdmfh 선언한다.
  * 클래스명은 관례상 테스트할 클래스명 + Test로 한다.
  * @Test를 선언한 테스트 메서드는 JUnit이 알아서 실행할 수 있게 한다.