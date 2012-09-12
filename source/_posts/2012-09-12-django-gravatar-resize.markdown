---
layout: post
title: "django自定义gravatar头像大小"
date: 2012-09-12 23:51
comments: true
categories: web
---

Django可以很方便的集成gravatar，官方教程<a href="https://en.gravatar.com/site/implement/images/django/" target="_blank">在这</a>
就是自定义一个**template tag**，在模板里加载使用就行了。

给的代码是只有一个默认尺寸，把代码稍微改了以下，可以添加一个尺寸参数，如果没有提供第三个参数，默认是32.

``` python
     
    from django import template
    import urllib, hashlib

    register = template.Library()

    class GravatarUrlNode(template.Node):
        def __init__(self, email, size=32):
            self.email = template.Variable(email)
            self.size = size
        def render(self, context):
            try:
                email = self.email.resolve(context)
            except template.VariableDoesNotExist:
                return ''

            default = "http://www.gravatar.com/avatar/00000000000000000000000000000000?d=mm&s=%s" % self.size

            gravatar_url = "http://www.gravatar.com/avatar/" + hashlib.md5(email.lower()).hexdigest() + "?"
            gravatar_url += urllib.urlencode({'d':default, 's':str(self.size)})

            return gravatar_url

    @register.tag
    def gravatar_url(parser, token):
        try:
            tag_name, email ,size = token.split_contents()
        except ValueError:
            try:
                tag_name,email = token.split_contents()
            except ValueError:
                raise template.TemplateSyntaxError, "useage:%r email [size]" % token.contents.split()[0]
            return GravatarUrlNode(email)
       
        return GravatarUrlNode(email,size) 
```
