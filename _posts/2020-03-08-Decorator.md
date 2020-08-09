---
title:  "Python Decorator"
excerpt: "Python Decorator에 대하여"

categories:
  - Python
tags:
  - Decorator
last_modified_at: 2020-03-07
---

* 데코레이터는 이미 만들어져 있는 기존의 코드를 수정하지 않고, Wrapper 함수를 이용하여 여러가지 기능을 추가할 수 있게 해준다.

```

def decorator_func(original_func):
    def wrapper_func():
        return original_func()
    return wrapper_func


def display():
    print('display')


decorated_display = decorator_func(display)
decorated_display()

```

1. 데코레이션 함수인 decorator_func 정의
2. 일반 함수인 display 정의
3. decorated_display 변수에 display 함수를 인자로 갖는 decorator_func 할당
 -> 리턴 값은 wrapper_func 
4. decorated_display()를 통해 wrapper_func 가 호출
5. wrapper_func 실행
6. 'display' 출력

* 함수 앞에 '@' 심볼과 데코레이터 함수의 이름을 붙여 쓰는 간단한 구문을 통해 사용
* 인수를 받는 함수를 데코레이팅 하고 싶은 경우
 -> wrapper_func 에 (*args, **kwargs) 를 인자로 받는다.
* 데코레이터는 클래스 형식을 사용할 수도 있다.

```

class DecoratorClass:
    def __init__(self, original_func):
        self.original_func = original_func

    def __call__(self, *args, **kwargs):
        print(f'{self.original_func.__name__} 함수 호출')
        return self.original_func(*args, **kwargs)


@DecoratorClass
def display():
    print('display')


@DecoratorClass
def display_info(age, name):
    print(f'{age}, {name} display_info')


display()
display_info(25, 'jin hyung')

``` 