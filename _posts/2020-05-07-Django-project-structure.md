---
layout: post
title: "유연한 Django 프로젝트 구조잡기"
author: "DONGWON KIM"
meta: "Springfield"
categories: "Django"
comments: true
---

## 1. 프롤로그 
<br/> 
![Image Alt 텍스트](/img/2019/04/28/Django-101/django.png)
django는 프로젝트 구성에 있어서 자유도가 높은 프레임워크이다. 하지만 자유도가 너무 높은 나머지, 나는 프로젝트 구조를 짜면서 항상 방황해왔었다.
프로젝트 구조에 있어서 항상 정답은 없지만 우리가 개발했던 대학교 축제사이트 '머동머동'을 통해 best-practice라고 생각한 구조에 대해서 다루어보고자 한다.

## 2. Django App
Django 공식홈페이지에 의하면 APP은 비슷한 기능을 제공하는 유틸리티들의 집합으로 정의할수 있다. APP은 다른 모듈과 독립적이어도되고 종속적이어도 상관없다. 그래서 보통 django에서 multiapp 구조로 설계할때는 비슷한 기능별로 묶은 여러 APP으로 설계한다. 만약 이런 구조가 익숙하지 않거나 프로젝트의 규모가 크지 않다면 싱글앱구조로 개발하는것도 좋은 방법이다. 

## 3. 우리가짠 머동머동의 프로젝트 구조
```bash
├── base
│   ├── static
│   └── templates
├── config
│   ├── credentials
│   ├── environments
│   │   ├── base.py
│   │   ├── development.py
│   │   ├── production.py
│   ├── urls
│   │   ├── base.py
│   │   ├── development.py
│   │   └── production.py
│   └── wsgi.py
├── foodtruck
├── friendboard
├── index
├── lostboard
├── manage.py
├── qnaknuch
├── starter
```

머동머동은 base, config, foodtruck, friendboard, index, lostboard, qnaknuch, starter의 총 8개 app으로 구성되어있다. 이중 config 앱은 환경설정 기능을 담당하고, starter 앱은 개발환경 및 배포환경을 실행해주는 스크립트들을 가지고 있다. 
나머지 앱들은 사용자가 사용하는 서비스를 제공하기 위한 model, view, controller 가 담겨 있는 앱이다. 그중 base 앱은 여러앱에서 공통적으로 사용하는 유틸리티를 가지고 있다.

## 4. settings, urls 환경분리
```bash
├── config
│   ├── credentials
│   ├── environments
│   │   ├── base.py
│   │   ├── development.py
│   │   ├── production.py
│   ├── urls
│   │   ├── base.py
│   │   ├── development.py
│   │   └── production.py
│   └── wsgi.py
```

개발환경과 배포환경의 설정과 urlconf가 다르기 때문에 이를 분리해서 관리해야 한다.
공통적으로 들어가는 코드들을 base.py에다가 전부 집어넣고 development, production 등의 이름으로 파일을 만들어서 base.py 를 import 한후 개별설정을 추가한다. 설정 분리후에는 반드시 manage.py 와 wsgi.py를 다음과 같이 수정해준다.

```python
# manage.py

import os
import sys


def main():
    # 아래의 코드를 본인이 만든 개발용 환경설정 파일로 수정해준다.
    os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'config.environments.development')
    try:
        from django.core.management import execute_from_command_line
    except ImportError as exc:
        raise ImportError(
            "Couldn't import Django. Are you sure it's installed and "
            "available on your PYTHONPATH environment variable? Did you "
            "forget to activate a virtual environment?"
        ) from exc
    execute_from_command_line(sys.argv)


if __name__ == '__main__':
    main()

```

```python
# wsgi.py
import os

from django.core.wsgi import get_wsgi_application

# 아래의 코드를 본인이 만든 배포용 환경설정 파일로 수정해준다.
os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'config.settings.production')

application = get_wsgi_application()


```