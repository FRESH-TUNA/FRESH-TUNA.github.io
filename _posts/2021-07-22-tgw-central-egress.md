---
layout: post
title: "Transit Gateway를 통한 중앙 집중식 egress 구축"
author: "DONGWON KIM"
meta: "Springfield"
categories: "Infra"
comments: true
---

## 1. 개요
Transit Gateway는 완전관리형 hub-spoke model 서비스 이다. Transit Gateway는 수천개의 VPC와 연결할수 있고 VPN, Direct Connect와 연결이 가능하며 VPC peering 과 다르게 전이적 라우팅이 가능하다.<br/>

이점을 활용하여 모든 VPC에 NAT Gateway를 설치하는것이 아닌 중앙VPC에 배치하고, 중앙 VPC와 다른 VPC를 Transit Gateway로 연결하여, 중앙 집중식 egress를 구축해보자 한다.
<br/><br/>

## 2. 프로젝트 주소
[https://github.com/lunacircle4/infra-projects/tree/master/transit-gateway-egress](https://github.com/lunacircle4/infra-projects/tree/master/transit-gateway-egress)
<br/><br/>

## 3. 구조도
![Image Alt 텍스트](/img/2021/7/7/tgw-central-egress.jpg)

## 4. 주요 테라폼 코드
```bash
resource "aws_ec2_transit_gateway" "networking" {
  description = "Transit Gateway"
  auto_accept_shared_attachments = "enable"
}

resource "aws_ec2_transit_gateway_vpc_attachment" "networking" {
  vpc_id             = var.vpc_id                  #서브넷이 위치하는 VPC
  subnet_ids         = var.subnets #TGW에 붙을 서브넷
  transit_gateway_id = aws_ec2_transit_gateway.networking.id #붙을 대상이되는 TGW
  tags = {
      Name = "test-tgw-attachment"
  }
}

resource "aws_ec2_transit_gateway_route" "egress" {
  destination_cidr_block         = "0.0.0.0/0"
  transit_gateway_attachment_id  = aws_ec2_transit_gateway_vpc_attachment.networking.id
  transit_gateway_route_table_id = aws_ec2_transit_gateway.networking.association_default_route_table_id
}
```

먼저 중앙 네트워킹 계정에서 transit gateway를 생성해줘야 한다. 그리고 attachment 생성을 통해 transit gateway에 접근할수 있는 interface를 생성한다.

그리고 transit gateway 라우트테이블에 "0.0.0.0/0" -> endpoint 경로를 추가하여 transit gateway 가 outbound로의 역활을 할수 있게 한다.

```bash
# route table
resource "aws_route_table" "public" {
  vpc_id = aws_vpc.networking.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.networking.id
  }

  route {
    cidr_block = var.client_1_cidr
    transit_gateway_id = var.transit_gateway_id
  }

  route {
    cidr_block = var.client_2_cidr
    transit_gateway_id = var.transit_gateway_id
  }

  tags = {
    Name = "main"
  }
}

resource "aws_route_table" "private" {
  vpc_id = aws_vpc.networking.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = var.nat_gateway_id
  }
}
```
client의 공인 ip에 대한 요청은 transit gateway를 통해 중앙 VPC의 private subnet에 도달하여 public subnet의 NAT gateway를 거쳐 outbound로 나간다.

따라서 private subnet의 공인 ip대역 -> nat의 경로를 생성해줘야 한다.
그리고 처리된 트래픽이 다시 public subnet을 통해 들어오기 때문에
public subnet에서 client ip -> tgw의 경로를 생성해줘야 한다.


```bash
resource "aws_ram_resource_share" "tgw" {
  name                      = "example"
  allow_external_principals = false
}

resource "aws_ram_principal_association" "tgw" {
  principal          = var.organization_arn
  resource_share_arn = aws_ram_resource_share.tgw.arn
}

resource "aws_ram_resource_association" "tgw" {
  resource_arn       = var.tgw_arn
  resource_share_arn = aws_ram_resource_share.tgw.arn
}
```
transit gateway를 다른계정에서도 사용할수 있도록 RAM을 통해 공유해주는 작업이 필요하다. 만약 계정들이 같은 조직에 속해있다면, 먼저 마스터 계정의 RAM 설정에 들어가 Organiations share 설정을 해줘야 한다.

만약 Organiations이 아니라면 share를 생성한후 상대방 계정에서 accept 하는 절차가 필요하다.

```bash
resource "aws_ec2_transit_gateway_vpc_attachment" "networking" {
  vpc_id             = var.vpc_id                  #서브넷이 위치하는 VPC
  subnet_ids         = var.subnets#TGW에 붙을 서브넷
  transit_gateway_id = var.shared_tgw_arn #붙을 대상이되는 TGW
  tags = {
      Name = "test-tgw-attachment"
  }
}

resource "aws_route_table" "private" {
  vpc_id = aws_vpc.client.id

   route {
    cidr_block = "0.0.0.0/0"
    transit_gateway_id = var.shared_tgw_arn
  }
}
```
이후 상대방 계정에서 공유받은 transit gateway와 연결되는 interface를 생성해야 한다. 

그리고 route table에서 공인 ip 대역이 destination인 경우 transit_gateway_id로 보내도록 라우팅 테이블을 설정해줘야 한다.

## 5. 시행착오
1. organizations 생성후 ram/설정 들어가서 org 와 리소스 공유 설정 허용해야한다.

2. ssh 키 bastion 으로 복사시 키 파일 이름을 변경하면 

3. 중앙 네트워킹 퍼블릭 서브넷에서 다른 서브넷을 향한 Transit Gateway의 경로를 생성해야한다.

4. bastion 호스트 연결 및 키 복사 예제, 안될시 sudo로 해본다.
```bash
# ssh 접속
ssh -i test.pem ec2-user@X.X.X.X
# scp copy
scp -i ./test.pem ./test.pem ec2-user@X.X.X.X:/home/ec2-user
```

5. network -> client 순으로 apply 해야 한다.

6. 실습이 끝나면 client -> network 순으로 delete 한다.

7. 조직 계정 삭제절차가 생각보다 까다롭다. 신용카드를 등록한후 삭제하는것이 가능한것으로 보인다.
