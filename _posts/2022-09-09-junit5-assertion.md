---
title:  "Junit5 Assertion"
excerpt: "Junit5 Assertion에 대하여 공부한 내용을 기술합니다."

categories:
  - Test
comments: true
last_modified_at: 2022-09-10T03:50:000-05:00
---

## 조건에 따른 테스트 실행
특정 OS 혹은 특정 환경변수에 따라 테스트를 실행하는 방법은 아래와 같다.

```
@Test
@DisplayName("환경변수 TEST ENV가 LOCAL이면 실행 (assumeTrue)")
void env_test() {
    String testEnv = System.getenv("TEST_ENV");
    System.out.println(testEnv);
    assumeTrue("LOCAL".equalsIgnoreCase(testEnv));

    Study study = new Study(10);
    assertThat(study.limit()).isGreaterThan(0);
}

@Test
@DisplayName("환경변수 TEST ENV가 LOCAL이면 실행 (assumeThat)")
void env_test_again() {
    String testEnv = System.getenv("TEST_ENV");
    System.out.println(testEnv);
    assumingThat("LOCAL".equalsIgnoreCase(testEnv), () -> {
        Study study = new Study(10);
        assertThat(study.limit()).isGreaterThan(0);
    });
}

@Test
@EnabledOnOs(OS.MAC)
@DisplayName("MAC에서만 실행")
void os_test() {
    System.out.println("ON MAC");
}

@Test
@DisabledOnOs(OS.MAC)
@DisplayName("MAC이 아니어야 실행")
void os_no_test() {
    System.out.println("ON MAC");
}

@Test
@EnabledOnJre(value = {JRE.JAVA_8, JRE.JAVA_9, JRE.JAVA_10, JRE.JAVA_11})
@DisplayName("jre가 8, 9, 10, 11이면 실행")
void jre_test() {
    System.out.println("ON MAC");
}

@Test
@DisabledOnJre(value = {JRE.JAVA_8, JRE.JAVA_9, JRE.JAVA_10, JRE.JAVA_11})
@DisplayName("jre가 8, 9, 10, 11 아니면 실행")
void jre_no_test() {
    System.out.println("ON MAC");
}

@Test
@EnabledIfEnvironmentVariable(named = "TEST_ENV", matches = "LOCAL")
@DisplayName("환경변수 TEST ENV가 LOCAL이면 실행 annotation")
void enabled_if_environment_variable() {
    System.out.println("execute");
}

@Test
@DisabledIfEnvironmentVariable(named = "TEST_ENV", matches = "LOCAL")
@DisplayName("환경변수 TEST ENV가 LOCAL이면 실행 annotation")
void disabled_if_environment_variable() {
    System.out.println("execute");
}
```

* assumeTrue - 인자값이 참인 경우 이후 검증을 수행한다.
* assumeThat - 첫번째 인자값이 참인 경우 다음 인자인 Executable 을 수행한다.
* @EnabledOnOs - 실행하는 OS가 인자로 전달한 OS 타입과 일치하는 경우 테스트를 실행한다.
* @DisabledOnOs - 실행하는 OS가 인자로 전달한 OS 타입과 일치하는 경우 테스트를 실행하지 않는다.
* @EnabledOnJre - 실행하는 자바 버전이 인자로 전달한 자바 버전과 일치하는 경우 테스트를 실행한다.
* @DisabledOnJre - 실행하는 자바 버전이 인자로 전달한 자바 버전과 일치하는 경우 테스트를 실행하지 않는다.
* @EnabledIfEnvironmentVariable - 인자로 전달한 환경 변수값이 실제 환경 변수와 같은 경우 테스트를 실행한다.
* @DisabledIfEnvironmentVariable - 인자로 전달한 환경 변수값이 실제 환경 변수와 같은 경우 테스트를 실행하지 않는다.


### 참고
[더 자바, 애플리케이션을 테스트하는 다양한 방법](https://www.inflearn.com/course/the-java-application-test)