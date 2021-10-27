---
title:  "querydsl"

categories:
  - querydsl
tags:
  - querydsl
  - Spring boot
comments: true
last_modified_at: 2021-06-19
---

> Back-end 개발을 할 경우 Spring boot, JPA, Spring data jpa를 대체로 많이 사용한다.<br>
하지만 이 기술 조합으로 해결하지 못하는 한계가 발생하기 마련인데, 이는 복잡한 쿼리와 동적 쿼리 문제다.<br>
실무에서는 특히 복잡한 쿼리를 많이 사용하고, 검색 조건에 따른 동적쿼리를 사용할 일이 많다.<br>
또한 필자가 개발하는 정산 도메인은 쿼리를 작성해야 하는 일이 대부분이다.

그 동안 위의 상황에서 JPA와 Querydsl의 학습이 제대로 되어 있지 않았기에 Sql Mapper인 MyBatis를 사용했지만
이번 기회에 querydsl을 학습하여 실무에까지 적용해보려 한다.

## querydsl이란
`type-safe`한 쿼리를 위한 `Domain Specific Language`

## 장점
* querydsl은 자바 언어의 한계를 넘어 쿼리를 문자가 아닌 자바 코드로 작성할 수 있도록 해줌. 
* 또한 이로 인해 문법 오류를 컴파일 시점에 잡아주며 IDE의 자동완성 기능을 지원받을 수 있다.
* 자바 코드이지만 문법이 sql과 비슷하여 쉽게 학습할 수 있다.
* 복잡한 쿼리도 작성할 수 있다.
* 동적 쿼리 문제를 해결해 준다.

## gradle setting

~~~
plugins {
    id 'org.springframework.boot' version '2.5.1'
    id 'io.spring.dependency-management' version '1.0.11.RELEASE'
    id 'java'
}

group = 'me.hjhng125'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '11'

configurations {
    compileOnly {
        extendsFrom annotationProcessor
    }
}

repositories {
    mavenCentral()
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    implementation 'org.springframework.boot:spring-boot-starter-web'

    implementation 'com.querydsl:querydsl-jpa'

    compileOnly 'org.projectlombok:lombok'
    runtimeOnly 'com.h2database:h2'

    annotationProcessor "com.querydsl:querydsl-apt:${dependencyManagement.importedProperties['querydsl.version']}:jpa"
    annotationProcessor 'org.projectlombok:lombok'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}

test {
    useJUnitPlatform()
}

def querydslDir = "$buildDir/generated/querydsl" // QClass가 저장될 빌드 아웃푹 디렉토리 경로

sourceSets {
    main.java.srcDirs += [ querydslDir ]
}

~~~

* querydsl-jpa : querydsl jpa 라이브러리
  * spring data가 다양한 저장소를 제공하듯 querydsl 또한 JPA, JDO, Lucene, MongoDB 등 다양한 모듈을 지원한다.
  * [http://www.querydsl.com](http://www.querydsl.com/)
* querydsl-apt : querydsl 관련 코드(`QClass`) 생성 기능 제공
  * annotation processor가 `@Entity` 어노테이션이 붙어있는 클래스를 기반으로 QClass를 생성한다.

## *apt*
* Annotation Processor Tool의 약자
* `java compile 단계`에서 유저가 정의한 annotation의 소스코드를 분석하고 처리하는 tool이다.
* querydsl-apt는 `@Entity`가 정의된 엔티티 클래스를 분석하여 QClass를 생성한다.

<hr>

참고: [실전! Querydsl - 프로젝트 환경설정](https://www.inflearn.com/course/Querydsl-%EC%8B%A4%EC%A0%84/lecture/30114)
참고: [실전! Querydsl - Querydsl 설정 과 검증](https://www.inflearn.com/course/Querydsl-%EC%8B%A4%EC%A0%84/lecture/30115?tab=curriculum)
참고: [실전! Querydsl - 라이브러리 살펴보기](https://www.inflearn.com/course/Querydsl-%EC%8B%A4%EC%A0%84/lecture/30116?tab=curriculum)