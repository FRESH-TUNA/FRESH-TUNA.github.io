---
layout: post
title: "Spring boot 기반의 도커파일 작성"
# author: "DONGWON KIM"
# meta: "Springfield"
tags: [Java, infra, container]
categories: "Infra"
comments: true
---

## 1. 프롤로그
컨테이너 가상화 기법을 사용하면 환경의 제약없이 큰 성능 저하 없이 프로젝트를 딜리버리 할수 있습니다.
이 포스팅에서는 가벼운 이미지를 만들기 위한 시도들에 대해 작성합니다.
<br><br>

## 2. Base 이미지의 선정
Dockerfile은 우리가 작업을 하거나 배포할 컨테이너를 만들수 있는 레시피라고 할수 있다.
Dockerfile 작성후 빌드를 통해 이미지가 생성되고, 그 이미지를 통해 우리가 원하는 컨테이너를 
마구마구 찍어낼수 있다!
Dockerfile이란 이름을 가진 파일을 생성한후 자주 사용하는 명령어들을 정리해보았다. 

```bash
# FROM: base로 선택할 이미지, 없으면 remote에서 pull해온다.
FROM        ubuntu:latest

# LABEL: 라벨을 생성하며 docker inspect 커맨드를 통해 label들을 확인 할 수 있다.
LABEL MAINTAINER="lunacircle4@gmail.com"

# ENV: 환경변수로 지정하여 사용할수 있다.
ENV     PIP /root/.pyenv/versions/floweryroad/bin/pip
RUN     $PIP install --upgrade pip

# WORKDIR: 컨테이너 안에서의 Working directory를 지정해준다. Linux의 cd 커맨드의 개념으로 생각하면 된다.
WORKDIR     /app

# USER: 컨테이너 실행시 user를 선택할수 있다.
USER ubuntu

# RUN: 컨테이너 안에서 명령어를 실행한다. 이미지가 빌드되면서 실행된다.
RUN         apt-get -y update
RUN         apt-get install -y tzdata
RUN         apt-get -y dist-upgrade
RUN         apt-get install -y python-pip git vim
RUN         apt-get install -y make build-essential libssl-dev zlib1g-dev libbz2-dev \
libreadline-dev libsqlite3-dev wget curl llvm libncurses5-dev libncursesw5-dev \
xz-utils tk-dev

# CMD: 빌드된 컨테이너가 실행되었을때 실행할 구문을 작성한다. RUN과는 다르다!!
CMD [ "npm", "start" ]

# COPY: 현재 context 경로 기준으로, 파일과 디렉토리를 호스트에서 docker image로 copy 한다
COPY    ./requirements.txt /app/floweryroad-backend/requirements.txt
```


<br><br>

## 3. 빌드를 통해 이미지 생성하기
도커파일 작성이 끝나면 docker build 명령어로 빌드를 실행할수 있다.
-t 옵션으로 이미지의 이름과 태깅을 지정할수 있고 -f 옵션으로 Dockerfile을 위치를 설정한다.
마지막의 . 은 빌드가 진행될 문맥을 뜻하는데 다음명령어를 보며 한번 생각해보자
```bash
COPY  ./requirements.txt /app/floweryroad-backend/requirements.txt
```
COPY나 VOLUME 명령어는 ../같이 상위폴더로의 접근이 불가능하다 따라서 requirements.txt를 복사하기 위해선
빌드 명령어의 마지막 옵션에, requirements.txt가 하위에 존재하는 경로를 집어넣어주어야한다.
현재경로 하위에 requirements.txt가 위치 하므로 이를 고려하여 다음과 같이 최종명령어를 작성해보았다.
```bash
docker build -t floweryroad-backend:develop  -f docker/dev/Dockerfile .
```

만약 현재 경로에 상관없이 빌드를 진행하고 싶으면 context의 절대경로와 dockerfile이 위치한 절대경로를 사용하면 된다. 
다만 dockerfile의 경로는반드시 빌드 context의 하위경로에 위치해야한다.
```bash
docker build -t lunacircle4/floweryroad-backend:1.3.2  -f  \
/Users/kimdongwon/Documents/WebProgramming/Projects/Floweryroad/floweryroad-backend/web/docker/prod/Dockerfile \
/Users/kimdongwon/Documents/WebProgramming/Projects/Floweryroad/floweryroad-backend/web
```

만약 태그를 여러개 붙이고 싶으면 어떻게 해야할까?? -t를 여러개 사용하면 된다! 
```bash
docker build -t lunacircle4/floweryroad-backend:1.3.2 -t lunacircle4/floweryroad-backend:latest -f  \
/Users/kimdongwon/Documents/WebProgramming/Projects/Floweryroad/floweryroad-backend/web/docker/prod/Dockerfile \
/Users/kimdongwon/Documents/WebProgramming/Projects/Floweryroad/floweryroad-backend/web
```
<br><br>

