---
layout:     post
title:      "解放F5 —— browsersync"
subtitle:   "Javascripe，coffeescript"
date:       2016-02-09
author:     "J3n5en"
tags:
    - Javascript
    - front-end
    - coffeescript
---

嘤嘤嘤.....双十一剁手剁了一个显示屏,想右边打代码左边即时显示....

## ![image](/img/post-img/327e1028-ce66-11e5-81c3-b16656d3f163.png)

这时可能会想到livereoad ~ ~......的确,配合浏览器插件来使用,livereload是一个很好用的自动刷新软件. 

但是这篇博文的主角是 "browsersync" 一个更新,更方便的开发工具... 这里给出中文网 http://www.browsersync.cn

Browsersync可以在不同的浏览器,不同设备下自动刷新修改的页面,更神器的是在其中一个浏览器中的动作可以同步到其他浏览器,其他设备中.

![image](/img/post-img/c0509bbe-ce66-11e5-81dc-e97785e15650.gif)

![image](/img/post-img/d9d2661c-ce66-11e5-99a1-24b5a9c4b90d.gif)

而且修改css后,browsersync不是刷新整个页面,而是重新请求该css,并将修改更新到页面中....

## 好了好了,,该说说怎么用了...

-安装Node.js

``` 
由于Browsersync是基于node.js的,,,,,,so....先安装好node.js和npm吧.
```

-安装BrowserSync

``` 
`#全局npm install -g browser-sync#当然,结合用gulp或者grunt的npm install --save-dev browser-sync`
```

-使用

``` 
`browser-sync start --server --files "css/*.css"`
这个命令用于纯静态站点，也就是仅一些`.html`文件的情况。后面的`--files "css/*.css"`，是指监听`css`目录中的后缀名为`.css`的文件。请注意这个命令里的`start --server`，这其实是BrowserSync自己启动了一个小型服务器。
```

如果是动态站点，则使用代理模式。例如PHP站点，已经建立了一个本地服务器如http://localhost:8080，此时会是这样的命令：

`browser-sync start --proxy "localhost:8080" --files "css/*.css"`

BrowserSync会提供一个新地址用于访问。

从BrowserSync的命令来看，很重要的一点就是通过`--files`指定需要监听的文件。有关这里的文件匹配模式（称为glob）的详情，请参考[isaacs's minimatch](https://github.com/isaacs/minimatch)。

经过尝试，如果简单只是想要监听整个项目，可以写成这样：

`browser-sync start --server  --files "**"`

## 此时，BrowserSync仍然会正确地判断文件变化是否是css。

### UI界面及其他

下面是一个BrowserSync的控制台输出示例：

![image](/img/post-img/1229440e-ce67-11e5-8702-b77acd509db0.png)

可以看到还有一个叫做UI的一个地址，它是BrowserSync提供的一个简易控制面板。BrowserSync最常用的几个配置选项，都可以在这个面板里调整。

在面板里面你还会发现那个经典的远程调试工具weinre也在这：

## ![image](/img/post-img/1ae76df0-ce67-11e5-8340-985c393ffa3f.png)

你可能会发现browsersync不像livereload需要安装浏览器插件,

这是因为browsersync会新建一个服务器对你的文件进行预览,并在页面中插入JS,

而这段JS用来新建web socket连接，一旦有监听的文件发生变化，BrowserSync会通知浏览器.

更多的玩法?读文档去吧!!!

happy hacking ~ ~ ~



