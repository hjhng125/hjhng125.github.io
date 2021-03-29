---
title:  "@Qualifier and @RequiredArgsConstructor"
excerpt: "Qualifier와 @RequiredArgsConstructor"

categories:
  - Spring
tags:
  - DI

last_modified_at: 2021-03-29
---

> 같은 타입의 빈이 여러개인 경우 객체명을 bean의 이름과 맞추거나, @Qualifier 어노테이션을 통해 사용할 bean을 선택한다.

[의존성 주입]({% link _posts/2020-07-18-dependency-injection.md %})

![1](/assets/images/Qualifier_and_RequiredArgsConstructor.png)
* 업무 중 주입받으려는 타입의 빈이 여러개라 @Qualifier와 @RequiredArgsConstructor를 이용하여 주입을 받으려하고 있었다.
* 하지만 실제로 확인해보니 실제 내가 가져오도록 Qualifier의 value로 지정한 값이 아닌 @Primary로 등록된 빈이 주입되었다.

![1](/assets/images/primary_bean.png)

* 아무래도 필드 주입을 통한 의존성 주입 과정에서 정상적으로 동작하는 것으로 보아 @Qualifier 어노테이션의 문제는 아닌것 같았다.
* @RequiredArgsConstructor 어노테이션을 사용하지 않고 생성자를 직접 만들었을 때도 정상적으로 작동하였다.
* 아래 링크를 확인한 결과 롬복 18.4버전부터 등장한 copyableAnnotations라는 기능이 소개되었다.
* 이 기능은 생성자에 복사하길 원하는 롬복 어노테이션 리스트를 append함으로써 빌드 시 생성자에 해당 어노테이션을 복사한다.

```
lombok.copyableAnnotations += org.springframework.beans.factory.annotation.Qualifier
```

[@Qualifier for @RequiredArgsConstructor](https://ath3nd.wordpress.com/2018/12/13/spring-lombok-or-injection-just-became-a-bit-easier-part-2-of-2/)