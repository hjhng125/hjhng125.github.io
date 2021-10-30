---
title:  "Open Session In View"
excerpt: "Open Session In View에 대하여 공부한 내용을 기술합니다."

categories:
  - JPA
comments: true
last_modified_at: 2021-09-05T01:50:000-05:00
---

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
  * [JPA N+1 문제](/jpa/jpa-N+1/)


### JPQL 패치 조인 사용
글로벌 패치전략을 수정하는 것은 해당 엔터티를 조회하는 모든 쿼리에 영향을 줄 수 있습니다.<br>
따라서 필자의 경우 fetchType.Eager를 기본으로 사용하며 필요한 경우만 패치조인을 사용하여 한번에 데이터를 조회하는 편입니다.<br>
JPQL 패치조인을 사용하게 되면 해당 시점에 SQL join을 통해 연관관계까지 한번에 조회할 수 있습니다.<br>
앞서 발생할 수 있는 N+1 문제를 해결할 수 있는 근본적 방법이기도 합니다.<br>
하지만 JPQL 패치조인을 남발하게 될 경우 프레젠테이션 영역에 필요한 데이터에 맞춘 레파지토리 메서드가 계속해서 늘어난다는 단점도 존재합니다. 

### 강제 초기화
강제 초기화는 트랜잭션 범위 내에서 연관관계 엔터티를 강제로 초기화하는 방법입니다.<br>
```
@Transactional
public Member findMember(Long memberId) {
    Member member = memberRepository.findById(memberId);
    member.getTeam().getName();
    
    return member;
}
```

이런 식으로 트랜잭션 범위 내에서 Member 엔터티의 연관관계인 Team 엔터티를 강제로 초기화할 수 있습니다.<br>
하지만 서비스 계층에서 초기화를 진행하게 되면 프레젠테이션 계층에 따라 서비스 영역이 침범을 받게 됩니다.<br>
이 말은 즉, 프레젠테이션 계층에서 보여주고자 하는 엔터티가 변경됨에 따라 서비스 로직도 변경이 불가피하게 됩니다.

### FACADE 계층 추가
FACADE 계층은 서비스 계층과 프레젠테이션 계층 사이의 논리적 분리를 위한 계층입니다.<br>
프록시의 초기화를 서비스 계층에서 진행하지 않고 사이의 FACADE 계층에서 진행하는 것입니다.<br>
프록시 초기화는 영속성 컨텍스트가 필요함으로 트랜잭션의 시작을 FACADE 계층에서 진행합니다.<br>
이 방법은 서비스 계층과 프레젠테이션 계층의 의존 관계를 제거할 수 있는 방법이긴 합니다. 하지만 이러한 추가 코드가 반드시 필요하며, 특정 화면에 대한 쿼리가 많아진다면 매번 FACADE 코드를 작성해야하는 불편이 있습니다.


## Open Session In View란?
![1](/assets/images/open-session-in-view.png)
OSIV(Open Session In View)는 `영속성 컨텍스트의 생존 범위를 다른 레이어까지 열어두는 기능`입니다.<br>
JPA 진영에서는 OEIV(Open EntityManager In View), Hibernate 진영에서는 OSIV라고 부르며, 관례상 둘다 OSIV로 불립니다.<br>

### 요청 당 트랜잭션
이 방법은 과거의 OSIV 방식으로 요청이 들어오는 servlet filter나 spring interceptor에서 트랜잭션을 시작하고 요청이 끝날때 트랜잭션을 종료하는 방법입니다.
이를 통해 프레젠테이션 계층에서 엔터티 지연 로딩이 가능했기에 강제 초기화나 FACADE 계층이 필요하지 않게 되었습니다.

하지만 이는 컨트롤러 계층과 프레젠테이션 계층에서 엔터티를 수정할 수 있다는 치명적인 단점이 존재했는데요, 이를 방지하기 위해 다음과 같은 방법들이 필요했습니다.
* 엔터티를 읽기 전용 인터페이스로 제공

```
@Entity
public class Member implements MemberView{

    @Id @GeneratedValue(strategy = IDENTITY)
    private Long id;
    
    private String name;

    @ManyToOne
    private Team team;

    @Override
    public String getName() {
        return name;
    }
}

public interface MemberView {

    String getName();
}

@Service
@RequiredArgsConstructor
public class MemberService {

    private final MemberRepository members;

    public MemberView findMember(Long memberId) {
        return members.findById(memberId).orElseThrow();
    }
}
```

```
@SpringBootTest
class MemberServiceTest {

    @Autowired
    private MemberService memberService;

    @Autowired
    private MemberRepository members;

    private String name = "hong";
    
    @BeforeEach
    void setUp() {
        Member member = Member.builder()
                .name(name)
                .build();

        members.save(member);
    }

    @Test
    void 일기전용_인터페이스() {
        MemberView member = memberService.findMember(1L);

        assertThat(member.getName()).isEqualTo(name);
    }
}
```

