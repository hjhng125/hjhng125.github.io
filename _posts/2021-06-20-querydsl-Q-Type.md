---
title:  "querydsl Q-Type"

categories:
  - querydsl
tags:
  - querydsl
  - Spring boot
last_modified_at: 2021-06-20
---

## 테스트할 Entity
> 앞으로 querydsl 게스글에 나올 Entity는 아래와 같다.

### 멤버 엔터티
~~~

@Entity
@Getter
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
* ToString()에 team과 같은 연관관계가 들어가면 안된다.
* 그렇게 되면 ToString()이 호출될때 team의 객체로 갔다가
* team에서 다시 ToString()에 의해 member로 무한 루프를 탈 수 있다.
* 따라서 소유하고 있는 필드만 포함시킨다.
* @NoArgsConstructor(access = AccessLevel.PROTECTED) 는 [여기를 참고](/jpa/jpa-constructor)

## 팀 엔터티
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

## Q-Type 선언

~~~
QMember qMember = new QMember("qMember"); // 별칭(alias) 사용
QMemver qMember = QMember.member; // 기본 인스턴스 사용

import static me.hjhng125.querydsl.member.QMember.*; // 기본 인스턴스와 static import 사용 (권장됨)
~~~

* Q-Type의 생성자에 들어가는 String값 ("qMember")는 JPQL에서의 table alias 값이다.
* Querydsl은 JPQL 빌더이기 때문에 실제 JPQL이 어떻게 발생했는지 확인할 수 있다.
* 이 부분을 확인하기 위해선 아래 옵션 추가
~~~
spring:
  jpa:
    properties:
      hibernate:
        use_sql_comments: true
~~~
* 같은 테이블을 조인(self join)해야 하는 경우 alias가 겹치면 안되기 때문에 각각 기본 인스턴스와 alias를 사용하여 만들어야 한다.

<hr>

참고: [실전! Querydsl - 기본 Q-Type 활용](https://www.inflearn.com/course/Querydsl-%EC%8B%A4%EC%A0%84/lecture/30123?tab=curriculum)