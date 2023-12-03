---
layout: post
title: "Dockerfile 작성 가이드 (python, Django 시나리오)"
# author: "DONGWON KIM"
# meta: "Springfield"
categories: "Infra"
comments: true
---

## 1. 프롤로그
컨테이너 가상화 기법을 사용하면 환경의 제약없이 큰 성능 저하 없이 프로젝트를 딜리버리 할수 있습니다.
이 포스팅에서는 Django 기반의 웹 어플리케이션을 동작하기 위한 가벼운 이미지를 만들기 위한 시도들에 대해 작성합니다.
<br><br>

## 2. Base 이미지 선택
base 이미지의 용량은 딜리버리의 속도와 성능과 관련이 있습니다. 불필요한 구성이나 패키지가 이미지에 포함되어 있을 수 있어 이미지 크기를 최소화하고 필요한 패키지만 포함하는 것이 중요합니다. 이를 위해 공식 이미지를 선택하고, 그중에서도 alpine등의 가벼운 리눅스 기반의 이미지를 선택하는것이 좋을수 있습니다.

회사의 사용 사례에 따라 오픈 소스 라이선스나 상용 라이선스로부터 비롯된 제약 사항을 고려해야 합니다. 이미지에 사용된 소프트웨어 또는 서비스의 라이선스를 확인해야 합니다.

이미지의 보안 업데이트 정책과 회사 정책 간의 일치 여부를 확인해야 합니다. 필요에 따라 보안 패치를 적용할 수 있도록 커스텀 이미지를 관리할 수 있는 능력이 중요할 수 있습니다.

이 포스팅에선 base 이미지를 alpine 으로 정했습니다. (가볍고 보안이 좋지만 호환성 이슈가 있습니다)

## 3. 파이썬 환경 구동을 위한 Dockerfile 작성
Dockerfile은 우리가 작업을 하거나 배포할 컨테이너를 만들수 있는 레시피라고 할수 있습니다.
Dockerfile 작성후 빌드를 통해 이미지가 생성되고, 그 이미지를 통해 우리가 원하는 컨테이너를 
마구마구 찍어낼수 있습니다.
Python3.9 기반 어플리케이션을 동작시키기 위한 Dockerfile 입니다.
```bash
# ARG: 환경변수로 지정하여 사용할수 있습니다.
# --build-arg 옵션을 사용하여 빌드 시에 환경 변수를 전달하거나 덮어쓸수 있습니다.
# ex) docker build --build-arg BASE_ALPINE_VERSION=1.11 -t my_image .
# 알파인 리눅스 3.14는 패키지매니저를 통해 3.9버전을 지원합니다.
ARG BASE_ALPINE_VERSION=3.14

# FROM: base로 선택할 이미지, 없으면 remote에서 pull해온다.
FROM alpine:${BASE_ALPINE_VERSION}

ARG PYTHON_VERSION=3

# LABEL: 라벨을 생성하며 docker inspect 커맨드를 통해 label들을 확인 할 수 있다.
LABEL MAINTAINER="tunakim1004@gmail.com"

# RUN: 컨테이너 안에서 명령어를 실행한다. 이미지가 빌드되면서 실행된다.
# 아래 명령어는 python2 버전을 삭제하고 새로 설치된 python3로 유도하는 심볼릭 링크를 생성합니다
# 이를 통해 python 이라 명령해도 python3를 실행시켜 줍니다.
# python 패키지를 실행하는 과정에서 자동으로 실행됩니다.
# ln -sf python3 /usr/bin/python
RUN apk add --update --no-cache python${PYTHON_VERSION}

# CMD: 빌드된 컨테이너가 실행되었을때 실행할 구문을 작성합니다.
CMD ["python3"]
```

## 4. Django 어플리케이션 구동을 위한 Dockerfile 작성
제가 만들었던 Django 어플리케이션을 동작시키기 위한 Dockerfile 을 작성합니다.
위에서 만들었던 Python 3.9 를 기반 이미지로 하여 빌드합니다.
```bash
## launcher 스테이지
# 어플리케이션의 동작에만 집중하고, 라이브러리 빌드, 설치는 library_builder 스테이지에 위임합니다.
# 이를 통해 레이어의 숫자를 줄이고, 컴파일러 환경설치를 피하여 이미지의 용량을 줄입니다.
ARG PROJECT_PYTHON_VERSION=3.9
FROM python:${PROJECT_PYTHON_VERSION}-slim-buster as library_builder
LABEL MAINTAINER="tunakim1004@gmail.com"

# RUN: 컨테이너 안에서 명령어를 실행합니다. 이미지가 빌드되면서 실행됩니다.
# FROM, RUN, COPY, CMD, ENTRYPOINT 명령어는 파일시스템상의 레이어의 기준이 됩니다. 
# RUN 스탭의 횟수를 최대한 줄여 레이어의 숫자를 줄이는것이 이미지의 용량을 낮추는데 도움이 됩니다.
# 이를 위해 명령어를 && 로 chaining 하고, cache를 clean하는것을 권장합니다.
RUN         apt-get update && apt-get install -y make build-essential libssl-dev zlib1g-dev libbz2-dev \
libreadline-dev libsqlite3-dev wget curl llvm libncurses5-dev libncursesw5-dev \
xz-utils tk-dev libpq-dev libjpeg-dev

COPY ./requirements.txt /requirements.txt
RUN pip3 install -r /requirements.txt


## launcher 스테이지
# 어플리케이션의 동작에만 집중하고, 라이브러리 빌드, 설치는 library_builder 스테이지에 위임합니다.
# 이를 통해 레이어의 숫자를 줄이고, 컴파일러 환경설치를 피하여 이미지의 용량을 줄입니다.
FROM lunacircle4/python:${PROJECT_PYTHON_VERSION} as launcher
ARG PROJECT_PYTHON_VERSION
LABEL MAINTAINER="tunakim1004@gmail.com"
WORKDIR     /app

COPY --from=library_builder /usr/local/lib/python${PROJECT_PYTHON_VERSION}/site-packages /usr/lib/python${PROJECT_PYTHON_VERSION}/site-packages
COPY ./ /app
CMD ["python3", "manage.py", "runserver"]
```

## 4. 빌드를 통해 이미지 생성
도커파일 작성이 끝나면 build 명령어로 이미지 빌드를 진행할수 있습니다.
-t 옵션으로 이미지의 이름과 태깅을 지정할수 있고 -f 옵션으로 Dockerfile을 위치를 설정합니다.

알파인 리눅스의 힘으로 python3.9 환경의 경우 55MB, django 환경의 경우 180MB 정도로 용량을 절약할수 있었습니다.
```bash
# python3.9 빌드
docker build -t lunacircle4/python:3.9 <python3.9 dockerfile path>

# django 빌드
docker build -t lunacircle4/myapp:test <Django dockerfile path>
```

