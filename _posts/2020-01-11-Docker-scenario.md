---
layout: post
title: "Docker를 활용한 개발환경 구축하기"
author: "DONGWON KIM"
meta: "Springfield"
categories: "Infra"
---

## 1. D O C K E R
![Image Alt 텍스트](/img/2019/08/27/Docker-scenario/New_Docker_logo_Logo.jpg)
도커는 사용자가 원하는 응용 프로그램들을 컨테이너(프로세스) 안에 격리시켜 활용할수 있게 해주는 오픈소스 프로젝트이다.<br/>
만약 여러분이 파이썬 환경에서 개발을 한다고 가정해보자. 기존 방식이라면 로컬환경에 루비를 깔고 개발을 시작했겠지만, Docker를 활용하게 되면
파이썬 개발환경을 가지고 있는 이미지로 부터 컨테이너를 생성하고 컨테이너의 환경만을 이용하여 개발을 진행하게 되는것이다.<br/>


## 2. 왜 도커를 쓰는가?
기존의 가상머신 (vmware)은 OS자체를 가상화하는 방식을 사용하고 있었다. 기존 방식은 간단하지만 무겁고 느려서 운영환경에서 사용하기
에는 무리가 있다. 반면 Docker는 프로세스(컨테이너)의 형태로 격리시키는 방법을 선택했고, 여러 컨테이너들은 Docker가 제공하는 Layer를 통해
하나의 OS위에서 동작할수 있게 만들어주었다. 이를 통해 Docker는 호스트 기준 95% 이상의 성능을 낼수 있는 장점을 가지고 있다.<br/>

도커는 특정서비스를 여러개 돌릴수 있는 장점을 가지고 있다. 기존의 로컬환경에서는 nginx를 하나만 깔수 있지만, Docker를 활용하면 
nginx기능을 제공하는 컨테이너를 n개 생성할수 있다. 이러한 점을 이용하여 로드밸런서를 구현한다던지, 무중단 배포를 해본다던지 재미있는일을
많이 시도할수 있다. <br/>

여러분이 자바언어를 공부하고 과제를 한후 jdk를 이용하여 binary 파일을 뽑아냈다고 가정해보자. 이 바이너리 파일은 JVM이 설치되어있는 어떤 환경에서도 똑같은 동작을 보장해준다. 마찬가지로 도커를 활용하여 빌드된 이미지가 있다고 가정했을때, 이미지를 이용하여 만들어진 컨테이너는 도커가 설치된 어떤 환경에서 똑같은 서비스의 제공을 보장해준다. <br/>

예를 들어 파이썬 개발을 위해서 이미지를 만들어 놓았는데, 나중에 파이썬 개발이 필요한 시점이 왔을때 그 이미지를 그대로 활용하여 컨테이너를 생성하여 개발하면 되는것이다. 즉 환경 구축에 팔요한 시간을 줄일수 있어 DEVOPS 의 지향점과 일치하는것이다.<br/>

## 3. 도커를 활용하는 과정
#### 1. Dockerfile을 작성한다. Dockerfile에는 사용자가 필요한 환경을 구축하기 위한 명령어들을 기술한다.
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
```

#### 2. 작성된 Dockerfile을 이용하여 원하는 환경이 포함된 이미지를 빌드한다.
```bash
docker build -t my_image .
```

#### 3. 빌드된 이미지를 이용하여 컨테이너를 생성하고, 생성된 컨테이너의 환경을 이용하여 원하는 작업을 진행한다.
```bash
docker run --rm -it my_image /bin/bash
```

## 4. 도커로 개발환경 구축하기
나는 도커를 활용하여 만들어진 컨테이너의 환경만을 이용하여 루비 개발을 진행하겠다는 야심찬 계획을 세워보았다.<br/>
가정먼저 아래의 자료를 참고하여 docker와 docker-compose를 설치해주었다.<br/>
참고자료: https://myjamong.tistory.com/105

### 1. 파이썬 Base 이미지 만들기 
```bash
FROM python:3.8.1-slim-buster

