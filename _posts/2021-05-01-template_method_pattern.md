---
title:  "템플릿 메소드 페턴"
excerpt: "템플릿 메소드 패턴이란?"

categories:
  - Clean Code

last_modified_at: 2021-05-01
---

업무 중 비즈니스 로직을 구현하며 템플릿 메소드 패턴을 사용하였다.
템플릿 메소드 패턴은 코드의 변하지 않는 부분은 템플릿으로 두고 다양하게 변할 수 있는 로직은 캡슐화하여 특정 단계에서 수행하는 내역을 바꾸는 패턴이다.
이하의 내용은 템플릿 메소드 패턴에 대하여 서술한다.

* 템플릿 메소드 패턴은 전체적으로 동일하지만 일부분 다른 구문으로 구성된 메소드의 코드 중복을 최소한으로 할 경울 유용한 패턴이다.
* 동일한 부분을 슈퍼 클래스로 두고 변화가 발생할 수 있는 부분만 서브 클래스로 확장한다.

개인적으로 템플릿 메소드 패턴을 사용하며 interface의 default로 구현할지 abstract 클래스로 구현할지 고민을 정말 많이 한 것같다.
결론적으로 필자는 default 키워드를 사용하기 보다 해당 부분을 abstract 클래스로 구현하였는데 이는 다음과 같은 이유로 그렇다.
* 확장된 메소드를 외부로 노출하지 않기 위하여
  * 템플릿에서 확장된 로직은 특정 상황에 발생한 데이터를 파싱하는 역할로 다른 패키지 혹은 외부에 노출이 필요하지 않았다.


프로젝트의 버전을 향상시켜야하는 상황을 맞이하여 그동안의 코드를 돌아보았다.
다시 본 코드는 중복도 상당했고, 비효율적이며 코드 가독성이 없이 코딩이 되어있다는 것을 알게되었다.
이는 분명 시간이 촉박해서라는 안일한 생각으로 만들어논 것이 분명했다.
버전업을 하기 전 확장을 위한 리팩토링이 필요함을 느끼게 되었고 비즈니스 로직의 불변하는 부분을 템플릿으로 구현하고, 특정 상황에 변할 수 있는 로직을 확장하여 구현하고 보니, 템플릿 메소드 패턴을 적용하고 있음을 알게되었다.