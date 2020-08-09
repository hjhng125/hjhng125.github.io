---
title:  "Private Constructor"
excerpt: "Private Constructor에 대하여 공부한 내용을 기술합니다."

categories:
  - Java
tags:
  - access modifier
last_modified_at: 2020-07-16
---

* Java에는 `private 기본 생성자`와 `정적 메소드` 혹은 `정적 필드`로 구성된 클래스가 존재한다. 
  * 이 클래스는 생성자를 다른 곳에서 호출할 수 없으며, 하위 클래스를 만들 수 없다. 
  * 이와 같이 정적 메서드나 정적 필드로 이루어진 객체는 `private 생성자`를 만들어 둠으로써 인스턴스 생성이 불가능한 클래스로 만들 수 있으며 이를 통해 `memory leak`을 막을 수 있다.

  ```
  public class UtilClass {
    private UtilClass() {
      throw new AssertionError();
    }
  }
  ```

* 예로, Arrays 클래스나 Math 클래스는 `유틸성 기능`을 제공하는 목적으로 만들어졌다.

## Tip
* 기능적으로 동일한 객체는 매번 재생성을 하기 보다 `재사용`을 하는 것이 효율적이다.
* `변경 불가능한 객체`는 언제든지 재사용될 수 있다.
* `생성자`와 `정적 팩토리 페서드`를 같이 제공하는 `변경 불가능 클래스`의 경우 생성자를 사용하여 매번 객체를 만들기 보다 정적 팩토리 메서드를 이용하면 불필요한 객체 생성을 막을 수 있다.
* 변경 가능한 클래스 또한 변경할 일이 없다면 재사용할 수 있다.
  * `정적 초기화 블록`을 만들어 클래스 초기화 시점에 한번만 생성한다.
