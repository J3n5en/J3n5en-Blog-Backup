---
layout:     post
title:      "在Django中使用Simditor"
subtitle:   "Python，Django，Simditor"
date:       2016-02-07
author:     "J3n5en"
tags:
    - Python
    - Django
    - Simditor
---



之前我的博客一直用的是DjangoUeditor作为我的富文本编辑器，然而由于种种原因 djangoueditor 的代码高亮失效了，

后来发现了simditor，UI 好有feel，就懒得改 djangoueditor 了，，直接换 simeditor，，，在这里记录一些过程。。。。

![image](/img/post-img/2.png)

来来来，，，，我们先看看她那撩人的外表

开始结合吧！！

``` python
#admin.py
#引入simditor文件
from django.contrib.staticfiles.storage import staticfiles_storage
def _wrap(*args, **kwargs):
    return tuple(staticfiles_storage.url(path) for path in args)
class SimditorMixin(object):
    class Media:
        js = _wrap(*('scripts/jquery.min.js',
                     'scripts/module.js',
                     'scripts/hotkeys.js',
                     'scripts/uploader.js',
                     'scripts/simditor.js'))
        css = {
            'all': _wrap(*('styles/simditor.css',
                           ))
        }
class BlogAdmin(SimditorMixin , admin.ModelAdmin):
      `其他内容`
admin.site.register(Blog,BlogAdmin)
```


``` script
#admin/templates/change_form.html
#用了 bootstrap_admin 的自己换成对应目录咯
#启用 simditor

<script>
var $editor = $('#editor') #注意这里
if(!$editor.length){}
else{
(function() {
  $(function() {
    var editor, mobileToolbar, toolbar;
    toolbar = ['title', 'bold', 'italic', 'underline', 'strikethrough', 'color', '|', 'ol', 'ul', 'blockquote', 'code', 'table', '|', 'link', 'image', 'hr', '|', 'indent', 'outdent'];
    mobileToolbar = ["bold", "underline", "strikethrough", "color", "ul", "ol"];
    if (mobilecheck()) {
      toolbar = mobileToolbar;
    }
    return editor = new Simditor({
      textarea: $editor,
      toolbar: toolbar,
      pasteImage: true,
      {% raw %}
      defaultImage: "{% static 'img/image.png' %}",
      {% endraw %}
    });
  });
}).call(this);
}
</script>

```


对于代码高亮的问题，我使用了highlightjs，，，so 在 base.html引入highlightjs的内容，然后加上

``` html
#这里是highlightjs的内容
<script>
$(document).ready(function(){
        hljs.configure({useBR: true});
        $("pre[class^='lang']").each(function(i, block){
            hljs.highlightBlock(block);    
        });
        hljs.initHighlightingOnLoad();
});
</script>
```

好了，，，快去看看成果吧！！！！