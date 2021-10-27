---
title:  "JPA4 - 관계 Mapping"
excerpt: "JPA 관계 Mapping에 대하여 공부한 내용을 기술합니다."

categories:
  - spring-data-jpa
tags:
  - JPA
  - Spring
comments: true
last_modified_at: 2021-04-05
---

* 관계에는 항상 두 엔터티가 존재한다.
  * 둘 중 하나는 관계의 주인이며,
  * 다른 한 쪽은 종속된 쪽이다.
  * 해당 관계의 반대쪽 레퍼런스를 갖고 있는 쪽인 주인이다.

# 단방향 관계
  * 한쪽에서만 반대쪽 정보를 필요로 하는 경우 사용
  * 관계를 정의한 쪽이 주인
  * 위에 언급한대로 반대쪽 레퍼런스를 갖고 있는 쪽

* @ManyToOne

```
@Getter
@Setter
@Entity
public class Account {

  @Id
  @GeneratedValue
  private Long id;

  private String name;
}

@Getter
@Setter
@Entity
public class Study {

  @Id
  @GeneratedValue
  private Long id;

  private String name;

  @ManyToOne // 현재 reference를 보면 Account는 Collection이 아니니까 ManyToOne이 맞음.
             // Collection이라면 OneToMany다.
  private Account account;

}

```

  * Study 클래스가 Account를 참조하고 있다.
  * 기본값은 Study 테이블이 Account 테이블의 FK를 생성하게된다.
  * 위의 관계에서의 주인은 Study이다. (자손 테이블)

* @OneToMany

```
@Getter
@Setter
@Entity
public class Account {

  @Id
  @GeneratedValue
  private Long id;

  private String name;

  @OneToMany
  private Set<Study> studies = new HashMap<>();
}

@Getter
@Setter
@Entity
public class Study {

  @Id
  @GeneratedValue
  private Long id;

  private String name;

}
```
  * Account 클래스가 Study를 참조하고 있다. 
  * 기본값은 Account와 Study의 Mapping Table이 생성되며 이 테이블은 Account의 key와 Study의 key를 FK로 생성한다.
  * 이 관계에서 데이터를 insert하게 되면 Account와 Study가 insert 되고 이후 account_studies 테이블이 채워진다.
  * 이 관계에서의 주인은 Account이다.

# 양방향 관계
  * 양쪽에서 반대 쪽 정보를 참조하고 싶은 경우 사용.
  * 기본적으로 다른 옵션을 주지 않으면 FK를 갖고 있는 쪽인 주인이다.
  * 이 말은 곧, @ManyToOne 어노테이션을 갖고 있는 쪽인 주인
  * 주인이 아닌 쪽(@OneToMany)에선 mappedBy를 사용하여 관계를 맺고 있는 필드를 설정해야 한다.
  
  ```
  @Getter
  @Setter
  @Entity
  public class Account {

    @Id
    @GeneratedValue
    private Long id;

    private String name;

    @OneToMany(mappedBy = "account_id")
    private Set<Study> studies = new HashMap<>();
  }

  @Getter
  @Setter
  @Entity
  public class Study {

    @Id
    @GeneratedValue
    private Long id;

    private String name;

    @ManyToOne
    private Account account;
  }
  ```
  * 앞서 말했듯 위의 관계의 주인은 FK를 갖고 있는 Study 이다.
  * 만약 애래와 같이 Account쪽에만 관계를 설정한다면 실제 table을 조회할 때 Study 테이블의 관계가 제대로 매핑되지 않는다.
  
  ```
  account.getStudies().add(study);
  ```

  * 따라서 관계의 주인인 Study에 관계를 설정해야한다. (FK를 주인인 Study가 갖고 있기 때문)

  ```
  study.setAccount(account);
  ```
  * account에의 관계 설정은 필수는 아니지만 객체 지향적 관점에서 보았을 때 설정해주는게 맞다.
  * 삭제의 경우에도 양쪽 모두 관계를 끊어준다.


  참고: [https://www.inflearn.com/course/스프링-데이터-jpa](https://www.inflearn.com/course/스프링-데이터-jpa)