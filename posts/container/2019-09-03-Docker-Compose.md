---
layout: post
title: "Docker compose를 활용하여 쉽게 컨테이너 생성하기"
# author: "DONGWON KIM"
# meta: "Springfield"
categories: "Infra"
comments: true
---

# 1. Docker compose를 활용하여 쉽게 컨테이너 생성하기
먼저 db_credential.env 를 만들어서 다음과 같이 작성해주자
```bash
POSTGRES_DB=5432
POSTGRES_USER=hoho
POSTGRES_PASSWORD=hoho
POSTGRES_INITDB_ARGS=--encoding=UTF-8
```

그리고 credential.env 를 만들어서 다음과 같이 작성해주자
```bash
WEB_HOST=127.0.0.1

DB_HOST=db
DB_PORT=5432
DB_NAME=hoho
DB_USER=hoho
DB_PASSWORD=1234

SECRET_KEY=hohohohhohhohohohhohohhoohohoho
MEDIA=http://localhost:8000/media/
```

db_credential.env, credential.env 작성이 끝났으면 이제 파일이름을 docker-compose.yml로 하여 생성하고 다음과 같이 작성해주자
db_credential.env, credential.env, docker-compose.yml는 같은 경로에 위치해야 한다!

```bash
version: '3'

services:
  db:
    image: postgres
    env_file: 
      - ./db_credential.env
    volumes:
      - ../db_volume:/var/lib/postgresql/data

  web:
    image: floweryroad-backend:develop
    env_file: 
      - ./credential.env
    volumes:
      - ../../:/app/floweryroad-backend
    ports:
      - "8000:8000"
    depends_on: 
      - db
    command: 
      - /bin/zsh
      - -c
      - |
        source /root/.pyenv/versions/floweryroad/bin/activate
        python /app/floweryroad-backend/manage.py migrate
        python /app/floweryroad-backend/manage.py runserver 0.0.0.0:8000
```

credential.env 에 정의한 내용들은 settings.py에서 다음과 같이 
os.environ['']을 통해서 불러쓰게 된다!
```python
SECRET_KEY = os.environ['SECRET_KEY']

ROOT_URLCONF = 'floweryroad.urls.development'
#media 설정
MEDIA_ROOT = os.path.join(BASE_DIR, 'media')

MEDIA_URL = os.environ['MEDIA']

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql_psycopg2',
        'NAME': os.environ['DB_NAME'],
        'USER': os.environ['DB_USER'],
        'PASSWORD': os.environ['DB_PASSWORD'],
        'HOST': os.environ['DB_HOST'],
        'PORT': os.environ['DB_PORT']
    }
}

```

이제 다음명령어로 개발환경을 돌려보자!!
```bash 
docker-compose -f docker/dev/docker-compose.yml up
```