---
layout: post
title: "Django Router 사용하기"
author: "DONGWON KIM"
meta: "Springfield"
comments: true
---

## 1. Django Router 란
Django rest framework를 사용하면 Json object로 decode하거나 object를 json으로 encode 해야하는
수고를 덜어준다. django-router는 drf가 제공하는 viewset과 연계하여 url의 하드코딩을 막을수 있는 신박한녀석
이라 볼수 있다.

## 2. 기본적인 사용법
```python
router = DefaultRouter(trailing_slash=False)
router.register(r'flowers', FlowerViewSet, basename='flower')
router.register(r'purposes', PurposeViewSet, basename='purpose')
router.register(r'colors', ColorViewSet, basename='color')
urlpatterns = router.urls
```
router.register 첫번째 인자인 prefix는 url에 들어갈 자원의 이름으로 지정하면 되고
두번째 인자로는 viewset을 넣어주면 된다. 세번째 인자의 경우 django에서사용하는 path name 처럼 
view에 이름을 지정해줄수 있다. 공란으로 남겨두면 viewset을 참고하여 자동으로 생성되지만, viewset에
queryset이 반드시 지정되어아한다.
trailing_slash 옵션을 통해서 url의 끝에 / 를 붙일지 말지 결정할수 있다.

```python
urlpatterns = [
    url(r'^api/', include((router.urls, 'app_name'))),
]
```
이런식으로 application 스페이스를 주는것이 가능하다. 이는 나중에 view_name='app_name:user-detail'
식으로 하이퍼링크 시리얼라이저에 사용된다.

## 3. NestedSimpleRouter
만약 flowers/1/comments 처럼 nested 되어있을때는 어떻게 해야할까
나는 drf-nested-routers 라는 서드파티 라이브러리를 이용하여 이문제를 해결했다.

```python
comments_router = NestedSimpleRouter(router, r'flowers', lookup='flower')
comments_router.register(r'comments', CommentFlowerViewSet, base_name='flower-comments')
```
NestedSimpleRouter의 첫번째 인자로는 부모 router를 집어넣고, 2번째인자로 prefix을 집어넣는다.
3번째인자의 lookup을 살펴보자. 지금까지 확인한바로는 lookup을 'yes'로 지정하면 viewset에서 'yes_pk' 형태로 
받을수 있어 다음과 같이 쓸수 있다.

```python
def get_queryset(self):
    flower = Flower.objects.get(pk=self.kwargs['yes_pk'])
    return Comment.objects.filter(flower=flower).order_by('-created_at')
```
register 시에는 첫번째로 인자로 자식의 prefix, 두번째로 viewset, 세번째인자로 함수의 이름을 지정한다.
