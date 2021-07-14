---
layout: post
title: "cloudfront signed cookie를 활용한 회원전용 동영상 서비스 구축"
author: "DONGWON KIM"
meta: "Springfield"
categories: "Infra"
comments: true
---

## 1. 개요
cloudfront는 edge location을 통한 컨텐츠의 빠른 다운로드, 업로드를 지원하는 서비스이다. 이외에도 signed cookie, signed url을 통해 허가된 사용자들에게만 컨텐츠를 제공하는 서비스를 구축할수 있다. 이번 시나리오에서 간단한 비디오 스트리밍 어플리케이션과 cloudfront를 사용하여 회원전용 동영상 서비스를 구축해보고자 한다.

### 1-1. 프로젝트 주소
[테라폼 코드](https://github.com/lunacircle4/infra-projects/tree/master/cloudfront-signed-cookie-example)<br/>
[백엔드 코드](https://github.com/lunacircle4/django_streaming_example)
<br/><br/>

## 2. 구조도
![Image Alt 텍스트](/img/2021/2/12/scenario_6.png)

## 3. 주요 테라폼 코드
```
resource "aws_cloudfront_origin_access_identity" "scenario_6" {
  comment = "scenario_6"
}

resource "aws_cloudfront_public_key" "scenario_6" {
  comment     = "test public key"
  encoded_key = var.cloudfront_public_key
  name        = "test_key"
}

resource "aws_cloudfront_distribution" "scenario_6" {
  origin {
    domain_name = var.bucket_regional_domain_name
    origin_id   = "app_s3"

    s3_origin_config {
      origin_access_identity = var.app_cloudfront_access_identity_path
    }
  }
...
```
당연하게도 이프로젝트에선 비디오 스트리밍 서비스를 위해 cloudfront를 도입했다. OAI를 통해 s3로의 요청이 cloudfront를 통해서만 이루어지도록 설정하는 테라폼 코드이다.

```
resource "aws_ssm_parameter" "app_envs" {
  name        = "/scenario_6/app"
  description = "app credential"
  type        = "SecureString"
  value       = templatefile("${path.module}/app.env.mock", 
  { 
    AWS_DEFAULT_REGION = var.AWS_DEFAULT_REGION,
    AWS_STORAGE_BUCKET_NAME = var.AWS_STORAGE_BUCKET_NAME,
    ALLOWED_HOST = "test.freshtuna.me",

    SECRET_KEY = var.SECRET_KEY,
    DB_HOST = var.DB_HOST,
    DB_NAME = var.DB_NAME,
    DB_USER = var.DB_USER,
    DB_PASSWORD = var.DB_PASSWORD,
    DB_PORT = var.DB_PORT,

    CLOUDFRONT_URL = var.CLOUDFRONT_URL,
    CLOUDFRONT_PRIVATE_KEY = var.CLOUDFRONT_PRIVATE_KEY,
    CLOUDFRONT_KEY_PAIR_ID = var.CLOUDFRONT_KEY_PAIR_ID })

  tags = {
    environment = "production"
  }
}

resource "aws_ssm_parameter" "app_cloudfront_private_key" {
  name        = "/scenario_6/app/cloudfront_private_key"
  description = "app_envs_cloudfront_private_key"
  type        = "SecureString"
  value       = templatefile("${path.module}/app_cloudfront_private_key.env.mock", 
  { CLOUDFRONT_PRIVATE_KEY = var.CLOUDFRONT_PRIVATE_KEY })

  tags = {
    environment = "production"
  }
}

resource "aws_iam_policy" "ssm_get_app_envs" {
  name        = "ssm_get_app_envs"
  description = "ssm_get_app_envs"

  policy = <<EOF
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": "ssm:GetParameter",
            "Resource": "${var.app_envs_arn}*"
        }
    ]
}
EOF
}
```

이번 프로젝트에서 처음으로 파라미터 스토어 서비스를 사용해보았다. ec2 인스턴스가 생성될때 paremeter store에서 환경변수들을 받아와서 활용할수 있도록 구현해 보았다. 또 인스턴스에서 파라미터 스토어에 요청을 할수 있도록 정책을 생성하여 role과 연결한다.

```
resource "aws_s3_bucket" "app" {
  bucket = "scenario6"
  acl    = "public-read"

  tags = {
    Name        = "My bucket"
    Environment = "Dev"
  }
}

resource "aws_s3_bucket_policy" "app" {
  bucket = aws_s3_bucket.app.id

  policy = <<POLICY
{
  "Version": "2012-10-17",
  "Id": "MYBUCKETPOLICY",
  "Statement": [
    {
      "Sid": "IPAllow",
      "Effect": "Allow",
      "Principal": {
        "AWS": "${var.app_role_id}"
      },
      "Action": "s3:*",
      "Resource": [
        "${aws_s3_bucket.app.arn}",
        "${aws_s3_bucket.app.arn}/*"
      ]
    },
    {
      "Sid": "IPAllow",
      "Effect": "Allow",
      "Principal": {
        "AWS": "${var.cloudfront_oai_app_iam_arn}"
      },
      "Action": "s3:*",
      "Resource": [
        "${aws_s3_bucket.app.arn}",
        "${aws_s3_bucket.app.arn}/*"
      ]
    }
  ]
}
POLICY
}
```
비디오들을 저장할 s3 버킷을 하나 생성해주었다. 이때 cloudfront oai, iam role principal로 설정하여 특정 cloudfront distribution 이나 인스턴스에서만 s3로 요청을 할수 있게 구성했다.
<br/><br/>

## 4. 백엔드 코드
[https://github.com/lunacircle4/django_streaming_example](https://github.com/lunacircle4/django_streaming_example)
