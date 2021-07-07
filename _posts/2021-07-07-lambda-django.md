---
layout: post
title: "Zappa 라이브러리를 사용하여 Django 프로젝트 lambda 배포하기"
author: "DONGWON KIM"
meta: "Springfield"
categories: "CI/CD"
comments: true
---

## 1. 개요
AWS Lambda는 서버를 프로비저닝하거나 관리하지 않고도 코드를 실행할 수 있는 서버리스 컴퓨팅 서비스이다. Lambda를 사용하면 대부분의 인터프리터 언어로 코딩된 애플리케이션(백엔드) 서비스를 실행할수 있다. 
Zappa는 파이썬 어플리케이션을 쉽게 서버리스 배포해줄수 있게 도와주는 파이썬 라이브러리 이다. 이번 실습에서는 zappa를 활용하여 강원대학교 축제 서비스를 lambda와 API gateway를 활용하여 배포해보자 한다.

## 2. 기본 인프라 구축을 위한 terraform 코드
```bash
.
├── aurora
│   ├── main.tf
│   └── variable.tf
├── aws_caller_identity
│   └── output.tf
├── main.tf
├── sg
│   ├── main.tf
│   ├── output.tf
│   └── variable.tf
├── terraform.tfstate
├── terraform.tfstate.backup
├── vpc
│   ├── igw.tf
│   ├── main.tf
│   ├── output.tf
│   ├── route_table.tf
│   ├── subnet.tf
│   └── variable.tf
└── vpc_endpoint
    ├── main.tf
    ├── output.tf
    └── variable.tf
```
lambda 함수가 통신시 사용할 eni가 배치될 VPC(서브넷, 라우팅테이블 등), 보안그룹, 게시물, 푸드트럭 정보를 저장할 오로라 데이터베이스, vpc_endpoint가 필요하다.
lambda 함수가 s3와 통신하기 위해선, nat gateway 혹은 vpc_endpoint가 필요하다. 다음은 vpc_endpoint를 생성하기위한 테라폼 코드이다.

```
resource "aws_vpc_endpoint" "s3" {
  vpc_id       = var.vpc_id
  service_name = "com.amazonaws.ap-northeast-2.s3"
}

resource "aws_vpc_endpoint_route_table_association" "s3_private" {
  vpc_endpoint_id = aws_vpc_endpoint.s3.id
  route_table_id = var.route_table_id
}
```

## 4. zappa 설치 및 config 작성
먼저 다음 명령어를 사용하여 zappa 라이브러리를 설치한다.
```
pip3 install zappa
```

프로젝트의 루트 폴더에서 다음 명령어를 사용하여 config 파일을 생성한다. config 파일의 이름은 zappa_settings.json으로 정해진다.
```
zappa init
```

config 파일의 이름을 다음과 같게 수정한다. 본인의 django 프로젝트의 settings.py 위치에 따라 django_settings 값을, 테라폼에 의해 생성된 vpc의 서브넷, 보안그룹 id를 수정해주면 된다. 필요한 환경변수가 있다면 키와 값을 environment_variables 섹션에 추가해주면 된다. 만약 안쓴다면 environment_variables 섹션을 제거한다.

```
{
    "dev": {
        "aws_region": "ap-northeast-2",
        "apigateway_enabled": true,
        "profile_name": "default",
        "runtime": "python3.8",
        "s3_bucket": "knufesta2019",
        "django_settings": "config.environments.production",
        "vpc_config": { // Optional Virtual Private Cloud (VPC) configuration for Lambda function
            "SubnetIds": [ 
                "subnet-029af834932299ece",
                "subnet-0f26e4c812b36ad2b",
                "subnet-0c9ded1abcffa86ab"
            ], // Note: not all availability zones support Lambda!
            "SecurityGroupIds": [ "sg-0750007a2c668e92b" ]
        },
        "slim_handler": true,
        "environment_variables": {
            "your_key_1": "your_value_1",
            "your_key_2": "your_value_2",
            ...
        }
    }
}
```

그리고 zappa를 비롯한 django 프로젝트 구동에 필요한 모든 파이썬 라이브러리에 대한 정보가 requirements.txt에 있어야 한다. 다음 명령어로 만들어주자.
```
pip3 freeze > requirements.txt
```

## 5. 배포
다음 명령어를 사용하여 api-gateway를 비롯한 lambda 함수를 배포할수 있
다.
```
zappa deploy dev

# 데이터베이스 마이그레이션
zappa manage dev migrate

# 슈퍼 유저 생성
zappa invoke --raw dev "from django.contrib.auth.models import User; User.objects.create_superuser('freshtuna', 'freshtuna@freshtuna.com', 'freshtuna')"
```

## 6. 짜잔
API gateway의 스테이징 주소에 접속하면 다음과 같이 배포된 웹사이트를 볼수 있다.
![Image Alt 텍스트](/img/2021/7/7/example.PNG)