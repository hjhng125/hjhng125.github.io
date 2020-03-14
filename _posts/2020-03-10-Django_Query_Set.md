---
title:  "Django QuerySet"
excerpt: "Django QuerySet"

categories:
  - Django
tags:
  - Django
  - ORM
  - QuerySet
last_modified_at: 2020-03-14T023:43:00
---

* Query란 데이터베이스에 정보를 요청해주는 것을 의미하며 Python으로 작성한 코드가 SQL로 매핑되어 QuerySet이라는 자료 형태로 값이 넘어온다.
* 이는 Iterable한 객체로써 이를 이용해 1개 이상의 데이터를 불러와 사용할 수 있다.
* QuerySet은 Lazy한 특성을 갖고 있는데, 이를 이해하여 DB access 최적화해야한다.

## QuerySet 생성
 
```

from django.db import models

class User(models.Model):
    id = models.IntegerField(primary_key=True, null=False, unique=True)
    name = models.CharField(max_length=30)

```
`QuerySet`을 생성하기 위해서는 ModelName.objects를 통해 가능하다.
위의 예시는 `User.objects`

## 조회
* all : 모든 데이터를 가져온다.

```
User.objects.all()
```

* filter : 특정 데이터를 필터링하여 가져온다. 조건이 2개 이상 `,` 로 구분되어 들어가면 조건을 `and`로 묶어준다. `or`로 묶기 위해선 Q 를 사용해야 한다.

```
from django.db.models import Q

User.objects.filter()
User.objects.filter(Q(id=1) | Q(name='이름'))

```

* exclude : filter와 상반되는 개념으로, 조건을 제외한 결과를 가져온다.

```
User.objects.exclude()
```

* get : 조건에 대한 값을 가져오며, 그 데이터는 유일해야 한다. 0개일 경우 `DoesNotExist`, 2개 이상일 경우 `MultipleObjectsReturned` 에러를 발생한다.

```
User.objects.get(id=1)
```

* first : 첫번째 데이터를 가져온다.

```
User.objects.first()
```

* last : 마지막 데이터를 가져온다.

```
User.objects.last()
```

* indexing, slicing 

```
User.objects[index]
User.objects[start:end:step]
```

* 조건을 통한 조회 방법
    * 필드명 <= 조건값 `필드명__lte`
    * 필드명 >= 조건값 `필드명__gte`
    * 필드명 < 조건값 `필드명__lt`
    * 필드명 > 조건값 `필드명__gt`

    * 필드명으로 시작하는 데이터 : `필드명__startswith`
    * 필드명으로 끝나는 데이터 : `필드명__endswith`
    * 필드명을 포함하는 데이터 : `필드명__contains`

   <br>
    **대소문자 구분 없이** 
    * 필드명으로 시작하는 데이터 : `필드명__istartswith`
    * 필드명으로 끝나는 데이터 : `필드명__iendswith`
    * 필드명을 포함하는 데이터 : `필드명__icontains`
    
## 정렬
> 데이터는 클래스 혹은 QuerySet에서 정렬가능하다.
* 모델 클래스에서 지정

```

from django.db import models

class User(models.Model):
    id = models.IntegerField(primary_key=True, null=False, unique=True)
    name = models.CharField(max_length=30)
    
    class Meta:
        ordering = ['id]
        # 역순
        # ordering = ['-id']
```

* QuerySet에서 지정

```

User.objects.all().order_by('id')
User.objects.all().order_by('-id')

```

두가지 방식을 모두 사용하면 QuerySet의 `order_by`지정을 따른다.