---
title:  "Transaction Isolation"
excerpt: "Transaction Isolation에 대하여 공부한 내용을 정리합니다"

categories:
  - spring
tags:
  - Transaction

last_modified_at: 2021-08-09
---

여러 사용자가 동시에 하나의 데이터에 접근하려할 경우 발생하는 문제는 다음과 같습니다.
### Dirty Read
  * 커밋되지 않은 데이터를 읽는 것
  * 트랜잭션이 롤백될 경우 이미 읽어온 데이터는 유효하지 않음
  * 동시성이 좋아 커밋을 기다리지 않고 바로 읽을 수 있는 장점이 있지만 데이터의 일관성이 떨어진다
  * 데이터가 롤백될 일이 없고 확실히 커밋된다고 보장할 수 있는 경우에만 사용해야한다.

### Non Repeatable Read
  * 같은 데이터를 다시 조회할 경우 다른 값을 얻음

### Phantom Read
  * 트랜잭션 내에서 일정 범위의 데이터를 두번 이상 조회할 때 두번째 쿼리에서 새로 입력된 데이터를 읽거나
  * 이전에 존재하던 데이터가 사라지는 것.

이러한 문제를 방지하기 위하여 아래와 같은 옵션을 인지하여야 합니다.

## Isolation
isolation 옵션은 해당 트랜잭션의 격리 수준을 지정하는 옵션입니다. <br>
격리 수준이란 일관성이 없는 데이터를 허용하는 수준을 나타내며 옵션은 다음과 같습니다.

### DEFAULT
사용하는 DB 벤더에 따른 기본 전략을 사용합니다.
* Postgresql : READ_COMMITTED
* MySQL : REPEATABLE_READ

### READ_UNCOMMITTED
이는 커밋되지 않은 레코드를 다른 트랜잭션에서 읽는 것을 허용합니다.<br>
그렇기 때문에 레코드가 커밋되지 않고 롤백된 경우 두번째 트랜잭션은 유효하지 않은 데이터를 갖게 됩니다.<br>
해당 레벨에서는 Dirty Read, Non Repeatable Read, Phantom Read가 발생할 수 있습니다.
> Postgresql 에서는 이 레벨을 지원하지 않습니다.

### READ_COMMITTED
이름 그대로 커밋된 결과만 읽는 격리 수준입니다.<br>
해당 레벨은 한 트랜잭션에서 레코드를 변경하는 동안 다른 트랜잭션에서 읽는 것을 방지합니다.<br>
하지만 하나의 트랜잭션내에서 동일한 레코드를 조회하는 사이에 다른 트랜잭션에서 레코드의 변경, 삽입, 삭제를 허용하며 이들이 커밋되면 다른 결과를 가져올 수 있습니다.<br>
이로 인해 Dirty Read는 방지할 수 있지만 여전히 Non Repeatable Read, Phantom Read가 발생할 수 있습니다.

### REPEATABLE_READ
MySQL InnoDB가 기본으로 사용하는 격리 수준으로, 반복해서 조회하더라도 항상 같은 레코드를 조회하는 격리 수준입니다.<br>
하지만 하나의 트랜잭션내에서 동일한 레코드를 조회하는 사이에 다른 트랜잭션에서 레코드의 변경은 방지하나 삽입, 삭제를 허용하며 이들이 커밋되면 다른 결과를 가져올 수 있습니다.<br>
따라서 Non Repeatable Read를 방지할 수 있지만 Phantom Read는 여전히 발생할 수 있습니다.
> MySQL에서는 Phantom Read가 발생하지 않습니다.

### MySQL에서는 Phantom Read가 왜 발생하지 않는 것일까요?
[MySQL 공식문서](https://dev.mysql.com/doc/refman/8.0/en/innodb-transaction-isolation-levels.html#isolevel_repeatable-read)<br>

InnoDB의 REPEATABLE_READ 수준은 트랜잭션 내에서 첫 조회 시 결과를 SNAPSHOT으로 만들어 놓고, 이후에 조회 시 해당 SNAPSHOT에서 레코드를 조회합니다.<br>
그렇기 때문에 매번 조회하여 나온 결과가 처음과 동일하여 Phantom Read가 발생하지 않는 것입니다.<br>

InnoDB의 READ_COMMITTED 격리 수준 또한 SNAPSHOT을 사용하나, 매 조회 시 SNAPSHOT을 새로 구축하기 때문에 결과가 다를 수 있습니다.

### SERIALIZABLE
해당 격리 수준은 동시 처리 성능을 포기하고 완벽한 읽기 일관성을 보장하는 격리 수준입니다.<br>
하나의 트랜잭션이 완료된 이후에 다른 트랜잭션이 실행되는 것처럼 지원합니다.

### 참고
[MySQL 공식문서](https://dev.mysql.com/doc/refman/8.0/en/innodb-transaction-isolation-levels.html#isolevel_repeatable-read)<br>
[https://nesoy.github.io/articles/2019-05/Database-Transaction-isolation](https://nesoy.github.io/articles/2019-05/Database-Transaction-isolation)