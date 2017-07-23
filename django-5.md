# django-5



---


# POST实现django表单
### **准备工作**

创建comment评论模型，并进行数据合并
```
class Comment(models.Model):                     #创建评论模型
    name = models.CharField(null=True, blank=True, max_length=50)
    comment = models.TextField()
    def __str__(self):
        return self.comment

python manage.py makemigrations                  #数据合并
python manage.py migrate


#此处应该存在admin注册，但是视频中并未体现
```

引入模块，编写视图函数

```
from firstapp.models import Article, Comment

def detail(request):                    #直接粘贴全的了
    if request.method == 'GET':
        form = CommentForm
    if request.method == 'POST':
        form = CommentForm(request.POST)
        if form.is_valid():
            name = form.cleaned_data['name']
            comment = form.cleaned_data['comment']
            c = Comment(name=name, comment=comment)
            c.save()
            return redirect(to='detail')
    context = {}
    comment_list = Comment.objects.all()
    context['comment_list'] = comment_list
    context['form'] = form
    return render(request,'detail.html',context)
```

引入模块，分配url

```
from django.conf.urls import url
from django.contrib import admin
from firstapp.views import index, detail

urlpatterns = [
    url(r'^admin/', admin.site.urls),
    url(r'^index', index, name='index'),
    url(r'^detail', detail, name='detail'),
]

```

templates引入静态文件
```

```
### **渲染表单**
**利用 context 向模板传递 form 变量，form 在传递过程中变成 input 输入框表单结构**

新建form.py，引入forms类，创建CommnetForm实例
```
from django import forms
from django.core.exceptions import ValidationError

def words_validator(comment):
    if len(comment) < 4:
        raise ValidationError('Not enough words')

def comment_validator(comment):
    if 'fuck' in comment:
        raise ValidationError('Do not use that word!')

class CommentForm(forms.Form):
    name = forms.CharField(max_length=50)
    comment = forms.CharField(
        widget=forms.Textarea(),
        error_messages={
            'required':'wow,such words'
            },
        validators=[words_validator, comment_validator]
        )

```
view层中引入Commentform实例，并编写detail视图函数

```
from firstapp.form import CommentForm

def detail(request):                   
    if request.method == 'GET':
        form = CommentForm
    if request.method == 'POST':
        form = CommentForm(request.POST)
        if form.is_valid():
            name = form.cleaned_data['name']
            comment = form.cleaned_data['comment']
            c = Comment(name=name, comment=comment)
            c.save()
            return redirect(to='detail')
    context = {}
    comment_list = Comment.objects.all()
    context['comment_list'] = comment_list
    context['form'] = form                       #都是利用context方法传递      
    return render(request,'detail.html',context)
```

模板层中，删除form表单中的input和textarea，保留提交button,传递form变量；评论模板替换变量，加入循环

```
#提交表单中的更改
{{ form }}

+{{ form as_p }}  #django提供多种表单渲染方式

#评论模板的更改
               {% for comment in comment_list %}              #模板标签
                    <div class="comment">
                        <div class="avatar">
                            <img src="http://semantic-ui.com/images/avatar/small/matt.jpg" alt="" />
                        </div>
                        <div class="content">
                            <a href="#" class="author">{{ comment.name }}</a>    #模板变量
                            <div class="metadata">
                                <div class="date">2 days ago</div>
                            </div>
                            <p class="text" style="font-family: 'Raleway', sans-serif;">
                                {{ comment.comment }}            #模板变量
                            </p>
                        </div>
                    </div>
                {% endfor %}

```

### **绑定表单**
**我们用post方法向服务器提交表单，在view层进行数据校验**
注：在此过程中，视图会接收两种不同的方法，第一是向模板层中传递form进行渲染的get方法，第二是渲染成功的页面进行form提交的post方法

绑定表单之前，我们需要做跨站攻击防护CSRF

```
#模板中,添加于{{ form }}之后

{% csrf_token %}
```


编写detail函数进行表单绑定
```

from firstapp.form import CommentForm

def detail(request):                   
    if request.method == 'GET':
        form = CommentForm               #未绑定表单，用于返回可填写信息的表单
    if request.method == 'POST':
        form = CommentForm(request.POST)  #用request.POST装载Commentform，绑定了表单，用于数据校验
        if form.is_valid():               #和前端渲染时的校验一样，都会触发校验器
            name = form.cleaned_data['name']  #通过验证的数据会存储在cleaned_data字典中
            comment = form.cleaned_data['comment']   #存储在变量中
            c = Comment(name=name, comment=comment)  #变量存于模型，模型构造实例
            c.save()                                 #实例可以进行保存，存于数据库
            return redirect(to='detail')             #redirect重定向
    context = {}                           #失败后，会自动跳出循环，错误数据会在渲染前再次触发校验             
    comment_list = Comment.objects.all()
    context['comment_list'] = comment_list
    context['form'] = form                       #都是利用context方法传递      
    return render(request,'detail.html',context) 
    
#绑定表单后，数据渲染时，会进行自动校验。当发现错误的时候，表单会重新构造，添加错误提示进去，并在前端渲染出来。可以通过print（form.errors）进行查看
```




### **表单定制**
定制表单的验证逻辑（可以重复使用）

```
from django import forms
from django.core.exceptions import ValidationError

def words_validator(comment):
    if len(comment) < 4:
        raise ValidationError('Not enough words')

def comment_validator(comment):
    if 'fuck' in comment:
        raise ValidationError('Do not use that word!')

class CommentForm(forms.Form):
    name = forms.CharField(max_length=50)
    comment = forms.CharField(
        widget=forms.Textarea(),         #修改widget可以改变前端的表现样式
        error_messages={
            'required':'wow,such words'  #自带的验证器验证基本错误，非空错误，非法字符错误，输入数量错误，这里我们修改默认的错误提示
            },
        validators=[words_validator, comment_validator]  #通常，我们手写验证器，列表形式
        )


```
### **定制表单的手工渲染**

django的表单样式上和semantic存在差异，多字段验证时报错信息会出现在每个字段前面
自动渲染的form，django会进行自动校验，手动渲染的时候，我们需要把每个信息陈列出来
```

                {% if form.errors %}      #如果存在错误列表
                    <div class="ui error message">   #顶层展示错误信息，填充到ui error message中
                        {{ form.errors }}       
                    </div>

                    {% for field in form %}
                        <div class="{{ field.errors|yesno:'error, '}} field">
                            {{ field.label }}   #模板过滤器映射判断error
                            {{ field }}
                        </div>
                    {% endfor %}

                {% else %}
                    {% for field in form %}
                        <div class="field">
                            {{ field.label }}      #标签
                            {{ field }}            #输入框
                        </div>
                    {% endfor %}

                {% endif %}

                {% csrf_token %}

```


