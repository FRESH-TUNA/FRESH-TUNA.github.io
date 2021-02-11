---
layout: post
title: "탄력적 ip 를 활용한 고가용성 환경 구축"
author: "DONGWON KIM"
meta: "Springfield"
categories: "CI/CD"
comments: true
---

## 1. 개요
elastic ip는 변하지 않는 정적 공인 ip 주소 서비스이다. 이미 ec2에 할당된 elastic ip를 다른 ec2에 할당해도 ip주소는 변하지 않기 때문에, 동일한 ip주소로 서비스를 유지할수 있는 장점이 있다. 하지만 로드밸런서와 오토스케일링그룹등의 서비스의 좋은점이 많아 elastic ip는 사용하지 않는것이 좋기는 하다. 이번 실습에서는 단일 인스턴스 환경에서, AZ 장애 발생시, 바로 다른 AZ에서 인스턴스를 생성하고 elastic ip를 할당하여 고가용성을 구현하는 시나리오를 구축해보고자 한다.

## 2. 구조도
![Image Alt 텍스트](/img/2021/2/11/terraform/scenario_4.png)

## 3. 주요 테라폼 코드
```sh
resource "aws_autoscaling_group" "logic" {
  name                      = "logic"
  max_size                  = 1
  min_size                  = 1
  health_check_grace_period = 300
  health_check_type         = "EC2"
  desired_capacity          = 1
  
  launch_template {
    id      = var.aws_launch_template_id
    version = "$Latest"
  }
  
  vpc_zone_identifier = var.subnet_ids

  tag {
    key                 = "asg"
    value               = "test"
    propagate_at_launch = true
  }

  timeouts {
    delete = "15m"
  }
}
```
이번 프로젝트에서 최초로 오토스케일링 그룹을 도입했다. 최소갯수, 최대갯수, 원하는갯수를 1로 설정해두면, 한개의 인스턴스로 서비스를 시작하게 된다. 이후 AZ Failure가 발생하면 새로운 인스턴스 1개를 다른 AZ에 자동으로 생성해주므로 가용성을 높일수 있다.

```bash
resource "aws_iam_role" "asg" {
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

resource "aws_iam_policy" "ec2_associate_elasticip" {
  name        = "associate_elasticip"
  description = "associate_elasticip"

  policy = <<EOF
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": "ec2:AssociateAddress",
            "Resource": "*"
        },
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

resource "aws_iam_role_policy_attachment" "asg_ec2_associate_elasticip" {
  role       = aws_iam_role.asg.name
  policy_arn = aws_iam_policy.ec2_associate_elasticip.arn
}

resource "aws_iam_instance_profile" "asg" {
  name = "asg_logic_associate_elasticip"
  role = aws_iam_role.asg.name
}
```
AZ Failure시 다른 AZ에 인스턴스가 생성되고 해야하는일이 있는데 바로 사용하던 elasticip를 재할당해주는것이다. 이를 위해 eip 할당 권한을 가진 인스턴스 프로필을 만들어주었다.