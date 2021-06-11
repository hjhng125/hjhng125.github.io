---
title:  "Spring Data Common - Null 처리"

categories:
  - 스프링-데이터-jpa
tags:
  - JPA
  - Spring
last_modified_at: 2021-06-10
---


 spring data 2.0부터 java 8의 Optional을 지원한다.
  * Optional<Entity> findById(Long id);
  * 되도록이면 단일 값을 가져와야 하는 경우 Entity타입이 아닌 Optional<Entity> 값으로 가져오면 아래의 Optional이 지원하는 메소드를 사용하여 null 체크를 조금 더 우아하게(?) 처리할 수 있다.
  
아래 두 코드를 비교해보자
  * ![1](/assets/images/findById.png)
  * ![1](/assets/images/findByIdOptional.png)
  * isEmpty()
  * isPresent()
  * orElse()
  * orElseThrow()
  * etc..
  * Spring data 가 리턴하는 콜렉션은 Null이 아닌 비어있는 콜렉션을 리턴한다.

spring 5.0 부터 지원하는 Null 어노테이션 지원한다.
  * @NonNullApi
  * @NonNull
  * @Nullable
  * 런타임 시에 null 체크 지원


package-info라는 java파일을 만들고 Null 어노테이션을 붙이면 해당 패키지 내의 모든 메서드, 모든 리턴타입, 모든 파라미터에 해당 어노테이션이 붙은 것과 같다.


참고: [https://www.inflearn.com/course/스프링-데이터-jpa](https://www.inflearn.com/course/스프링-데이터-jpa)