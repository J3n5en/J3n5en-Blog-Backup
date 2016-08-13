---
layout:     post
title:      "在sae上使用DjangoUeditor 1.8的图片上传功能"
subtitle:   "Python，Django"
date:       2016-02-06
author:     "J3n5en"
tags:
    - Python
    - Django
---

SAE上是没有目录读写权限的，所以要在SAE使用Ueditor的图片上传功能需要借助SAE的Storage服务。并对DjangoUeditor 进行修改，，，

1、开通Sae的Storage功能：



在SAE控制台开通Storage服务，并新增一个domain，按需求填写即可。

![image](/img/post-img/1.png)



2、修改 DjangoUeditor 的代码：

``` 
# DjangoUeditor/Views.py
# 保存上传图片的部分
# 原因:将图片上传至Storage并返回图片地址
# 注意修改 domain
#coding:utf-8
def save_upload_file(PostFile,FilePath):
    try:
        f = open(FilePath, 'wb')
        for chunk in PostFile.chunks():
            f.write(chunk)
    except Exception,E:
        f.close()
        return u"写入文件错误:"+ E.message
    f.close()
    return u"SUCCESS"
# 改为:    
def save_upload_file(PostFile,FilePath):
    try:
        import sae.const
        access_key = sae.const.ACCESS_KEY
        secret_key = sae.const.SECRET_KEY
        appname = sae.const.APP_NAME
        domain_name = "注册的domain"
        import sae.storage
        s = sae.storage.Client()
        ob = sae.storage.Object(PostFile)
        #此处返回的url是文件在Storage上的url
        url = s.put(domain_name, FilePath, ob)
        return u"SUCCESS",url
    except Exception,e:
        return u'FAIL'
# 注释以下两行代码
# DjangoUeditor/Views.py
# 原因:sae没有读写权限，不能makedirs
# 
# if not os.path.exists(OutputPath):
#  os.makedirs(OutputPath)
#所有检测完成后写入文件
if state==u"SUCCESS":
#改为
if state!=u"FAIL":
# 原因:由于上传成功还会返回url所以不能通过SUCCESS来判断是否上传成功
state=save_upload_file(file,os.path.join(OutputPath,OutputFile))
# 改为:
state,url=save_upload_file(file,os.path.join(OutputPath,OutputFile))
# 作用:
# 提取上传状态和图片地址
'url': urllib.basejoin(USettings.gSettings.MEDIA_URL , OutputPathFormat) ,                # 保存后的文件名称
# 修改为：
'url': url,                # 保存后的文件地址
# 作用:原来的是返回文件名，修改后返回文件地址
```

这样你就可以在SAE上通过Ueditor将图片上传到SAE Storage上去了。

enjoy it！~~