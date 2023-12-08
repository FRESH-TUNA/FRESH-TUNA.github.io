---
layout: post
title: "적절한 인프라 기술/장비 선택"
# author: "DONGWON KIM"
# meta: "Springfield"
tags: [infra]
comments: true
---
## OS 선정

### 유닉스

- 전용 RISC 칩 사용
- IBM AIX, 솔라리스, HP-UX

### 리눅스

- 레드헷 계열(yum, rpm, dnf, amazon linux), 데비안 계열(apt, 컨테이너, 클라우드 환경에서 많이 사용), 슬랙웨어 계열(suse → SAP 구동을 위해 필요), 맨드레이크 계열

### 윈도서버

- 닷넷 프레임워크, 액티브 디렉토리, MS-SQL등의 MS의존 환경을 사용하고 싶은 경우

### 현재 이슈

- centos 지원이 끝나고, centos stream 등장
- fedora → centos stream → rhel(redhat enterprise linux) 순으로 안정화

### OS의 버전, 커널버전 체크

```jsx
# OS 버전 체크
CentOS Stream release 8
NAME="CentOS Stream"
VERSION="8"
ID="centos"
ID_LIKE="rhel fedora"
VERSION_ID="8"
PLATFORM_ID="platform:el8"
PRETTY_NAME="CentOS Stream 8"
ANSI_COLOR="0;31"
CPE_NAME="cpe:/o:centos:centos:8"
HOME_URL="https://centos.org/"
BUG_REPORT_URL="https://bugzilla.redhat.com/"
REDHAT_SUPPORT_PRODUCT="Red Hat Enterprise Linux 8"
REDHAT_SUPPORT_PRODUCT_VERSION="CentOS Stream"

# 커널버전 체크
uname -a
```

## 네트워크

### 라우터

- 라우터는 네트워크와 네트워크를 이어주는 L3 장비다.
- 라우팅을 통해 최적화된 목적지를 정하여 라우팅 테이블을 만들고, 라우팅테이블에 의해 입력포트에서 출력포트로 포워딩한다.
- 라우팅테이블이 만들어지는 방식은 수동(static), 다이내믹 방식이 있다.
- 상위 회선(ISP, 데이터센터)과 일치하는 인터페이스를 가질것
- WAN 통신 대역
- 쓰루풋: 단위시간당 데이터전송량
- 보안 기능 적용 여부, 네트워크 장비의 기능이 적어질수록 ASIC(커스텀 cpu) 제작 난이도가 떨어져 성능개선 및 가격적 이점이 있다.

### 스위치

- L2 스위치, L3 스위치 (라우팅 기능이 제공되는 l2 스위치, VLAN을 통해 가상으로 망을 분리하여 쓸수 있다)

### 패브릭구조

- 이중화, 삼중화하기위해 트렁크포트를 통해 서로 연결하는 구조

### 장비별 failover 구성 용어

- 본딩: 리눅스에서 failover구성
- 티밍: nic단에서의 failover구성 (active standby)
- link aggregation: active active로 구성하여 대역폭도 늘리는 효과, 스위치포트에서 지원해야 한다.
- 이더채널: 시스코 장비에서의 failover 구성

## 스토리지

### 스토리지 유형

- **Block Storage**
    
    블록 저장소는 데이터를 일정한 크기의 블록으로 분할하고 각 블록에 고유한 주소를 할당하여 저장하는 방식이다. 각 블록은 독립적으로 관리되며 파일 시스템에 의해 조직된다.
    
    주로 데이터베이스, 가상 머신, 워크로드 등과 같은 대용량 및 고성능의 응용 프로그램을 지원하는 데 사용된다. 블록 수준의 액세스를 통해 데이터를 읽고 쓰는 데 효율적이다.
    
- **File Storage (NFS)**
    
    파일 저장소는 데이터를 파일의 형태로 저장하며, 파일 시스템을 사용하여 데이터를 조직한다. 여러 사용자가 동시에 파일에 액세스할 수 있도록 공유 파일 시스템을 제공한다.
    
    네트워크 환경에서 파일 공유 및 협업이 필요한 경우에 주로 사용된다. 여러 사용자가 동일한 파일에 동시에 액세스할 수 있다.
    
    페타바이트 단위 스토리지로 확장이 용이한다. 파일스토리지를 하드웨어 제품으로 판매하는것을 NAS라고 한다.
    
- **Object Storage (cloud native storage)**
    
    **Object Storage** 는 데이터를 물체 또는 객체로 관리한다. 각 객체에는 메타데이터와 고유한 key가 포함되어 있다. 파일이나 블록과는 다르게 계층 구조가 없이 데이터를 저장하며, 대규모 분산 스토리지를 제공하는것이 가능하다.
    
    대량의 정형 및 비정형 데이터를 효율적으로 저장하고 검색하는 데 사용된다. 대규모 웹 애플리케이션, 백업 및 아카이브 시스템에서 많이 사용된다.
    
    쓰기 작업이 적은 읽기 위주의 작업에 유리하다.
    

### 스냅샷 방식

처음 백업시에는 전체를 백업한후 변경분에 대한 내용만 백업하는 증분 백업 방식을 사용한다.

