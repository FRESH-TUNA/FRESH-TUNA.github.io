---
layout: post
title: "Cookie와 Session"
# author: "DONGWON KIM"
# meta: "Springfield"
categories: "Infra"
---

## 1. HTTP의 간략한 정의

HTTP는 Hypertext Transfer Protocol 의 약자로  OSI 7, TCP/IP의 응용계층에 속한 텍스트 기반의 통신 규약(프로토콜)이다. 1989년 CERN의 Tim Berners-Lee에 의해 개발이 시작되었으며 HTTP/1.1, HTTP/2를 거쳐서 HTTP/3 까지 발표되었다. 

HTTP는 TCP를 기반으로 하는 프로토콜이다. (HTTP/3은 UDP를 사용한다)  클라이언트와 서버 사이에 통신시에 필요한 규칙을 정의하고 있으며 일반적으로 TCP 80 Port를 이용하여 통신한다. HTTP는 이미지, 동영상, 오디오, 텍스트 문서 등 종류를 가리지 않고 전송 가능하다. 우리가 즐겁게 사용하는 World Wide Web도 HTTP를 기반으로 하여 오늘도 수많은 통신이 이루어지고 있다.

## 2. HTTP의 비연결성(Connectionless)과 비상태성(Stateless)

비연결성이란, 클라이언트와 서버가 연결을 맺은후, 클라이언트 요청에 대하여 서버가 응답을 마치면 맺었던 연결을 끊어 버리는 성질을 의미한다. 따라서 클라이언트, 서버 입장에서 보았을때 새로운 연결 발생시, 이전에 서버와 연결했던 사실을 알수 없다. (항상 새로운 연결이 발생했다고 인지한다.)

비상태성이란, 연결이 종료된후, 연결 기간동안의 상태를 유지하지 않는 성질을 의미한다. 즉 다음에 서버와 클라이언트가 연결되었을때 이루어지는 통신은 이전에 있었던 연결에 영향을 받지 않는다.

## 3. Stateless 의 문제점

우리가 인터넷 뉴스 게시판에 댓글을 다는 상황을 예시로 들어보자. 댓글을 달기 위해서는 해당 웹서비스에 회원가입을 한후 로그인을 먼저 진행해야 한다. HTTP는 Stateless 하다는 원칙을 따져보면 사용자와 서버사이에 연결이 이루어지고 로그인을 위한 정보교환을 진행했지만, 다음번 댓글을 달기 위해 연결을 설정했을때는 이전 연결의 state가 존재하지 않는다. 즉 로그인을 아무리해도 댓글을 달수가 없는 사태가 벌어질수 있는것이다.
이문제를 해결하기 위해서 Cookie, JWT등의 개념이 도입되었다.

## 4. Cookie
Cookie란 서버가 클라이언트의 웹브라우저에 보내는 작은 용량의 데이터이다. 웹브라우저는 전송된 쿠키를 저장했다가 다시 서버와 연결시, 필요한정보들과 request.header에 쿠키를 담아 전송한다. 쿠키를 통해 사용자 인증, 개인화, 트래킹 등을 할수 있어 stateless의 단점을 보완할수 있다.

다만 엘리스와 밥 사이에 오가는 쿠키 데이터를 이브가 나쁜 목적으로 가로챌수 있으므로 보안상 취약하며, 쿠키는 300개까지 만들 수 있으며, 최대 크기는 4,096바이트이고, 하나의 호스트나 도메인에서 최대 20개까지 만들수 있는 제약이 있다.

## 5. Session
![Image Alt 텍스트](/img/2020/02/23/Session-Cookie/Cookies_Explainied.png)
세션은 서버에서 저장하여 관리하는 연결과 관련된 데이터이다. 쿠키와 다른점은 데이터를 클라이언트 뿐만아니라 서버에서도 관리하게 되고, 서버는 클라이언트가 보내준 쿠키를 key로 사용하여 저장소에서 연결과 관련된 데이터를 읽어오게 된다.
예를 들어 쿠키만 사용하는 환경을 가정해보자, 엘리스가 웹서비스에 로그인하기 위해 아이디와 패스워드를 전송하면, 서버는 유저와 관련한 정보들을 쿠키에 담아 클라이언트에 전송한다. 유저는 쿠키를 받아 향후 서비스 이용에 활용한다.

