---
layout: post
title: "Django QuerySet의 지연된 실행"
date: 2019-03-24
---

Django ORM을 더 효율적으로 사용하기 위해 **[쿼리셋(QuerySet)](https://docs.djangoproject.com/en/2.1/ref/models/querysets/#django.db.models.query.QuerySet)**을 만드는 경우 언제 쿼리문을 실행하는지에 대해 정리했습니다.

### 쿼리셋(QuerySet)의 지연된 실행(lazy evaluation)

다음 Django 모델 코드가 있습니다.

```
# https://docs.djangoproject.com/en/2.1/topics/db/queries/#making-queries

from django.db import models


class Blog(models.Model):
    name = models.CharField(max_length=100)
    tagline = models.TextField()

    def __str__(self):
        return self.name


class Author(models.Model):
    name = models.CharField(max_length=200)
    email = models.EmailField()

    def __str__(self):
        return self.name


class Entry(models.Model):
    blog = models.ForeignKey(Blog, on_delete=models.CASCADE)
    headline = models.CharField(max_length=255)
    body_text = models.TextField()
    pub_date = models.DateField()
    mod_date = models.DateField()
    authors = models.ManyToManyField(Author)
    n_comments = models.IntegerField()
    n_pingbacks = models.IntegerField()
    rating = models.IntegerField()

    def __str__(self):
        return self.headline
```

아래의 코드는 Django ORM을 사용하여 **Entry** 모델 쿼리셋을 만듭니다

```
# https://docs.djangoproject.com/en/2.1/topics/db/queries/#querysets-are-lazy

>>> q = Entry.objects.filter(headline__startswith="What")
>>> q = q.filter(pub_date__lte=datetime.date.today())
>>> q = q.exclude(body_text__icontains="food")

>>> print(q)
```

쿼리셋을 만들면서 몇 번 쿼리문을 실행할까요?

**fitler**를 두 번, **exclude** 한 번 실행했으니까 총 세 번일까요? 그렇지 않습니다.

`print(q)`를 할 때 단 한 번 다음 쿼리문을 실행합니다.

```
# raw query

SELECT "blog_entry"."id",
       "blog_entry"."blog_id",
       "blog_entry"."headline",
       "blog_entry"."body_text",
       "blog_entry"."pub_date",
       "blog_entry"."mod_date",
       "blog_entry"."n_comments",
       "blog_entry"."n_pingbacks",
       "blog_entry"."rating" 
FROM "blog_entry" 
WHERE ("blog_entry"."headline" LIKE \'What%\' ESCAPE \'\\\' 
AND "blog_entry"."pub_date" <= \'2019-03-24\' 
AND NOT ("blog_entry"."body_text" LIKE \'%food%\' ESCAPE \'\\\'))  
LIMIT 21'
```
**[QuerySets are lazy](https://docs.djangoproject.com/en/2.1/topics/db/queries/#querysets-are-lazy)** 문서에서 볼 수 있듯이 쿼리셋을 만든다고 바로 쿼리문을 실행하지 않습니다.

사용자가 쿼리셋의 결과를 요청하기 전까지 쿼리문을 실행하지 않습니다. 위의 코드에서는 쿼리셋을 출력하기 위해 `print(q)`를 실행하는 시점에 쿼리문을 실행했습니다.

그러면 언제 쿼리문을 실행하는 것일까요? 


### 쿼리문을 실행하는 시점

실제 실행하는 raw query를 확인하려면 다음 패키지를 사용하면 됩니다.

```
from django.db import connection
from django.db import reset_queries


connection.queries
reset_queries()
```

`connection.queries`는 지금까지 실행한 raw query에 대한 정리를 리스트 형식으로 다음과 같이 담고 있습니다.

```
[{'sql': 'SELECT "blog_entry"."id", "blog_entry"."blog_id", "blog_entry"."headline", "blog_entry"."body_text", "blog_entry"."pub_date", "blog_entry"."mod_date", "blog_entry"."n_comments", "blog_entry"."n_pingbacks", "blog_entry"."rating" FROM "blog_entry" WHERE "blog_entry"."rating" > 5',
  'time': '0.003'},
 {'sql': 'SELECT "blog_entry"."id", "blog_entry"."blog_id", "blog_entry"."headline", "blog_entry"."body_text", "blog_entry"."pub_date", "blog_entry"."mod_date", "blog_entry"."n_comments", "blog_entry"."n_pingbacks", "blog_entry"."rating" FROM "blog_entry"',
  'time': '0.003'}]
```

주의할 점은 대화형 쉘에서 실행한 raw query를 계속해서 하나씩 추가하면서 저장합니다. 따라서 어떤 특정 로직에 대한 raw query 만 확인하고 싶다면 중간 중간 `reset_queries()` 를 실행하면 기존에 있던 raw query 정보를 삭제할 수 있습니다.

#### 반복문 실행 시

쿼리셋은 반복 가능한 객체입니다.

```
>>> qeuryset = Entry.objects.all()
```

쿼리셋을 만들었지만 어떠한 쿼리문을 실행하지 않습니다.

```
>>> queryset = Entry.objects.all()

>>> for query in queryset:
>>>     pass
```

위와 같이 쿼리셋을 반복하는 경우에는 아래의 쿼리문을 실행합니다.

```
# raw query

SELECT "blog_entry"."id", 
       "blog_entry"."blog_id", 
       "blog_entry"."headline", 
       "blog_entry"."body_text", 
       "blog_entry"."pub_date", 
       "blog_entry"."mod_date", 
       "blog_entry"."n_comments", 
       "blog_entry"."n_pingbacks", 
       "blog_entry"."rating" 
FROM "blog_entry"
```

#### Slicing

쿼리셋에 파이썬 슬라이스 구문을 사용할 수 있습니다. (제약 사항이 있습니다.)

```
>>> queryset = Entry.objects.all()
>>> slice_queryset = queryset[:6]
```

위의 코드처럼 슬라이스를 하는 경우 아직 쿼리문을 실행하지 않는 쿼리셋을 반환합니다.

하지만 슬라이스에 스텝을 넣으면 쿼리문을 실행한 쿼리셋을 반환합니다.

```
>>> queryset = Entry.objects.all()
>>> slice_queryset = queryset[:6:2]
```

```
# raw query

SELECT "blog_entry"."id", 
       "blog_entry"."blog_id", 
       "blog_entry"."headline", 
       "blog_entry"."body_text", 
       "blog_entry"."pub_date", 
       "blog_entry"."mod_date", 
       "blog_entry"."n_comments", 
       "blog_entry"."n_pingbacks", 
       "blog_entry"."rating" 
FROM "blog_entry"  
LIMIT 6
```

#### Pickling/Caching
다음 글에서 자세히 설명하겠습니다.

#### repr() 메서드 실행 시

`__repr()__` 메서드 실행 시 쿼리문을 실행합니다.

```
>>> queryset = Entry.objects.all()
>>> queryset.__repr__()
```

```
# raw query

SELECT "blog_entry"."id", 
       "blog_entry"."blog_id", 
       "blog_entry"."headline", 
       "blog_entry"."body_text", 
       "blog_entry"."pub_date", 
       "blog_entry"."mod_date", 
       "blog_entry"."n_comments", 
       "blog_entry"."n_pingbacks", 
       "blog_entry"."rating" 
FROM "blog_entry"  
LIMIT 21
```

#### len() 함수 실행 시

`len()` 함수를 사용하여 쿼리셋의 길이를 반환할 때 쿼리문을 실행합니다.

```
>>> queryset = Entry.objects.all()
>>> len(queryset)
```

```
# raw query

SELECT "blog_entry"."id", 
       "blog_entry"."blog_id", 
       "blog_entry"."headline", 
       "blog_entry"."body_text",
       "blog_entry"."pub_date", 
       "blog_entry"."mod_date", 
       "blog_entry"."n_comments", 
       "blog_entry"."n_pingbacks", 
       "blog_entry"."rating" 
FROM "blog_entry"'
```

#### list() 함수 실행 시

쿼리셋을 `list()` 함수를 사용하여 리스트를 반환하는 경우 쿼리문을 실행합니다.

```
>>> list(Entry.objects.all())
```

```
# raw sql

SELECT "blog_entry"."id", 
       "blog_entry"."blog_id", 
       "blog_entry"."headline", 
       "blog_entry"."body_text", 
       "blog_entry"."pub_date", 
       "blog_entry"."mod_date", 
       "blog_entry"."n_comments", 
       "blog_entry"."n_pingbacks", 
       "blog_entry"."rating" 
FROM "blog_entry"
```

### 쿼리셋의 참, 거짓 여부를 테스트하는 경우(if 문, bool() 함수, boolean operation)

if 문 또는 bool() 함수를 사용하여 특정 조건으로 참, 거짓을 테스트하는 경우에 쿼리문을 실행합니다.
```
>>> bool(Entry.objects.filter(rating__gt=5))
```

```
# raw sql

SELECT "blog_entry"."id", 
       "blog_entry"."blog_id", 
       "blog_entry"."headline", 
       "blog_entry"."body_text", 
       "blog_entry"."pub_date", 
       "blog_entry"."mod_date", 
       "blog_entry"."n_comments", 
       "blog_entry"."n_pingbacks", 
       "blog_entry"."rating" 
FROM "blog_entry" 
WHERE "blog_entry"."rating" > 5
```

### 참고
- [QuerySets are lazy](https://docs.djangoproject.com/en/2.1/topics/db/queries/#querysets-are-lazy)
- [When QuerySets are evaluated](https://docs.djangoproject.com/ko/2.1/ref/models/querysets/#when-querysets-are-evaluated)




