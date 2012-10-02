---
layout: post
title: "Django集成uploadify"
date: 2012-09-14 09:10
comments: true
categories: web
---
**django 1.4 ;uploadify 3.1**

uploadify实现多文件上传很简单，一个js，一个css和一个swf就行了。不过要想在Django里使用flash上传文件，还得动点手脚才行。

问题就出<a href="http://www.uploadify.com/documentation/uploadify/using-sessions-with-uploadify/" target=_blank>在这</a>,使用flash无法传递cookies到服务端,
服务端程序也就无法验证上传者的身份，如果在视图函数(views)上加了**@login_required**,请求就会一直被重定向，然后uploadify得到的就是一堆http 302或者IO error.

上面的链接里已经指出了问题并给了解决方法，但那是针对php的，django要麻烦的多，当然有个不是办法的办法，就是不用@login_required，这样不需要验证就能上传文件了...

既然没有cookies，那就需要手动添加cookies，Django的middleware正好适合干这事，uploaidfy的初始化选项里有个**formData**可以用来post数据，把要设置的cookies值，放在这里，
然后自定义一个middleware在**process_requrest()**里构造自己需要的cookies。

   一个小问题:django 1.4里的session cookies默认都有httponly选项，无法用js脚本获得这个值,还需要一个可以获得的sessionid
<!-- more -->
总结一下:

1. 在视图函数里手动建立一个没有httponly选项的sessionid，值和浏览器里的**sessionid**一样

2. 在uploadify初始化里传递必要的参数

3. 自定义一个middleware，在其中构造需要的cookies,自定义的middleware要放在settings.py里SessionMiddleware的前面


```python views.py
upload_sessionid=request.COOKIES['sessionid']
...
response.set_cookie('upload_sessionid',upload_sessionid,httponly=False)
return response
```

```javascript main.js
$(function() {
   $('#file_upload').uploadify({
        'swf'      : '/static/uploadify/uploadify.swf',
        'uploader' : '', // 设置为空，则为当前url
        // Put your options here
        'fileTypeExts':'*.jpg;*.bmp;*.png',
        'formData':{'upload_sessionid':$.cookie('upload_sessionid'),'selected_album':$('.selected_data').val()},
        'multi':true,
        'auto':true,
        'buttonText':'添加照片',
        'height':21,
        'width':81,
    });
});
```

```python middleware.py
 from django.conf import settings
 
 class UploadifyMiddleware(object):
    def process_request(self,request):
        if request.method == 'POST' and request.POST.has_key('upload_sessionid'):
            request.COOKIES[settings.SESSION_COOKIE_NAME] = request.POST['upload_sessionid']
```

```python settings.py
MIDDLEWARE_CLASSES = (
    'django.middleware.common.CommonMiddleware',
    'albums.middleware.UploadifyMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    ...
```

这样应该就没问题了，不过视图函数还是要加上**@csrf_exempt**不然会出现403错误，我试着用同样的方法设置了csrftoken的cookies，不过貌似不起什么作用，先这样用着了。
