---
title:  "Junit5 Assertion"
excerpt: "Junit5 Assertion에 대하여 공부한 내용을 기술합니다."

categories:
  - Test
comments: true
last_modified_at: 2022-09-10T03:50:000-05:00
---

## Assertion

```
@Test
@DisplayName("스터디 객체 만들기")
void create_new_study() {
    Study study = new Study(10);

    assertAll(
            () -> assertNotNull(study),
            () -> assertEquals(DRAFT, study.status(), "스터디를 처음 만들면 상태값이 DRAFT 이다."),
            () -> assertEquals(DRAFT, study.status(), () -> "스터디를 처음 만들면 상태값이 " + DRAFT + " 이다."), // lambda를 사용하면 문자열 연산이 실패했을때만 수행된다.
            () -> assertTrue(study.limit() > 0, "스터디 정원은 항상 양수이다.")
    );

    System.out.println("create1");
}

@Test
@Disabled
void create_new_study_again() {
    Study study = new Study(-10);
    assertNotNull(study);
    System.out.println("create2");
}

@Test
void throw_exception_test() {
    IllegalArgumentException illegalArgumentException = assertThrows(IllegalArgumentException.class, () -> new Study(-10));
    String errorMessage = illegalArgumentException.getMessage();

    assertEquals("limit은 0보다 커야한다.", errorMessage);
}

@Test
void timeout_test() {
    assertTimeout(Duration.ofMillis(100), () -> {
        Thread.sleep(300); // 이 코드 블럭이 실행된 후 시간을 계산하고 비교하고자 하는 timeout 시간과 비교한다.
        new Study(10);
    });
}

@Test
void timeout_preemptively_test() {
    assertTimeoutPreemptively(Duration.ofMillis(100), () -> {
        Thread.sleep(300); // 위와 다르게 설정한 timeout 시간이 도래하면 바로 예외를 발생시키고 테스트를 종료한다.
        new Study(10);
    });

    // TODO ThreadLocal - 람다는 별도의 쓰레드에서 실행될 수 있기 때문에 ThreadLocal을 사용하는 코드는 원하는 결과를 얻을 수 없을 수 있다.
}
```

* assertEquals - 실제 값이 기대값과 같은지 확인
* assertNotNull - 값이 not null 인지 확인
* assertTrue - 값이 true 인지 확인
* assertAll - 모든 구문 확인
* assertThrows - 예외 발생 확인
* assertTimeout - 특정 시간 안에 실행이 완료되는지 확인

위의 메서드들은 Supplier<String> 타입의 인스턴스를 람다 형태로 제공할 수 있다.
* 복잡한 메세지를 생성해야 하는 경우 실패한 경우에만 해당 메세지를 만들 수 있도록 지연시킨다.

AssertJ, Hamcrest, Truth 등의 라이브러리를 사용할 수도 있음.
* AssertJ, Hamcrest는 spring-boot-starter-test에 들어있음.


### 참고
[더 자바, 애플리케이션을 테스트하는 다양한 방법](https://www.inflearn.com/course/the-java-application-test)