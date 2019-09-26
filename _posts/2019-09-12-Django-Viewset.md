---
layout: post
title: "DRF Viewset 과 mixin을 활용하여 깔끔하게 모듈화해보기"
author: "DONGWON KIM"
meta: "Springfield"
comments: true
---

## 1. 프롤로그
DRF가 제공하는 Viewset을 활용하면 APIView와 달리 router와 filter class, 인증 클래스와 연계하여 하이레벨틱한 
코딩을 할수 있다. 하지만 풀꽃길 프로젝트를 진행하면서 꽃들을 반환하는 Viewset 로직을 보고 걱정에 휩싸였다.
만약 특수한상황이 발생해서 로직을 재정의해야할일이 다수 발생한다면 Viewset이 더러워질것을 시간문제인듯 싶었다.
쪼개고 쪼개라고 했던가. GenericView와 직접 제작한 mixin들을 상속하여 viewset을 만들면 로직을 역활별로 정리할수 
있다.<br/><br/>

## 2. ViewSet과 GenericViewSet이란 무엇인가?
하나의 모델에 대한 컨트롤러(뷰)들을 하나의 클래스로 모아서 사용할수 있도록 해준다.
ViewSet은 APIView를 상속하여 APIView의 필드와 메소드를 그대로 쓰거나 재정의해서 사용하기때문에 APIView의
작동원리를 그대로 계승한다. <br/>

GenericViewSet는 DRF가 기본으로 제공하는 편리한 제네릭 ViewSet을 제공하기 위한 기반 클래스이다.
ViewSet은 GenericAPIView를 상속하기 때문에 get_queryset, get_object, paginated response등의
기능을 그대로 계승한다.  <br/>

개인적으로  GenericViewSet이나 GenericAPIView는 filter_backend와 paginate 등 기능별로 모듈화가 잘되어있어,
APIview, genericAPIVIew, viewset를 모두 공부 했다면 GenericViewSet를 적절히 고쳐 활용하는것을 추천해주고 싶다.

## 3. ViewSet과 APIView와 다른점은 무엇인가?
##### 1. ViewSet은 Router와 연계하여 편리한 url mapping이 가능하다
##### 2. ViewSet은 APIView와 달리 extra_action을 삽입할수 있다.
##### 3. ViewSet은 APIView와 달리 컨트롤러 메소드의 이름을 http method의 이름으로 사용하지 않는다.


## 2. 시나리오
꽃들에 대한 댓글들을 반환하고, 꽃에 대한 댓글을 작성할수 있는 Viewset을 mixin을 이용해 분리하여
깔끔하게 만들어보자

```python
class _CommentViewSet():
    queryset = Comment.objects.all()
    serializer_class = CommentSerializer
    pagination_class = CommentPaginator

class CommentFlowerViewSet(_CommentViewSet, viewsets.ModelViewSet):
    def get_queryset(self):
        flower = Flower.objects.get(pk=self.kwargs['flower_pk'])
        return Comment.objects.filter(flower=flower).order_by('-created_at')

    def perform_create(self, serializer, flower_pk):
        flower = Flower.objects.get(pk=flower_pk)
        if serializer.is_valid():
            serializer.save(user=self.request.user, flower=flower)
        else:
            raise Exception(serializer.errors)

    def create(self, request, flower_pk):
        serializer = CommentCreateSerializer(data=request.data)
        self.perform_create(serializer, flower_pk)
        return self.list(request)
```
먼저 GenericViewset의 기능을 활용하기 위해 get_object() 함수와
get_serializer_class() 함수를 재정의 해주는 작업이 필요하다. 그리고 상속하는 모듈도 변경해보자
<br/><br/>

```python
class CommentFlowerViewSet(mixins.ListModelMixin,
                           CreateModelMixin,
                           viewsets.GenericViewSet
                          ):
    pagination_class = CommentPaginator

    def get_queryset(self):
        flower = Flower.objects.get(pk=self.kwargs['flower_pk'])
        return Comment.objects.filter(flower=flower).order_by('-created_at')

    def get_serializer_class(self):
        if self.action == 'list':
            return CommentSerializer
        else:
            return CommentCreateSerializer
```
우리의 막강한 기능을 위해 viewsets.GenericViewSet 를 상속 해주어야한다!
댓글들을 읽어오는행위는 따로 변경해줄필요가 없기 때문에 (이미 get_queryset()에서 cover를 쳐서 어헠)
이미 잘만들어진 mixins.ListModelMixin 를 그래도 가져오고 CreateModelMixin를 다음과 같이
우리가 만들어서 상속해주도록 하자.
<br/><br/>

```python
class CreateModelMixin:
    def perform_create(self, serializer, flower_pk):
        flower = Flower.objects.get(pk=flower_pk)
        if serializer.is_valid():
            serializer.save(user=self.request.user, flower=flower)
        else:
            raise Exception(serializer.errors)

    def create(self, request, flower_pk):
        serializer = CommentCreateSerializer(data=request.data)
        self.perform_create(serializer, flower_pk)
        return self.list(request)
```
원래 내가 작성했던 create 로직이다. 그대로 써도 사실 상관이 없지만 genericViewset의 get_queryset()과
get_serializer_class()를 활용해야 하지 않겠는가 호호! 결합성을 낮추어 아래와 같이 수정해보자.
<br/><br/>

```python
from rest_framework import status
from rest_framework.response import Response
from rest_framework.settings import api_settings

from core.models import Flower

class CreateModelMixin:
    def perform_create(self, serializer, flower):
        serializer.save(user=self.request.user, flower=flower)

    def create(self, request, *args, **kwargs):
        flower = Flower.objects.get(pk=kwargs['flower_pk'])
        serializer = self.get_serializer(data=request.data)
        serializer.is_valid(raise_exception=True)
        self.perform_create(serializer, flower)
        self.action = 'list'
        return self.list(request)
```
이렇게 하면 위의 Viewset에서 우리가 맨든 CreateModelMixin을 상속해서 사용할수 있다.
꽃에 대한 댓글들을 읽어오는기능과 댓글을 쓰는 기능이 잘 작동한다. 호호 신난다!
