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
Read Replica 시 비동기복제, Multi AZ 시 동기 복제가 이루어진다.

## 2. Aurora (relational database service)
PostgreSQL / MySQL API 호출을 지원하는 아마존이 자체개발한 데이터베이스 서비스가 구동되는 RDS 서비스이다. 데이터는 6개의 리플리카셋(글로벌 가능ㄴ, 최대 15 리플리케이션 가능), 3개의 가용존에서 기본으로 다루어지며 오토힐링을 통한 자동복구 기능까지 갖추고 있다. 스토리지는 오토스케일되며, 인스턴스 유형먼 정해주면 된다. 
하지만 기본 RDS 서비스보다 다소 비싸다는 단점이 있다.

## 3. ElastiCache (Redis / Memecashed)
매니지드 key-value 타입 (Redis / Memecashed) NOSQL 서비스이다. 인메모리 데이터베이스로 레이턴시가 우수하며 인스턴스 유형을 지정받는다. Redis 타입의 경우 클러스터팅을 지원하고 Multi AZ, 리드 리플리카를 지원한다. RDS와 같은 복구기능을 지원한다. 성능과 가격이 미리 프로비전된 인스턴스 유형에 따라 결정된다.

## 4. DynamoDB (Serverless database service)
매니지드 서버리스 NoSQL 데이터베이스이다. 사용자가 서버 유형을 정하지 않는 서버리스 서비스이다. (알아서 오토스케일링 되고 사양이 결정된다. Multi AZ 도 기본값으로 적용된다.) ElastiCache 대용으로 사용할수 있고 Lambda와 함께 쓰일수 있다. 쿼리를 할때 오직 기본키와 정렬키, 인덱스로만 쓸수 있다.

## 5. S3 (Object storage)
서버리스 오브젝트 스토리지 서비스이다. 최대 객체사이즈는 5TB까지 지원하며 무한히 scale 되는것이 장점이다. 주로 정적 파일 서빙이나 용량이 큰 파일을 저장할때 많이 사용되며 SPA 배포에도 주로 사용된다.
한 prefix 당 5500 read 요청을 감당할수 있으므로 만약 더 많은 요청이 필요하면 고객별로 prefix를 나누는것이 좋은 전략이다.
S3 라이프사이클 설정을 통해 자주사용하는 데이터는 좋은 티어를 쓰고 잘 안쓰는 데이터는 낮은 티어를 쓰는 방식으로 설정할수 있다. 티어는 Standard, Standard_IA, Intelligent_tiering, Onezone_IA, Glacier, Deep archive 가 있다.
versioning을 통해 여러버전의 동일한 객체를 지원하여 삭제된 데이터를 복구하는것이 가능하지만 , 설정시 약간의 다운타임이 존재한다. MFA를 활성화하여 삭제시 반드시 더 확인하는것이 좋다.
S3를 암호화할때는 KMS, S3, C, client side encryption 방법이 있다. KMS, S3, C는 서버사이드 암호화라는점에서 공통점을 가지고 있지만 Cloudtrail를 사용하고 싶다면 KMS를 사용해야한다.

## 6. Athena
서버리스 데이터베이스 서비스의 일종으로 S3 기록 데이터를 요청해서 저장할수 있는 기능을 제공한다.

## 7. Redshift
PostgreSQL 기반의 데이터 웨어하우싱 서비스로 데이터 분석(Online Analytic Processing)에 특화된것이 장점이다. 다른 데이터 웨어하우스와 비교했을때 10배 빠르며 컬럼 기반이다. 데이터는 S3, DynamoDB 등의 다양한 데이터베이스로부터 읽어들일수 있다. Redshift Spectrum를 이용하여 Redshift 테이블을 생성하지 않고 S3, DynamoDB 등의 다양한 데이터베이스로부터 읽어들여 쿼리를 할수 있는 장점을 가지고 있다.

## 8. Neptune 
Neo4j 같은 완전관리형 그래프 database 서비스이다. 소셜네트워킹 같은 그래프 구조를 쓰는 서비스에 유리하다. 

## 9. ElasticSearch  
DynamoDB 와 다르게 기본키, 인덱스를 제외한 다른 필드를 이용한 탐색이 가능하다. 빅데이터 서비스에 많이 사용되며 주로 키바나나 로그스태시와 함께 많이 운용된다.
