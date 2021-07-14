---
layout: post
title: "우분투 환경에서 Docker 설치하기"
author: "DONGWON KIM"
meta: "Springfield"
categories: "Infra"
---

# 1. docker 설치하기

Ubuntu 18.04 기준으로 작성되어있으며 다음 명령어들 타이핑하여 우분투를 docker를 설치할수 있다.

    sudo apt update
    sudo apt install apt-transport-https ca-certificates curl software-properties-common
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
    sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable"
    sudo apt update
    apt-cache policy docker-ce
    sudo apt install docker-ce
    sudo systemctl status docker

도커를 sudo 권한 없이 실행하기 위해서 아래와 같이 명령어를 사용하면 된다.

    #현재 로그인한 user에게 권한을 줄때 사용한다.
    sudo usermod -aG docker ${USER}
    
    #다른 user에게 권한을 줄때 사용한다
    sudo usermod -aG docker username
    
    #로그아웃후 다시 로그인하여 아래명령어를 적용해야 한다.
    su - ${USER}

다음 명령어를 쳐서 권한이 부여되어 있는지 확인한다.

    id -nG