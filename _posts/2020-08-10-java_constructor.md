---
title:  "생성자(Constructor)"
excerpt: "생성자(Constructor)에 대하여 공부한 내용을 기술합니다."

categories:
  - Java
tags:
  - Overloading
  - OOP

last_modified_at: 2020-08-10
---

> 생성자는 인스턴스가 생성될 때 호출되는 `인스턴스 초기화 메서드`이다.
> 인스턴스 변수의 초기화 작업에 주로 사용되며, 인스턴스 생성 시에 실행되어야 할 작업을 수행한다.
> 메서드와 같이 클래스 내에 선언되며, 클래스와 이름이 같고 리턴값이 없다.
> 생성자 또한 오버로딩이 가능하며, 하나의 클래스에 여러 생성자가 존재할 수 있다.

```
  Object object = new Object();

  1. 연산자 new에 의해 메모리(heap)에 인스턴스가 생성된다.
  2. 생성자가 호출되어 수행된다.
  3. 연산자 new의 결과로 생성된 객체 인스턴스의 주소가 반환되어 참조변수에 저장된다.
```

### 기본 생성자 (Default Constructor)
* 모든 클래스에는 하나 이상의 생성자가 정의되어야 한다.
* 컴파일 할 때, 클래스에 생성자가 정의되어 있지 않은 경우, 컴파일러는 자동으로 기본 생성자를 추가하여 컴파일한다.
* 하나 이상의 생성자가 정의되어 있는 경우, 컴파일러는 기본 생성자를 추가하지 않는다.

### 매개변수가 있는 생성자
* 생성자도 메서드처럼 매개변수를 선언하여 호출 시 값을 넘겨받음으로써 인스턴스 초기화 작업에 사용할 수 있다.
* 인스턴스마다 각기 다른 값으로 초기화 작업을 하는 경우가 많기 때문에 매개변수를 사용한 초기화는 매우 유용하다.
* 기본 생성자를 사용하면 인스턴스 생성 이후에 인스턴스 변수들을 초기화 해주어야 한다.
* 매개변수가 있는 생성자를 사용하면 인스턴스를 생성함과 동시에 인스턴스 변수들을 원하는 값으로 초기화가 가능하다.
* 그렇기에 매개변수를 갖는 생성자를 사용하는 것이 코드를 보다 간결하고 직관적으로 만들 수 있다.

### 생성자에서 다른 생성자 호출 - this(), this
* 같은 클래스의 멤버들 간 서로 호출할 수 있는 것처럼 생성자 간에도 서로 호출이 가능하다.
  - 생성자의 이름으로 클래스 이름 대신 this를 사용한다.
  - 한 생성자에서 다른 생성자를 호출할 때는 반드시 첫줄에서만 호출이 가능하다.

```
Car(String color) {
    door = 5;
    Car(color, "auto", 4); // error! 첫줄에서 생성자를 호출해야한다.
                           // error! Car가 아닌 this를 사용해야한다.
}
```

* 같은 클래스 내의 생성자들은 서로 연관 관계가 깊은 경우가 많다.
* 따라서 서로 호출하도록 하여 유기적으로 연결해주면 더 좋은 코드를 얻을 수 있다.
* 또한 수정이 필요한 부분이 적어지기에 유지보수가 쉬워진다.
* 생성자의 매개변수와 인스턴스 변수의 이름이 같은 경우 인스턴스 변수 앞에 `this.`를 붙여준다.
* `this`는 참조변수로 인스턴스 자신을 가리킨다. 
* 참조 변수를 통해 인스턴스의 멤버에 접근할 수 있는 것처럼 `this`로 인스턴스 변수에 접근할 수 있다.
* static 메서드에서는 인스턴스 멤버를 사용할 수 없는 것처럼, this를 사용할 수 없다. 

```
  this : 인스턴스 자신을 가리키는 참조변수. 인스턴스의 주소가 저장되어 있다.
  this(), this(매개변수) : 생성자. 같은 클래스의 다른 생성자를 호출할 때 사용한다.
```

### 생성자를 이용한 인스턴스의 복사
* 현재 사용하고 있는 인스턴스와 같은 상태를 갖는 인스턴스를 하나 더 만들고자 할 때 생성자를 이용할 수 있다.
* 하나의 클래스로부터 생성된 모든 클래스 멤버는 서로 동일하기에 인스턴스간의 차이는 인스턴스 변수뿐이다.

```
Car(Car c) {
  color = c.color;
  gearType = c.gearType;
  door = c.door;
}
```

* Object 클래스에 정의된 `clone`이라는 메서드는 이용하면 간단히 인스턴스를 복사할 수 있다.