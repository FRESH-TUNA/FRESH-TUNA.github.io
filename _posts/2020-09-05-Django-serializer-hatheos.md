---
layout: post
title: "HATEOAS를 고려한 Django Serialzier 사용"
author: "DONGWON KIM"
meta: "Springfield"
categories: "Django"
comments: true
---

## 1. HATHEOS
클라이언트가 서버에 요청을 할때, 서버에서 미리 받아온 URL을 사용하는 기법 
<br><br>

## 2. 예제 1
Django rest framework 가 제공해주는 HyperlinkedIdentityField 를 사용한다. 다음예제는 요청결과를 반환할때 게시글의 리스트를 반환할때 각각의 게시글로 이동할수 있는 url을 제공한다.

```python
from rest_framework import serializers
from lostboard.models import Post, Comment

class PostsListSerializer(serializers.ModelSerializer):
    # inline class
    class CommentsSerializerMethodField(serializers.SerializerMethodField):
        def to_representation(self, value):
            method = getattr(self.parent, self.method_name)
            return {'count': method(value)}

    comments = CommentsSerializerMethodField('get_comments_count')
    url = serializers.HyperlinkedIdentityField(view_name='lostboard:posts-detail')

    class Meta:
        model = Post
        fields = '__all__'
        extra_kwargs = {
            'password': {'write_only': True},
        }

    def get_comments_count(self, obj):
        return obj.comments.count()
```

```json

  "links": {
    "next": null,
    "previous": null,
    "first": "http://localhost:8000/lostboard/posts.json?page=1",
    "last": "http://localhost:8000/lostboard/posts.json?page=1",
    "pages": [
      {
        "index": 1,
        "url": "http://localhost:8000/lostboard/posts.json?page=1"
      }
    ]
  },
  "index": 1,
  "count": 1,
  "posts": [
    {
      "id": 11,
      "comments": {
        "count": 4
      },
      "url": "http://localhost:8000/lostboard/posts/11.json",
      "content": "못생김",
      "image": null,
      "found": true,
      "created_at": "2019-09-17T21:35:47.425046+09:00"
    }
  ]
}
```

## 2. 예제 2
댓글들을 읽어올때 각 댓글의 대댓글들을 읽어올수 있는 url과 수정과 삭제를 위해 필요한 자기 자신을 나타내는 url을 포함해서 읽어올수 있게 해주는 예제이다.

```python
from rest_framework import serializers
from lostboard.models import Comment

class CommentsListSerializer(serializers.ModelSerializer):
    # view_name: 대댓글들을 요청할수 있는 view_name
    # lookup_url_kwarg: view_name을 통해 얻은 url에 필요한 인자의 키
    # lookup_field: view_name을 통해 얻은 url에 필요한 인자
    comments = serializers.HyperlinkedIdentityField(
        view_name='lostboard:comments_comments-list',
        lookup_url_kwarg='comment_pk',
        lookup_field='pk'
    )

    url = serializers.HyperlinkedIdentityField(view_name='lostboard:comments-detail')

    class Meta:
        model = Comment
        fields = '__all__'
        extra_kwargs = {
            'password': {'write_only': True},
        }
```

```json
// http://localhost:8000/lostboard/posts/11/comments.json

[
  {
    "id": 23,
    "comments": "http://localhost:8000/lostboard/comments/23/comments.json",
    "url": "http://localhost:8000/lostboard/comments/23.json",
    "content": "호호d",
    "active": true,
    "depth": 0,
    "created_at": "2020-09-15T03:45:36.139793+09:00",
    "post": 11,
    "parent": null
  },
  {
    "id": 90,
    "comments": "http://localhost:8000/lostboard/comments/90/comments.json",
    "url": "http://localhost:8000/lostboard/comments/90.json",
    "content": "1234",
    "active": true,
    "depth": 1,
    "created_at": "2020-09-20T23:45:49.664816+09:00",
    "post": 11,
    "parent": 23
  },
  {
    "id": 94,
    "comments": "http://localhost:8000/lostboard/comments/94/comments.json",
    "url": "http://localhost:8000/lostboard/comments/94.json",
    "content": "1234",
    "active": true,
    "depth": 0,
    "created_at": "2020-09-21T16:28:55.898952+09:00",
    "post": 11,
    "parent": null
  },
  {
    "id": 95,
    "comments": "http://localhost:8000/lostboard/comments/95/comments.json",
    "url": "http://localhost:8000/lostboard/comments/95.json",
    "content": "1234",
    "active": true,
    "depth": 1,
    "created_at": "2020-09-21T16:29:00.541315+09:00",
    "post": 11,
    "parent": 94
  }
]
```

## 2. 예제 3
직접 production에 사용하지는 않았지만 특정 포스트의 특정 댓글의 정보를 읽어올때 자기자신을 나타내는 url을 포함시키는 기법을 적용한 예제이다.
drf-nested-router의 NestedHyperlinkedIdentityField 를 사용했다.

```python
from rest_framework import serializers
from rest_framework_nested.relations import NestedHyperlinkedIdentityField
from lostboard.models import Comment

class CommentsDetailSerializer(serializers.ModelSerializer):
    # post_pk: url에서 받는 인자, 'post__pk': parent model의 기본키 접근을 위한 ORM 문법
    url = NestedHyperlinkedIdentityField(
        view_name='lostboard:posts_comments-detail',
        parent_lookup_kwargs={'post_pk': 'post__pk'}
    )

    class Meta:
        model = Comment
        fields = '__all__'
        extra_kwargs = {
            'password': {'write_only': True},
        }

```

```json
// http://localhost:8000/lostboard/posts/11/comments/23.json

{
  "id": 23,
  "url": "http://localhost:8000/lostboard/posts/11/comments/23.json",
  "content": "호호d",
  "active": true,
  "depth": 0,
  "created_at": "2020-09-15T03:45:36.139793+09:00",
  "post": 11,
  "parent": null
}
```
