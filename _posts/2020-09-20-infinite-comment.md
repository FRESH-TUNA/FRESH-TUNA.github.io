---
layout: post
title: "무한 대댓글 구현하기"
author: "DONGWON KIM"
meta: "Springfield"
categories: "Django"
comments: true
---

## 1. 개요
미국 유명 커뮤니티 사이트인 reddit의 무한 대댓글을 모방하여 구현하는 과정을 정리했다.
<br><br>

## 2. 모델
```python
import os
from django.db import models
from django.conf import settings

class Post(models.Model):
    content = models.TextField(null=False)
    image = models.ImageField(null=True, upload_to="lostpost")
    found = models.BooleanField(default=False)      # 주웠어요=True, 잃어버렸어요=False
    password = models.CharField(null=False, max_length=50)
    created_at = models.DateTimeField(auto_now_add=True)

    class Meta:
        ordering=['-created_at']

    def __str__(self):
        return ("{}").format(self.content)

class Comment(models.Model):
    # 댓글은 반드시 게시물 대상의 외래키를 가진다.
    post = models.ForeignKey(
        'Post', 
        on_delete=models.CASCADE, 
        related_name='comments', 
        blank=True,
    )

    # 댓글은 자신의 부모 댓글 대상의 외래키를 가질수도 있다.
    parent = models.ForeignKey('Comment', on_delete=models.CASCADE, related_name='parent_comments', null=True)
    content = models.TextField(null=False)

    # depth를 제한할 필요성이 있을때 사용한다. 꼭 제한 하지 않더라고 depth를 쓰면 유용할때가 많다.
    active = models.BooleanField(default=True)
    depth = models.IntegerField(default=0)
    created_at = models.DateTimeField(auto_now_add=True)
    password = models.CharField(max_length=50)

```
<br><br>

## 3. API 설계
```sh
# 특정 게시글의 댓글들: GET
# 특정 게시글에 댓글 생성: POST
/posts/<post_pk>/comments

# 특정 댓글의 댓글 생성: POST
/comments/<parent_comment_pk>/comments

# 특정 댓글 삭제: DELETE
/comments/<comment_pk>
```
<br><br>

## 4. 프론트엔드에 댓글들 띄우기
### 1. 부모 컬럼이 null 인 댓글들을 기준으로 탐색을 시작한다.
### 2. 부모 컬럼을 기준으로 깊이 우선 탐색을 시행하면 된다.
### 3. depth 컬럼을 UI 표시에 활용한다.
<br><br>

## 5. 향후 발전시키기
댓글이 너무 많으면 백엔드와 프론트에 부하가 생길수 있으므로, 한정된 depth와 댓글들을 읽어오고
더 보고 싶으면 또 요청을 보내는식으로 구현하면 좋을것 같다.
