# djnago-7 



---

# 分页器，分类模块，图片上传

假数据生成

```
from django.db import models
# from faker import Factory                        #假数据库
from django.contrib.auth.models import User

class Video(models.Model):
    title = models.CharField(null=True, blank=True, max_length=300)
    content = models.TextField(null=True, blank=True)
    url_image = models.URLField(null=True, blank=True)    #这里使用的网络图片，URLField
    cover = models.FileField(upload_to='cover_image', null=True)     #上传图片字段   
    editors_choice = models.BooleanField(default=False)     #这是使用的文章分类，BooleanField

    def __str__(self):
        return self.title
        

# f = open('/Users/Hou/Desktop/web_url.txt', 'r')        #假数据生成
# fake = Factory.create()
# for url in f.readlines():
#     v = Video(
#         title=fake.text(max_nb_chars=90),
#         content=fake.text(max_nb_chars=3000),
#         url_image=url,
#         editors_choice=fake.pybool()
#         )
#     v.save()

```
---
分页

```
from django.shortcuts import render, Http404, redirect, HttpResponse
from website.models import Video, Ticket
from django.core.paginator import Paginator, EmptyPage, PageNotAnInteger  #分页机器人自带异常处理
from django.contrib.auth import authenticate, login
from website.form import LoginForm
from django.contrib.auth.forms import UserCreationForm, AuthenticationForm
from django.core.exceptions import ObjectDoesNotExist



def listing(request, cate=None):               #添加了cate分类参数
    context = {}
    if cate is None:                                   #分类的判断，和get文章分类
        vids_list = Video.objects.all()
    if cate == 'editors':
        vids_list = Video.objects.filter(editors_choice=True)
    else:
        vids_list = Video.objects.all()

    page_robot = Paginator(vids_list, 9)               #分页机器人：数据，每页个数
    page_num = request.GET.get('page')                 
    
    try:
        vids_list = page_robot.page(page_num)          #分页机器人异常处理
    except EmptyPage:
        vids_list = page_robot.page(page_robot.num_pages)
        # raise Http404('EmptyPage!')
    except PageNotAnInteger:
        vids_list = page_robot.page(1)

    context['vids_list'] = vids_list                   #渲染
    return render(request, 'listing.html', context)
```
---
url添加

```
from website.views import listing

urlpatterns = [
    url(r'^admin/', admin.site.urls),
    url(r'^list/$', listing, name='list'),


]


```
---
异常处理

 - 超出范围
 - 输入类型错误




```

from django.core.paginator import Paginator, EmptyPage, PageNotAnInteger  #分页机器人自带异常处理


    try:
        vids_list = page_robot.page(page_num)          #取页
    except EmptyPage:
        vids_list = page_robot.page(page_robot.num_pages)   #不存在的页码，加载最后一页
        # raise Http404('EmptyPage!')                       #返回404
    except PageNotAnInteger:
        vids_list = page_robot.page(1)     #非整数，返回第一页

```
---

template层实现分页按钮

- 按钮状态判断

```
    <div class="ui center aligned very padded vertical segment container">
        <div class="ui pagination menu">      #这里是前端分页部分
            {% if vids_list.has_previous %}           #判断有没有前页
                <a href="?page={{ vids_list.previous_page_number }}" class="item"> #返回上一页
                    <i class="icon red left arrow"></i>      #page会跳转到url后面
                </a>
            {% else %}
                <a href="?page={{ vids_list.start_index }}" class="disabled item"> #返回第一页
                    <i class="icon  left arrow"></i>         #返回第一页后变成不可点击状态
                </a>                                        #此时class为disabled状态
            {% endif %}

            {% if  vids_list.has_next %}              #判断有没有后页
                <a href="?page={{ vids_list.next_page_number }}" class="item">  #返回后一页
                    <i class="icon red right arrow"></i>
                </a>

            {% else %}

                <a href="?page={{ vids_list.end_index }}" class="disabled item">  #返回最后一页
                    <i class="icon  right arrow"></i>         #按钮状态改变
                </a> 
            {% endif %}




        </div>

```
文章分类

url混合
```
from website.views import listing, index_login, index_register, detail, detail_vote

urlpatterns = [
    url(r'^admin/', admin.site.urls),
    url(r'^list/$', listing, name='list'),
    url(r'^list/(?P<cate>[A-Za-z]+)$', listing, name='list'),  #同种名字，django会一直向下找寻
  
# 分类和页数都是使用了，模板传递url参数，然后view层接收url分类参数，从而实现分类；使用get实现文章分类一章使用模板层直接添加了url后缀，通过request的get方法获取参数，没有定义url。
]
```

view实现逻辑

```
def listing(request, cate=None):
    context = {}                                   #文章分类代码实现在分页机器人之前
    if cate is None:
        vids_list = Video.objects.all()
    if cate == 'editors':
        vids_list = Video.objects.filter(editors_choice=True)
    else:
        vids_list = Video.objects.all()

    page_robot = Paginator(vids_list, 9)
    page_num = request.GET.get('page')
    try:
        vids_list = page_robot.page(page_num)
    except EmptyPage:
        vids_list = page_robot.page(page_robot.num_pages)
        # raise Http404('EmptyPage!')
    except PageNotAnInteger:
        vids_list = page_robot.page(1)

    context['vids_list'] = vids_list
    return render(request, 'listing.html', context)


```

模板层的模板标签传入url

```
 <a class="item" href="{% url 'list' %}editors">
                            Editor's
 </a>

```
判断分类是否被激活

```
                    {% if 'editors' in request.path %}    #根据path是否被传递，判断是否点击
                        <a class="active item" href="{% url 'list' %}editors">
                            Editor's                      #此处class为active class
                        </a>

                    {% else %}

                        <a class="item" href="{% url 'list' %}editors">
                            Editor's
                        </a>

                    {% endif %}

```
---

上传图片

```
#model中添加本地上传属性
class Video(models.Model):
    title = models.CharField(null=True, blank=True, max_length=300)
    content = models.TextField(null=True, blank=True)
    url_image = models.URLField(null=True, blank=True) 
    cover = models.FileField(upload_to='cover_image', null=True)   #此处
    editors_choice = models.BooleanField(default=False)

    def __str__(self):
        return self.title


#告诉系统设置的文件夹
MEDIA_URL = '/upload/'
MEDIA_ROOT = os.path.join(BASE_DIR, 'upload').replace("//", "/")


# url中插入模块，debug处于开启状态，项目会来定义位置寻找；debug关闭，服务器会自动查找，无需记忆
from django.conf import settings
from django.conf.urls.static import static

#合并数据库
makemigrations   migrate

tempalte层处理逻辑

{# 框框 #}
        <div class="ui basic segment container content"  >

            <div class="ui three column grid">

                {% for vid in vids_list %}
                    <div class="column">
                        <a class="ui fluid card" href="#">
                            <div class="image" >
                                {% if vid.cover %}
                                    <img src="/upload/{{ vid.cover }}" alt="" style="height:200px;object-fit: cover;">
                                {% else %}
                                    <img src="{{ vid.url_image }}" alt="" style="height:200px;object-fit: cover;">
                                {% endif %}


                            </div>
                        </a>

                        <div class="title header" href="#">{{ vid.title|truncatechars:30 }}</div>
                        <i class="icon grey unhide"></i>
                        <span style="color:#bbbbbb">10K</span>
                        <span class="" style="color:rgb(226, 226, 226)">|</span>
                        <i class="icon grey checkmark"></i>
                        <span style="color:#bbbbbb">10 people got it</span>

                    </div>

                {% endfor %}
            </div>
        </div>
```