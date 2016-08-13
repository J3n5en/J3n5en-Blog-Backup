---
layout:     post
title:      "《Python Web开发测试驱动方法》读书笔记之一"
subtitle:   "Python，Django，《Python Web开发测试驱动方法》"
date:       2016-02-09
author:     "J3n5en"
tags:
    - 《Python Web开发测试驱动方法》
    - Python
    - Django
---
#### 前言:

在双十一剁了一本《Python Web开发测试驱动方法》。从11.16看了1、2个星期。感觉吸收的不多。效率太低了。前几天看了个文章叫“写作驱动学习”。感觉有点道理。so，决定重头再看，一边看一边写笔记。希望能有不错的效果。

环境：debian + python3 + django1.9 + selenium

我的情况：懂python（在写这篇文章之前一直用python2），略懂django

####  第一章：
TDD ：测试驱动开发（Test-driven development）是极限编程中倡导的程序开发方法，以其倡导先写测试程序，然后编码实现其功能。

在TDD过程中第一步始终是：编写测试。

也就是说，我们首先要先编写测试程序，and run，看看是否和预期一样失败，只有失败了才能去编写程序 to fix the error。

ok～ 介绍TDD就介绍到这里，开始安装django，here we go！
```python
from selenium import webdriver
browser = webdriver.Firefox()
browser.get("http://localhost:8000")
assert 'Django' in browser.title
```
yo ~man~ what the hell are you doing?? 不是说安装django吗?

哈哈..看来你又忘了 在TDD过程中第一步始终是：编写测试。恩,跑起来报错了.of course~ 我们还没安装'占狗'嘛~ 
那就安装django ： 
`pip install django   # 现在是1.9版本`
为了fix 前一个test的坑，这里我们需要新建一个项目并跑起django
```python
django-admin startproject superlists
cd sperlists
manage.py runserver
```
跑起来后我们再运行测试程序看看。

![image](/img/post-img/ba180f66-ce7f-11e5-91a1-d75ccc3fbb8d.png)

Cool～ ～测试通过了。

为了纪念第一次测试成功，我们是不是应该把他保存下来呢？

https://coding.net/u/jensen/p/Python-web-TDD

Cool　～～
#### -第二章：　使用unittest模块扩展功能测试
ｏｋ～　我们继续完善我们的function_test.py

```python
from selenium import webdriver
browser = webdriver.Firefox()
# j3n5en 进入首页
browser.get("http://localhost:8000")

# 标题有"TO-Do"字样
assert 'To-Do' in browser.title

# 在一个文本框输入　"购买显示屏"
# 点击回车，完成输入
# 表格中出现"1、购买显示屏"
# Ｊ再次输入“安装显示屏”
# 页面再次更新
# 页面中显示两条事项
# 滚去睡觉，关闭浏览器
browser.quit()
```
先做到显示标题有“To-Do”吧。不用说跑起来肯定是失败的（所谓的“预期失败”），然后我们通过自己的努力去修复这个错误。。。哇。。听着就爽。
```
➜  superlists :python3 function_test.py   
Traceback (most recent call last):
  File "function_test.py", line 7, in <module>
    assert 'To-Do' in browser.title
AssertionError
```
不过我们先看看这个错误，我们只看到“AssertionError“这个没什么用的报错，也不知道哪里错，而且跑完了浏览器还是在打开状态。不爽。

处理方法有很多，但我们这里用unittest模块的现成方法试试去解决他把。
```python
from selenium import webdriver
import unittest

class NewVisitorTest(unittest.TestCase):
    def setUp(self):   # 特殊方法，在测试前运行
            self.browser = webdriver.Firefox()

    def tearDown(self):    # 特殊方法，在测试完成后运行。出错了也会运行
            self.browser.quit()

    def test_can_start_a_list_and_retrieve_it_later(self):
            self.browser.get("http://localhost:8000")    # j3n5en 进入首页
            self.assertIn("To-Do",self.browser.title)        # 标题有"TO-Do"字样
        self.fail("finish the test!")     # 不管是否测试成功都会输出
if __name__ == "__main__":
    unittest.main(warnings='ignore')
```
恩，跑起来的确完成了想要的效果了。（哪里出错，结束时浏览器关闭）
#### 隐式等待
万一机子慢，页面没加载完就进行判断了那怎么办？　　可以加入
```python
self.browser = webdriver.Firefox()
self.browser.implicitly_wait(3)　　＃　等待３秒
```
这样的话就让selenium等待3秒,让页面出现.
#### 使用单元测试测试简单的首页
创建django应用
`python3 manage.py startapp lists`
为此app编写一个单元测试

```python
#lists/tests.py
from django.core.urlresolvers import resolve
from lists.views import home_page
from django.test import TestCase


class HomePageTest(TestCase):
    
    def test_root_url_resolves_to_home_page_view(self):
        found = resolve('/')
```
我们测试下解析根目录时，是否能找到home_page函数.明显是不行的,因为我们的url和view都还没写,哇哈哈....
```python
#superlists/urls/py
urlpatterns = [
    url(r'^admin/', admin.site.urls),
    url(r'^$', 'lists.views.home_page', name="home"),
]
```
```python
# lists/views.py
def home_page():
    pass
```
好了,终于有写一点django的代码了....现在我们再来运行一下单元测试吧.
![image](/img/post-img/0788aee0-ce80-11e5-99e7-ad390d9fa00c.png)
cool ~ 测试通过了.
### 编写视图层单元测试
```python
# lists/test.py
from django.core.urlresolvers import resolve
from django.http import HttpRequest
from lists.views import home_page
from django.test import TestCase


class HomePageTest(TestCase):

    def test_root_url_resolves_to_home_page_view(self):
        found = resolve('/')
        self.assertEqual(found.func, home_page)

    def test_home_page_returns_correct_html(self):
        request = HttpRequest()  # 创建HttpRequest对象(请求后django看到的就是HttpRequest对象)
        response = home_page(request)  # 把HttpRequest对象丢给home_page处理
        self.assertTrue(response.content.startswith(b'<html>'))
        self.assertIn(b'<title>To-Do lists</title>', response.content)
        self.assertTrue(response.content.endswith(b'</html>'))
# 注意b''  因为response.content是原始字符串,而不是python字符串,因此对比时要用b''    
```
运行看看效果,Nope~ ~ TypeError: home_page() takes 0 positional arguments but 1 was given,看到错误了,我们就去编写代码,去修复他..这就是TDD的一个体现: 遇红  --> 变绿 -->  重构

现在我们遇到了 home_page() takes 0 positional arguments but 1 was given这个红,我们就去编写最少量的代码让他变绿。
```python
# lists/views.py
def home_page(request):
    pass
```
刚刚说不需要参数,却提供了一个参数,那么我们就使他需要参数.再运行测试看看
```
AttributeError: 'NoneType' object has no attribute 'content' 
```
yes~ 刚刚的红变绿了,,但是又有一个红了。再写
```
# lists/views.py
from django.http import HttpResponse
def home_page(request):
    return HttpResponse()
```
还有红,
```
 self.assertTrue(response.content.startswith(b'<html>'))
AssertionError: False is not true
```
再来一发~  哈哈,,终于全绿了,,,哈哈哈。再跑function_test.py。也是毫无错误。。。

哈哈，好刺激。成功编写一段HTML。。。。醉了。。

这一篇文就到这里。。

> happy hacking～ ～