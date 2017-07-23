# django-6



---

# 正则实现文章跳转

## **不够多的url**



### **url分配**
**我们创建文章时，并不需要在model层手动定义，数据库会自动给每篇文章分配一个id，即article.id**
```
from django.conf.urls import url
from django.contrib import admin
from firstapp.views import index, detail, detail_comment

urlpatterns = [
    url(r'^admin/', admin.site.urls),
    url(r'^index', index, name='index'),
    url(r'^detail/(?P<page_num>\d+)$', detail, name='detail'),   #我们给数字起了一个名字page_num
    url(r'^detail/(?P<page_num>\d+)/comment$', detail_comment, name='comment'),
    ]
```

### **view层传递**

使用get方法取到唯一id等同于page_num的文章，并将其装载于context中

get方法带着page_num传入，view处理逻辑进行数据返回
```
def detail(request, page_num, error_form=None):
    context = {}
    form = CommentForm
    
    a = Aritcle.objects.get(id=page_num)
    best_comment = Comment.objects.filter(best_comment=True, belong_to=a) 
    if best_comment:
        context['best_comment'] = best_comment[0]
    
    article = Aritcle.objects.get(id=page_num)            #主要看这行
    context['article'] = article
   
    if error_form is not None:
        context['form'] = error_form
    else:
        context['form'] = form
    return render(request, 'article_detail.html', context)
```

> 流程---------
template传入数据 ，然后使用url捕获，url返回给我们的view，然后view传递数据进行加载

模板替换
```
            <h1 class="ui header">
                {{ article.headline }}
            </h1>
            <p>
                {{ article.content }}    #模板文章替换
            </p>
```

跳转捕获

```
            {% for article in article_list %}
                <div class="ui container vertical segment">
                    <a href="{% url 'detail' article.id %}">     #标签会渲染成完整的url，并被捕获
                        <h1 class="ui header">
                            {{ article.headline }}
                        </h1>
                    </a>

                    <i class="icon grey small unhide">10,000</i>
                    <p>
                        {{ article.content|truncatewords:100 }}
                        <a href="{% url 'detail' article.id %}">  #捕获
                            <i class="angle tiny double grey right icon">READMORE</i>
                        </a>
                    </p>

                    <div class="ui mini  tag label">
                        {{ article.tag }}
                    </div>
                </div>
            {% endfor %}
```

## **文章和评论的对应关系**

**普通评论**

Comment中设置普通评论对应属性，并将其设置为Article的外键(模板渲染可以直接使用主键，不在view中体现)
```
class Comment(models.Model):
    name = models.CharField(null=True, blank=True, max_length=50)
    comment = models.TextField()
    belong_to = models.ForeignKey(to=Aritcle, related_name='under_comments', null=True, blank=True)
    best_comment = models.BooleanField(default=False)    #bool属性
    def __str__(self):
        return self.comment
```

view中删除detail中的comment_list属性

template中修改comment_list

```
{% for comment in article.under_comments.all %}
```

**最优评论**
定义完成后，我们在后台标记最优评论

```
best_comment = True
```
然后在view视图中，书写渲染逻辑

```
def detail(request, page_num, error_form=None):
    context = {}
    form = CommentForm
    
    a = Aritcle.objects.get(id=page_num)             #主要看这一部分
    best_comment = Comment.objects.filter(best_comment=True, belong_to=a) #两个评论的对应属性
    if best_comment:
        context['best_comment'] = best_comment[0]   #可能不存在最优评论，存在的时候，返回列表第一个值
    
    article = Aritcle.objects.get(id=page_num)
    context['article'] = article

    if error_form is not None:
        context['form'] = error_form
    else:
        context['form'] = form
    return render(request, 'article_detail.html', context)
```

template中进行渲染

```
                {% if best_comment %}             #判断是否存在
                    <div class="ui mini red left ribbon label">   #存在最优评论的时候，存在小标签
                        <i class="icon fire"></i>
                        BEST
                    </div>
                    <div class="best comment">     #复制的comment的内容      
                        <div class="avatar">
                            <img src="http://semantic-ui.com/images/avatar/small/matt.jpg"  alt="" />
                        </div>
                        <div class="content">
                            <a href="#" class="author">{{ best_comment.name }}</a>  #模板变量
                            <div class="metadata">
                                <div class="date">2 days ago</div>
                            </div>
                            <p class="text" style="font-family: 'Raleway', sans-serif;">
                                {{ best_comment.comment }}     #模板变量
                            </p>
                        </div>
                    </div>
                    <div class="ui divider"></div>         #加了分割线
                {% endif %}

```
提交评论，文章和评论没有针对性

```
#拆分后的视图，主要看评论的绑定
def detail_comment(request, page_num):
    form = CommentForm(request.POST)
    if form.is_valid():
        name = form.cleaned_data['name']              #筛选名字
        comment = form.cleaned_data['comment']         #筛选评论 
        a = Aritcle.objects.get(id=page_num)            #筛选对应文章
        c = Comment(name=name, comment=comment, belong_to=a)    #存储保存
        c.save()
    else:
        return detail(request, page_num, error_form=form)

    return redirect(to='detail', page_num=page_num)    #评论返回对应的文章

```

## **视图分离**
> 存在多个form的情况，我们需要进行视图分离

我的理解：在不分离视图的情况下，template层中，django渲染form有自己的一套逻辑，form或者form error都有相对应的显示，但都是针对detail函数的；视图分离以后，detail负责get方法的接收，负责comment的显示，detail_comment负责post方法的接收，负责form校验，并将返回的form传递给detail，表单正确时候直接传递，表单错误的时候原有基础上传递error_form。此时因为是post方法，所以，网址后缀comment并不显示。
```
def detail(request, page_num, error_form=None):
    context = {}
    form = CommentForm
   
    a = Aritcle.objects.get(id=page_num)
    best_comment = Comment.objects.filter(best_comment=True, belong_to=a)
    if best_comment:
        context['best_comment'] = best_comment[0]
    
    article = Aritcle.objects.get(id=page_num)
    context['article'] = article
 
    if error_form is not None:
        context['form'] = error_form
    else:
        context['form'] = form
    return render(request, 'article_detail.html', context)

def detail_comment(request, page_num):
    form = CommentForm(request.POST)
    if form.is_valid():
        name = form.cleaned_data['name']
        comment = form.cleaned_data['comment']
        a = Aritcle.objects.get(id=page_num)
        c = Comment(name=name, comment=comment, belong_to=a)
        c.save()
    else:
        return detail(request, page_num, error_form=form)

    return redirect(to='detail', page_num=page_num)
```