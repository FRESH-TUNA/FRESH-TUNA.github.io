---
layout: post
title: "Django 시리얼라이저"
author: "DONGWON KIM"
meta: "Springfield"
categories: "Django"
comments: true
---

## 1. 프롤로그
Django DRF의 꽃(?)인 시리얼라이저에 대해 연구해본 흔적을 남겨보려 한다.
<br><br>

## 2. Serializer
### 시나리오 1
flowers/<int:pk>의 'GET 접근했을때 꽃에 대한 자세한 정보를 반환하는 시리얼라이저를 구현하고자 한다.

```python
class FlowerDetailSerializer(serializers.Serializer):
    id = serializers.IntegerField()
    name = serializers.CharField()
    description = serializers.CharField()
    season = serializers.IntegerField()
    star = serializers.FloatField()
    purposes = serializers.PrimaryKeyRelatedField(many=True, read_only=True)
    languages = serializers.PrimaryKeyRelatedField(many=True, read_only=True)
    colors = serializers.PrimaryKeyRelatedField(many=True, read_only=True)
    images = serializers.PrimaryKeyRelatedField(many=True, read_only=True)
```
![Image Alt 텍스트](/img/2019/08/25/serializer/simple.png)

이렇게 하면 purposes, languages. colors, images의 다대다 관계를 가진 필드들이
serializers.PrimaryKeyRelatedField() 를 통과하면서 기본키로 바뀌어서 반환된다.

다대다모델은 여러개의 값이 존재할수 있으므로 many=True로 설정하여야 하며, 일단 꽃에 대한 데이터를 읽어들여야 하기때문에
read_only=True로 설정하였다. 다대다모델을 create, update할때는 다음에 공부해서 다루어보겠다.

하지만 꽃의 데이터뿐만아니라 purposes, languages. colors, images에 대한 자세한 정보가 필요하기때문에 
코드를 아래와 같이 고쳐서 써보자


```python
class FlowerDetailSerializer(serializers.Serializer):
    id = serializers.IntegerField()
    name = serializers.CharField()
    description = serializers.CharField()
    season = serializers.IntegerField()
    star = serializers.FloatField()
    purposes = PurposeSerializer(many=True)
    languages = LanguageSerializer(many=True)
    colors = ColorSerializer(many=True)
    images = ImageSerializer(many=True)

class ImageSerializer(serializers.ModelSerializer):
    class Meta:
        model = Image
        fields = ['url']


class ColorSerializer(serializers.ModelSerializer):
    class Meta:
        model = Color
        fields = '__all__'


class LanguageSerializer(serializers.ModelSerializer):
    class Meta:
        model = Language
        fields = '__all__'


class PurposeSerializer(serializers.ModelSerializer):
    class Meta:
        model = Purpose
        fields = ['name', 'image']
```
![Image Alt 텍스트](/img/2019/08/25/serializer/detail.png)

이러한 방식을 nested serializer라고 한다. 대대다 모델의 값들이 각각의 시리얼라이저를 통과하면서
purposes, languages. colors, images에 대한 자세한 정보를 출력할수 있게 되었다.
다대다 모델은 값이 여러개가 될수 있기 때문에 many=True로 설정해주자
<br><br>

### 시나리오 2
flowers/flower_pk/comments에 'GET' 으로 접근했을때 접속한 유저가 댓글에 좋아요를 했는지의 여부, 이 댓글의 소유자인지의 여부를 보함한
댓글의 정보를 반환하게 한다.

```python
class CommentSerializer(serializers.Serializer):
    id = serializers.IntegerField()
    user = serializers.IntegerField()
    content = serializers.CharField()
    star = serializers.CharField()
    created_at = serializers.DateTimeField()

    is_like = serializers.SerializerMethodField('get_is_like')
    is_owner = serializers.SerializerMethodField('get_is_owner')
    user = UserNicknameSerializer(read_only=True)
    flower = CommentFlowerSerializer(read_only=True)

    def get_is_like(self, obj):
        if self.context['request'].user.is_authenticated:
            
            like_comment = CommentLike.objects.all().filter(
                comment=obj, user=self.context['request'].user).first()
            if like_comment:    
                return True
        return False
    
    def get_is_owner(self, obj):
        if obj.user == self.context['request'].user:
            return True
        else:
            return False

```

- serializers.SerializerMethodField를 이용하면 사용자가 정의한 메소드의 반환값을 serializer.data에 추가할수 있다.
serializers.SerializerMethodField()의 인자론 사용자가 정의한 함수의 이름이 들어간다. 
사용자 정의 함수는 serializer에 대한 참조 self, model 객체에 대한 참조 obj가 들어간다. self를 통해 serializer의 context에
접근할수 있고 obj을 통해 모델의 데이터에 접근할수 있다.
<br><br>


### 시나리오 3
flowers/flower_pk/comments에 'POST' 'PUT' 으로 접근했을때 댓글을 create, update 한다.

```python
class CommentCreateSerializer(serializers.Serializer):
    content = serializers.CharField(max_length=200, allow_blank=True)
    star = serializers.FloatField(validators=[MinValueValidator(0.0), MaxValueValidator(5.0)], default=0.0)

    def create(self, validated_data):
        return Comment.objects.create(**validated_data)
    
    def update(self, instance, validated_data):
        instance.content = validated_data['content']
        setattr(instance, 'star', validated_data['star'])
        return instance
```
- serializers.Serializer의 경우 에는 create, update를 하기위해선 create(), update()를 위와 같이 재정의
해주어야만 한다. 이번에 새롭게 알았는데 setattr()을 통해 딕셔너리 타입의 값을 설정하는게 가능하다.
