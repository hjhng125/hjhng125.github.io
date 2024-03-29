---
title:  "Maven과 Gradle"
excerpt: "Maven과 Gradle에 대하여 공부한 내용을 기술합니다."

categories:
  - spring
tags:
  - maven
  - gradle
comments: true
last_modified_at: 2020-07-11
---

> 의존성 관리 도구

## Maven
* 자바용 프로젝트 관리 도구
* 아파치 라이센스로 배포되는 오픈소스 소프트웨어
* `Dependency`를 관리하고, 표준화된 프로젝트를 제공
* 네트워크를 통해 연관된 라이브러리까지 관리해주기 때문에 과거 개발자가 수동으로 업데이트 혹은 연결시켜주던 과정을 자동으로 해주어 편리함을 제공한다.
* `아파치 앤트(ANT)`는 전적으로 전처리, 컴파일, 패키징, 테스팅, 배포하는데 초점이 맞춰져있지만, 메이븐과 같은 프로젝트 관리 툴은 빌드 툴을 바탕으로 여러 기능들을 종합적으로 제공한다.
* `POM.xml` 이라는 파일에 필요한 `Jar` 혹은 `Class Path`를 선언해주면 직접 다운로드할 필요 없이 Maven이 Repository 레파지토리에 필요한 모든 파일들을 해당 프로젝트로 불러준다. 

## Gradle
* `JVM`기반으로 `ANT`와 `Maven`의 장점을 모아 만들어진 프로젝트 관리 도구
* `Java`와 `Groovy`를 이용해 개발자의 의도에 맞는 logic을 설계할 수 있는 빌드 자동화 시스템