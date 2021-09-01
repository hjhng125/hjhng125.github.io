---
title:  "Spring Bean Scope"
excerpt: "Spring Bean Scope에 대하여 공부한 내용을 기술합니다."

categories:
  - spring
tags:
  - Bean


last_modified_at: 2020-08-10
---

스프링 기반 어플리케이션을 개발하면 Spring Bean을 사용하지 않을 수 없습니다.<br>
Spring Bean은 기본적으로 Singleton Scope로 관리되지만 요구에 따라 그 외에 다른 Scope를 사용해야 할 수 있습니다.<br>
이번 포스트에서는 Bean Scope에 어떤 종류가 있고 어떻게 동작하는지에 대하여 정리할 것입니다.

## Bean 이란?
Bean은 스프링에서 IOC 컨데이너에 의해 생성 및 관리하는 객체를 의미합니다. <br>
사실 이 특징을 제외하면 일반 객체와 다를게 없습니다. <br>

따라서 객체를 스프링 컨테이너가 관리하는 빈으로 만들기 위해서 다음과 같이 할 수 있습니다.
* Configuration 클래스에 등록한다 (`@Bean`) - 보통 직접 만든 객체가 아닌 외부에서 제공하는 객체에 사용
~~~
@Configuration
public class AppConfig {

    @Bean
    public MyService myService() {
        return new MyService();
    }
}
~~~

* 컴포넌트 스캐닝 - 직접 만든 객체에 사용
  * @Component, @Controller, @Service, @Repository 어노테이션을 붙인다.

위 처리를 하게되면 ServletContext 혹은 ApplicationContext에 빈이 등록되어 다른 빈에 주입되거나, getBean() 메서드를 통해 가져올 수 있습니다.

## Bean Scope
스프링은 이 빈에 대해 6가지 Scope를 제공합니다.
* singleton
* prototype
* web aware scopes
  * request
  * session
  * application
  * websocket

Scope을 정의하는 방법은 아래와 같습니다.
~~~
@Bean
@Scope("prototype")
public MyService myService() {
    return new MyService();
}
~~~

Type Safe한 코딩을 원한다면 이렇게 할 수도 있습니다. (singleton, prototype만 지원)
~~~
@Bean
@Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
public MyService myService() {
    return new MyService();
}
~~~

Web Aware Scope를 Type Safe하게 사용하는 방법은 아래와 같습니다. 
~~~
@Bean
@Scope(WebApplicationContext.SCOPE_REQUEST)
public MyService myService() {
    return new MyService();
}
~~~

또한 @RequestScope, @SessionScope, @ApplicationScope가 있습니다.

websocket scope를 Type Safe하게 사용하는 방법은 찾지 못했습니다.
혹시 알고 계시다면 댓글 부탁드립니다.

이제 이 Scope들을 차례대로 알아보겠습니다.

### Singleton
우리가 빈의 scope을 싱글톤으로 정의하면, 스프링 컨테이너는 스프링 설정에 정의된 하나의 객체를 생성합니다.
따라서 어떠한 요청에도 우리는 같은 객체를 사용하게 됩니다.
이 Scope는 아무런 설정을 하지 않은 경우 기본적으로 사용됩니다.
> 참고: 실제 GOF 패턴에서 정의된 싱글톤 패턴과는 다른 개념입니다.

### Prototype
이 Scope은 스프링 컨테이너가 매 요청에 다른 객체를 리턴합니다.
그렇기 때문에 다른 빈에서 주입받거나, getBean() 메서드를 호출할 때마다 응답된 객체는 항상 새로운 객체입니다.
만약, 빈에서 상태 정보를 갖고 있어야 하는 경우 prototype scope을 사용해야 합니다.

prototype scope을 사용할 시 주의해야할 사항이 있습니다.

