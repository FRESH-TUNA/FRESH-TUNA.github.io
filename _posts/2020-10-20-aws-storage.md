---
layout: post
title: "AWS/스토리지"
author: "DONGWON KIM"
meta: "Springfield"
categories: "infra"
comments: true
---

## 1. EBS
### Instance Store
물리적으로 연결된 ssd 저장장치로 캐시, 버퍼, 임시 데이터를 저장할때 사용하며 저렴한 가격에 비해 강한 성능과 폴트에 강한 특성을 지닌다

## General Purpose SSD (gp2)
일반적인 목적으로 사용되는 스토리지, root 볼륨 가능

## Provisioned IOPS SSD (io1)
고성능을 요구하는 목적으로 사용되는 스토리지, root 볼륨 가능

## Throughput Optimized HDD (st1)
규칙적인 사용되는 데이터를 저장하는 스토리지

## Cold HDD (sc1)
잘 안사용하는 데이터를 저장하는 스토리지
