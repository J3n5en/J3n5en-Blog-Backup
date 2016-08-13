---
layout:     post
title:      "《Python Web开发测试驱动方法》读书笔记之三"
subtitle:   "Python，Django，《Python Web开发测试驱动方法》"
date:       2016-02-09
author:     "J3n5en"
tags:
    - 《Python Web开发测试驱动方法》
    - Python
    - Django
---

上一篇博文中，我们利用作弊的代码把用户的输入写死了，欺骗了测试joy。很明显这是不可行的。那么我们要怎么做的？其实我们需要让用户的点击变成一个请求，然后让django接受到这个请求并保存起来，最后渲染到模板。这篇博文主要记录的有“编写表单，发送请求”、“服务器处理请求”、“模板渲染变量”还有“Django的ORM和模型”。
---
### 一、编写表单、发送请求。
这里我们编写表单，让用户的输入和提交变为POST请求。这里我们修改模板文件。
```html
# lists/home.html<html>
    <head>
        <title>To-Do lists</title>
    </head>
    <body>
        <h1>To-Do</h1>
        <form method="post">
            <input id="id_new_item" type="text" placeholder="输入备忘录">
        </form>
        <table id="id_list_table">
            <tr><td>1:购买显示屏</td></tr>
        </table>
    </body>
</html>
```
运行测试看看变绿没。oh ～ no。。红了。。。而且只知道没找到“id_list_table”但是不知道为什么会这样。
![image](/img/post-img/f1344a4e-ce81-11e5-86c0-4083d3a0070f.png)
那样，我们在判断id_list_table的时候停下来一阵，让我们肉眼能看见吧。
```python
import time
time.sleep(10)
table = self.browser.find_element_by_id('id_list_table')
```
这样的话我们看到了403错误。
![image](/img/post-img/ff348adc-ce81-11e5-9e79-8d46afe03a57.png)
这是Django一个很有趣的错误。其实他是为了防止跨站请求伪造（csrf）而做的一些限制。我们需要在表单中加入`{% raw %}{% csrf_token %}{% endraw%}`
```html
<form method="post">
    <input id="id_new_item" type="text" placeholder="输入备忘录">
    {% csrf_token %}
</form>
```
ok ～ 删了sleep 。没问题。

下面我们在服务器中接收并处理这个请求

先编写测试
> 待续
