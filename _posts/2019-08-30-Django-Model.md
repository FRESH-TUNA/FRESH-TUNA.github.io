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

## 2. One to One Model
모델에 대해 추가적인 정보를 저장하고 싶지만 핵심적인 내용은 아닐때 일대일 대응 관계를 사용한다.
One-to-one 모델의 역참조는 하나의 model obejct를 반환하지만, ForeignKey의 역참조는 QuerySet 을 반환하는 점이 차이점이다.
ForeignKey(모델이름, unique=True)로도 unique-field를 쓸수 있지만 queryset을 써야하기때문에 그냥 onetoonefield 쓰자 

Django에서는 사용자 Entity를 구현하는 방법이 3가지가 있는데 하나는 주어진대로 사용하기, 
둘째는 custom user model 만들기, 셋째는 one to one model로 주어진대에 덧붙여 사용하는방법이다.

## 3. 참고한 자료
https://cjh5414.github.io/extending-user-model-using-one-to-one-link/
https://cjh5414.github.io/extending-user-model-using-one-to-one-link/