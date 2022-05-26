---
layout: post
title: "Django 라이프사이클 요약"
# author: "DONGWON KIM"
# meta: "Springfield"
categories: "Django"
comments: true
---

## 1. wsgi.py
웹서버와 비즈니스 로직 사이를 연결해주는 인터페이스

## 2. middleware
http request를 처리하기전에 거치는 관문, settings에 설정된 middleware들을 순차적으로 통과한다.

## 3. urls.py
request와 matching 되는 callable object 를 반환하여 실행한다. callable object는 함수형 뷰일수도 있고 class based view의 함수 일수도 있다.

## 3. views.py
만약 함수형 뷰가 아니라 클래스 뷰라면 dispatch 메소드를 통해 최종 request를 처리할 함수가 선택된다.
이후 함수를 실행하면서 models.py 에서 필요한 데이터를 읽어온다. 적절한 template renderer를 선택되어 response가 생성된다.

## 4. middleware
response는 다시 미들웨어들을 통과한다. 이번에는 순차적으로가 아니라 역방향으로 미들웨어 리스트의 마지막 원소부터 거치게 된다.

## 5. wsgi.py
인터페이스를 통해 response가 웹서버로 전달된다.