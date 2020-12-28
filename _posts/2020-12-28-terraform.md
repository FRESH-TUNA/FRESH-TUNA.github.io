---
layout: post
title: "Terraform을 활용한 aws 인프라 구축해보기"
author: "DONGWON KIM"
meta: "Springfield"
categories: "CI/CD"
comments: true
---

## 1. 개요
Infrastructure as a code 도구 를 활용하면 GUI가 아닌 코드를 통해 인프라를 구축할수 있다. 여기까지 들어보면 어떤 장점이 있을가 싶은데, 마우스를 통해 클릭하여 인프라를 생성하는것보다 코드를 작성하는것이 조금더 빠르고, 작성해둔 코드를 똑같은 인프라를 구축할때 재사용할수 있는 장점이 있다. 이번글에서는 iaac 도구중 하나인 테라폼을 통해 간소한 형태의 AWS 인프라를 구축하면서 중요하다고 생각했던 키포인트들을 정리해보고, 차후 여러 시나리오를 가정해서 좋은 인프라 구조를 구축하는 후속 포스팅을 해보려 한다.

## 2. 시나리오
간단한 게시판 서비스를 위한 인프라 구축을 하려고 한다. 정적파일을 저장하고 처리하는 계층을 분리해야 하고, 게시물들을 저장할 데이터베이스와 사용자의 요청을 처리할 웹서버가 필요하다. 일일사용자수가 적고 시간대별 사용자수역시 일정한것을 감안하여 최소비용으로 구축하고자 한다. 예상되는 구성도는 다음과 같다.
<br>
![Image Alt 텍스트](/img/2020/12/28/terraform/aws.jpg)


## 3. 네트워크 구축 과정
```
resource "aws_vpc" "scenario_1_vpc" {
  cidr_block = "172.16.0.0/26"
  tags = {
    Name = "scenario-1-vpc"
  }
}
```
VPC는 AWS에서 다루는 네트워크의 기본단위이다. 사용자들이 앞으로 만들 서버와 인터넷을 통해 통신하기 위해선 네트워크 구축이 필요한데 가장 먼저 VPC를 만들어주어야 한다. cidr_block 을 통해 어떤 ip 대역을 사용할지 결정하고 나는 172.16.0.0/26 를 사용했다. 보통 10.0.0.0/8, 172.16.0.0/16, 192.168.0.0/24의 사설 ip 대역을 사용하지만 공인 ip 대역 역시 사용이 가능하다. 다만 공인 ip 대역 사용시 인터넷을 통한 통신이 불가능하므로 사설 ip대역을 사용하는것을 강력히 권장한다. VPC를 구축한다음엔 다음 코드로 서브넷팅을 할 차례다.

```
resource "aws_subnet" "scenario-1-public-subnet" {
  vpc_id = aws_vpc.scenario_1_vpc.id
  cidr_block = "172.16.0.0/28"
  availability_zone = "ap-northeast-2a"
  tags = {
    Name = "scenario-1-public-subnet"
  }
}

resource "aws_subnet" "scenario-1-private-subnet" {
  vpc_id = aws_vpc.scenario_1_vpc.id
  cidr_block = "172.16.0.16/28"
  availability_zone = "ap-northeast-2a"
  tags = {
    Name = "scenario-1-private-subnet"
  }
}

resource "aws_subnet" "scenario-1-private-subnet-2" {
  vpc_id = aws_vpc.scenario_1_vpc.id
  cidr_block = "172.16.0.32/28"
  availability_zone = "ap-northeast-2b"
  tags = {
    Name = "scenario-1-private-subnet-2"
  }
}
```
VPC가 허용하는 대역 안에서 웹서버를 배치할 퍼블릭 서브넷 1개, 데이터베이스를 배치할 프라이빗 서브넷 2개를 생성했다. 서브넷을 사용할때는 가용영역을 설정할수 있는데, 여러 가용영역에 서브넷을 만들어서 장애발생시도 고가용성을 유지하도록 구현할수 있다. 프라이빗 서브넷의 경우 인터넷 게이트웨이와 연결되어있지 않기 때문에 인터넷을 통한 외부 공격을 방지할수 있는 장점이 있다. 반면 퍼블릭 서브넷은 인터넷과의 통신을 위해 인터넷게이트웨이가 필요하므로 생성해보자.

