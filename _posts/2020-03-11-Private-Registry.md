---
layout: post
title: "Amazon EC2, S3와 harbor를 활용한 도커 레지스트리 구축"
author: "DONGWON KIM"
meta: "Springfield"
categories: "Infra"
---

## 1. 서론 
![Image Alt 텍스트](/img/2020/03/11/Private-Registry/harbor.png)
CI/CD 파이프라인은 크게 형상관리, 빌드, 테스트, 배포, 모니터링으로 구성되어있다. 
그중 '레지스트리' 는 CI/CD 파이프라인을 통해 빌드된 이미지를 관리하는 서비스이며, 대표적으로 우리가 가장 많이 사용하는 docker hub, AWS ECR 등의 서비스가 있다. 
<br/>docker hub의 경우 무료로 사용할수 있지만, 배포된 이미지가 공개 되기 때문에 누구나 다운로드 해서 받아볼수 있는 단점을 가지고 있다. 
이를 위해선 docker hub의 유료 플랜을 이용하거나 ECR등의 서비스를 이용해 이미지를 관리하는 방법이 있지만, 이 글에선 aws ec2, s3
서비스와 오픈소스 레지스트리 서비스인 "Harbor"를 활용하여 직접 레지스트리를 구축해보자 한다. 

## 2. harbor registry 구축 준비
### 1. ec2 인스턴스 생성
Amazon Elastic Compute Cloud(EC2)는 컴퓨팅 엔진 (원하는 연산 능력을 가진 컴퓨터)을 제공해주는 서비스이다. 
harbor registry를 동작 시키기 위해선 CPU, RAM, STORAGE등의 인프라가 필요한데 이를 위해 ec2 인스턴스 한대가 필요하다. 
Harbor 공식 github 문서에 따르면 다음과 같은 pre-requirements들이 필요하다. 
![Image Alt 텍스트](/img/2020/03/11/Private-Registry/requirements.png)

하지만 테스트 용도의 레지스트리 구축이며, 도커 이미지가 실제로 저장되는 스토리지는 어차피 s3를 사용할것이기 때문에 
Amazon Linux 2 AMI 기반의 t2.micro 인스턴스 한대를 기본 옵션으로 생성하였다. 인스턴스에 필요한 보안그룹의 inbound 설정은 다음과 같이 설정해주었다. 
![Image Alt 텍스트](/img/2020/03/11/Private-Registry/security_group.png)

### 2. s3 버켓 생성
모든 레지스트리들은 이미지를 저장하기 위한 스토리지가 필요하다. 
AWS가 제공하는 스토리지 서비스는 EBS, EFS, S3 등이 있는데, 이중에 나는 s3를 선택하여 버켓 하나를 생성하였다. 
ec2의 디폴트 스토리지는 EBS는 속도가 빠르지만 비용이 s3에 비해 많이 들고, 스케일 업이 어려운 단점이 있가 때문이다. 

### 3. 인증서 발급
도커 레지스트리의 보안 연결을 위해서 반드시 인증서 발급이 필요하다. 
AWS Certificate Manager에 접속하여 내가 원하는 도메인 이름을 바탕으로 인증서를 생성해주었다. 
예를 들어 내가 소유하고 있는 도메인이 hello.com인데 원하는 레지스트리 도메인 이름이 registry.hello.com이라면 인증서 생성시 
도메인 이름을 'registry.hello.com'으로 설정하여 발급해야 한다. 

### 4. 로드 밸런서 생성
향후 레지스트리의 확장성과 SSL 인증서 연동의 편리함을 이유로 ec2와 route53 사이에 로드밸런서를 한대 생성하였다. 
https(443 port) 연결을 위해 필요한 인증서는 위의 ACM에서 만든것을 사용하였다. 

aws는 여러 종류의 로드밸런서를 제공하는데 그중에 나는 Application 타입의 로드밸런서를 생성하였다. 
로드밸런서의 대상그룹에 아까 생성했던 ec2를 추가시킨후, 대상그룹의 80번 포트로 요청을 받으면 ec2의 80번 포트로 보내도록 설정하였다.

