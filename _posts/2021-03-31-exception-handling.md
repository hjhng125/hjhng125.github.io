---
title:  "예외 처리"
excerpt: "예외 처리에 관하여"

categories:
  - java
tags:
  - Exception Handle

last_modified_at: 2021-04-03
---

* Error
  - 에러는 시스템에 뭔가 비정상적인 상황이 발생할 경우 사용된다.
  - 주로 JVM에서 발생시키며 애플리케이션 코드에서 대응 방법이 없기에 잡으면 안된다.
  - 시스템 레벨에서 틀별한 작업을 하는게 아니라면 애플리케이션에서 에러에 대한 처리는 신경쓰지 않아도 된다.

* Exception
  - 에러와 달리 개발자들이 만든 애플리케이션 코드의 작업 중 발생한 예외 상황에 사용된다.
  - Checked Exception
    * Exception 클래스의 서브클래스이며, RuntimeException 클래스를 상속하지 않은 예외
    * Checked Exception이 발생할 수 있는 메소드를 사용한다면 예외를 처리하는 코드를 함께 써야한다.
    * 예를 들어, try~catch 블록을 사용하거나, throws를 통해 메소드 밖으로 던져야 한다.
    * 그렇지 않을 경우, 컴파일 에러가 발생한다.
    * SQLException, IOException이 이에 해당된다.
  - UnChecked Exception
    * RuntimeException을 상속한 예외
    * 명시적인 예외처리를 강제하지 않는다
    * try~catch나 throws를 사용하지 않아도 된다.
    * NullPointerException, IllegalArgumentException 등이 이에 해당된다.
    * 이 예외는 코드에서 미리 조건을 체크하도록 주의한다면 피할 수 있다.
    * RuntimeException은 예상치 못한 상황에 발생하는 예외가 아니므로 굳이 try~catch나 throws를 강제하지 않는다.

* 예외 처리 방법
  1. 예외 복구
    - 예외 상황을 파악하고 문제를 해결하여 정상 상태로 돌리는 것.
    - 만약 파일을 읽으려 시도할 때 파일이 없어 IOException이 발생했다. 이때 사용자에게 상황을 알리고, 다른 파일을 이용하게 하여 예외를 해결한다.

  2. 예외 처리 회피
    - 예외를 처리하는 것이 자신의 역할이 아니라고 판단된 경우 자신을 호출한 곳으로 던지는것 
    - JdbcTemplate이나 JdbcContext가 사용하는 콜백 오브젝트는 작업하다 발생할 수 있는 SQLException을 템플릿으로 던진다. 해당 예외 처리는 콜백 오브젝트가 아닌 템플릿 레벨의 역할이라고 판단하기 때문에 해당 예외를 회피하여 템플릿에서 처리하도록 한다.
    - 이러한 긴밀한 역할분담을 하고 있는 관계가 아니라면 자신의 예외는 자신이 해결하는게 낫다. 그렇지 않으면 무책임한 책임회피일 수 있다.

  3. 예외 전환
    - 내부에서 발생한 예외를 그대로 던지는 것이 예외에 대한 의미를 부여해주지 못하는 경우에 의미를 정확히 해줄 수 있는 예외로의 전환을 위해 사용한다.
  ![1](/assets/images/exception_transfer.png)
    - 위의 케이스는 restTemplate.exchange 메소드에서 던지는 예외를 받아 UserGuideException으로 전환하여 처리한다.
    - 서비스 레이어에서 의미가 명확한 예외를 받았을 때 적절한 복구 작업을 시도할 수 있다. 물론 서비스 계층에서도 원인을 해석하여 대응할 수 있지만 원인을 해석하는 코드를 비즈니스 로직에 포함하는 것은 매우 어색하다.
    - 예외를 전환하는 경우에 기존 예외를 담아서 중첩 예외로(nested Exception)으로 만드는 것이 좋다. 
    - 예외를 추적하기 쉽게 해주기 때문이다.
    - 중첩 예외는 getCause()를 통해 처음 발생한 예외가 무엇인지 확인할 수 있다.
    
    ```
    catch (IOException e) {
      throw new UserGuideException(e);
    }
    ```

  * 처리할 수 없는 예외는 가능한 한 빨리 런타임 예외로 포장하여 다른 계층에서 메소드를 작성할 때 불필요한 throws를 선언하지 않도록 해야한다.
  * 무분별한 throws를 통하여 의미없고 지저분한 코드를 만들지 않도록 하자