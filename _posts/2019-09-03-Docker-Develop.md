---
layout: post
title: "Docker로 Django 개발환경 구축하기"
author: "DONGWON KIM"
meta: "Springfield"
categories: "CI/CD"
comments: true
---

## 1. 프롤로그
Docker를 활용하면 컨테이너로 가상화 되어 환경의 제약없이 배포를 할수 있다.
웹서비스는 개발 -> 테스트 -> 배포의 과정을 거쳐 제작된다.
그냥 일반적인 환경에서 작업을 해도 무방하나 개발환경과 배포환경의 차이를 줄이고 싶어서
(나의 스트레스를 줄여 수명도 늘리면서) 개발환경도 도커로 돌려보려한다.
<br><br>

## 2. 도커 설치
다음 링크를 참조하여 도커설치를 진행해보자!!
윈도우: https://steemit.com/kr/@mystarlight/docker
맥: https://myjamong.tistory.com/105
<br><br>

## 3. 도커 컴포즈 설치
다음 링크를 참조하여 도커 컴포즈를 진행해보자!!
https://17billion.github.io/docker/2017/04/02/docker_compose_install_exec.html
<br><br>

## 4. Dockerfile 작성
Dockerfile은 우리가 작업을 하거나 배포할 컨테이너를 만들수 있는 레시피라고 할수 있다.
Dockerfile 작성후 빌드를 통해 이미지가 생성되고, 그 이미지를 통해 우리가 원하는 컨테이너를 
마구마구 찍어낼수 있다!
Dockerfile이란 이름을 가진 파일을 생성한후 다음과 같이 작성했다. 
```bash
FROM        ubuntu:latest
MAINTAINER  lunacircle4@gmail.com

RUN         apt-get -y update
RUN         apt-get install -y tzdata
RUN         apt-get -y dist-upgrade
RUN         apt-get install -y python-pip git vim

# pyenv
RUN         apt-get install -y make build-essential libssl-dev zlib1g-dev libbz2-dev \
libreadline-dev libsqlite3-dev wget curl llvm libncurses5-dev libncursesw5-dev \
xz-utils tk-dev
RUN         curl -L https://raw.githubusercontent.com/yyuu/pyenv-installer/master/bin/pyenv-installer | bash
ENV         PATH /root/.pyenv/bin:$PATH
RUN         pyenv install 3.6.4

# zsh
RUN         apt-get install -y zsh
RUN         wget https://github.com/robbyrussell/oh-my-zsh/raw/master/tools/install.sh -O - | zsh || true
RUN         chsh -s /usr/bin/zsh

# pyenv settings
RUN         echo 'export PATH="/root/.pyenv/bin:$PATH"' >> ~/.zshrc
RUN         echo 'eval "$(pyenv init -)"' >> ~/.zshrc
RUN         echo 'eval "$(pyenv virtualenv-init -)"' >> ~/.zshrc

# virtualenv floweryroad
COPY    ./requirements.txt /app/floweryroad-backend/requirements.txt
RUN     pyenv virtualenv 3.6.4 floweryroad
RUN     /root/.pyenv/versions/floweryroad/bin/pip install --upgrade pip
RUN     /root/.pyenv/versions/floweryroad/bin/pip install -r /app/floweryroad-backend/requirements.txt 
```
<br><br>

## 5. 빌드를 통해 이미지 생성하기 
도커파일 작성이 끝나면 docker build 명령어로 빌드를 실행할수 있다.
-t 옵션으로 이미지의 이름과 태깅을 지정할수 있고 -f 옵션으로 Dockerfile을 위치를 설정한다.
마지막의 . 은 빌드가 진행될 문맥을 뜻하는데 다음명령어를 보며 한번 생각해보자
```bash
COPY  ./requirements.txt /app/floweryroad-backend/requirements.txt
```
COPY나 VOLUME 명령어는 ../같이 상위폴더로의 접근이 불가능하다 따라서 requirements.txt를 복사하기 위해선
빌드 명령어의 마지막 옵션에 requirements.txt가 존재하는 경로를 집어넣어주어야한다.
이를 고려하여 다음과 같이 최종명령어를 작성해보았다.
```bash
docker build -t floweryroad-backend:develop  -f docker/dev/Dockerfile .
```
<br><br>

## 6. Docker compose를 활용하여 쉽게 컨테이너 생성하기
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