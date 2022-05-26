---
layout: post
title: "쿠버네티스 시크릿 업데이트 하기"
# author: "DONGWON KIM"
# meta: "Springfield"
categories: "Infra"
comments: true
---

## 1 인증서 제발급후 반영 예제
```sh
sudo kubectl create secret tls freshtuna-me --key="/etc/letsencrypt/live/freshtuna.me/privkey.pem" --cert="/etc/letsencrypt/live/freshtuna.me/cert.pem" --dry-run=client -o yaml | sudo kubectl apply -f -
```
