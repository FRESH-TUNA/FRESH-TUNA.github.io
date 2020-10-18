---
layout: post
title: "AWS/마이그레이션"
author: "DONGWON KIM"
meta: "Springfield"
categories: "infra"
comments: true
---

## 1. SnowBall
엣지 컴퓨팅 및 대용량 데이터 마이그레이션 서비스 이다. 기존의 네트워크 전송 방식과 비교했을때 대용량의 데이터를 빠른속도로 옮기거나 높은 성능을 요구하는 작업을 빠르게 처리할수 있는것이 장점이다. Snowball Edge Compute Optimized의 고성능 서비스, Snowball Edge Storage Optimized의 마이그레이션 서비스를 제공하며, 작업 결과물을 S3로 저잗한다. (Glacier로 보내기 위해선 라이프사이클 정책을 사용해야 한다.)