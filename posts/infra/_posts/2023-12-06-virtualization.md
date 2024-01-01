---
layout: post
title: "가상화 개론"
# author: "DONGWON KIM"
# meta: "Springfield"
tags: [infra]
comments: true
---

## Virtualization

가상화는 컴퓨터의 자원을 여러 가상머신으로 나누어서 유휴자원의 이용률을 높이기 위한 전략이다.

왜 유휴자원의 이용률이 떨어지는가? 각각의 어플리케이션마다 소프트웨어 라이센스, 다른 보안전략을 가지고 있다. 이를 위해선 각각의 앱마다 물리서버를 구축해야 하는데, 비용이 증가하고 자원의 이용률이 떨어지게 되는것이다.

전가상화/반가상화 의 성능이 비슷해졌으므로 이제 비용과 사용환경에 따라 가상화방식을 선택하면 된다.

## Hypervisor

가상화 시스템을 운용하기 위한 소프트웨어, VMM이라고 부른다. 가상화시트템과 운영체제간의 시스템콜을 중계한다.

### Hosted Hypervisor (type2)

일반 운영체제에 VMM을 설치하는 방식 (Xen, vmware workstation)

### Bare-metal Hypervisor (type1)

- 가상화전용운영체제 사용하는 방식
- Xen, VMware vSphere(EXSI hypervisor, 윈도우, 리눅스환경에 유리), Hyper-v(윈도우 중심 환경에유리)

## 가상화의 모드

### 전가상화 (FULL virtualization)

- 전체를 가상화하는 기법
- 하이퍼바이저를 구동하면 DOM0(virtual machine manager) 라고 하는 관리용 가상 머신이 실행되며, 모든 가상머신들의 하드웨어 접근이 DOM0을 통해서 이루어진다.
- KVM (2010년 이후 하드웨어 자체 가상화 지원으로 성능 개선)
- 호환성이 좋으나 성능이 떨어지지만, 최근 하드웨어 자체적으로 가상화 기능(intel vt-x/ept, amd-v/rvi)을 제공하여 하이퍼바이저의 부담을 덜어주어서 성능은 비등비등해졌다.

### 반가상화 (Para virtualization)

- IO(disk, network) 를 별도의 특화된 모듈에 위임하여 오버헤드를 줄이다. 하이퍼콜(Hyper Call)이라는 인터페이스를 통해 가상머신과  하이퍼바이저 사이에 직접 요청을 중개할수 있다.
- 모든 명령을 DOM0를 통해 하이퍼바이저에게 요청하는 전가상화에비해 성능이 빠르다.
- 하이퍼바이저에게 Hyper Call 요청을 할 수 있도록 각 OS의 커널을 수정해야하며 오픈소스 OS가 아니면 반가상화를 이용하기가 쉽지 않다. (주로 리눅스가 반가상화를 지원한다)
- Xen (amazon ec2에서 초기에 사용, 지금은 자체개발 하이퍼바이저 nitro 사용)

## 클라우드

가상화등의 소프트웨어 기술을 통해 온디맨드, 매니지드 형태로 인프라를 대여해주는 서비스를 클라우드라고 한다.

클라우드는 Iaas(인프라, 구축한 인프라를 직접 관리), Paas(플랫폼 형태), Saas(앱)의 형태로 서비스를 고객사들에게 제공하며 멀티테넌시(자원을 공동관리), 광대역망 엑세스를 제공하는 특징이 있다.

퍼블릭, 하이브리드, 프라이빗, 커뮤니티(여러조직이 데이터를 공유하여 사용, 국가기간에서 많이 사용) 의 형태로 서비스를 제공한다.

console(웹브라우저, 앱 …), cli(쉘), sdk(프로그래밍언어) 등의 방법으로 클라우드에 엑세스 할수 있다.