prototype scope는 스프링이 빈의 전체 생명주기를 관리하지 않는다는 것입니다. 즉 생성 과정 이후 소멸 과정에는 관여하지 않습니다. 
이는 빈 생성 이후 객체의 소멸은 GC에 의해 처리되게끔 설계된 것인데요, 만약 빈이 DB Connection pool과 같은 비싼 자원을 점유하고 있다면 해당 빈은 GC에 의해 소멸되지 않습니다.

이런 경우 Custom Bean Post Processor를 구현하여 객체의 소멸을 직접 관리해야 합니다.

빈의 생명주기는 차후에 더 자세히 다루겠습니다.

### Singleton 빈에서 Prototype 빈을 주입받는 경우
만약 Singleton 빈에서 Prototype 빈을 주입받아 사용하는 경우에 prototype 빈이 어떻게 동작할까요?

결론부터 말씀드리자면 singleton 빈에 종속적으로 동작하게 됩니다.
즉, singleton 빈 내부에 어떤 메소드에서 prototype 빈을 사용하던 같은 객체일 것입니다.

스프링은 singleton 빈을 한 번만 생성하고, 이 빈에 prototype 빈을 주입하려할 때 한번만 생성할 것입니다.
그렇기 때문에 필요할 때마다 prototype 빈의 새 인스턴스를 제공할 수 없게 됩니다.

이런 경우 [Dependency Lookup]([DL, Dependency Lookup](/spring/dependency-lookup/))을 통해 해결할 수 있습니다. 
또한 `@Scope.proxyMode`를 통해 Prototype Bean을 프록시로 감싸주어 빈에 접근하려 할때 새로운 인스턴스를 생성하는 방법도 있습니다.
~~~
@Scope(scopeName=ConfigurableBeanFactory.SCOPE_PROTOTYPE, proxyMode = ScopedProxyMode.TARGET_CLASS)
~~~


### Web Aware Scopes
이 Scopes에 포함되는 것들은 일반 스프링 애플리케이션에서는 사용할 수 없으며, 스프링 MVC 애플리케이션에서만 사용가능합니다.(Web Aware)<br>
만약 이 scopes를 MVC가 아닌 스프링 어플리케이션에서 사용하려하면 알 수 없는 Bean Scope라는 IllegalStateException이 발생하게 됩니다.

* Request
  * 빈을 HTTP 요청 lifecycle의 범위로 지정합니다. 
    * 요청에 대해 새로운 빈 인스턴스를 생성하고 요청 종료 시 삭제합니다.
  * 즉, 각각의 HTTP 요청마다 고유한 빈을 주입받을 수 있습니다.
    * 요청마다 고유한 객체이기 때문에 내부 상태를 원하는 만큼 변경할 수 있습니다.
  * 빈 정의 시, @RequestScop 어노테이션으로 scope 설정 가능합니다.
* Session
  * 빈을 HTTP Session lifecycle의 범위로 지정합니다.
    * 세션마다 새로운 빈 인스턴스를 생성하고 세션이 종료되면 삭제힙니다.
  * 즉, 다른 HTTP 요청에도 같은 Session을 갖고 있다면 같은 빈 인스턴스를 사용합니다.
    * Request Scope과 마찬가지로 세션마다 고유한 객체이기 때문에 내부 상태를 원하는 만큼 변경할 수 있습니다.
  * 빈 정의 시, @SessionScop 어노테이션으로 scope 설정 가능합니다.
* application
  * 빈을 ServletContext lifeCycle의 범위로 지정합니다.
  * ApplicationContext가 아닌 ServletContext라는 타입의 객체에 의해 관리됩니다.
  * 이 scope은 singleton과 비슷하지만 영역이 다릅니다.
    * singleton : ApplicationContext 내에서 singleton
    * application : ServletContext 내에서 singleton
  * 빈 정의 시, @ApplicationScope 어노테이션으로 scope 설정 가능합니다.
* websocket
  * 빈을 WebSocket lifeCycle의 범위로 지정합니다.

### 참고
[Spring 공식 문서](https://docs.spring.io/spring-framework/docs/3.0.0.M3/reference/html/ch04s04.html)

[Baeldung](https://www.baeldung.com/spring-bean-scopes)
