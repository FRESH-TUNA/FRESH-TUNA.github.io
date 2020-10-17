---
layout: post
title: "AWS/데이터베이스"
author: "DONGWON KIM"
meta: "Springfield"
categories: "infra"
comments: true
---

## 1. RDS (relational database service)
매니지드 관계형 데이터베이스 서비스로, PostgreSQL, MySQL, Oracle, SQL Server 등을 지원한다. 인스턴스 유형과 볼륨 유형, 사이즈를 반드시 지정해야 하며, Read Replica와 Multi AZ 를 지원한다.
IAM, 보안 그룹, KMS, SSL 을 이용하여 통신의 보안을 유지하며 백업, 스냅샷,포인트를 사용하여 복구 할수 있고 CloudWatch를 이용하여 모니터링 한다.
성능과 가격이 미리 프로비전된 인스턴스, 볼륨 유형에 의해 결정되는 관계형 데이터베이스인것이 특징이다.

## 2. Aurora (relational database service)
PostgreSQL / MySQL API 호출을 지원하는 아마존이 자체개발한 데이터베이스 서비스가 구동되는 RDS 서비스이다. 데이터는 6개의 리플리카셋(글로벌 가능ㄴ, 최대 15 리플리케이션 가능), 3개의 가용존에서 기본으로 다루어지며 오토힐링을 통한 자동복구 기능까지 갖추고 있다. 스토리지는 오토스케일되며, 인스턴스 유형먼 정해주면 된다. 
하지만 기본 RDS 서비스보다 다소 비싸다는 단점이 있다.

## 3. ElastiCache (Redis / Memecashed)
매니지드 key-value 타입 (Redis / Memecashed) NOSQL 서비스이다. 인메모리 데이터베이스로 레이턴시가 우수하며 인스턴스 유형을 지정받는다. Redis 타입의 경우 클러스터팅을 지원하고 Multi AZ, 리드 리플리카를 지원한다. RDS와 같은 복구기능을 지원한다. 성능과 가격이 미리 프로비전된 인스턴스 유형에 따라 결정된다.

## 4. DynamoDB (Serverless database service)
매니지드 서버리스 NoSQL 데이터베이스이다. 사용자가 서버 유형을 정하지 않는 서버리스 서비스이다. (알아서 오토스케일링 되고 사양이 결정된다. Multi AZ 도 기본값으로 적용된다.) ElastiCache 대용으로 사용할수 있고 Lambda와 함께 쓰일수 있다. 쿼리를 할때 오직 기본키와 정렬키, 인덱스로만 쓸수 있다.