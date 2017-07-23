# django-4



---
# get实现文章分类，主要思路：反推

### **Model层需要多少数据字段**

定义分类字段
```
class Article(models.Model):
    headline = models.CharField(null=True, blank=True, max_length=500)
    content = models.TextField(null=True, blank=True)
    TAG_CHOICES = (                     #元组结构，tech参数的值，Tech是后台的值
    ('tech','Tech'),
    ('life','Life'),
    )
    tag = models.CharField(null=True, blank=True, max_length=5, choices=TAG_CHOICES)
    def __str__(self):
        return self.headline
```
数据库同步

```
python manage.py makemigrations
python manage.py migrate
```
### **View层根据什么请求返回什么结果**

```
可以通过dir(request)来打印出其所有的方法和属性
可以通过type(request)来打印其类型

from django.shortcuts import render, HttpResponse
from firstapp.models import People, Article

def index(request):
    queryset = request.GET.get('tag')
    if queryset:
        article_list = Article.objects.filter(tag=queryset)
    else:
        article_list = Article.objects.all()
    context = {}
    context['article_list'] = article_list
    index_page = render(request, 'first_web_2.html', context)
    return index_page

```

### **Templete层交互**

设置交互
```
<a class="item" href="?tag=life">life</a>
<a class="item" href="?tag=tech">tech</a>
```

tag模板标签
```
{% for article in article_list %}
                <div class="ui container vertical segment">
                    <a href="#">
                        <h1 class="ui header">
                            {{ article.headline }}
                        </h1>
                    </a>

                    <i class="icon grey small unhide">10,000</i>
                    <p>
                        {{ article.content|truncatewords:100 }}
                        <a href="#">
                            <i class="angle tiny double grey right icon">READMORE</i>
                        </a>
                    </p>

                    <div class="ui mini  tag label">
                        {{ article.tag }}
                    </div>
                </div>
            {% endfor %}
```





