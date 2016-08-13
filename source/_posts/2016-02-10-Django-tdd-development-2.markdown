---
layout:     post
title:      "《Python Web开发测试驱动方法》读书笔记之二"
subtitle:   "Python，Django，《Python Web开发测试驱动方法》"
date:       2016-02-09
author:     "J3n5en"
tags:
    - 《Python Web开发测试驱动方法》
    - Python
    - Django
---

前一篇笔记中,我们已经实际演练了基本的TDD流程,那么这篇笔记我们就先来谈谈编写这些测试的用处吧。

相信很多的读者和心中存在很多疑问:
 + 编写的测试会不会有点多,
 + 测试好像有点重复了吧.? 单元测试和功能测试里面
 + 每次修改都要循环"单元测试/编写代码",这有必要吗?
这里把项目开发比喻成用绳子在水井里面打一桶水.

如果井不是很深,那么打起水来是比较简单的  ( 就是说在开发小型项目的时候,并不困难)

但是井越来越深了,打起水来就不是一件简单的事情。（好比在开发大型项目）。

而TDD理念就好比是一个棘轮，使用它，你可以保存当前的进度，而且保证进度绝不倒退。这样打起水来就不必要一直这么聪明了。
- 但是测试真的要写的这么细吗？（棘轮的齿轮有必要这么细吗？）
我们现在先为简单的函数写好测试，占好位置，当这个函数变得越来越复杂，测试也不会变得很乱，那也就不会产生心理障碍了。

或许一个函数本来只是一个if语句，但是也许几天后他会再添加一个for循环，或许日后会越来越复杂，不知不觉间变成一个基于元素的多态树结构解析器了。

理都懂，然并卵？ok～ 那我们继续实践咯。
---
好了，继续我们的实战。

### 使用selenium测试用户交互

完了，之前写代码写到哪里了？运行测试看看吧。
![image](/img/post-img/fdffb534-ce80-11e5-8c93-14e3cb0d3aaa.png)
>这也是TDD的优点之一，我们永远不会忘记接下来要干嘛。


测试结束了，那么我们继续编写测试吧。
```python
#coding:utf-8
from selenium import webdriver
import unittest

from selenium.webdriver.common.keys import Keys


class NewVisitorTest(unittest.TestCase):
    def setUp(self):  # 特殊方法，在测试前运行
        self.browser = webdriver.Firefox()
        # self.browser.implicitly_wait(3)

    def tearDown(self):  # 特殊方法，在测试完成后运行。出错了也会运行
        self.browser.quit()

    def test_can_start_a_list_and_retrieve_it_later(self):
        # j3n5en 进入首页
        self.browser.get("http://localhost:8000")
        # 标题有"TO-Do"字样
        self.assertIn("To-Do", self.browser.title)
        header_text = self.browser.find_element_by_tag_name('h1').text
        # 头部也有To-Do字眼
        self.assertIn('To-Do', header_text)
        # 在一个文本框输入　"购买显示屏"
        inputbox = self.browser.find_element_by_id('id_new_item')
        self.assertEqual(
            inputbox.get_attribute('placeholder'),
            '输入备忘录'
        )
        inputbox.send_keys("购买显示屏")
        # 点击回车，完成输入
        inputbox.send_keys(Keys.ENTER)
        # 表格中出现"1、购买显示屏"
        table = self.browser.find_element_by_id('id_list_table')
        rows = table.find_element_by_tag_name('tr')
        self.assertTrue(
            any(row.text == '1:购买显示屏' for row in rows)
        )
        self.fail("完成测试")
    # Ｊ再次输入“安装显示屏”
    # 页面再次更新
    # 页面中显示两条事项
    # 滚去睡觉，关闭浏览器
if __name__ == "__main__":
    unittest.main(warnings='ignore')
```
运行看看。
![image](/img/post-img/191e2508-ce81-11e5-8104-db3c98ccaa58.png)
未找到<h1>标签。那么我们给他加h1呗。
> 单元测试规则之一：不测试常量、其实要测试的是：逻辑、流程控制和配置。
之前我们把html插在了views.py里面了，其实正确的方法应该是使用模板。那么我们使用模板重构一下之前的代码吧。

在重构之前我们先运行下下测试，python3 manage.py test 以确保重构前后变现一致。ok～ 测试是通过的，上，写代码去。
```
# lists/templates/home.html
<html>
    <title>To-Do lists</title>
</html>
```
```
# /lists/views.py
def home_page(request):
    return render(request, 'home.html')
```
重构完了，跑下测试，看看重构前后表现是否一致。 ok～ 没问题

接下来我们编写模板使代码能通过功能测试。
```html
<html>
    <head>
        <title>To-Do lists</title>
    </head>
    <body>
        <h1>To-Do</h1>
        <input id="id_new_item" type="text" placeholder="输入备忘录">
        <table id="id_list_table">
            <tr><td>1:购买显示屏</td></tr>
        </table>
    </body>
</html>
```

![1111](/img/post-img/bcba8c7e-ce81-11e5-9a2f-5ad81f2fa059.png)

啊哈，，我用假的代码骗过了测试。但是很明显这是假的。下一篇将学习如何保存用户输入，和渲染到模板。

> Happy Hcking ～ ～ ～