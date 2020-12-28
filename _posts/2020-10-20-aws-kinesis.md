---
layout: post
title: "AWS/Kinesis"
author: "DONGWON KIM"
meta: "Springfield"
categories: "infra"
comments: true
---

## 1. Kinesis
완전관리형 스트리밍 데이터 서비스 이다. 데이터를 수집하고 이를 다양한 포맷으로 변환하여 통계를 내거나 다른 소프트웨어가 사용할수 있도록 다른 저장소에 전달하는 기능을 제공한다.

### Kinesis Stream
디바이스에서 데이터 스트림을 안전하게 실시간으로 전달하는 서비스, 하나의 스트림은 여러개의 샤드로 구성되어 높은 속도로 전송된다. (write: 1MB per shard, read: 2MB per shard)

### Kinesis Firehose
데이터 스트림을 다른 저장소(S3, Redshift, ElasticSearch) 로 전달하는 서비스, 최대 1분 지연시간의 실시간 전송을 지원한다.

### Kinesis Data Analytics
Kinesis Stream 으로부터 온 데이터를 SQL을 통해 실시간으로 분석해주는 서비스
