---
layout: post
title: "Cookie와 Session, JWT"
# author: "DONGWON KIM"
# meta: "Springfield"
categories: "Infra"
---

## 1. Stateless 의 문제점
우리가 인터넷 뉴스 게시판에 댓글을 다는 상황을 예시로 들어보자. 댓글을 달기 위해서는 해당 웹서비스에 회원가입을 한후 로그인을 먼저 진행해야 한다. HTTP는 Stateless 하다는 원칙을 따져보면 사용자와 서버사이에 연결이 이루어지고 로그인을 위한 정보교환을 진행했지만, 다음번 댓글을 달기 위해 연결을 설정했을때는 이전 연결의 state가 존재하지 않는다. 즉 로그인을 아무리해도 댓글을 달수가 없는 사태가 벌어질수 있는것이다.
이문제를 해결하기 위해서 Cookie, 세션, JWT등의 개념이 도입되었다.

## 2. Cookie
HTTP 통신에서 서버가 클라이언트(웹브라우저)에 제공하고, 정해진 기간동안 클라이언트에 저장되는 데이터가 쿠키다. 쿠키를 통해 사용자 인증, 개인화, 트래킹 등을 할수 있어 stateless의 단점을 보완할수 있다. 다만 쿠키의 최대 크기는 4,096바이트 라는 제약이 있고, CSRF(cross site request forgery)를 통한 광고글게시등의 공격과 XSS(cross site scripting) 를 통한 쿠키탈취에 취약하다. 다만 쿠키설정시 httponly로 해주면 자바스크립트를 사용한 쿠키의 접근을 막을수 있다.

## 3. Session
![Image Alt 텍스트](/img/2020/02/23/Session-Cookie/Cookies_Explainied.png)
세션은 서버에서 저장하여 관리하는 데이터이다. 쿠키와 다른점은 데이터를 서버에서 관리하고 서버는 클라이언트가 보내준 쿠키를 key로 사용하여 저장소에서 연결과 관련된 데이터를 읽어오게 된다. 예를 들어 쿠키만 사용하는 환경을 가정해보자, 엘리스가 웹서비스에 로그인하기 위해 아이디와 패스워드를 전송하면, 서버는 유저와 관련한 정보들을 쿠키에 담아 클라이언트에 전송한다. 유저는 쿠키를 받아 향후 서비스 이용에 활용한다.

세션을 이용하는 환경을 가정해보자. 엘리스가 웹서비스에 로그인하기 위해 아이디와 패스워드를 전송하면, 서버는 유저와 관련한 정보들을 특정 저장소(메모리, DB...) 에다 저장한다. 이후 저장된 데이터에 해당하는 key값을 쿠키에 넣어 엘리스에게 반환한다. 이처럼 세션을 활용하면 사용자의 정보를 쿠키에 저장해서 쿠키를 통한 인증과 비교 했을때 보안이 우수하다고 생각한다. 하지만 사용자가 상당히 많은 웹서비스 환경을 가정했을때 서버의 성능이 뒷받침되어야 한다는 보장이 있다.

## 4. Session 인증 예제
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

## 5. Cookie와 Session의 한계
Cookie와 Session을 운용한다는것은  HTTP가 갖는 비연결성, 비상태성을 보완하기 위한 방법이다. 이를 역으로 생각해보면 비연결성, 비상태성 원칙을 위배하게 되는것이다. 또한 쿠키는 보안상 취약점이 있고, 세션 또한 대용량의 트렌젝션을 처리하는 환경을 가정했을때 강력한 컴퓨팅 성능을 필요로 한다. 이 같은 단점을 보완하기 위해 token 라는 개념이 도입이 되었다.

## 6. JSON Web token
jwt 토큰은 데이터를 가지고 있는 json 형식의 데이터를 의미하며 공개키/기본키 혹은 단일키로 서명되어있다.
jwt 토큰은 header(서명방식와 토큰의 타입), payload, 서명으로 구성된다. payload에 토큰의 데이터가 담기게 되는데 크게 registered, public, private 클레임으로 구성된다. registered는 이미 표준으로 전해진 클레임으로 토큰발급주체(iss), 토큰의 제목(subject, 주로 유저id 정보가 들어간다), 만료시간등의 정보가 포함되어있다. 그리고 공개(키 중복금지), 비공개 클레임을 통해 서버 클라이언트간에 필요한 정보를 집어넣게 된다. 마지막으로 header와 payload를 비밀키로 서명한 signature가 들어가게 된다.

### 장점
- JSON 이라는 범용적인 포맷덕분에 웹, 앱등의 다양한 플랫폼에서 토큰을 활용할수 있다.
- 쿠키와 다르게 payload의 크기 제한이 없다. (단점도 될수 있음)
- 토큰의 클레임에 쿠키와 마찬가지로 유저관련 정보를 포함하고 있으므로 세션의 도움없이 사용자 인증을 할수 있다. 만약 공격자가 사용자정보를 가지고 토큰을 위조해도, 공격자는 비밀키를 모르기때문에 서명까지 위조하는것이 불가능하다. 그래도 비밀번호같은 민감한정보는 토큰을 까보면 알수 있으므로 넣지 않도록 하자.

