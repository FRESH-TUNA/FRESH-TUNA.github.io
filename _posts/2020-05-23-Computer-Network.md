---
layout: post
title: "Computer Network"
author: "DONGWON KIM"
meta: "Springfield"
categories: "network"
comments: true
---

## 1. Internet
인터넷은 전세계에 걸친 수백만대의 컴퓨터들의 데이터통신을 위한 네트워크이며 분산 어플리케이션을 위한 infrastructure를 의미한다. 인터넷의 edge 부분에 연결되어있는 컴퓨터들을 host나 endsystem 이라고 부른다. endsystem들은 네트워크의 링크와 패킷스위칭을 통하여 데이터 교환을 한다. 링크는 케이블, 구리선, 광섬유, 전파 대역 등으로 구현된다. endsystem들은 ISP를 통해 인터넷을 접근한다. ISP는 케이블 모뎀, 무선등의 기법을 활용하여 endsystem들이 인터넷에 접근할수 있도록 도와준다. host들은 첫번째로 연결되어있는 라우터를 통해 인터넷연결을 시작하는데 이를 access network라고 한다. 가정에선 DSL, Cable, FTTH, Dial-Up, 위성 등의 access network를 사용한다. 

endsystem들에게 직접 인터넷에 접속할수 있게 해주는 ISP를 access ISP라고 한다. 이 access ISP들이 얽히고 얽혀서 인터넷을 구성할수도 있지만 access ISP만 사용하면 너무 구조가 mess해지는 단점이 있기 땨문애 국가마다 운영되는 regional ISP에 연결되고 regional ISP는 글로벌 대기업이 운영하는 tier1 ISP와 연결되어 계층적으로 구성된다.
최근에는 POP(customer ISP가 provider ISP에 연결을 위한 라우터 그룹), multi-homing(2개이상의 provider ISP에 연결하는 기법), peering(같은 레벨에 있는 ISP들이 직접 연결을 통해 비용을 절감하는 기법), IXP(peering을 위핸 exchange point)와 거대 그룹들의 content provider network를 통해 하위 ISP에 직접 서비스를 제공하는등의 기법으로 5세대 네트워크를 구성한다.

## 2. Protocol
프로토콜이란 두개이상의 통신장치 사이에 오가는 메시지의 포멧과 순서를 정의한것이다. 

## 3. packet switching
endsystem들이 서로 메시지를 교환하기위해선 message를 packet의 작은 단위로 쪼개야한다. 모든 패킷들은 2계층의 스위치나 3계층의 라우터들에 의해 전송된다. 모든 패킷들은 자신의 패킷의 모든 데이터가 모이기 전까지는 다음 노드로 전송되지 않는다. 패킷 스위칭시 L bit의 데이터를 rate가 R인 N개의 링크를 사용했을때 전송지연은 다음과 같다.

$$d = N {L \over R}$$

## 4. Packet의 명칭
7계층에서는 메시지, 4계층에서는 세그먼트, 3계층에서는 데이타그램, 2계층에서는 프레임이라 부른다.