### 4. Django image를 만들어서 개발, 배포 해보기
#### 1. django_base 이미지 만들기
```bash
FROM        ubuntu:latest
MAINTAINER  lunacircle4@gmail.com

RUN         apt-get -y update
RUN         apt-get install -y tzdata
RUN         apt-get -y dist-upgrade
RUN         apt-get install -y python-pip git vim
RUN         apt-get install -y make build-essential libssl-dev zlib1g-dev libbz2-dev \
libreadline-dev libsqlite3-dev wget curl llvm libncurses5-dev libncursesw5-dev \
xz-utils tk-dev libpq-dev libjpeg-dev

# pyenv
RUN         curl -L https://raw.githubusercontent.com/yyuu/pyenv-installer/master/bin/pyenv-installer | bash
ENV         PATH /root/.pyenv/bin:$PATH
RUN         pyenv install 3.6.4
RUN         echo 'export PATH="/root/.pyenv/bin:$PATH"' >> ~/.zshrc
RUN         echo 'eval "$(pyenv init -)"' >> ~/.zshrc
RUN         echo 'eval "$(pyenv virtualenv-init -)"' >> ~/.zshrc

# zsh
RUN         apt-get install -y zsh
RUN         wget https://github.com/robbyrussell/oh-my-zsh/raw/master/tools/install.sh -O - | zsh || true
RUN         chsh -s /usr/bin/zsh

# supervisord install
RUN         apt-get -y install supervisor
```

나는 Django 프레임워크를 가지고 풀꽃길, 족제비, 머동머동등의 프로젝트를 진행하는중이다.
따라서 Django 개발시 공통으로 사용할법한 모듈들과 개발환경 셋팅을 마친 이미지가 필요하다
생각해 가장먼저 위와 같이 django_base 이미지 파일을 만들었다.


#### 2. 개발에 필요한 이미지 만들기
```bash
FROM        django_base
MAINTAINER  lunacircle4@gmail.com

COPY    ./requirements.txt /app/web/requirements.txt
RUN     pyenv virtualenv 3.6.4 floweryroad
RUN     /root/.pyenv/versions/floweryroad/bin/pip install --upgrade pip
RUN     /root/.pyenv/versions/floweryroad/bin/pip install -r /app/web/requirements.txt 
```

위에서 만든 django_base 이미지를 바탕으로 개발에 필요한 이미지인<br/>lunacircle4/floweryroad-backend:develop 를 만든다.
django_base 이미지에 개발에 필요한 가상환경및 파이썬 모듈들을 설치하여 개발을 위한 이미지가 생성된다.
이미지 이름앞에 lunacircle4/를 붙여서 개발이 끝난후 docker hub에 push 할수 있게 하였다.
개발중에 새로운모듈을 설치하여 변경사항이 생기면 아래의 명령어를 통해 이미지를 커밋할수 있다!!
```bash
docker commit CONTAINER_NAME lunacircle4/floweryroad-backend:develop
```

#### 3. docker-compose로 개발 진행하기
```bash
version: '3'

services:
  db:
    image: postgres
    env_file: 
      - /Users/kimdongwon/Documents/WebProgramming/Projects/Floweryroad/floweryroad-backend/.credential/dev/db_credential.env
    volumes:
      - ../../db_volume:/var/lib/postgresql/data

  web:
    image: lunacircle4/floweryroad-backend:develop
    env_file: 
      - /Users/kimdongwon/Documents/WebProgramming/Projects/Floweryroad/floweryroad-backend/.credential/dev/credential.env
    volumes:
      - ../../:/app/
    ports:
      - "8000:8000"
    depends_on: 
      - db
    command: 
      - /bin/zsh
      - -c
      - |
        source /root/.pyenv/versions/floweryroad/bin/activate
        python /app/web/manage.py migrate
        python /app/web/manage.py runserver 0.0.0.0:8000
```

docker-compose를 활용하면 docker 명령어를 파일속에 모아놓고 편하게 실행할수 있다. <br/>
env_file 옵션으로 노출이 되면 안되는 값들을 파일에 모아서 .gitignore에 추가한다. <br/>
volumes 옵션으로 비즈니스로직을 복사하지 않고 mount할수 있다!! db도 마찬가지로 내용을 <br/>
마운트 해서 이미지나 컨테이너가 없어져도 db데이터는 남아있게 할수 있다. <br/>

#### 4. 배포 이미지 만들기
```bash
FROM        floweryroad-backend:develop
MAINTAINER  lunacircle4@gmail.com

COPY ./ /app/web
RUN         cp -f /app/web/.server/supervisor_app.conf /etc/supervisor/conf.d/
```

위에서 만든 개발용 이미지를 바탕으로 개발에 필요한 이미지인 배포용 lunacircle4/floweryroad-backend 를 만든다.
배포시에는 volume을 쓰지 않고 직접 비즈니스 로직을 복사해서 사용하며 supervisor setting도 추가해주었다.


#### 5. docker-compose로 배포 진행하기
```bash
version: '3'

services:
  nginx:
    image: lunacircle4/nginx:floweryroad-backend # nginx 서비스에서 사용할 도커 이미지
    ports:
      - "80:80"
    volumes:
      - /app/static:/app/static  
      - /app/media:/app/media
    
  db:
    env_file: 
      - /app/.credential/db_credential.env
    image: postgres
    volumes:
      - /app/db_volume:/var/lib/postgresql/data
  
  web:
    env_file: 
      - /app/.credential/credential.env
    image: lunacircle4/floweryroad-backend

    volumes:
      - /app/static:/app/static  
      - /app/media:/app/media

    command: 
      - /bin/bash
      - -c
      - |
        supervisord
```

개발과 달리 nginx 컨테이너가 추가된다. nginx을 통해 static과 media 파일에 대한 접근속도를 높이고
로드밸런싱, 무중단 배포등의 기능을 수행할수 있다! static과 media에 대한 volume옵션을 web 컨테이너와
nginx 컨테이너에 적용하여 static과 media의 접근을 가능케 하였다.