### 해결되지 않은 단점
- 토큰이 유출되었을때 따로 세션을 통해 관리하지 않았다면 토큰을 유효기간이 끝날때까지 정지시킬수 없다.

### refresh token
서버는 유저에게 token과 함께 token의 갱신에 사용되는 refresh token 을 같이 발급한다. token의 유효기간을 최소로 하고, refresh token의 유효기한은 길게 한다. 이때 포인트는 refresh 토큰은 세션을 통해 따로 관리한다. 만약 토큰이 유출되면 유효기간이 짧아 피해를 최소화할수 있고, refresh 토큰은 세션으로 관리되기 때문에 유저가 이상행동을 하거나 refresh 토큰이 유출되면 세션에서 삭제해서 갱신을 방지할수 있다.

### RTR (Refresh Token Rotation)
Refresh Token을 한번만 사용할 수 있게(One Time Use Only) 만드는 방법이다. Refresh Token을 사용하여 새로운 Access Token을 발급받을 때 Refresh Token 도 새롭게 발급받게 된다.

만약 이미 사용된 Refresh Token을 사용하게 되면 서비스측에서 탈취를 확인하여 조치할 수 있게된다. 다만, 사용되지 않은 Refresh Token을 훔쳐 사용하거나, 그냥 지속적으로 Access Token만을 탈취한다면 막을 수 없다.
또한 탈취시 사용정지를 위해 token들의 chain을 따로 관리해야 하는 단점이 있다.

### 클라이언트에서의 토큰 저장
클라이언트에서 jwt토큰을 관리하는방법은 여러가지가 있다.

- private 변수
    - 다른 방식에 비해 보안에서 가장 안전하지만 페이지를 이동하거나, 새로고침만 하여도 토큰 정보가 휘발되어 사용자 경험에 좋지 않아 사실상 단독적으로 사용이 불가능하다.

- 웹 스토리지
    + 클라이언트에 데이터를 저장할 수 있도록 HTML5부터 나온 새로운 방식의 데이터 저장소로 세션스토리지, 로컬스토리지가 있다.

    + 세션 스토리지 의 장단점
        - 페이지를 새로고침하거나, 이동하여도 토큰이 유지되지만 새로운 탭에서 접속 시 세션이 나뉘어지고, 브라우저가 종료되는 순간 휘발되어 세션 스토리지도 사용자 경험에 좋지 않다. 
        - 세션 스토리지는 모든 자바스크립트 코드를 통해 액세스 할 수 있음으로 XSS 공격에도 취약하나 자바스크립트 코드로 제어가 필요하기에 CSRF 공격에서는 안전하다.

    + 로컬 스토리지 의 장단점
        - 로컬 스토리지는 페이지를 이동하거나, 브라우저를 다시 시작하여도 만료 없이 유지된다.
        - 하지만, 세션 스토리지와 동일하게 모든 자바스크립트 코드를 통해 액세스 할 수 있음으로 XSS 공격에 취약하고 CSRF 공격에 안전하다.

- 쿠키
    - 쿠키는 사실 XSS, CSRF 공격에 모두 취약하다!
    - 하지만, 백엔드에서의 설정을 통해 쿠키에 httpOnly를 사용하면 자바스크립트 코드 상에서 쿠키 접근을 막을수 있고,    SameSite 속성으로 CSRF 공격에 대한 대비, secure로 http 메시지를 암호화하여 탈취공격이 소용없도록 할수 있다.

완벽한 토큰관리 방법은 없는것 같다. private 변수를 사용하면 사용자경험이 악화되고, 스토리지의 경우 XSS공격, 쿠키는 XSS, CSRF에 취약하다. 필자는 프로젝트를 개발할때, refresh 토큰은 쿠키에 저장을 하고, access 토큰의 경우 스토리지에 저장하고 있다.



## 7. 참고자료

[https://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol](https://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol)<br/>
[https://upload.wikimedia.org/wikipedia/commons/0/00/Cookies_Explainied.png](https://upload.wikimedia.org/wikipedia/commons/0/00/Cookies_Explainied.png)<br/>
[https://stackoverflow.com/questions/359434/differences-between-cookies-and-sessions](https://stackoverflow.com/questions/359434/differences-between-cookies-and-sessions)<br/>
[https://cjh5414.github.io/cookie-and-session/](https://cjh5414.github.io/cookie-and-session/)<br/>
[https://developer.mozilla.org/ko/docs/Web/HTTP/Cookies](https://developer.mozilla.org/ko/docs/Web/HTTP/Cookies)<br/>
[https://developer.mozilla.org/ko/docs/Web/HTTP/Headers/Set-Cookie](https://developer.mozilla.org/ko/docs/Web/HTTP/Headers/Set-Cookie)