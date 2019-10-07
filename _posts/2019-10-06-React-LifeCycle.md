---
layout: post
title: "리액트 라이프사이클"
author: "DONGWON KIM"
meta: "Springfield"
categories: "데이터베이스"
comments: true
---

## 1. 생명주기
리액트에서 아용되는 모든 컴포넌트들은 라이프사이클을 따라서 통해 생성, 업데이트, 제거 된다.
개발시 생명주기에 대해 이해하는것은 중요하다고 생각하여 이번기회에 정리해보고자 한다.
그리고 개발하면서 새로 알은 지식이 있디면 추가적으로 수정해서 보완해 나가려 한다.

## 2. Render 단계
이때는 React에 의해 컴포넌트가 DOM에 탑재되는 과정이므로, DOM으로 컴포넌트를 접근하여 
제어하는 행위가 불가능하다.
<br/> 
![Image Alt 텍스트](/img/2019/04/28/Django-101/django.png)

django는 python으로 작성된 backend web framework의 일종이다. 프레임워크를 사용하여 웹을 구축하는 이유는 무엇일까? 웹 개발자들이 새로운 웹사이트를 개발시 항상 유사한 로직이 필요하게 된다. (예: 폼, 회원가입...) 
이런 유사한 로직들을 빠르게 개발할수 있는 프레임워크를 개발하게 된것이다!

유사 프레임워크로 Rails(ruby), spring(java), ASP.NET(C#) 등이 사용된다.