### 5. Route53 으로 도메인과 로드밸런서 연동하기
Route53은 aws가 제공하는 DNS 웹서비스이다. 도메인을 구매하거나 등록, 라우팅을 할수 있는 재미있는 친구인데
내가 구매했던 도메인을 활용하여 생성했던 로드밸런서와 연결시키는 작업을 진행했다. 

### 6. s3 접근을 위한 IAM key 생성
harbor registry 접근을 위해선 s3에 읽기, 쓰기 권한을 가진 programmable key가 필요하다. 
IAM 서비스를 이용하여 S3Fullaccess 권한을 가진 계정을 하나 생성하여 credential정보롤 기록해두었다. 
![Image Alt 텍스트](/img/2020/03/11/Private-Registry/iam.png)

## 3. harbor registry 구축하기
### 1. ec2 인스턴스에 docker, docker-compose 설치
harbor를 구동시키기 위해선 docker와 docker-compose가 필요하다. 
amazon linux AMI 2 버전 기준으로 필요한 명령어를 작성해보았다. 
아래 명령어를 차례로 실행해준다음에 ssh에 재접속 해주면 된다. 

```bash
sudo amazon-linux-extras install -y docker

sudo service docker start

sudo usermod -a -G docker ec2-user

docker version

sudo curl -L https://github.com/docker/compose/releases/download/1.22.0/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose

sudo chmod +x /usr/local/bin/docker-compose

sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose

docker-compose version
```

### 2. harbor 설치 프로그램 다운로드 및 압축풀기
https://github.com/goharbor/harbor/releases<br/>
harbor github에 접속해서 최신 release 설치용 파일 링크을 찾은후 wget으로 다운로드 해준다.
![Image Alt 텍스트](/img/2020/03/11/Private-Registry/harbor_release.png)

```bash
wget https://github.com/goharbor/harbor/releases/download/v1.10.1/harbor-offline-installer-v1.10.1.tgz

tar xvzf harbor-offline-installer-v1.10.1.tgz
```

이과정이 끝나면 다음과 같은 폴더 구조가 될것이다. harbor폴더 안으로 이동해주자
![Image Alt 텍스트](/img/2020/03/11/Private-Registry/harbor_download.png)

<!-- ### 3. openssl을 활용한 인증서 발급
harbor를 https로 접근할때 필요한 인증서를 발급해주는 과정이 필요하다. 
다음 명령어를 입력하여 ca.key, ca.crt 파일을 생성해주자. 
-subj 옵션은 각자의 상황에 맞게 적절히 수정해서 적용하면 된다.

```bash
openssl genrsa -out ca.key 4096

openssl req -x509 -new -nodes -sha512 -days 3650 \
  -subj "/C=KR/ST=Seoul/L=Seoul/O=Org/OU=Registry/CN=myregistry.knufesta2019.de" \
  -key ca.key \
  -out ca.crt
``` -->

### 3. harbor.yml 수정
harbor.yml은 harbor 구축시 필요로 하는 설정들이 들어있는 파일이다. 
hostname을 본인의 원하는 도메인으로 설정해주고, 아까 s3에 접근하기 위해 발급받았던 credentials를 넣어주자. 
아까 생성했던 s3 bucket에 관한 정보도 집어넣어어야 하는데, 본인이 만든 bucket의 region과 이름을 잘확인하여 기입한다. 

```
hostname: myregistry.knufesta2019.de

storage_service:
  s3:
    accesskey: xxxxxxxxxxxxxxxxxxxxxxxxxxxx
    secretkey: xxxxxxxxxxxxxxxxxxxxxxxxxxxx
    region: ap-southeast-1
    bucket: tuna-registry-test
```

harbor.yml 수정이 끝나면 다음 명령어를 실행한다.
```bash
sudo ./prepare
```

