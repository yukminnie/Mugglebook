# djnago-1



---

### **理解MTV**
**django利用MTV模型参与到网站的请求和回应当中**

**意义：M 数据  T 渲染 V 逻辑**

**总结顺序： P A D M V T U**
```
1.创建站点

django-admin startproject firstsite
#!/usr/bin/env python3   --设置兼容

2.创建app

python manage.py startapp firstapp

'firstapp',   ---添加app到setting

3.创建数据库，并运行

python manage.py migrate
python manage.py runserver

4.Model中创建表

from django.db import models

class People(models.Model):         ---此处所有的表类都继承自models.Model
    name = models.CharField(null=True, blank=True, max_length=200)
    job = models.CharField(null=True, blank=True, max_length=200)
    ---null  数据库中暂时没有此数据无碍  ---blank 可空 
    
python manage.py makemigrations   数据库同步
python manage.py migrate          数据库合并

5.视图函数

from django.shortcuts import render, HttpResponse            ---引入渲染函数，http回复
from firstapp.models import People                           ---引入Model模块
from django.template import Context, Template                ---引入模板函数和上下文函数

--- templete模式有三种   模板变量  模板标签  模板过滤器
# Create your views here.
def first_try(request):
    person = People(name='Spock', job='officer')
    html_string = '''
        <html>
            <head>
                <meta charset="utf-8">
                <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/semantic-ui/2.2.6/semantic.css" media="screen" title="no title">
                <title>firstapp</title>
            </head>
            <body>
                <h1 class="ui center aligned icon header">
                    <i class="hand spock icon"></i>
                    Hello, {{ person.name }}
                </h1>
            </body>
        </html>

    '''

    t = Template(html_string)
    c = Context({'person':person})  ---'person'对应模板名称  person对应数据库变量
    web_page = t.render(c)

    return HttpResponse(web_page)   ---视图函数都会返回一个response对象，此处将网页变为http对象

6.设置网址

from django.conf.urls import url
from django.contrib import admin
from firstapp.views import first_try    ---引入视图函数

urlpatterns = [
    url(r'^admin/', admin.site.urls),
    url(r'^first_try/', first_try),    ---^之前代表服务器地址，first_try/代表我们分配的网址，first_try代表对应的视图函数，将来我们还会给对应的网址起名字
]

```





