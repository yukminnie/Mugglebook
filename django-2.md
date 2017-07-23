# django-2



---
#实现第一个django网站

### **理解上下文**

```
render（request,x.html,context） -----将数据库中的数据和网页中要显示的数据进行关联
```

### **创建模板**   

建立专门存放模板的文件夹，和存放静态文件的文件夹,
```
app/templates
app/static
```
告诉项目模板目录
```
'DIRS': [os.path.join(BASE_DIR, 'templates').replace('\\', '/')]  
```
用模板标签，给模板引入静态文件

```
静态文件最终都会以304重定向的方式输出

<!DOCTYPE html>
{% load staticfiles %}

<link rel="stylesheet" href="{% static 'css/semantic.css' %}"  media="screen" title="no title" charset="utf-8">

background: url({% static 'images/star_banner.jpg' %});
```
### **创建后台**

创建超级管理员

```
python manage.py createsuperuser
```

Model注册到管理后台，实现显示

```
from django.contrib import admin
from firstapp.models import People, Article

admin.site.register(People)
admin.site.register(Article)

注：此时并不需要合并数据库
```
修改后台名称显示,并同步数据库

```
from django.db import models


class People(models.Model):
    name = models.CharField(null=True, blank=True, max_length=200)
    job = models.CharField(null=True, blank=True, max_length=200)
    def __str__(self):            -----后台名称修改
        return self.name

class Article(models.Model):
    headline = models.CharField(null=True, blank=True, max_length=500)
    content = models.TextField(null=True, blank=True)
    def __str__(self):            -----后台名称修改
        return self.headline
        
注：此时修改了Model，需要进行同步数据库

python manage.py makemigrations
python manage.py migrate
```

### **引入数据**

Model中正确引入数据，通过view函数，将其表现到模板中

```
from django.shortcuts import render, HttpResponse
from firstapp.models import People, Article

def index(request):
    context = {}                             ---创建上下文字典
    article_list = Article.objects.all()     ---数据库中取数据到数据列表(不加分辨，取所有数据)
    context['article_list'] = article_list   ---利用上下文字典装载数据
    index_page = render(request, 'first_web_2.html', context)  ---将装载的数据传入到渲染函数render
    return index_page

```
### **模板语言**
设置完模板语言

```
{% for article in article_list %}       ---模板标签
                <div class="ui container vertical segment">
                    <a href="#">
                        <h1 class="ui header">
                            {{ article.headline }}     ---模板变量
                        </h1>
                    </a>

                    <i class="icon grey small unhide">10,000</i>
                    <p>
                        {{ article.content|truncatewords:100 }}    ---模板过滤器
                        <a href="#">
                            <i class="angle tiny double grey right icon">READMORE</i>
                        </a>
                    </p>

                    <div class="ui mini  tag label">
                        life
                    </div>
                </div>
            {% endfor %}
```
最后添加url

```
from django.conf.urls import url
from django.contrib import admin
from firstapp.views import first_try, index

urlpatterns = [
    url(r'^admin/', admin.site.urls),
    url(r'^first_try/', first_try),
    url(r'^index', index, name='index'),
]

```


