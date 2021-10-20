---
title:  "Open Session In View"
excerpt: "Open Session In View에 대하여 공부한 내용을 기술합니다."

categories:
  - JPA

last_modified_at: 2021-09-05T01:50:000-05:00
---

미완
얼마전, 파일럿 프로젝트를 진행하며 엔터티를 응답하는 api를 만들었습니다.<br>
이를 분석하는 과정에서 OSIV가 관계가 있음을 알게되었는데요,<br>
이번 포스트에서는 OSIV(Open Session In View)에 대하여 알아보겠습니다.

그전에 먼저 준영속 상태에서의 지연로딩에 대해 잠깐 짚고 넘어가겠습니다.

## 준영속 상태에서의 지연로딩
만약 영속성 컨텍스트가 트랜잭션 범위 내에서만 살아있다고 가정해보겠습니다.<br>
그렇다면 트랜잭션 범위를 벗어난 컨트롤러 계층이나 뷰 계층에서 영속성 컨텍스트는 종료될 것이고, 엔터티는 준영속상태가 될 것입니다.<br>
따라서 변경 감지, 지연 로딩이 동작하지 않게됩니다.<br>

물론 변경 감지 기능은 서비스 계층의 비즈니스 로직에서 수행되며 발생합니다.<br>
이 기능이 컨트롤러 계층이나 프레젠테이션 계층에서 발생한다면 데이터를 어디서 어떻게 변경했는지 관리 포인트가 많아지게 되며 유지보수가 어려워집니다.<br>
따라서 변경 감지가 타 계층에서 동작하지 않는 것은 특별히 문제되지 않습니다.

하지만 뷰에 엔터티를 리턴하여 렌더링할 때 연관된 엔터티도 함께 사용해야 하는 경우 연관 엔터티가 지연 로딩으로 설정되어 있다면 어떨까요?<br>
지연로딩으로 설정된 연관 엔터티는 초기화되지 않은 프록시 객체일 것이고 이 객체를 사용하려 하면 실제 데이터를 불러오려고 초기화를 시도할 것입니다.<br>
하지만 프레젠테이션 계층엔 영속성 컨텍스트가 없기 때문에 지연로딩을 할 수 없으며, 이 과정에서 Hibernate 구현체를 사용하고 있다면 `org.hibernate.LazyInitializationException`이 발생하게 됩니다.

준영속 상태에서 지연 로딩 문제를 해결하기 위한 방법은 다음과 같습니다.
* 뷰가 필요한 엔터티를 미리 로딩하는 방법
* OSIV를 사용하여 엔터티를 항상 영속상태로 유지하는 방법

## 엔터티를 미리 로딩하는 방법
### 글로벌 패치 전략 수정
이 방법은 가장 간단한 방법으로 엔터티의 연관관계의 로딩 전략을 즉시 로딩으로 변경하는 것입니다.<br>
엔터티에 있는 fetch 타입을 변경하면 애플리케이션 전체에서 해당 엔터티를 로딩할 때마다 적용한 로딩 전략을 사용하므로 글로벌 패치 전략이라고 합니다.
~~~
@Entity
public class Member {

    @Id @GeneratedValue
    private Long id;

    private String username;
    private int age;

    @ManyToOne(fetch = FetchType.EAGER)
    private Team team;
}
~~~

글로벌 패치 전략은 가장 간단한 방법이지만 다음과 같은 단점이 있습니다.
* 모든 상황에 연관 엔터티를 즉시 로딩하므로 굳이 필요없는 상황에서도 엔터티를 로딩합니다.
* JPQL N+1 문제가 발생할 수 있습니다.
  * entityManager.find() 메서드는 엔터티를 조회할 때 연관 엔터티를 로딩하는 방법이 즉시 로딩이면 JOIN 쿼리를 사용하여 한 번에 연관 엔터티를 가져옵니다.
  * JPA는 JPQL을 분석해서 SQL을 생성할 때 글로벌 패치 전략을 참고하지 않고 오직 JPQL 자체만 사용합니다.
  * entityManager.createQuery("SELECT m FROM Member m", Member.class).getResultList(); 


### JPQL 패치 조인 사용
### 강제 초기화
### FACADE 계층 추가

## Open Session In View란?
![1](/assets/images/open-session-in-view.png)
OSIV(Open Session In View)는 영속성 컨텍스트의 생존 범위를 다른 레이어까지 열어두는 기능입니다.<br>
JPA 진영에서는 OEIV(Open EntityManager In View), Hibernate 진영에서는 OSIV라고 부르며, 관례상 둘다 OSIV로 불립니다.<br>

### Open Session In View가 필요한 이유

스프링에서는 영속성 컨텍스트를 servlet filter나 spring interceptor까지 유지하기 위해 이 기능이 기본으로 true로 설정되어 있습니다.