```
resource "aws_internet_gateway" "scenario_1_igw" {
  vpc_id = aws_vpc.scenario_1_vpc.id

  tags = {
    Name = "main"
  }
}
```
인터넷 게이트웨이는 VPC 와 인터넷 사이를 연결해주는 고마운 존재이다. 인터넷 게이트웨이에 연결된 서브넷은 퍼블릭 서브넷이라고 하며 인터넷을 통한 통신이 가능하며, 이때 공인 ip주소를 VPC내의 사설ip 주소로 변환해주는 NAT로써의 역활을 수행한다. 인터넷 게이트웨이를 생성 했다면 라우터가 참조할수 있는 라우팅 테이블들을 생성해주는 작업이 필요하다.

```
resource "aws_route_table" "scenario_1_public_route_table" {
  vpc_id = aws_vpc.scenario_1_vpc.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.scenario_1_igw.id
  }

  tags = {
    Name = "main"
  }
}

resource "aws_route_table" "scenario_1_private_route_table" {
  vpc_id = aws_vpc.scenario_1_vpc.id

  tags = {
    Name = "main"
  }
}

resource "aws_route_table_association" "scenario_1_public_rt_association" {
  subnet_id      = aws_subnet.scenario-1-public-subnet.id
  route_table_id = aws_route_table.scenario_1_public_route_table.id
}

resource "aws_route_table_association" "scenario_1_private_rt_1_association" {
  subnet_id      = aws_subnet.scenario-1-private-subnet.id
  route_table_id = aws_route_table.scenario_1_private_route_table.id
}

resource "aws_route_table_association" "scenario_1_private_rt_2_association" {
  subnet_id      = aws_subnet.scenario-1-private-subnet-2.id
  route_table_id = aws_route_table.scenario_1_private_route_table.id
}
```

나는 라우팅 테이블은 서브넷의 라우터가 통신할때 참조하는 테이블로 이해했다. 여기서 퍼블릭 서브넷과 프라이빗 서브넷의 차이점이 한번더 드러나는데, 퍼블릭 서브넷이 참조하는 라우팅 테이블은 목적지가 공인 ip 대역인 요청을 인터넷게이트웨이로 보내는 규칙이 필요하다. 라우팅 테이블들을 생성하고 필요한 규칙을 추가하면 네트워크 구성은 마무리 된다.

## 4. 보안 그룹 구성
보안그룹은 인스턴스의 방화벽의 역활을 담당하는 중요한 오브젝트이다. 이 시나리오에서는 웹서버와 데이터베이스와 웹서버를 위해 각각 1개의 보안그룹이 필요하다. 웹서버는 고객들이 접근할수 있도록 0.0.0.0/0 에 대해 443 포트를, ssh 연결을 위해 x.x.x.x/32(관리자가 접속하는 ip)의 22번 포트 연결을 허용해야 한다.AWS가 제공하는 보안그룹은 특정 IP 대역뿐만 아니라 특정 보안그룹에 속해 있는 인스턴스와의 통신만 허용하는것이 가능하다. 이점에 착안하며 웹서버를 위한 보안그룹을 생성한후 데이터베이스는 웹서버가 속한 보안그룹의 소스들만 통신을 허용하도록 구성해야 한다.
```
resource "aws_security_group" "scenario_1_ec2" {
  description = "Allow"
  vpc_id      = aws_vpc.scenario_1_vpc.id

  ingress {
    description = "communication"
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    description = "ssh"
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["<your_source_ip/32"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

resource "aws_security_group" "scenario_1_rds" {
  description = "Allow"
  vpc_id      = aws_vpc.scenario_1_vpc.id

  ingress {
    from_port   = 5432
    to_port     = 5432
    protocol    = "tcp"
    security_groups = [aws_security_group.scenario_1_ec2.id]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "with logic"
  }
}
```

## 5. 참고자료
### [https://www.44bits.io/ko/post/terraform_introduction_infrastrucute_as_code](https://www.44bits.io/ko/post/terraform_introduction_infrastrucute_as_code)
### [https://registry.terraform.io/providers/hashicorp/aws/latest/docs](https://registry.terraform.io/providers/hashicorp/aws/latest/docs)
