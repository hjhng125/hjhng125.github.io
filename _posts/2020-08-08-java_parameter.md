---
title:  "기본형 매개변수와 참조형 매개변수"
excerpt: "기본형 매개변수와 참조형 매개변수에 대하여 공부한 내용을 기술합니다."

categories:
  - Java
tags:
  - Parameter

last_modified_at: 2020-08-08T0018:51:00
---

> 자바에서는 메서드를 호출할 때 매개변수로 지정한 값을 메서드의 매개변수에 복사해서 넘겨준다. 이 때, 매개변수의 타입이 기본형(primitive type)이면 기본형 값이 복사되겠지만, 참조형(reference type)일 때는 인스턴스의 주소가 복사된다.

```
  기본형 매개변수 : 변수의 값을 읽기만 할 수 있다. (read only)
  참조형 매개변수 : 변수의 값을 읽고 변경할 수 있다. (read & write)
```

 ![1](/assets/images/parameter_data.png)
 ![1](/assets/images/parameter_main.png)
 ![1](/assets/images/parameter_result.png)

 위 결과를 보면 기본형 매개변수로 넘겨받은 값을 변경했지만 main 메서드 내의 객체의 x값이 변하지 않았다.

 * changeX 메서드가 호출되면서 data 객체의 x가 changeX()의 매개변수 x에 복사
 * changeX 내에서 x의 값을 변경
 * changeX 메서드가 종료되면서 매개변수 x는 스택에서 제거

 data의 x 의 값이 변경된 것이 아니라, changeX 메서드의 매개변수 x의 값이 변경되었기에 main 메서드의 원본에는 영향을 미치지 않은 것이다.
 이처럼 기본형 매개변수는 변수의 저장된 값만을 읽을 수 있으며 변경할 수는 없다.

 ![1](/assets/images/parameter_refer.png)
 ![1](/assets/images/parameter_refer_result.png)

 위 결과에선 참조형 매개변수로 넘겨받은 값을 변경함으로써 main 메서드의 원본값에 영향을 미치는 것을 확인할 수 있다.

 * changeData 메서드가 호출되면서 참조변수 data의 값(주소값)이 매개변수 data에 복사
 * 매개변수 data의 x값을 변경
 * changeData 메서드가 종료되며 매개변수 data는 스택에서 제거

 매개변수가 값이 아닌 `값이 저장된 주소`를 매개변수로 넘겨주었기 때문에 값을 읽을 수 있을뿐 아니라, 변경하는 것 또한 가능하다.
 main 메서드의 data와 changeData 메서드의 매개변수 data는 같은 객체를 가리킨다.

 * 배열 또한 객체와 같이 참조변수를 통해 데이터가 저장된 공간에 접근한다. 
 * 임시적으로 간단히 처리할 떄는 클래스를 선언하는 것보다 배열을 이용할 수도 있다.

 ![1](/assets/images/parameter_array.png)
 ![1](/assets/images/parameter_array_result.png)

 * `반환값이 있는 메서드는 하나의 값만을 반환할 수 있지만 참조형 매개변수를 활용하면 여러개의 값을 반환받는 것과 같은 효과를 얻을 수 있다.`