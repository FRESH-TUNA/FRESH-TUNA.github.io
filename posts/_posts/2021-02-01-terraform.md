---
layout: post
title: "vpc gateway endpoint를 활용하여 안전하게 s3 활용하기"
author: "DONGWON KIM"
meta: "Springfield"
categories: "Infra"
comments: true
---

## 1. 개요
s3 는 기본적으로 아마존 VPC에서 다루는 오브젝트로, ec2에서 s3를 접근하고자 할때 인터넷을 통해서 접근한다. 이걸 방지하기 위해서 나온 개념이 vpc endpoint이며, 이를 통해 ec2에서 아마존 사설망을 통해 s3에 접근할수 있고 인터넷을 통한 부정한 엑세스를 방지할수 있다.

### 1-1. 프로젝트 주소
[테라폼 코드](https://github.com/lunacircle4/infra-projects/tree/master/s3-endpoint)<br/>
<br/><br/>

## 2. 구조도
![Image Alt 텍스트](/img/2021/2/1/terraform/scenario_3.PNG)
s3 에 엑세스할수 있는 인스턴스는 프라이빗 서브넷에 위치해 있고 오르지 퍼블릭 서브넷의 bastion 인스턴스를 통해서만 접근할수 있다. s3는 인터넷이 아닌 gateway endpoint를 통한 사설망 통신을 통해 접근한다.

## 3. 주요 테라폼 코드
```sh
resource "aws_vpc_endpoint" "s3" {
  vpc_id       = var.vpc_id
  service_name = "com.amazonaws.ap-northeast-2.s3"
}

resource "aws_vpc_endpoint_route_table_association" "s3_private" {
  vpc_endpoint_id = aws_vpc_endpoint.s3.id
  route_table_id = var.route_table_id
}
```
s3 gateway 엔드포인트를 생성한후 프라이빗 서브넷의 라우트테이블과 연동해주는 작업이 필요하다.
이 작업을 통해 인스턴스에서 s3로 가는 요청이 인터넷이 아닌 프라이빗 엔드포인트 게이트웨이로 전달된다.

```sh
resource "aws_s3_bucket" "scenario_4" {
  bucket = "scenario.4.bucket"
  acl    = "private"

  tags = {
    Name        = "My bucket"
    Environment = "Dev"
  }
}

resource "aws_s3_bucket_policy" "scenario_4" {
  bucket = aws_s3_bucket.scenario_4.id

  policy = <<POLICY
{
  "Version": "2012-10-17",
  "Id": "MYBUCKETPOLICY",
  "Statement": [
    {
      "Sid": "IPAllow",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:*",
      "Resource": ["${aws_s3_bucket.scenario_4.arn}/*",
                  "${aws_s3_bucket.scenario_4.arn}"],
      "Condition": {
        "StringEquals": {
          "aws:sourceVpce": "${var.aws_vpc_endpoint_id}"
        }
      }
    }
  ]
}
POLICY
}

resource "aws_iam_role" "s3_access" {
  name = "asg_logic_associate_elasticip"

  assume_role_policy = <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Action": "sts:AssumeRole",
      "Principal": {
        "Service": "ec2.amazonaws.com"
      },
      "Effect": "Allow",
      "Sid": ""
    }
  ]
}
EOF
}

resource "aws_iam_policy" "s3_access" {
  name        = "s3_access"
  description = "s3_access"

  policy = <<EOF
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowStsDecode",
            "Effect": "Allow",
            "Action": "sts:DecodeAuthorizationMessage",
            "Resource": "*"
        }
    ]
}
EOF
}

resource "aws_iam_role_policy_attachment" "s3_access" {
  role       = aws_iam_role.s3_access.name
  policy_arn = aws_iam_policy.s3_access.arn
}

resource "aws_iam_instance_profile" "s3_access" {
  name = "s3_access"
  role = aws_iam_role.s3_access.name
}
```

s3 버킷 생성후 bucket policy를 생성해서 vpc endpoint으로 부터의 요청만 허용하도록 한다. 그리고 ec2 인스턴스에서 s3에 요청을 보낼수 있도록 iam role 생성이 필요하다.

