---
layout: post
title: "AWS/EC2"
author: "DONGWON KIM"
meta: "Springfield"
categories: "infra"
comments: true
---

## Cluster Placement group
하나의 물리적 랙에 ec2를 배치시키는 논리적 그룹, 짧은 지연시간을 필요로 하는 서비스에 권장된다.

## Partition Placement group
여러 파티션(실제 랙)으로 나누어 배치가 되어 내가용성을 높아는 논리적 그룹

## Spread Placement group
ec2 인스턴스를 모든 랙으로 나누어 배치하여 내가용성 상당히 높이는 기법, 여러 가용존으로 확장할수 있고 한 가용존 당 7개의 인스턴스를 생성할수 있다.

## auto scaling group
auto scaling group에서 인스턴스가 생성되거나 삭제될때 hook을 걸어 특정 작업을 실행할수 있다.
scheduled action을 통해 특정시간대 인스턴스를 생성하거나 줄일수 있다.