향후 복구시에는 가장 처음 스냅샷부터 사용해서 쌓아나가며 복구한다.

블록스토리지에서 이런 백업 방식을 많이 사용한다.

### DAS

- 서버에 직접연결하는 스토리지
- RAID 컨트롤러가 서버에 있는 경우(엔트리, 미들레인지), 각각의 디스크에 있는 경우가 있다.

### NAS

- 네트워크를 통해 여러대의 서버가 엑세스할수 있는 스토리지
- NFS, SMB/CIFS등의 파일 레벨의 프로토콜 사용

### SAN

- FC-SAN
    - 전용 스위치 및 광섬유 사용
    - Fibre Channel 프로토콜
- IP-SAN(일반스위치)
    - iSCSI(iSCSI Over IP, 블록 레벨 데이터 전송) 프로토콜을 사용
    - 일반 스위치 사용 및 다양한 케이블 사용 가능
    - 비교적 저렴한 비용으로 스토리지 액세스를 제공하므로 중소기업 및 중소 규모의 데이터 센터에서 많이 사용

### 핫스페어

빈디스크를 미리 꼽아, 장애 발생시 바로 활성화시키는 기법

### 씬 프로비저닝

씬 프로비저닝(Thin Provisioning)"은 스토리지 관리 기술 중 하나로, 스토리지 리소스를 효율적으로 사용하기 위한 방법이다.

씬 프로비저닝은 물리적인 스토리지 용량을 미리 할당하지 않고 필요한 만큼 동적으로 할당함으로써 스토리지 공간을 절약하고 스냅샷 및 백업 효율성을 높일수 있다. 

overcommit의 문제가 있으며 정확히 필요한 용량의 예측이 힘들다.

### 자동 계층화

서로 다른 성능의 디스크를 조합한후 데이터들의 엑세스 빈도를 분석하여 이용빈도가 높은 데이터는 고가의 장비, 이용빈도가 낮은 데이터는 싸고 느린 장비에 사용한다.

### 디둡

백업시 데이터를 압축하거나, 이미 백업되 데이터는 백업하지 않는 방식이다.

## 솔루션

나기오스, 자빅스, cloudwatch 등이 있다.

지표의 종류가 많거나 지표를 scan하는 간격이 줄인다면, 중요한 신호를 놓칠 가능성이 있다.

## CDN

- PoP를 통한 빠르게 백본에 접근할수 있고, 캐싱을 통한 응답속도의 향상을 기대할수 있다.
- 주로 읽기 위주의 정적 콘텐츠 배포에 사용한다.
- AWS cloudfront

## DSR 구성을 통한 부하분산

- l4 스위치(로드밸런서)를 통한 부하분산을 하고, 서버에서 응답할때 는 l2 스위치를 사용하여 다이렉트하게 소스로 쏴주는 방식
- 로드밸런서가 소스, dest를 변환하는 부하를 줄일수 있다. 이를 위해 로드밸런서가 서버에 요청을 할때 source ip 주소에 대한 정보를 전달한다. 현재는 많이 사용하지 않는다.
- AWS ELB는 지원하지 않는다.

## 클라우드

### 서비스의 유형

- IAAS (infrastructure as a service)
    - 고객이 인프라를 구축하고 소프트웨어를 운영해야 한다.
    - IAAS의 핵심은 가상머신 서비스이다.
- PAAS
    - 인프라는 준비되어있으며 소프트웨어만 가져오면 된다.
- SAAS
    - 소프트웨어 서비스를 제공한다.
    - 최근에는 구독형 으로 서비스를 많이 제공한다.

## openstack

- 클라우드 컴퓨팅을 구현하기 위한 오픈소스다.
- private, public, edge의 형태로 서비스가 가능하다.
- qemu/kvm기반으로 동작
- VM, bare metal, container 관리
- 오픈스택위에 쿠버네티스, 쿠버네티스위에 오픈스택의 형태로 서비스가 가능하다.

### openstack 주요 서비스

- horizon(콘솔)
- nova(컴퓨팅)
- cinder(블록스토리지), swift(오브젝트 스토리지), manilar(파일스토리지)
- glance(이미지)
- keystone(IAM)
- Neutron(VPC), Octavia(ELB)

## kubernetes

- 컨테이너 오케스트레이션 툴
- cluster architecture (여러 노드의 그룹)
- etcd(nosql): 콘피그 관리한다.
- pod: container 배포 단위
- kubelet: 각 노드당 api를 통해 컨트롤 플레인과 소통하기 위한 모듈
- container engine: 도커, 팟맨, containerd(요즘 표준 기술)
- cloud native network(컨테이너간 통신): calico, flannel, cilllium
- cloud native storage(컨테이너에 스토리지 제공): ceph, gluster(nfs)
- master: control plain 이자 워커 노드
  

- l4 스위치(로드밸런서)를 통한 부하분산을 하고, 서버에서 응답할때 는 l2 스위치를 사용하여 다이렉트하게 소스로 쏴주는 방식
- 로드밸런서가 소스, dest를 변환하는 부하를 줄일수 있다. 이를 위해 로드밸런서가 서버에 요청을 할때 source ip 주소에 대한 정보를 전달한다. 현재는 많이 사용하지 않는다.
- AWS ELB는 지원하지 않는다.
  