RUN apt update \
    && apt install -y gcc wget \            
    && rm -rf /var/lib/apt/lists/*
```

```bash
docker build -t lunacircle4/python:3.8.1 .
```
FROM 명령어를 이용하여 base가 되는 이미지를 설정할수 있다.
Dockerfile을 작성할때 RUN 명령어를 활용하여 컨테이너 환경에 필요한 요소들을 구축할수 있다.<br/>

이때 주의할점은 RUN 명령어를 실행시 임시파일이 생길 가능성이 있으면 && 를 활용하여 바로 지워줘야한다.
지워지지 않으면 그대로 layer에 남게되어 빌드후 이미지의 용량이 커지게 된다.<br/>

파이썬 기반의 환경을 직접 구축하는것도 좋지만 도커 허브가 제공하는 공식 이미지를 기반으로 하여 Dockerfile을 작성하면 빠르고 안정적으로 인프라를 구성할수 있다. 나는 python:3.8.1-slim-buster 라는 데비안 기반의 공식 이미지를 기반으로 원하는 이미지를 만들었다.
<br/>

### 2. Django 개발환경 이미지 만들기 
```bash
FROM lunacircle4/python:3.8.1 as dev
WORKDIR /web
EXPOSE 8000
COPY requirements.txt ./
RUN apt update \
    && apt install -y postgresql postgresql-contrib libpq-dev \         
    && rm -rf /var/lib/apt/lists/*
    
RUN pip3 --no-cache-dir install -r requirements.txt
ENTRYPOINT ["sh", "./docker/dev/entrypoint.sh"]
```
위에서 만들었던 파이썬 환경 이미지인 lunacircle4/python:3.8.1를 기반으로 하여 프로젝트 개발을 위한 Django 환경 이미지를 만들었다. requirements.txt에 django 프레임워크를 포함하여 개발에 필요한 라이브러리에 대한 정보가 담겨있고, pip를 통해 설치할수 있다.

이미지로부터 컨테이너가 생성되는순간 실행되는 명령어를 ENTRYPOINT를 통해 정의할수 있다. 아래의 entrypoint.sh를 살펴보자

```bash
#entrypoint.sh
#!/bin/sh

EXPOSE_PORT=8000

until pg_isready -h db; do
  >&2 echo "postgres is unavailable - sleeping"
  sleep 3
done

python3 /web/manage.py runserver 0.0.0.0:${EXPOSE_PORT}

```
먼저 pg_isready 명령어를 통해 개발환경에서 사용할 데이터베이스가 준비되었는지 check할수 있다. 데이터베이스가 준비되면 루프를 빠져나와 개발환경을 구동하게 된다. 

### 4. 이미지로 부터 컨테이너를 만드는 작업을 간편하게 해보자
다음 명령어를 입력하면 컨테이너를 실행하여 개발을 진행할수 있다.
```bash
docker create network -d bridge knufestival2019

docker run \
--env-file $(pwd)/db/.env \
-v $(pwd)/db/volume:/var/lib/postgresql/data \
--network knufestival2019 \
--network-alias db\
postgres:12.2-alpine

docker run --rm -it \
--env-file $(pwd)/web/.env \
-p 8000:8000 -v $(pwd):/web \
--network knufestival2019 \
--network-alias web\
my_project_image
```
하지만 매번 입력하면 손이 아프다는 단점을 가지고 있다. 하지만 도커가 제공하는 compose tool을 활용하면 파일로 한번 만들어서 계속 쉽게 활용할수 있다.'start.yml' 이라는 이름으로 프로젝트 루트폴더에 다음과 같이 작성해보았다.

```bash
version: "3.7"

services:
  db:
    env_file: 
      - env/develop/db.env
    image: postgres:12.2-alpine
    container_name: knufestival2019-db
    volumes:
      - ./db/volume:/var/lib/postgresql/data

  web:
    env_file:
      - env/develop/web.env
    container_name: knufestival2019-web
    build:
      context: ./web
      dockerfile: ./docker/dev/Dockerfile
    volumes:
      - "./web:/web"
    ports:
      - "8000:8000"

```
이제 다음 명령어를 치면 개발을 쉽게 진행할수 있게 되었다.

```bash
docker-compose -f start.yml up
```

### 5. 처음 개발환경 구축시 마이그레이션 진행하기
개발환경을 처음 시작하면 데이터베이스 마이그레이션이 하나도 되있지 않기 때문에 당연히 정상적으로 실행되지 않는다. 따라서 먼저 웹 컨테이너의 entrypoint를 바꿔서 마이그레이션을 먼저 진행해주는 작업이 필요하다. 아래는 그 역활을 위한 임시 entrypoint 파일이다.

```bash
#!/bin/sh
# temperory entrypoint
until pg_isready -h db; do
  >&2 echo "postgres is unavailable - sleeping"
  sleep 3
done

python3 /web/manage.py makemigrations
python3 /web/manage.py migrate
```

위의 entrypoint 파일을 사용하여 처음 개발환경 시작시 마이그레이션을 진행하는 쉘 스크립트를 만들고 이를 실행해주면 된다.

```bash
#!/bin/sh
# development image build
echo "1. development image build"
docker-compose build

# migrate
echo "2. db migrate"
docker-compose up -d db
docker-compose run --entrypoint="sh ./docker/init.sh" web

# finish
docker-compose down
echo "development environment finished!"
```

### 6. shell_script로 주요 명령어 딘순화하기
```sh
#!/bin/sh
function dev_start_func {
  docker-compose -f start.yml up
}

function dev_stop_func {
  docker-compose -f start.yml down
}

alias dev_start=dev_start_func
alias dev_stop=dev_stop_func
```
이렇게 쉘 스크립트를 만들어서 source 명령어로 로딩해주면 긴 명령어를 하나의 단어로 축약해서 사용할수 있다. 하지만 매번 shell을 시작할때마다 로딩해줘야 해서 direnv라는 신박한 물건을 도입하게 되었다.

### 7. direnv
direnv를 활용하면 프로젝트폴더에 진입했을때 자동으로 .envrc를 찾아서 스크립트를 로딩해주므로 편하게 개발할수 있다.<br/>
만약 프로젝트 폴더를 빠져나오면 다시 스크립트를 언로딩해주는 편리함을 가지고 있었다. 아래와 같이 direnv를 설치해주고
나만의 명령어 모음집을 완성할수 있었다.

```bash
brew install direnv

# ~/.zshrc에 추가해줍니다
eval "$(direnv hook zsh)"
```

```bash
# .envrc 파일을 만들어서 프로젝트 폴더의 루트 에 넣어줍니다
function export_alias() {
  local name=$1
  shift
  local alias_dir=$PWD/.direnv/aliases
  local target="$alias_dir/$name"
  mkdir -p "$alias_dir"
  PATH_add "$alias_dir"
  echo "#!/usr/bin/env bash -e" > "$target"
  echo "$@" >> "$target"
  chmod +x "$target"
}

export_alias dev_start "docker-compose -f start.yml up \$@"

export_alias dev_stop "docker-compose -f start.yml down \$@"

export_alias docker_shell "docker exec -it my_project_image_web_1 /bin/bash \$@"

export_alias image_update "docker commit my_project_image_web_1 example \$@" 
```

.envrc를 만들고 direnv allow를 해주면, 앞으로 프로젝트 폴더에 진입시 자동으로 스크립트가 로딩되어 미리 정의해둔 단어로 명령을 실행할수 있다.

### 8. (부록) docker-sync로 개발환경 속도 높이기
이제 신나게 개발만 하는일이 남았는데 윈도우와 맥환경에서 volume 사용시 60배정도의 성능저하 현상이 있어서 이를 해결하기 위한
방법을 찾아야만 했다.<br/>
여러 자료를 조사해본끝에 docker-sync라는 서드파티앱을 활용하기로 결정했다.<br/>
docker-sync는 rubygem의 일종이었고, 결국 로컬환경에 루비가 필요하다는 슬픈사실을 알았다. 사실 이런문제때문에 docker-sync 도입을
망설였으나 속도저하현상을 막기위해 한번 도입해보았다.

```bash
# 이 명령어로 docker-sync를 설치한다.
gem install docker-sync
```

docker-sync를 설치한후 'docker-sync.yml' 이라는 파일을 생성해주고 다음과 같이 작성하였다.
```bash
version: "2"
syncs:
  codes:
    notify_terminal: true
    src: './web'
    sync_excludes: ['.git', '.idea']
```
syncs 아래에 'codes' 라는 이름을 적어주어서 아래 start.yml에서 'codes'라는 이름으로 볼륨을 사용할수 있게 해주었다.
그리고 start.yml 파일을 아래와 같이 수정해주었다.

```bash
version: "3.7"

services:
  db:
    env_file: 
      - env/develop/db.env
    image: postgres:12.2-alpine
    container_name: knufestival2019-db
    volumes:
      - ./db/volume:/var/lib/postgresql/data

  web:
    env_file:
      - env/develop/web.env
    container_name: knufestival2019-web
    build:
      context: ./web
      dockerfile: ./docker/dev/Dockerfile
    volumes:
      - codes:/web
    ports:
      - "8000:8000"

volumes:
  codes:
    external: true
```

이제 아래 명령어를 입력하고 개발을 시작하면 빠른 퍼포먼스를 경험할수 있다.
```bash
docker-sync start
```

### 9. 으아아악
도커로 개발환경을 구축하면서 Infrastructure as a code의 장점을 조금이나마 느낄수 있었던것 같다.<br/>
하지만 도커를 개발을 시작하면서 관리해야할 파일들이 늘어났고, 여러개의 컨테이너로 돌릴때 docker-compose 만으로 커버하기 힘든 상황이 나중에
발생할것 같아 수많은 고민이 들었다.<br/>
이러한 문제들 때문에 '컨테이너 오케스트레이션' 이라는 개념이 나온것 같고 쿠버네티스를 공부해야겠다는 생각을 하며 글을 마쳐보려 한다.

### 10. 내가 참고한 자료
1. https://subicura.com/2017/01/19/docker-guide-for-beginners-1.html<br/>
2. https://medium.com/myrealtrip-product/docker-%EB%A1%9C-%EC%BE%8C%EC%A0%81%ED%95%9C-%EA%B0%9C%EB%B0%9C%ED%99%98%EA%B2%BD-%EA%B5%AC%EC%B6%95%ED%95%98%EA%B8%B0-e484b80947a3<br/>
3. https://myjamong.tistory.com/105<br/>
4. https://dreisbach.us/articles/a-favorite-development-tool-direnv/<br/>