### 4. config.yml, nginx.conf 수정
./prepare 명령어 실행이 완료되면 common 폴더가 생성 되었을텐데, common 폴더의 일부 파일들의 설정을 바꿔주는 작업을 진행해야 한다. 
가장먼저 common/config/registry 폴더에 있는 config.yml 의 auth 부분을 다음과 같이 수정해주자.
```bash
# before
auth:
  token:
    issuer: harbor-token-issuer
    realm: http://myregistry.knufesta2019.de/service/token
    rootcertbundle: /etc/registry/root.crt
    service: harbor-registry

# after: "realm"에 여러분이 설정한 도메인이 있을것이다, 앞에 http를 https로 바꾸어주자
auth:
  token:
    issuer: harbor-token-issuer
    realm: https://registry.knufesta2019.de/service/token
    rootcertbundle: /etc/registry/root.crt
    service: harbor-registry
```
다음으로 common/config/nginx 폴더에 존재하는 nginx.conf 를 수정해줘야한다. 
nginx.conf 에 존재하는 특정 설정들을 모주 주석처리 해주면 된다.
```bash
# before
proxy_set_header X-Forwarded-Proto $scheme;

# after: 위 설정을 보이는대로 모두 주석처리 해주자
# proxy_set_header X-Forwarded-Proto $scheme;
```


### 5. Harbor 배포 하기
다음명령어를 입력하여 Harbor를 배포 할수 있다.
```bash
sudo docker-compose up -d
```

먼저 우리가 정했던 도메인 (예: myregistry.knufesta2019.de)으로 접속되는지 확인해보자. 
다음 화면이 나온다면 harbor 구축의 큰산을 넘은것이다. 
아이디로 admin, 패스워드로 Harbor12345 (기본계정)을 입력해서 로그인 하도록하자.
![Image Alt 텍스트](/img/2020/03/11/Private-Registry/harbor_intro.png)

메인화면이 뜨면 화면 우측 상단의 admin메뉴의 'Change Password'를 클릭하여 계정의 기본 패스워드를 빠르게 바꿔주자
![Image Alt 텍스트](/img/2020/03/11/Private-Registry/main.png)

패스워드가 바뀌면 Project 메뉴를 들어가서 원하는 이름으로 project를 추가해주자. 
project란 이미지 이름 앞에 붙은 namespace 정도로 생각하시면 될것 같다.
<br/> ex) project_1/image:tag, project_2/image:tag ...
![Image Alt 텍스트](/img/2020/03/11/Private-Registry/project.png)

### 6. Harbor 레지스트리에 push 해보기
먼저 우리가 구축한 registry에 로그인한다.
```bash
#Username: admin
#Password: xxxxxxxx (아까 재설정한 패스워드)
# 본인이 route53에서 만든 url
docker login myregistry.knufesta2019.de
```

그리고 로컬에 가지고 있던 이미지를 우리가 만든 레지스트리에 푸시할수 있도록 tagging 한다.
```bash
#Username: admin
#Password: xxxxxxxx (설정한 패스워드)
docker login myregistry.knufesta2019.de
docker tag myimage:hoho myregistry.knufesta2019.de/organization/myimage:hoho
```

tagging이 끝나면 push가 잘 되는지 확인한다.
```bash
docker push myregistry.knufesta2019.de/organization/myimage:hoho
```

### 7. Harbor 레지스트리에 pull 해보기
pull 테스트용 이미지를 docker hub에서 골라서 받아주도록 하자. 
그리고 우리가 만든 레지스트리에 push 하기 위해서 tagging 해보자
```bash
docker pull mariadb:latest
docker tag mariadb:latest registry.knufesta2019.de/organization/mariadb:latest
```

tagging이 끝나면 우리가 만든 레지스트리에 푸시해보자
```bash
# push 테스트
docker push myregistry.knufesta2019.de/organization/mariadb:latest
```

푸시가 완료되었다면 로컬에 있는 이미지를 지우고 다시 풀을 받아보자
풀이 정상적으로 완료된다면 성공이다!
```bash
# pull 테스트
docker push myregistry.knufesta2019.de/organization/mariadb:latest
```

## 4. 참고자료 및 출처
### 1. 이미지
https://goharbor.io/ 
<br/>https://www.44bits.io/ko/post/news--docker-found-unauthorized-access-to-docker-hub-database

### 2. 참고자료
https://www.44bits.io/ko/post/running-docker-registry-and-using-s3-storage
<br/>https://novemberde.github.io/2017/04/09/Docker_Registry_0.html
<br/>https://github.com/goharbor/harbor
<br/>https://engineering.linecorp.com/ko/blog/harbor-for-private-docker-registry/
<br/>https://github.com/goharbor/harbor/issues/3114
