---
layout: post
title: "Dockerfile 작성 및 이미지 빌드 가이드"
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

## 2. Dockerfile 작성법
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


