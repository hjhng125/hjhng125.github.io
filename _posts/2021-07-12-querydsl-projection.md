---
title:  "querydsl Projection"

categories:
  - querydsl
tags:
  - querydsl
  - Spring boot
last_modified_at: 2021-07-12
---

## Projection
* select절에 어떤 데이터를 가져올지 대상을 지정하는 것.
* 대상이 하나인 경우 해당 타입으로 지정할 수 있다.
* 대상이 여러개인 경우 Tuple이나 DTO로 받아야 한다.

### 대상이 하나인 경우
~~~
List<String> result = queryFactory
    .select(member.username)
    .from(member)
    .fetch();
~~~ 

### 대상이 여러건인 경우
~~~
List<Tuple> result = queryFactory
    .select(member.age, member.username)
    .from(member)
    .fetch();
~~~

* Tuple은 `querydsl`이 지원하는 객체이다. 따라서 레파지토리 계층이 아닌 다른 계층에서 사용하는 것은 타 계층에 레파지토리가 JPA, querydsl에 의존하게 만들며, 내부 구현 기술이 어떤 것인지 알리게 된다. 
* Spring이 추구하는 설계는 내부 구현 기술을 최대한 숨기는 것. 즉, 의존도를 낮추는 것이다.
* Tuple을 사용함에 따라 계층 간 의존도가 높아지게 되면, 레파지토리에서 다른 기술을 사용할 경우 여러 계층에서의 수정도 불가피해진다.
* 따라서 계층 간의 데이터는 DTO로 변환하여 반환하는 것이 의존관계를 없앨 수 있는 방법이다.

### MemberDTO
~~~
@Getter
@Setter
@ToString
@NoArgsConstructor
public class MemberDto {

    private String username;
    private int age;

    public MemberDto(String username, int age) {
        this.username = username;
        this.age = age;
    }
}

~~~

### 순수 JPA에서 DTO 사용
* JPQL에서 반환하는 객체는 Entity이기에 바로 DTO로 반환할 수 없다.
* DTO를 사용하고 싶다면 DTO 객체 내부에 Projection으로 가져올 필드를 받는 생성자를 정의한다.
* 이 DTO 객체를 `JPQL이 제공하는 new operation`으로 주입한다.
* DTO의 package을 모두 적어주어야 한다.
* 생성자 방식만 지원하며 `field, setter 주입`은 지원하지 않는다.
~~~
List<MemberDto> resultList =
    em.createQuery("select new me.hjhng125.querydsl.model.dto.MemberDto(m.username, m.age) from Member m", MemberDto.class)
        .getResultList();
~~~

### querydsl에서 DTO로 반환하는 방법
#### Property 접근
* Setter를 사용하는 방법
* Setter를 사용하는 프로퍼티와 이름이 같아야한다.
* 기본 생성자로 객체를 생성한 후 Setter를 사용하기에 기본 생성자가 없을 경우 에러를 리턴한다.

~~~
List<MemberDto> result = queryFactory
    .select(Projections.bean(MemberDto.class,
        member.username,
        member.age))
    .from(member)
    .fetch();
~~~

#### Field 직접 접근
* Getter, Setter가 없어도 필드에 바로 주입된다.
  
~~~
List<MemberDto> result = queryFactory
    .select(Projections.fields(MemberDto.class,
        member.username,
        member.age))
    .from(member)
    .fetch();
~~~
~~~
List<UserDto> result = queryFactory
    .select(Projections.fields(UserDto.class,
        member.username.as("name"), 
        member.age))
    .from(member)
    .fetch();
~~~
* field 명이 맞아야 하기에 alias를 사용해야함.

~~~

List<UserDto> result = queryFactory
    .select(Projections.fields(UserDto.class,
        member.username.as("name"), 
        ExpressionUtils.as(
            JPAExpressions
                .select(memberSub.age.max())
                .from(memberSub), "age")
    ))
    .from(member)
    .fetch();
~~~
* field 주입을 해야하는데 서브쿼리를 쓰고 싶은 경우 ExpressionUtils를 사용해야 한다. 
* 서브쿼리는 field이름과 안맞으니깐 그 자체를 alias한다고 생각하자. 
* 서브쿼리가 아닌 단순 필드도 사용할 수 있으나 그냥 as가 간편함.

#### 생성자 사용
* 생성자를 통한 주입은 projection하는 값들의 순서가 맞아야 한다.
* 생성자를 통한 주입은 생성자의 타입만을 보기 때문에 필드명이 달라도 된다.
* 타입이 안맞거나, DTO에 선언되지 않은 타입을 추가로 넣은 경우 런타임 시점에 `com.querydsl.core.types.ExpressionException` 오류를 리턴한다.

~~~
@Getter
@ToString
@AllArgsConstructor
public class UserDto {

    private final String name;
    private final int age;
}
~~~
~~~
List<MemberDto> result = queryFactory
    .select(Projections.constructor(MemberDto.class,
        member.username,
        member.age))
    .from(member)
    .fetch();

List<UserDto> result = queryFactory
    .select(Projections.constructor(UserDto.class,
        member.username,
        member.age))
    .from(member)
    .fetch();
~~~
* 위의 결과 모두 정상적으로 주입된다.

### @QueryProjection

#### 장점
* Q-Type DTO를 new 연산자를 통해(생성자 그대로) 주입받기 때문에 IDE의 도움을 받아 훨씬 안정적으로 값을 받을 수 있다.
* 타입이 안맞는 경우 컴파일 시점에 오류를 잡을 수 있다.

#### 단점
* Q-Type을 무조건 선언해야한다.
* DTO 자체가 querydsl 기술에 의존하게 된다.
* DTO는 모든 계층에 걸쳐 사용되고 외부로의 통신에도 사용된다. 따라서 여러 계층에 영향을 줄 수 있다.
* DTO 자체가 순수성을 잃는다는 의미이기도 하다.
* DTO가 순수하길 원한다면 이 방법은 고려되어야 한다.
* 하지만 앞으로도 계속 querydsl을 사용할 수 있고, 그럴 예정이라면 사용에 문제가 되지 않을 것이다.

~~~
@Getter
@ToString
@NoArgsConstructor
public class MemberDto {

    private String username;
    private int age;

    @QueryProjection
    public MemberDto(String username, int age) {
        this.username = username;
        this.age = age;
    }
}
~~~

<hr>

참고: [실전! Querydsl - 프로젝션과 결과 반환 - 기본](https://www.inflearn.com/course/Querydsl-%EC%8B%A4%EC%A0%84/lecture/30136?tab=curriculum)

참고: [실전! Querydsl - 프로젝션과 결과 반환 - DTO 조회](https://www.inflearn.com/course/Querydsl-%EC%8B%A4%EC%A0%84/lecture/30137?tab=curriculum)

참고: [실전! Querydsl - 프로젝션과 결과 반환 - @QueryProjection](https://www.inflearn.com/course/Querydsl-%EC%8B%A4%EC%A0%84/lecture/30138?tab=curriculum)
