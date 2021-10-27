---
title:  "Dependency Injection"
excerpt: "Dependency Injection에 대하여 공부한 내용을 기술합니다."

categories:
  - spring
tags:
  - Dependency
comments: true
last_modified_at: 2020-07-18
---

> 요즈음 의존성 주입을 해야하는 경우 @Autowired의 사용을 하지 않고 생성자를 사용한다.
> @Autowired는 간단하지만 치명적 단점이 존재하는데,,,, 아래와 같은 이유로 스프링팀에서는 생성자 주입을 추천한다.


1. 순환참조가 방지할 수 있다.
  * 개발을 하다보면 다른 객체가 서로 의존하게 되는 경우가 있다. 이는 런타임 시에 순환참조를 발생시켜 StackOverFlowError를 발생시키게 된다.
  * 하지만 생성자 주입 방식은 컴파일 시에 BeanCurrentlyInCreattionException 이 발생하여 어플리케이션이 구동되지 않는다. 따라서 에러를 사전에 잡을 수 있다.
2. 단위 테스트를 하기 쉽다.
  * DI의 핵심은 관리되는 클래스가 DI Container에 의존적이지 않은 것이다. 이 말은 독립적으로 인스턴스화가 가능한 POJO여야 한다는 것이다.
  * 필드 주입 방식은 컨테이너에 의해 의존성을 주입받아야 하기 때문에 간단한 테스트를 하기 어렵다.

```
Some some = new Some();
Another another = new Another(some);
another.something();
```

3. 필드에 final을 사용할 수 있다.
  * 필드에 final을 사용하여 수정할 수 없게 만드는 것은 Side Effect를 막을 수 있다.
4. 리팩토링 요소를 파악하기 쉽다.
  * 클래스의 크기가 커짐에 따라 의존 받는 객체가 많아지게 되었을 경우 개인적으로 필드 주입을 통한 방식은 그리 이상해보이지 않았다.
  * 하지만 생성자 주입을 통해 많은 인자를 받게 된 경우 이는 코드의 분리 및 리펙토링 대상으로 여겨지게 되어 더 깔끔한 코드를 만들 수 있었다.
