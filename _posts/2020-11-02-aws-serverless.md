---
layout: post
title: "AWS/Serverless"
author: "DONGWON KIM"
meta: "Springfield"
categories: "infra"
comments: true
---

## Lambda
서버를 프로비저닝하거나 관리하지 않고도 코드를 실행할 수 있게 해주는 컴퓨팅 서비스, 다만 실행시간이 30분으로 제한이 되어있어서, 오래걸리는 작업은 스팟 인스턴스를 사용하는것이 좋다. 동시 호출 횟수가 1000회로 기본적으로 제한되어있어 더많은 횟수가 필요하면 aws에 문의하여 증가시킨다.

## SQS
FIFO Queue 는 초당 300개의 메시지를 처리할수 있는 성능을 가지고 있다. 따라서 최대 10개 까지 가능한 batch를 이용해서 3000개의 메시지를 처리할수 있다.

## API Gateway
Stateless 한 클라이언트-서버 통신을 제공하는 Restful API를 생성하거나 Stateful 한 통신이 가능하게 하는 WebSocket API를 생성할수 있다.
