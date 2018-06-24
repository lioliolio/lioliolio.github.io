---
layout: post
title: Django 환경 설정 및 파이썬 패키지 관리하기
date: 2018-06-24
---
### Django 설정 분리하기
개발 환경과 프로덕션 환경에서의 설정 값들은 달라집니다. 예를 들어, [DEBUG](https://docs.djangoproject.com/en/2.0/ref/settings/#std:setting-DEBUG) 같은 경우 절대 프로덕션 환경에서는 `True` 이면 안됩니다. 반면에, 개발환경에서 `True`로 설정해야 원할히 디버깅 할 수 있습니다.

[DATABASES](https://docs.djangoproject.com/en/2.0/ref/settings/#std:setting-DATABASES) 설정값 역시 프로덕션 환경과 개발 환경이 다릅니다. 따라서 개발 환경과 프로덕션 환경 설정을 분리하는 것이 좋습니다.

```
# settings/
#   __init__.py
#   base.py
#   development.py
#   production.py
```

```
# settings/development.py

from base import *

DEBUG = True

...
```
```
# settings/production.py

from base import *

DEBUG = False

...
```
- 공통으로 적용하는 설정은 `settings/base.py`에 있습니다.
- 환경 별로 다르게 설정해야 하는 경우는 각 설정 파일에 위와 같은 방법으로 적용하면 됩니다.
- 특정 환경을 사용할 경우 `--settings=` 에 사용할 환경을 입력하면 됩니다.

```
# blog라는 Django 프로젝트를 만들었다고 가정

$ python manage.py runserver --settings=blog.settings.development
```

### **DJANGO\_SETTINGS\_MODULE** 사용하기
매번 옵션으로 `--settings=`를 사용하는 일은 귀찮습니다. 이때 환경변수로 `DJANGO_SETTINGS_MODULE`을 설정하면 편합니다.

```
$ export DJANGO_SETTINGS_MODULE=blog.settings.development
```
- 위와 같이 `DJANGO_SETTINGS_MODULE` 환경 변수를 설정하면 `runserver`, `migrate` 등 **Django** 관련 명령어를 사용할 때 따로 `--settings` 옵션값을 주지 않아도 `blog.settings.development` 설정을 사용합니다.

### 파이썬 패키지 관리하기
**Django**로 개발을 하다보면 외부 패키지를 사용하는 경우가 많습니다. 외부 패키지를 관리하기 위해서 `requirements.txt` 파일을 사용합니다. `requirements.txt` 파일 안에 다음과 같이 작성하면 됩니다.

```
# requirements.txt

# 설치할 패키지==버전

Django==2.0.6
```

`reqirements.txt` 파일에 있는 패키지를 설치하기 위해서는 `$ pip install -r requirements.txt` 라는 명령어를 입력하면 설치할 수 있습니다.

### requirements 파일 분리하기
하나의 `requirements.txt`파일 만으로도 외부 패키지를 관리할 수 있는 경우도 있지만, 아닌 경우도 있습니다. 

개발환경을 프로덕션 환경과 개발 환경 두가지로 분리했을 때를 생각해봅시다. 개발환경에서는 개발할 때 도움을 받기 위해 사용하는 패키지가 있습니다. **[Django Debug Toolbar](https://django-debug-toolbar.readthedocs.io/en/stable/)**, **[Django Extensions](https://django-extensions.readthedocs.io/en/latest/)** 같은 패키지들이 그 예입니다. 이 패키지들은 개발환경에서 사용하고 프로덕션 환경에서는 일반적으로 필요하지 않은 패키지입니다.

이럴 경우 프로덕션 환경에서 사용할 `requirements` 파일과 개발 환경에서 사용할 `requirements`파일을 분리하는 것이 좋습니다.

```
# requirements/
#   production.txt
#   development.txt
# requirements.txt
```
```
# requirements.txt

-r requirements/production.txt
```
```
# requirements/production.txt

Django==2.0.6
...
```
```
# requirements/development.txt

-r production.txt

django-debug-toolbar==1.9.1
```
- 프로덕션 환경에서 사용하는 패키지 설치는 `pip install -r requirements.txt` 명령어를 사용하면 됩니다.
- 프로덕션 환경에서 사용하는 패키지는 `requirements/production.txt` 파일에 저장합니다.
- 개발 환경에서 사용하는 패키지 설치는 `pip install -r requirements/development.txt` 명령어를 사용하면 됩니다.
- 개발 환경에서 사용하는 패키지는 `requirements/development.txt` 파일에 저장합니다.
- 위의 방법은 개인적으로 사용하고 있는 방법입니다. 외부에 `requirements.txt` 파일을 두지 않고 `pip install -r requirements/production.txt`로 바로 설치해도 됩니다.

### 참고
- Two Scoops Of Django
- [Django Documentation](https://docs.djangoproject.com/en/2.0/)