* 엔터티를 래핑

```
@Builder
public class MemberWrapper {

    private Member member;

    public String getName() {
        return member.getName();
    }
}

@Test
void 엔터티를_래핑() {
    MemberWrapper wrapper = MemberWrapper.builder()
            .member(member)
            .build();

    assertThat(wrapper.getName()).isEqualTo(name);
}
```

* DTO 사용
가장 고전적인 방법으로 프레젠테이션 계층에 엔터티가 아닌 데이터를 복사한 객체를 제공하는 것입니다.<br>
이 방법은 OSIV의 장점을 살릴 수 없을 뿐더러 엔터티를 그대로 복사한 DTO를 양산하게 됩니다.


지금까지 알아본 과거 OSIV는 이러한 불편이 존재했습니다.<br>
다음으로 Spring에서는 이러한 문제점들을 어떻게 극복해냈는지 알아보겠습니다.

### Spring OSIV
위 사진처럼 스프링은 영속성 컨텍스트를 `servlet filter나 spring interceptor`까지 열어두고 트랜잭션은 `비즈니스 계층`에서만 유지하는 방식을 사용합니다.<br>

1. 요청이 들어오면 `servlet filter나 spring interceptor`에서 영속성 컨텍스트를 생성하고 트랜잭션은 시작하지 않습니다.
2. 서비스 계층에서 `@Transactional`을 통해 트랜잭션을 시작하며 이 때 미리 생성한 영속성 컨텍스트를 사용하여 트랜잭션을 시작합니다.
3. 서비스 계층이 끝나면 트랜잭션을 커밋하고 영속성 컨텍스트를 플러시합니다. 이 때 트랜잭션은 종료되지만 영속성 컨텍스트는 종료하지 않습니다.
4. `servlet filter나 spring interceptor`로 응답이 오면 영속성 컨텍스트를 종료합니다. 이 때 플러시는 일어나지 않고 바로 종료합니다.

### 트랜잭션 없이 읽기
엔터티를 변경하지 않고 단순히 읽는 작업은 트랜잭션이 없어도 할 수 있는데요, 이를 트랜잭션 없이 읽기라 합니다.<br>
프록시를 초기화하는 작업 또한 조회를 하는 일이기 때문에 트랜잭션 없이 읽기가 가능합니다.

여기서 주의해야할 사항이 있는데요, Spring OSIV는 영속성 컨텍스트가 트랜잭션 단위가 아닌 요청 단위로 생성되기 때문에 여러 트랜잭션이 하나의 영속성 컨텍스트를 공유한다는 것입니다.<br>
만약 컨트롤러에서 서비스에서 받은 엔터티를 어떠한 이유로 수정하고 그 엔터티를 서비스를 호출하여 트랜잭션을 시작한다면 어떻게 될까요???

```
@Controller
@RequiredArgsConstructor
public class MemberController {

    private final MemberService memberService;

    @GetMapping("/members/{memberId}")
    public String add(@PathVariable Long memberId) {
        Member member = memberService.findMemberBy(memberId);

        member.setName("****");
        memberService.add(member);

        return "member";
    }
}
```

위 예시는 서비스 계층에서 받아온 엔터티를 어떠한 이유로 인해 컨트롤러 계층에서 수정을 해서 프레젠테이션 계층으로 내보내고, 이후에 다시 트랜잭션을 시작한 코드입니다.<br>
위의 경우에는 가져온 member의 이름이 `****`로 변경되서 저장이 되버리죠.<br>
이처럼 영속성 컨텍스트를 공유하는 두 트랜잭션 사이에 엔터티가 변경되면 치명적인 오류를 발생시킬 수 있습니다.<br>

또한 프레젠테이션 계층에서 지연 로딩이 발생하기 때문에 SQL이 실행되는데요, 만약 성능 튜닝을 해야할 경우 확인해야하는 범위가 더 늘어나게 됩니다.


이렇게 OSIV가 나온 배경과 과거 OSIV와 스프링에서 사용하고 있는 OSIV에 대하여 알아보았습니다.<br>
이전에 OSIV를 자세히 알지 못해 많은 에러를 보았는데요, 그 중 하나가 [Controller에서 JPA Entity를 반환하는 경우](/jpa/jpa-entity-by-controller/) 였습니다.<br>
많은 분들이 이번 포스트를 통해 OSIV에 대해 확실히 개념을 잡으셨으면 좋겠습니다.<br>
혹시 잘못된 내용이나 궁금한 사항이 있으신 분은 댓글 부탁드립니다.

