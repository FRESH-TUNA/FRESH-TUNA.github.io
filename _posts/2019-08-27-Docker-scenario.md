---
layout: post
title: "Docker를 활용한 개발 및 배포"
author: "DONGWON KIM"
meta: "Springfield"
categories: "CI/CD"
---

## 1. 프롤로그
![Image Alt 텍스트](/img/2019/08/27/Docker-scenario/New_Docker_logo_Logo.jpg)
도커는 마치 JVM과 같아서, 우분투 이미지를 가지고 있다면 도커가 설치된 환경이라면 어디든지 우분투를 돌릴수 있다
다른 말로 표현하면 개발과 운영환경의 서로 다른 리눅스 상에서도 애플리케이션 수정없이 devops를 실현할수 있다는것이다. <br/>
우리가 일반적으로 써왔던 가상머신(vmware)에 비해서 가볍고 호스트 비교 98퍼센트까지 제 성능을 낼수 있고
컨테이너간은 완전 격리된 구조로 각 애플리케이션이 독립적으로 실행할 수 있다. <br/>
가상머신(VM)들과 달리, 기존 리눅스 자원(디스크, 네트워크 등)을 그대로 활용 할수 있어서 여러 서비스들을 한 서버에 때려 박아 돌리기가 좋은 편이다. <br/>
내가 풀꽃길을 Docker로 돌리면서 마음속으로 정했던 시나리오를 적어보려한다.
<br/><br/>


## 2. 내가 생각해본 Docker를 활용한 시나리오 
### 1. Django 베이스 이미지 만들기 
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


### 2. 개발에 필요한 이미지 만들기 
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
개발중에 

### 3. docker-compose로 개발 진행하기 
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

### 4. 배포 이미지 만들기 
```bash
FROM        floweryroad-backend:develop
MAINTAINER  lunacircle4@gmail.com

COPY ./ /app/web
RUN         cp -f /app/web/.server/supervisor_app.conf /etc/supervisor/conf.d/
```

위에서 만든 개발용 이미지를 바탕으로 개발에 필요한 이미지인 배포용 lunacircle4/floweryroad-backend 를 만든다.
배포시에는 volume을 쓰지 않고 직접 비즈니스 로직을 복사해서 사용하며 supervisor setting도 추가해주었다.


### 5. docker-compose로 배포 진행하기 
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