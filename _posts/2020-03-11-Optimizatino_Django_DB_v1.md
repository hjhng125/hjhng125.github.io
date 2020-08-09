---
title:  "Django DB 최적화 #1"
excerpt: "Django DB 최적화 에 대하여"

categories:
  - Django
tags:
  - Django
  - QuerySet 
  - ORM
  - 최적화
last_modified_at: 2020-03-07
---

Django에서 DB를 최적화하는 것은 성능면에서 상당한 장점을 갖지만 코드의 가독성을 흐리게 되는 trade off가 존재한다.
그렇기에 맹목적인 최적화가 아닌 상황에 알맞게 최적화를 사용하는 것이 중요하다.

## Basic
Django에서 진행할 최적화를 하기 전, DB의 index나, field를 잘 정의하는 DB 최적화가 우선적으로 수행되어야 한다.

## QuerySet의 특징
이전 포스트에서 언급했듯, QuerySet은 Lazy한 특성을 갖고 있다.

* 코드 상에서 QuerySet을 생성하는 작업은 DB에 아무런 영향을 끼치지 않는다.
* DB의 query는 QuerySet이 계산될 때까지 실제로 query를 생성하지 않는다.
* 실제로 계산되는 시점
    * 반복 (Iteration)
    * 슬라이싱 (Slicing)
    * 피클링/캐싱 (Pickling/Caching)
        * 피클링 : 캐싱의 타이밍을 임의로 정의하는 기법
    * repr(), len(), list(), bool() etc.

## QuerySet Caching 
* QuerySet이 연산되면 그 결과값은 메모리에 캐싱된다. 따라서 그 후에 같은 QuerySet을 연산하려고 할 경우, 캐시된 값이 불려진다.

```
# DB query가 두번 발생함
print([e.headline for e in Entity.objects.all()])
print([e.pub_date for e in Entity.objects.all()])

# DB query가 한번 발생함
querySet = Entity.objects.all()
print([p.headline for p in querySet]) # QuerySet이 연산됨 
print([p.pub_date for p in querySet]) # 캐시되어 있는 결과값 사용
```

* Python에서 non-callable한 속성들은 캐싱된다.

```
entry = Entry.objects.get(id=1)
entry.blog # blog 객체가 DB에서 불려옴
entry.blog # 캐시에서 불려옴

entry = Entry.objects.get(id=1)
entry.authors.all() # DB query가 진행됨
entry.authors.all() # DB query가 다시 한번 진행됨
```

* 커스텀으로 생성한 property들은 `cached_property` 데코레이터를 사용하여 캐시되게 할 수 있음.
* QuerySet의 일부만 연산된다면(slicing or indexing) 캐시된 값이 있는지 확인은 하지만, 결과값을 캐싱하진 않는다.
 => QuerySet 전체가 연산된 경우가 있을 경우만 생성된 캐시값을 사용한다.
 
```
querySet = Entry.objects.all()
print(querySet[5]) # DB query가 진행됨
print(querySet[5]) # DB query가 다시 한번 진행됨

querySet = Entry.objects.all()
[item for item in querySet] # DB query가 진행됨
print(querySet[5]) # 캐시값 사용
print(querySet[5]) # 캐시값 사용
```

