---
title:  "Spring proxy 호출 위치와 @Transactional"
excerpt: "Spring proxy 호출 위치와 @Transactional"

categories:
  - spring
tags:
  - proxy
  - Transactional
comments: true
last_modified_at: 2021-07-13
---

@Transactional 애노테이션은 @PostConstructor 애노테이션이 선언된 메소드 내에서 실행되지 않는다. <br/>
[Spring 공식 문서](https://docs.spring.io/spring-framework/docs/current/reference/html/data-access.html#transaction-declarative-annotations) 

* 위의 문서에 의하면 기본적으로 **spring proxy**는 **외부 메서드에서 호출**되었을 때만 **프록시가 동작**하고(프록시 빈이 요청을 **intercept**), **해당 객체 내부에 의한 호출(self-invocatino)**에는 @Transactional 어노테이션이 선언되었다해도 런타임 시에 실제 **트랜잭션이 동작하지 않을 수 있다**고 한다.
* 즉, 내부에 의한 호출에는 프록시 객체가 아닌 proxy 빈이 아닌 target 빈이 동작하여 예상하는 결과를 볼 수 없는 것이다.
* 또한 **프록시가 예상되는 동작을 하기 위해선 완전히 초기화**가 되어야 하기에 @PostConstruct가 선언된 메서드와 같은 **초기화 메서드에서(빈이 완전한 초기화를 보장할 수 없는 상태)** 프록시 기능을 의존해선 안된다.

## 해결 방법
* Spring AOP 대신 AspectJ의 사용을 고려한다.
  * AOP 라이브러리를 변경하는 것은 다른 Side Effect를 일으킬 여지가 있으므로 신중히 고려되어야한다.
* `@Reource`를 통한 Self Injection을 사용한다.
  * 자기 자신의 빈을 주입받아 target 빈이 아닌 proxy를 통해 호출되는 효과를 만든다.
* 메서드에서 Transaction이 필요한 코드를 분리하여 **외부에서 호출**하도록 하는 것이다.
  
### 참고
[Spring 공식 문서](https://docs.spring.io/spring-framework/docs/current/reference/html/data-access.html#transaction)

[Baeldung](https://www.baeldung.com/spring-transactional-propagation-isolation)