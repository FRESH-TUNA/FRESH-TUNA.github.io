---
layout: post
title: "Django response, reverse"
# author: "DONGWON KIM"
# meta: "Springfield"
categories: "Django"
comments: true
---

## 1. HttpResponse
모든 view는 요청에 대해 httpResponse를 반환하도록 설계해야 한다.

```python
return HttpResponse("Here's the text of the Web page.")
return HttpResponse("Text only, please.", content_type="text/plain")
return HttpResponse(b'Bytestrings are also accepted.')
```

```python
response = HttpResponse()
response.write("<p>Here's the text of the Web page.</p>")
response.write("<p>Here's another paragraph.</p>")
return response
```

다음과 같이 딕셔너리를 활용하여 response의 header도 설정하거나
attributes 접근을 통해 status_code를 바꿀수도 있다.
```python
response['Age'] = 120
response.status_code = 201
```

일반적으로 HttpResponse 보다 HttpResponse를 상속한 모듈을 자주 사용한다.
```python
return HttpResponseRedirect('/home')         #풀path, 절대경로, 상대경로 모두 가능 #301
return HttpResponsePermanentRedirect('/home') #풀path, 절대경로, 상대경로 모두 가능 #302
return HttpResponseRedirect(reverse('home')) #name을 home으로 햇을때 사용
```

restful한 api를 만들기 위해선 HttpResponse를 상속한 JsonResponse를 사용한다.
```python
return JsonResponse({'foo': 'bar'})
```

## 2. Reverse
reverse 기능을 활용하면 url을 하드코딩하지 않고 이름으로 추상화하여 다룰수 있다.
```python
return HttpResponseRedirect(reverse('flowers', args=[2010, 2011]))
return HttpResponseRedirect(reverse('flowers', kwargs={'purpose': 'medicine'}))
```