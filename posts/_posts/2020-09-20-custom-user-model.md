---
layout: post
title: "Django 커스텀 User Model 사용하기"
author: "DONGWON KIM"
meta: "Springfield"
categories: "Django"
comments: true
---

## 1. 개요
Django가 기본적으로 제공해주는 User 모델에 부족함을 느낄때 직접 정의하여 사용할수 있다.
<br><br>

## 2. Custom User Model
```python
from django.db import models
from django.utils import timezone
from django.contrib.auth.models import (BaseUserManager, AbstractBaseUser)
from core.models import (Purpose, Color)

# UserManager 재정의
class UserManager(BaseUserManager):
    def create_user(self, **kwargs):
        if not "email" in kwargs:
            raise ValueError('Users must have an email address')
        if not "password" in kwargs:
            raise ValueError('Users must have an password')

        user = self.model(
            email=self.normalize_email(kwargs.get("email")),
        )
        user.set_password(self.normalize_email(kwargs.get("password")))
        user.save(using=self._db)
        return user

    def create_superuser(self, **kwargs):
        new_superuser = self.create_user(**kwargs)
        new_superuser.is_admin = True
        new_superuser.save(using=self._db)
        return new_superuser

# 커스텀 유저 모델
class User(AbstractBaseUser):
    # username으로 쓰고싶은것에 반드시 unique 지정
    email = models.EmailField(
        verbose_name='email',
        max_length=255,
        unique=True,                                
    )
    nickname = models.CharField(max_length=30, blank=True)
    purpose = models.ForeignKey(Purpose, on_delete=models.SET_NULL, null=True, blank=True)
    color = models.ForeignKey(Color, on_delete=models.SET_NULL, null=True, blank=True)
    is_active = models.BooleanField(default=True)
    is_admin = models.BooleanField(default=False)
    date_joined = models.DateTimeField(default=timezone.now)

    # 위에서 정의한 UserManager() 사용
    objects = UserManager()

    # username으로 사용되는 필드. 반드시 unique일것
    USERNAME_FIELD = 'email'                 

    # createsuperuser 명령어에서 반드시 필요한 필드들을 정의.
    # REQUIRED_FIELDS = ['nickname']                  

    def __str__(self):
        return self.email

    def has_perm(self, perm, obj=None):
        return True

    def has_module_perms(self, app_label):
        return True

    @property
    def is_staff(self):
        return self.is_admin

```
<br><br>

## 3. apps.py
```python
from django.apps import AppConfig

class AuthConfig(AppConfig):
    # name is app folder name
    name = 'auth'

    # label is unique seperator on database
    label = 'floweryroad-auth' 
```
<br><br>

## 4. settings.py
```python
# Application definition
PROJECT_APPS = [
    'core',
    'auth.apps.AuthConfig'
]

INSTALLED_APPS = PROJECT_APPS + [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'rest_framework',
    'django_filters',
    'storages',
    'corsheaders',
]

# AUTH_USER_MODEL must be of the form 'app_label.model_name'
AUTH_USER_MODEL = 'floweryroad-auth.User'
```

## 5. 커스텀 유저 모델 사용 예제
```python
import config.environments.base as settings
from django.db import models
from django.core.validators import MaxValueValidator, MinValueValidator
from . import Purpose, Color, Language, Flower

class Comment(models.Model):
    # settings 에서 가지고 온다.
    user = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE)
    flower = models.ForeignKey(Flower, on_delete=models.CASCADE, related_name='comments')
    content = models.CharField(max_length=200, blank=True)
    star = models.FloatField(validators=[MinValueValidator(0.0), MaxValueValidator(5.0)], default=0.0)
    created_at = models.DateTimeField(auto_now=True)

    @property
    def like(self):
        #좋아요를 취소하면 반드시 지운다고 가정하여 설계했음
        return self.comment_likes.count()
```
