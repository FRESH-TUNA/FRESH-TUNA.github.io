---
layout: post
title: "서버(computing)"
author: "DONGWON KIM"
meta: "Springfield"
categories: "infra"
comments: true
---

## 0. 서버
사용자로부터 요청을 받아 처리된 결과를 반환하는 장치를 말한다.
서버는 프로세서, 메모리, 스토리지, 운영체제등의 요소로 구성되며 요구되는 사양 따라 달라진다. 고가용성, 많은 요청을 처리하기 위해 여러대의 인스턴스를 여러개의 AZ에 배치하기도 하며 사용자가 서버의 사양을 결정하는 불편함을 덜어주는 패러다임인 서버리스 제품을 사용하기도 한다.

## 1. auto scaling group
min, max, desired capacity 지정을 통해 유동적으로 인스턴스를 늘렸다 줄였다 할수 있는 기능이다.
lifecycle hook 을 통해 인스턴스가 생성되고 종료될때 시행해야할 작업을 지정할수 있다.
scheduled action을 통해 정해진 시각에 인스턴스 생성, 종료를 명령할수 있다. (AWS) 오토스케일링 그룹의 launch configuration은 한번 생성하면 수정할수 없다. 새로운 configuration을 생성하여 대체한다.
launch template 을 통해 온디맨드와 스팟 인스턴스를 섞어쓰는것이 가능하다

## 2.placement group
### cluster
하나의 AZ의 서버렉에 모든 인스턴스를 위치시키는 그룹
### partition
여러 서버렉에 인스턴스를 분산시키는 그룹 (AWS) AZ당 최대 7개의 파티션(서버렉) 허용
### spread
모두 다른 서버렉에 인스턴스를 위치시키는 그룹, (AWS) AZ당 최대 7개의 인스턴스

## 1-2. auto scaling policy
### simple scailing policy
특정 threshold에 도달하면 instance 생성/삭제를 시행한다.

### step scailing policy
simple scailing policy 와 유사하지만 확장 활동이나 상태 확인 교체가 진행되는 동안에도 추가 경보에 계속 대응할 수 있는 장점이 있다.
### target scailing policy
target 값을 지정하여 자동으로 instance를 생성/삭제 한다.

## 2. load balencer
### Application Load Balancer
### Classic Load Balancer
### Network Load Balencer
### cross-zone-load-balencing
여러 가용존에 걸쳐서 로드밸런싱을 제공하는 기능으로 Application Load Balancer 에서 기본적으로 제공하며 Network Load Balencer 는 추가적인 설정을 통해 활성화 할수 있다. Classic Load Balancer 의 경우 API를 통해 생성하면 비활성화, 콘솔을 통해 생성하면 활성화된다.

## 3. Fargate와 EC2의 차이점
Fargate의 경우 vCPU와 memory 사양에 따라 과금되고 ec2 인스턴스의 경우 유형과 사용되는 볼륨의 종류에 따라 과금된다.

## 4. AMI
인스턴스를 생성할때 사용하는 이미지이다. AMI 는 스냅샷 기반으로 생성이 되며 다른 계정과 공유가 가능하며 소유권은 변동되지 않는다.
AMI 를 복사하기 위해선 소유자로부터 허가를 받아야 한다. 암호화된 AMI는 바로 복사가 불가능하고 스냅샷을 복사하여 키를 이용해 다시 암호화한후 새로운 AMI와 키를 만들어 공유해야 한다.

## 5. placement group
### Cluster Placement group
하나의 물리적 랙에 ec2를 배치시키는 논리적 그룹, 짧은 지연시간을 필요로 하는 서비스에 권장된다.

### Partition Placement group
여러 파티션(실제 랙)으로 나누어 배치가 되어 내가용성을 높이는 논리적 그룹

### Spread Placement group
ec2 인스턴스를 모든 랙으로 나누어 배치하여 내가용성 상당히 높이는 기법, 여러 가용존으로 확장할수 있고 한 가용존 당 7개의 인스턴스를 생성할수 있다.

## 6. Elastic Fabric Adapter
ec2 인스턴스간의 고성능의 통신 성능이 필요할깨 사용한다. 주로 HPC가 필요한 환경에서 cluster placement group으로 많이 쓰인다.


## 6. instance type
### ondemend
### reserved
### spot
### dedicated host
### dedicated instance
인스턴스를 시작할때 유형을 지정할수 있는데 (dedicated, host, default) 일단시작하면 default와 dedicated, host로 쌍방향 변경은 불가능하다.
