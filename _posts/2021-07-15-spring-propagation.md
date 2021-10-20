---
title:  "Transaction Propagation"
excerpt: "Transaction Propagation에 대하여 공부한 내용을 정리합니다"

categories:
  - spring
tags:
  - Transaction
comments: true

last_modified_at: 2021-08-12T01:50:000-05:00
---

미완
스프링에서 트랜잭션 범위 내의 모든 코드는 해당 트랜잭션에서 실행됩니다.<br>
이때 트랜잭션 내부에서 다른 트랜잭션을 호출한다면 어떻게 될까요? 이는 설정에 따라 기존의 트랜잭션에서 실핼되거나 새 트랜잭션이 생성될 수 있습니다.
이렇게 트랜잭션의 범위를 정의하는 것을 트랜잭션의 전파(Propagation)라고 합니다.

이번 포스트에서는 Spring이 지원하는 트랜잭션의 `전파` 속성에 대해서 알아보겠습니다.

### REQUIRED
* 이 옵션은 스프링 트랜잭션 전파 기본 옵션입니다.
* 만약 현재 시작된 트랜잭션이 존재하면 참여하며, 존재하지 않으면 새로운 트랜잭션을 시작합니다.
* 선언된 메서드에 대해 논리적 트랜잭션 범위가 생성됩니다.
* 하나의 트랜잭션으로 진행하기 때문에 롤백 발생 시 모두 롤백됩니다.

~~~
@Slf4j
@Service
@RequiredArgsConstructor
public class ChildService {

    private final MemberRepository members;

    @Transactional
    public void saveByPropagationRequired(Member member) {
        log.info("current transaction name: {}", TransactionSynchronizationManager.getCurrentTransactionName());

        if (member.getId() != null) {
            throw new Exception(member.getId() + " is already present");
        }
        
        members.save(member);
    }
}
~~~

~~~
@Slf4j
@Service
@RequiredArgsConstructor
public class ParentService {

    private final ChildService childService;
    private final MemberRepository members;

    @Transactional
    public void transactionParent(Member member) {
        log.info("current transaction name: {}", TransactionSynchronizationManager.getCurrentTransactionName());

        members.save(Member.builder()
            .username("parent")
            .build());
        childService.saveByPropagationRequired(member);
    }
}
~~~

~~~
@BeforeEach
void before() {
    memberRepository.deleteAll();
    member = Member.builder()
        .username("child")
        .build();
}

@Test
void transaction_propagation_required_test() {
    parentService.transactionParent(member);
}
~~~

위 테스트를 실행하면 ParentService의 트랜잭션과 ChildService의 트랜잭션이 같음을 확인할 수 있고,
![1](/assets/images/transaction_propagation_required_success.png)

값도 예상하는대로 들어간 것을 확인할 수 있습니다.
![1](/assets/images/transaction_propagation_required_success_table.png)

이번엔 강제로 예외를 발생시켜 보겠습니다.
~~~
@Test
    void transaction_propagation_required_exception_test() {
        Member save = memberRepository.save(member); // 미리 데이터를 저장해둠.
        parentService.transactionParent(save);
    }
~~~

예외로 인해 트랜잭션 내에서의 쿼리가 모두 롤백되어 데이터베이스 데이터가 저장되지 않은 것을 확인할 수 있습니다.
![1](/assets/images/transaction_propagation_required_fail_table.png)

여기서 만약 예외를 try-catch 블록으로 감싼다면 어떻게 될까요?
~~~
@Slf4j
@Service
@RequiredArgsConstructor
public class ParentService {

    private final ChildService childService;
    private final MemberRepository members;

    @Transactional
    public void transactionParent(Member member) throws RuntimeException{
        log.info("current transaction name: {}", TransactionSynchronizationManager.getCurrentTransactionName());

        try {
            members.save(Member.builder()
                .username("parent")
                .build());
            childService.saveByPropagationRequired(member);
        } catch (RuntimeException e) {
            log.error("exception");
        }

    }
}
~~~

![1](/assets/images/transaction_propagation_required_fail_table.png)

예외 처리를 해주어 ParentService에서 저장하려한 데이터는 저장될 것 같았으나, try-catch 블록으로 예외를 감쌌지만 롤백이 수행되어 데이터가 저장되지 않았습니다.<br>
왜 이렇게 된것일까요?

JPA에서 Rollback 대상으로 인지된 예외(UnChecked Exception)가 발생한 경우 해당 예외의 try-catch로 예외 처리 여부에 관계없이 해당 트랜잭션은 rollback-only로 마킹됩니다.<br>

이 경우에 try-catch을 처리를 통해 해당 예외를 rollback을 위한 예외로 throw하지 않으면 에러가 발생하게 됩니다.
(org.springframework.transaction.UnexpectedRollbackException: Transaction silently rolled back because it has been marked as rollback-only)

하지만 Checked Exception의 경우 트랜잭션에서 롤백을 처리하지 않습니다. 




### SUPPORTS
* 이미 시작한 트랜잭션이 있다면 참여하고, 그렇지 않은 경우 트랜잭션 없이 진행합니다.

### MANDATORY
* 이미 시작한 트랜잭션이 있다면 참여하고, 그렇지 않은 경우 Exception을 발생시킵니다.
* 트랜잭션을 단독적으로 사용해야할 이유가 없는 경우에 사용합니다.

### REQUIRES_NEW
* 이미 시작한 트랜잭션이 있으면 일시중지한 뒤, 새 트랜잭션을 시작합니다.

### NOT_SUPPORTED
* 이미 시작한 트랜잭션이 있으면 일시중지한 뒤, 비즈니스 로직을 트랜잭션 없이 실행합니다.

### NEVER
* 트랜잭션이 있으면 Exception을 발생시킵니다.
* 트랜잭션을 사용하지 않는 것을 강제하고 싶을 때 사용합니다.

### NESTED
* 트랜잭션이 존재할 경우 중첩된 트랜잭션을 개시하고 존재하지 않을 경우 REQUIRED와 동일하게 동작합니다.
* SavePoint 기능을 지원하는 




### 참고
[Spring 공식문서](hthttps://docs.spring.io/spring-framework/docs/current/reference/html/data-access.html#tx-propagation)

[Baeldung](https://www.baeldung.com/spring-transactional-propagation-isolation)

[KimJS님 블로그](http://blog.breakingthat.com/2018/04/03/springboot-transaction-%ED%8A%B8%EB%9E%9C%EC%9E%AD%EC%85%98/)