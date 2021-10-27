---
title:  "Dependency Lookup"
excerpt: "Dependency Lookup에 대하여 공부한 내용을 기술합니다."

categories:
  - spring
tags:
  - Dependency
comments: true
last_modified_at: 2020-07-18
---

Spring을 이용하여 개발을 하면 [의존성 주입(DI, Dependency Injection)](/spring/dependency-injection/)을 굉장히 많이 사용합니다.<br>
DI는 Spring의 핵심 기술 중 하나인데요, 이 뿐 아니라 Spring은 의존성을 직접 검색하는 기능도 제공합니다.<br/>
이번 포스트에서는 스프링이 지원하는 의존성 검색(DL, Dependency Lookup)에 대해 알아보겠습니다.

## Dependency Lookup 이란?
`DL`은 말 그대로 `의존성 검색`입니다.<br>
이는 필요한 시점에 직접 Spring Bean에 접근하기 위해 컨테이너가 제공하는 API를 이용하여 Lookup하는 것입니다.
스프링은 DL을 위해 다음과 같은 방법들을 제공합니다.

## ApplicationContext
이 방법은 DI 컨테이너인 ApplicationContext를 통해 필요한 빈을 검색하는 방법입니다.
~~~
@Service
@RequiredArgsConstructor
public class LookupService {

    private final ApplicationContext applicationContext;
    
    public LookupTarget lookupDependency() {
        return applicationContext.getBean(LookupTarget.class);
    }
}
~~~

이 방법은 ApplicationContext를 직접 주입받은 후 `getBean()` 메서드를 호출하면 됩니다.
비교적 간단히 Lookup 할 수 있지만 코드 내에 스프링 API를 직접 사용하게 됩니다.

스프링이 추구하는 개발은 특정 환경과 기술에 종속되지 않은 POJO 방식의 개발인데, 이 방식은 스프링에 종속적이게 되버립니다.

## ObjectFactory, ObjectProvider
ObjectProvider는 ObjectFactory를 상속받은 interface로 `getObject` 메서드를 통해 DL을 제공합니다.<br>

~~~
@Component
@Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
public class PrototypeBean {

    private int num = 0;

    @PostConstruct
    void init() {
        System.out.println("Prototype Bean init");
    }

    @PreDestroy
    void preDestroy() {
        System.out.println("Prototype Bean destroy");
    }

    public void addNum() {
        num++;
    }

    public int getNum() {
        return num;
    }
}

@Component
@RequiredArgsConstructor
public class SingletonBean {

    private final ObjectProvider<PrototypeBean> prototypeBeans;

    @PostConstruct
    void init() {
        System.out.println("Singleton Bean init");
    }

    @PreDestroy
    void preDestroy() {
        System.out.println("Singleton Bean destroy");
    }

    public int getPrototypeBeanNum() {
        PrototypeBean prototypeBean = prototypeBeans.getObject();
        System.out.println(prototypeBean);

        prototypeBean.addNum();
        return prototypeBean.getNum();
    }
}
~~~

ObjectProvider는 앞서 설명한대로 ObjectFactory를 상속한 interface입니다.<br>
ObjectFactory는 단순히 `getObject` 메서드만을 제공하며, 
ObjectProvider는 추가로 Stream 처리 등의 편의 기능을 제공합니다.

하지만 두 interface 모두 스프링이 제공해주는 인터페이스로 스프링에 의존합니다.

## @Lookup
해당 어노테이션을 빈을 반환하는 메서드에 작성할 경우 Spring은 ApplicationContext의 getBean() 메서드를 호출하여 빈을 가져오는 메소드로 오버라이드합니다.
~~~
@Lookup
public PrototypeBean prototypeBean() {
    return null;
}
~~~

위 코드에서 빈을 호출할 경우 null을 반환하지만, 실제 Spring 실행 시 Lookup 어노테이션이 붙은 메서드를 오버라이드합니다.<br>
이 방식은 Spring API를 호출하는 코드를 숨김으로써 스프링에 의존적이지 않습니다.

## Provider
Provider는 JSR-330에 추가된 자바 표준 기술로, ObjectFactory와 유사하게 T 타입 파라미터와 `get()`이라는 팩토리 메서드를 제공합니다.
이 방식은 자바 표준 기술이기에 스프링에 의존적이지 않습니다.

get() 메서드는 호출 시 여러타입의 빈이 존재하는 경우 `org.springframework.beans.factory.NoUniqueBeanDefinitionException`를 빌셍시킵니다.
~~~
@Component
@RequiredArgsConstructor
public class SingletonBean {

    private final Provider<PrototypeBean> prototypeBeans;

    @PostConstruct
    void init() {
        System.out.println("Singleton Bean init");
    }

    @PreDestroy
    void preDestroy() {
        System.out.println("Singleton Bean destroy");
    }

    public int getPrototypeBeanNum() {
        PrototypeBean prototypeBean = prototypeBeans.get();
        System.out.println(prototypeBean);

        prototypeBean.addNum();
        return prototypeBean.getNum();
    }
}
~~~

## 마지막으로
Dependency Lookup을 사용하면 Singleton scope의 빈 내에 Prototype scope 빈을 사용할 경우 발생하는 [요청마다 같은 인스턴스를 제공받는 문제](/spring/bean-scope/)를 해결할 수 있습니다.

~~~
@RunWith(SpringRunner.class)
@ContextConfiguration(classes = {SingletonBean.class, PrototypeBean.class})
public class SingletonBeanTest {

    @Autowired
    SingletonBean singletonBean;

    @Test
    public void singletonUsePrototype() {
        int firstCall = singletonBean.getPrototypeBeanNum();
        assertThat(firstCall).isEqualTo(1);

        int secondCall = singletonBean.getPrototypeBeanNum();
        assertThat(secondCall).isEqualTo(1);
    }
}
~~~

![1](/assets/images/prototype-bean-with-singleton.png)

위와 같이 Singleton Bean 내부에서 호출한 Prototype Bean이 매 요청마다 다름을 확인할 수 있습니다.

## 참고
[망나니개발자님 블로그](https://mangkyu.tistory.com/168)

[춍춍's 블로그](https://chung-develop.tistory.com/63)