세션을 이용하는 환경을 가정해보자. 엘리스가 웹서비스에 로그인하기 위해 아이디와 패스워드를 전송하면, 서버는 유저와 관련한 정보들을 특정 저장소(메모리, DB...) 에다 저장한다. 이후 저장된 데이터에 해당하는 key값을 쿠키에 넣어 엘리스에게 반환한다. 이처럼 세션을 활용하면, 쿠키만 활용했을때와 비교했을때 보안이 우수하다. 하지만 사용자가 상당히 많은 웹서비스 환경을 가정했을때 , 서버의 성능이 뒷받침되어야 한다는 보장이 있다.

## 6. Session 인증 예제
```python
from django.contrib import auth
from django.contrib.auth import authenticate
from django.contrib.auth.models import User
from django.http import HttpResponse
import json
    
def json_parse(request_body):
    return json.loads(request_body.decode("utf-8"))
    
def signup(request):
    data = json_parse(request.body)
    User.objects.create_user(
        username=data.get('username'), 
        password=data.get('password')
    )
    return HttpResponse(status=201)

def login(request):
    data = json_parse(request.body)
    user = authenticate(
        username=data.get('username'), 
        password=data.get('password')
    )
    if user is not None:
        auth.login(request, user)
        return HttpResponse(status=200)
    else:
        return HttpResponse(status=401)
    
def logout(request):
    auth.logout(request.user)
    return HttpResponse(status=200)        
```
다음과 같이 회원가입, 로그인, 로그아웃을 진행하는 간단한 컨트롤러를 만들어보았다. 그리고 로그인 기능을 이용했을때 클라이언트가 받는 response와 서버가 관리하는 세션 table 을 관찰했다.

```bash
Date: Sat, 22 Feb 2020 17:06:53 GMT
Server: WSGIServer/0.2 CPython/3.8.1
Content-Type: text/html; charset=utf-8
X-Frame-Options: DENY
Content-Length: 0
Vary: Cookie
X-Content-Type-Options: nosniff
Set-Cookie: sessionid=rimuthh7vd6our6m1o5a1yfiqqwfdtcs; expires=Sat, 07 Mar 2020 17:06:53 GMT; HttpOnly; Max-Age=1209600; Path=/; SameSite=Lax
```
백엔드가 제공하는 회원가입기능을 통해, 새로운 유저를 등록한후, 유저의 정보를 이용하여 로그인을 시도한후 받은 response.header이다.
기존의 response와 다르게 header의 Set-Cookie가 설정되어있음을 확인할수 있었다. 웹브라우저는 Set-Cookie를 인식하여 쿠키를 임시저장소에 저장하게 된다.

![Image Alt 텍스트](/img/2020/02/23/Session-Cookie/db.png)

로그인이후, django가 관리하는 데이터베이스의 session table을 관찰한 결과이다. 클라이언트가 받았던 response.header에 위치한 Set-Cookie의 sessionid 랑 일치하는 값이 생성되었음을 확인할수 있다.
향후 클라이언트가 쿠키에 sessionid를 담아서 request를 하면, 백엔드에서 session table을 확인하여 sessionid에 대응되는 데이터를 가져와서 비즈니스 로직을 시행하게 되는것이다.

## 6. Cookie와 Session의 한계

Cookie와 Session을 운용한다는것은  HTTP가 갖는 비연결성, 비상태성을 보완하기 위한 방법이다.이를  역으로 생각해보면 비연결성, 비상태성 원칙을 위배하게 되는것이다. 

또한 쿠키는 보안상 취약점이 있고, 세션 또한 대용량의 트렌젝션을 처리하는 환경을 가정했을때 강력한 컴퓨팅 성능을 필요로 한다.
이 같은 단점을 보완하기 위해 JSON WEB TOKEN(JWT) 라는 개념이 도입이 되었으며 다음에 작성하는 글에서 알아보도록 하겠다.

## 7. 참고자료

[https://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol](https://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol)<br/>
[https://upload.wikimedia.org/wikipedia/commons/0/00/Cookies_Explainied.png](https://upload.wikimedia.org/wikipedia/commons/0/00/Cookies_Explainied.png)<br/>
[https://stackoverflow.com/questions/359434/differences-between-cookies-and-sessions](https://stackoverflow.com/questions/359434/differences-between-cookies-and-sessions)<br/>
[https://cjh5414.github.io/cookie-and-session/](https://cjh5414.github.io/cookie-and-session/)<br/>
[https://developer.mozilla.org/ko/docs/Web/HTTP/Cookies](https://developer.mozilla.org/ko/docs/Web/HTTP/Cookies)<br/>
[https://developer.mozilla.org/ko/docs/Web/HTTP/Headers/Set-Cookie](https://developer.mozilla.org/ko/docs/Web/HTTP/Headers/Set-Cookie)