---
title:  "Spring Boot Test"
excerpt: "Spring Boot Test에 대하여 공부한 내용을 기술합니다."

categories:
  - Spring
tags:
  - test
  - JUnit
  - Unit Tests
last_modified_at: 2020-07-18
---

## JUnit
- Java에서 독립된 단위테스트(Unit Test)를 지원해주는 프레임워크.
- `단정(assert) 메서드`로 테스트 케이스 수행 결과를 판별한다.
(ex: assertEquals(예상값, 실제값))
- `어노테이션(@blahblah)`을 통해 간결하게 사용가능하다.

- `@Before` 어노테이션을 이용하면 테스트 클래스 안의 메소드들이 테스트 전에 실행할 코드를 정의할 수 있다.
  * Junit5에선 @BeforeEach로 사용
- `@BeforeClass` 어노테이션을 이용하면 메소드들이 몇번 실행되던 간에 해당 클래스에서 `한번`만 실행하도록 할 수 있다.

- `@Test` 메서드가 호출될 때 마다 새로운 인스턴스를 생성하여 독립적인 테스트가 이루어지게 한다. 
  * `timeout`을 어노테이션 속성으로 선언하면 수행시간을 제한할 수 있다.
  * `expected`를 어노테이션 속성으로 선언하면 해당 메소드가 선언된 `exception`을 발생시켜야 테스트가 성공되도록 할 수 있다.

- `@After` 어노테이션은 테스트 클래스 안의 메소드들이 테스트 후 실행할 코드를 정의한다.
  * Junit5에선 @AfterEach로 사용
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

* Mock 객체란?
  * 가짜 객체
  * 개발을 하다보면 객체간의 의존성이 생기기 마련인데, 이는 완전한 유닛테스트를 하기 힘들다.
  * Mock 객체는 이러한 상황에 의존되는 객체를 대체해준다.

* 스프링부트에서 제공하는 `@SpringBootTest` 어노테이션을 사용하는 것은 스프링을 실행시키기에 `스프링 의존성`이 생기게된다. 
* 이는 스프링이 실행되며 모든 설정파일을 읽고, 선언된 모든 bean을 생성한다.
* 따라서 이를 사용하는 것은 `단위테스트`라기 보단 `통합테스트`에 가깝다.

* 단위 테스트는 다른 `의존성`이 전혀 없는 상황에서 실행할 수 있는 테스트를 의미한다.
* Spring을 사용하지 않기때문에 스프링에서 제공하는 `@Autowired`는 사용할 수 없다. 따라서 `field injection`이 아닌 `constructor injection`을 사용한다.
* 유닛 테스트를 지원하는 어노테이션으로는 @WebMvcTest, @DataJpaTest, @RestClientTest, @JsonTest 등이 있다.

* `@Ignore` 어노테이션은 해당 메서드의 테스트를 무시하고 지나가게 해준다.

* @ContextConfiguration
  * 자동으로 만들어줄 어플리케이션 컨텍스트의 설정파일 위치를 지정한다.
* @Import
  * @Congiguration 클래스를 import하는데 유용하댜.

* @RunWith 
  * Junit 테스트 러너 확장을 위한 annotation으로 Junit5부터 ExtendWith으로 대체되었다.
  * Junit은 매 테스트 메소드를 실행할 때마다 새로운 테스트 오브젝트를 만든다.
  * 스프링의 Junit 확장 기능은 테스트가 실행되기 전에 딱 한번만 애플리케이션 컨텍스트를 생성하고, 애플리케이션 컨텍스트는 테스트 오브젝트가 생성될 때마다 특별한 방법을 통하여 자신 스스로를 테스트 오브젝트의 필드에 주입한다.
  * ExtendWith(SpringExtension.class)는 Junit5가 제공하는 모든 테스트용 어노테이션에 포함되어 있어 선언을 생략할 수 있게 되었다.
* SpringRunner
  * 테스트 컨텍스트 확장 클래스로 테스트가 사용할 테스트용 어픞리케이션 컨텍스트를 만들고 관리해준다.
  * Junit5부터 SpringExtension으로 대체 되었다.
  
- 테스트 코드 작성
  * 테스트 클래스는 반드시 `public`으로 선언한다. - `JUnint`에서 접근 하기 위해
      * Junit5부터 public을 선언하지 않아도된다.
  * 클래스명은 관례상 테스트할 `클래스명 + Test`로 한다.
  * @Test를 선언한 테스트 메서드는 JUnit이 알아서 실행할 수 있게 한다.