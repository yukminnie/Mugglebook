# vue.js-1



---
# JS语法入门

### **前言**
js版本

```
ES5
```

js库和框架
```
Angular.js   Vue.js   React.js   JQuery
```

JS运行环境

```
Node.js    集成了chrome的内核和附加的一堆库，可以运行于后端
```

为什么使用Vue.js

```
减少代码量，代码变得更加可维护，克服不同浏览器对JS的兼容性

Vue.js： 用库更方便，学习成本低，关注界面变化(及时反映页面状态变化，数据变化，页面交互)

```
### **JS语法入门**

赋值

```
a = 1   #python

var a = 1;    
```

输出

```
print()   #python

alert();   
```

函数

```
def x(a):      #python
    return a
    
function x(a){
    return a;
}

例子：
     function foo(bar){
     alert(bar);
     return bar;
     }
     
     var z = foo("hi");
     z
```
条件判断

```
if true:
 ...
else:
 ...
 
if(true){
}
else{
}

例子：

function foo(bar){
	if(bar > 3){
		alert(bar + 1);
	}else{
		alert("not enough");
    }
}
```

列表

```
l = [1, 2, 3]    #python

var l = [1, 2, 3];

例子：

arr = []

function foo(bar){
	if(bar > 3){
		arr.push(bar);       #python中使用 append
	}else{
		alert("not enough");
    }
}
```

字典和对象

```
d = {                           #python，键不需要添加双引号
    "a":1,
    "b":2

}

o = {                     #js中这是个对象,可以使用点语法
    a:1,
    b:2
}
```

DOM   :document object model

```
var el = document.querySelector('#random-ads');
el.setAttribute('style', 'display:none;')   #设置属性
```

事件绑定

- where 特定区域
- how   触发事件
- what  做一些事情

```
var b = document.querySelector('body');
b.setAttribute('style', 'background-color:black');

例子:
var dark = 'background-color: black; color: white;';
var day = 'background-color: white; color: black;';

var button = document.querySelector('.nav');
var web = document.querySelector('body');

function lightSwitch(){
    if(web.style.cssText == dark){          
        web.style.cssText = day;
        alert('Day Mode');
    }else{
        web.style.cssText = dark;
        alert('Night Mode');
    }
}

button.onclick = lightSwitch
```