---
layout: post
title: "도커 명령어 정리"
# author: "DONGWON KIM"
# meta: "Springfield"
categories: "Infra"
---

## 1. network를 이용한 이미지 실행
```bash
docker network create floweryroad

docker run --rm -d \
    --name db \
    --network floweryroad \
    -e POSTGRES_DB=floweryroad \
    -e POSTGRES_USER=floweryroad \
    -e POSTGRES_PASSWORD=floweryroad \
    postgres
```

db컨테이너를 작동시키면서 db이름, 유저, 패스워드를 동시에 설정할수 있다.

## 2. 참고 자료
https://www.44bits.io/ko/post/almost-perfect-development-environment-with-docker-and-docker-compose
(정말 감사합니다 호호)