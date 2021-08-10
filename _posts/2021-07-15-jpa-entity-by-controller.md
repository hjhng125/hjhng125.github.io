---
title:  "Controller에서 JPA Entity를 반환하는 경우"

categories:
  - JPA
tags:
  - Spring
comments: true

last_modified_at: 2021-08-10
---

이번 포스트는 Controller에서 api 응답을 Entity로 하는 경우에 맞닥뜨린 이슈를 공유하며, 어떤 방법이 적절한가에 대한 생각을 정리하기 위해 작성하였습니다.<br>
개인적 견해가 많이 포함되어 잘못되었을 수 있으니 다른 의견을 갖고 계시다면 댓글로 달아주시면 감사드리겠습니다.

## Entity를 응답할 경우 문제점
파일럿 프로젝트를 JPA를 사용하여 개발하며 엔터티를 만들고 양방향 맵핑을 해주었습니다.<br>
이 엔터티를 컨트롤러를에서 ResponseEntity 객체에 담아 응답을 했는데 맵핑된 관계가 순환 참조를 하며 뻗어버리는 현상을 확인했습니다.<br>
분명 레파지토리 단위 테스트를 했을때는 정상이었는데 컨트롤러에서 응답이 이상하게 나오는 것이 이상하여 이유를 찾아보았습니다.

~~~
@Entity
@Getter
@Setter
@ToString(of = {"id", "username", "age"})
@NoArgsConstructor(access = AccessLevel.PROTECTED) 
public class Member {

    @Id @GeneratedValue
    @Column(name = "member_id")
    private Long id;

    private String username;
    private int age;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "team_id") // 관계의 주인
    private Team team;

    public Member(String username) {
        this(username, 0);
    }

    public Member(String username, int age) {
        this(username, age, null);
    }

    public Member(String username, int age, Team team) {
        this.username = username;
        this.age = age;
        if (team != null) {
            changeTeam(team);
        }
    }

    public void changeTeam(Team team) {
        // 멤버의 팀을 바꾸면 해당 팀의 멤버도 바꿈.
        this.team = team;
        team.getMembers().add(this);
    }
}
~~~

~~~
@Getter
@Entity
@ToString(of = {"id", "name"})
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class Team {

    @Id @GeneratedValue
    @Column(name = "team_id")
    private Long id;

    private String name;

    @OneToMany(mappedBy = "team") // 관계의 주인이 아님.
    private Set<Member> members = new HashSet<>();

    public Team(String name) {
        this.name = name;
    }
}
~~~

~~~
@RestController
@RequiredArgsConstructor
public class MemberController {

    private final MemberRepository memberRepository;

    @GetMapping("/v7/members/{member_id}")
    public ResponseEntity<Member> searchMemberV7(@PathVariable("member_id") Member member) throws UserPrincipalNotFoundException{
        return ResponseEntity.ok(memberRepository.findById(member.getId()).orElseThrow(
            () -> new UserPrincipalNotFoundException("not found")
        ));
    }
}
~~~

* 위 두 테이블은 ManyToOne 양방향 관계입니다.
* 컨트롤러는 @ResponseBody가 포함된 @RestController 어노테이션을 사용했습니다.
* 컨트롤러에서 @ResponseBody를 통해 응답할 경우 스프링 부트는 HttpMessageConverter로 Jackson 라이브러리를 이용합니다.
* 엔터티를 응답으로 내보내려 하면 Jackson의 ObjectMapper는 객체를 Json으로 변환하는데, 이 때 맵핑 관계까지 직렬화를 위해 값을 참조하게 됩니다.
* 때문에 지연 로딩으로 Team 엔터티를 가져왔고 Team 엔터티에서도 맵핑 관계인 Member를 다시 호출하는 과정이 발생하여 결국 무한으로 순환하게 됩니다.
 
이 문제는 아래와 같은 방법으로 해결할 수 있습니다.
* @JsonIgnore 
  * 해당 어노테이션이 붙은 필드는 직렬화되지 않고 null이 할당됩니다.
  * 양방향 순환 참조를 해결하기 위한 용도는 아니고, 해당 필드를 Json으로 직렬화하고 싶지 않은 경우 사용하는 용도입니다.
* @JsonManagedReference, @JsonBackReference
  * 해당 어노테이션은 순환 참조를 방어하기 위한 어노테이션입니다.
  * @JsonManagedReference는 정상적으로 직렬화를 진행합니다.
  * @JsonBackReference는 직렬화에서 제외합니다.
* DTO 사용
  * 개인적 생각을 아래에서 다루겠습니다.

## 응답으로 DTO를 사용해야 하는 이유
Entity를 Controller layer에서 외부로 내보내는 것과 더불어 layer간 데이터를 주고 받을 시 Entity를 그대로 사용하는 것은 바람직하지 않다고 생각합니다.

그 이유는 첫째로, repository layer를 제외한 다른 계층에서 특정 기술에 의존성이 생길 수 있기 때문입니다.<br>
스프링에서 추구하는 설계는 내부 기술을 최대한 숨기고 의존성을 줄이는 데 있습니다. <br>
Entity를 계층 간 주고 받는 의존 관계가 강한 설계는 JPA를 다른 기술로 바뀔 경우 많은 곳에서 수정이 불가피해집니다.<br>

둘째로, Controller에서 외부로 나가는 응답에 Entity를 사용하면 데이터 설계와 기술 노출하게 됩니다.<br>
이 문제는 보안상으로 바람직한 구조가 아니라고 생각합니다.

마지막으로, DTO의 사용은 불필요한 데이터를 사용하지 않을 수 있습니다.<br>
보통 실무에서 테이블을 구성할 때 공통적으로 모든 테이블에 넣는 칼람이 있거나, 내부 비즈니스 로직을 위해 필요하지만 노출할 필요갸 없는 컬럼이 있습니다.<br>
이러한 컬럼까지 굳이 외부에 노출하는 것은 비효율적이며 클라이언트에 혼란을 줄 수 있습니다.<br>
따라서 정말 필요한 컬럼만을 프로젝션하여 사용하는 것이 좋은 설계라고 생각합니다.