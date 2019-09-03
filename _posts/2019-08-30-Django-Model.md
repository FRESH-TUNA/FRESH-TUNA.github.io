---
layout: post
title: "Django ORM - Models"
author: "DONGWON KIM"
meta: "Springfield"
comments: true
---

## 1. 프롤로그
ORM이란 프로그래밍언어상의 객체와 관계형 데이터베이스 간의 매핑을 의미한다.
RDBMS를 연동할 때, SQL이나 Stored Procedure를 사용할수 있지만 
ORM을 사용하면, 좀더 효율적으로 개발이 가능하다.

ORM에선 SQL문을 사용하지 않기 때문에 추상화되어 다양한 DBMS에서 돌아갈수 있는
확률이 높다. (추상화되어서 이식성이 높다.)
<br><br>

## 1. INSTALLED_APP
만약 flower라는 app이 있다고 가정했을때 flower.models를 통해 model 접근이 가능하다면 
다음과 같이 정의해야한다.

```python
INSTALLED_APPS = [
    'flower',
]
```
만약 floweryroad.apps.flower.models를 통해 model 접근이 가능하다면 다음과 같이 정의해야한다.

```python
INSTALLED_APPS = [
    'floweryroad.apps.flower',
]
```

위와 같은 상황에서 만약 appconfig를 통해 직접접근 하고 싶다면 다음과 같이 
INSTALLED_APPS 변수와 appconfig를 수정해줘야 한다.
name에 floweryroad.apps.flower.models를 통해 접근할수 있으므로 name은
floweryroad.apps.flower가 된다.

```python
INSTALLED_APPS = [
    'floweryroad.apps.flower.apps.Flowerconfig',
]
```

```python
from django.apps import AppConfig

class Test4Config(AppConfig):
    name = 'floweryroad.apps.flower' 
    

```
<br><br>