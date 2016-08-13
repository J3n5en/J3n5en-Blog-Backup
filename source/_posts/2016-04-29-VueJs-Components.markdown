---
layout:     post
title:      "VueJs组件[译]"
subtitle:   "VueJs: Components"
date:       2016-04-29
author:     "J3n5en"
http-header-img: "http://ww4.sinaimg.cn/large/6bf00bd8gw1f4nucnvaafj21ig0qojs1.jpg"
tags:
    - VueJs
    - front-end
    - Javascript
---

> 原文：[VueJs: Components](https://coligo.io/vuejs-components/) 
>
> PS：非直译，根据译者的理解来翻译。



# 实用Vue组件指南:

在这篇文章中，我们将通过一系列容易上手的实例来学习和理解Vue的组件。在我们学习完组件的基础后，我们用之前所学知识创建一个简单版的reddit（类似于国内的煎蛋、豆瓣小组之类的）。让他有一些文章，并添加赞和差评的功能。最后，我们将使用组件的方式，创建一个和reddit类似的应用——Disqus(类似国内的多说)，用户可以点赞和差评其他用户的评论。

你刚刚接触Vue？没关系!可以看看我博客上一篇文章。

---

# 了解Vue组件

组件可以帮助我们把复杂的事情分拆成简单的事情来做；或者积小成多把简单的应用最后累积成庞大的系统。组件也可以帮助你跨应用地代码复用。

那我们和以前一样，我们先创建一个简单的HTML页面，并创建Vue实例作为Vue的演练场吧。

```html
<!DOCTYPE html>
<html>
  <head>
    <title>VueJs Tutorial - coligo</title>
  </head>
  <body>

  <div id="app">
    <!-- VueJs将会挂载在这个DOM元素上  -->
  </div>

  <script src="http://cdnjs.cloudflare.com/ajax/libs/vue/1.0.21/vue.js"></script>

  <script>
    // 在Vue实例中绑定DIV的ID
    var vm = new Vue({
      el: '#app'
    });
  </script>

  </body>
</html>
```

我们的演练场准备好了，现在就我们开始创建一个简单的、可复用的组件吧。在Vue中我们使用构造函数`Vue.component()` 创建组件，它接受两个参数：

1. 组件的名字
2. 包含选项的组件对象

组件对象的选项和`Vue()`的选项几乎是一模一样的，主要就是`el`和`data` 有微笑的差异。我们来通过组件来跑个`Hello VueJS！`吧。

- 给组件命名为`greeting`
- 给组件传递`template ` 选项

```html
Vue.component('greeting', {
    template: '<h1>Welcome to coligo!</h1>'
});
```

现在我们已经新建了一个简单的组件，那么在我们的应用里就可以用`<greeting>`和`</greeting>`像常规的HTML标签一样很简单地使用它了。放在一起看就是这样：

```html
...
<body>
    <div id="app">
        <greeting></greeting>
    </div>

    <script src="http://cdnjs.cloudflare.com/ajax/libs/vue/1.0.21/vue.js"></script>
    <script>
        Vue.component('greeting', {
            template: '<h1>Welcome to J3n5en\'Blog!</h1>'
        });
        var vm = new Vue({
            el: '#app'
        });
    </script>
</body>
...
```

不出所料的话，他和我们预期一样会显示一个“Welcome to J3n5en'Blog!”。要是我们想要调用三/多次组件呢？

我们可以这样：

```html
...
<div id="app">
    <greeting></greeting>
    <greeting></greeting>
    <greeting></greeting>
</div>
...
```

通过组件，我们的应用会更加高效、简洁。

# 使用`<template>` 标签创建更加复杂的组件

现在或许你会想，要是我们的组件里面不只只存在一个`<h1>`标签而是非常复杂呢？直接在`template`属性里定义？那还没等到你觉得组件是高效的、简洁的时候你就已经抓狂、崩溃了吧。

为了避免这种问题，我们可以利用HTML5里面的`<template>`标签，它可以被客户端获取，却不会被渲染。通过这个标签的这种特性，那么当我们需要他的时候，我们就可以获取他，使用他。

我们可以在HTML页面的任意位置添加`<template>`,并通过的的ID在Vue组件中引用它。

```html
...
<body>
    <div id="app">
        <greeting></greeting>
    </div>

    <template id="greeting-template">
        <h1>Welcome to J3n5en's Blog!</h1>
        <button>login</button>
        <button>signup</button>
        <a href="https://blog.j3n5en.com">Check out the other tutorials!</a>
    </template>


    <script src="http://cdnjs.cloudflare.com/ajax/libs/vue/1.0.21/vue.js"></script>
    <script>
        Vue.component('greeting', {
            template: '#greeting-template'
        });

        var vm = new Vue({
            el: '#app'
        });
    </script>
</body>
...
```

现在我们已经可以通过这个方法储存更多，更复杂的模板，同时还可以避免混乱、保持较高的可读性。我们在后面的文章中还会介绍一些更加优雅的方法来组织组件例如：vueify 。

# 使用`Props`传递数据

**组件实例的作用域是孤立的**。这意味着不能并且不应该在子组件的模板内直接引用父组件的数据。可以使用 **`props`** 把数据传给子组件。让我们创建一个简单的、基本例子来学习怎样通过`Props`传递数据。

我们需要在子组件的`props`选项中显式地声明希望从父组件获得的数据。像这样：

```javascript
Vue.component('greeting', {
    template: '<h1>{{message}}</h1>',
    props: ['message']
});
```

这里的`props`是一个字符串组成的数据。这个例子中，我们的组件只会从父组件中仅接受`message`的数据。然后我们需要在父组件给他传递`message`的数据。这里传入一个普通的字符串：

```html
<greeting message="Welcome to the VueJs Components Tutorial"></greeting>
```

`props`也可以被定义为数据，可以指定验证要求。例如在这个例子中，我们希望`message`是必须存在的，而且只能是字符串类型，那么我们可以这样：

```javascript
Vue.component('greeting', {
    template: '<h1>{{message}}</h1>',
    props: {
        message: {
            type: String,
            required: true
        }
    }
});
```

在这篇文章中，我们不会太过于深入地讲解**Prop验证**。如果你感兴趣可以去[VueJs的官网的指南](http://cn.vuejs.org/guide/components.html#Prop-验证)中了解更多关于**Prop验证**的知识。

下面我们看看在实际的场景中组件的用法。实际场景中，我们会调用数据库获得一些如下的博客文章信息对象：

```json
{
    author: 'Johnnie Walker',
    title: 'Aging Your Own Whisky',
    content: 'A bunch of steps and a whole lot of content'
}
```

这里为了方便演示，我们就不调用数据库了，直接模拟已经获得数据的情况。像：

```javascript
var vm = new Vue({
    el: '#app',
    data: {
        author: 'Johnnie Walker',
        title: 'Aging Your Own Whisky',
        content: 'A bunch of steps and a whole lot of content'
    }
});
```

我们的博客一般会有很多文章，这里我们创建一个简易的文章组件：

```html
<template id="post-template">
    <h1>{{ title }}</h1>
    <h4>{{ author }}</h4>
    <p>{{ content }}</p>
</template>
```

那么我们怎么将标题等内容替换成从数据库中获得的信息呢？

首先，我们应该像刚刚一样，在组件中引用id为`post-template`的模版。

```javascript
Vue.component('post', {
    template: '#post-template'
});
```

接下来我们在组件中声明`props`选项，

```javascript
Vue.component('post', {
    template: '#post-template',
    props: ['title', 'author', 'content']
});
```

现在我们可以在我们的博客应用中到处引用`post`组件了：

```html
<post :title="title" :author="author" :content="content"></post>
```

在这个例子中我们从父组件中动态地绑定组件的3个数据。（这里的`:*`是`v-bind:*`的缩写）。现在只要父组件的数据发生改变，子组件也会随之马上发生改变。

# 创建一个类似Reddit的文章管理系统

通过这篇文章，我们已经掌握了Vue的基础知识，现在我们就用这篇和前一篇文章所学的知识编写一个类似Reddit的文章管理系统。

- 创建Vue实例
- 绑定`#app`DIV
- 定义将要显示的文章数据

```javascript
// voter.js
var vm = new Vue({
  el: "#app",
  data: {
    posts: [{
                title: "A post for our reddit demo starting at 15 votes",
                votes: 15
            },
            {
                title: "Try out the upvoting, it works, I promise",
                votes: 53
            },
            {
                title: "coligo is the bomb!",
                votes: 10
            }]
  }
});
```

`data`由文章标题和文章的票数组成，现在我们编写HTML和CSS。

```html
<!-- index.html -->
<div id="app">
    <div class="container-fluid">
        <ul class="list-group">

        </ul>
    </div>
</div>
```

在HTML 中使用`<template>` 定义一个单篇文章的组件，这样我们就能使用`v-for` 很简单地把所有的文章都显示出来。

```html
<!-- index.html -->
<template id="post-template">
  <li class="list-group-item">
    <i class="glyphicon glyphicon-chevron-up"></i>
    <span class="label label-primary">{{ post.votes }}</span>
    <i class="glyphicon glyphicon-chevron-down"></i>
    <a>{{ post.title }}</a>
  </li>
</template>
```

最后我们的HTML就将会是这样的:

```html
<!-- index.html -->
<!doctype html>
<html>

<head>
    <meta charset="utf-8">
    <title>VueJs Components Tutorial - coligo.io</title>

    <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.6/css/bootstrap.min.css">
    <link rel="stylesheet" href="style.css">
</head>

<body>

    <div id="app">
        <div class="container-fluid">
            <ul class="list-group">

            </ul>
        </div>
    </div>

    <template id="post-template">
        <li class="list-group-item">
            <i class="glyphicon glyphicon-chevron-up"></i>
            <span class="label label-primary">{{ post.votes }}</span>
            <i class="glyphicon glyphicon-chevron-down"></i>
            <a>{{ post.title }}</a>
        </li>
    </template>

    <script src="https://code.jquery.com/jquery-2.2.0.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/vue/1.0.16/vue.js"></script>
    <script src="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.6/js/bootstrap.min.js"></script>
    <script src="voter.js"></script>
</body>

</html>
```

现在我们已经定义好的Vue实例，和template。现在我们就在Js文件中定义Vue组件。并在组件中定义`props`。这样我们就可以在子组件中接受来自父组件的`data`.

```javascript
// voter.js
Vue.component('post', {
  template: "#post-template",
  props: ['post']
});

var vm = new Vue({
  el: "#app",
  data: {
    posts: [{
                title: "A post for our reddit demo starting at 15 votes",
                votes: 15
            },
            {
                title: "Try out the upvoting, it works, I promise",
                votes: 53
            },
            {
                title: "coligo is the bomb!",
                votes: 10
            }]
  }
});
```

现在我们就可以使用`v-for`指令把`<post>`全部循环出来。

```html
<!-- index.html -->
<div id="app">
  <div class="container-fluid">
    <ul class="list-group">
      <post v-for="post in posts" :post="post"></post>
    </ul>
  </div>
</div>
```

现在我们做的就是循环输出`data`里面的`posts`对象数组。那么现在就能在页面中渲染数据的所有内容。接下来我们要做的就是编写点赞的逻辑，当用户点击赞时，我们需要判断他是否已经点过赞。

首先，我们给文章添加赞/踩按钮：

```html
<i class="glyphicon glyphicon-chevron-up" @click="upvote"></i>
<i class="glyphicon glyphicon-chevron-down" @click="downvote"></i>
```

任何时候用户点击了赞/踩，都会调用响应的`method`,但是当用户已经点击过了，按钮就变成不可点击状态。

```javascript
Vue.component('post', {
  template: "#post-template",
  props: ['post'],
  data: function () {
    return {
      upvoted: false,
      downvoted: false
    };
  },
  methods: {
    upvote: function () {
      this.upvoted = !this.upvoted;
      this.downvoted = false;
    },
    downvote: function () {
      this.downvoted = !this.downvoted;
      this.upvoted = false;
    }
  }
});
```

现在我们需要添加点击赞/踩后`votes`增加/改变的功能：

使用`computed`属性计算，检查文章是否已经赞/踩，从而使`votes` 增加或减少1。

```javascript
...
  computed: {
    votes: function () {

      if (this.upvoted) {
        return this.post.votes + 1;
      } else if (this.downvoted) {
        return this.post.votes - 1;
      } else {
        return this.post.votes;
      }

    }
  }
...
```

最后Javascript的全部代码是：

```javascript
// voter.js
Vue.component('post', {
  template: "#post-template",
  props: ['post'],
  data: function () {
    return {
      upvoted: false,
      downvoted: false
    };
  },
  methods: {
    upvote: function () {
      this.upvoted = !this.upvoted;
      this.downvoted = false;
    },
    downvote: function () {
      this.downvoted = !this.downvoted;
      this.upvoted = false;
    }
  },
  computed: {
    votes: function () {
      if (this.upvoted) {
        return this.post.votes + 1;
      } else if (this.downvoted) {
        return this.post.votes - 1;
      } else {
        return this.post.votes;
      }
    }
  }
});

var vm = new Vue({
  el: "#app",
  data: {
    posts: [{
                title: "A post for our reddit demo starting at 15 votes",
                votes: 15
            },
            {
                title: "Try out the upvoting, it works, I promise",
                votes: 53
            },
            {
                title: "coligo is the bomb!",
                votes: 10
            }]
  }
});
```

最后我们要做的就是把`post`中的`votes`显示到页面中。

```html
<template id="post-template">
    <li class="list-group-item">
        <i class="glyphicon glyphicon-chevron-up" @click="upvote"></i>
        <span class="label label-primary">{{ votes }}</span>
        <i class="glyphicon glyphicon-chevron-down" @click="downvote"></i>
        <a>{{ post.title }}</a>
    </li>
</template>
```

判断用户已经投票然后使得按钮失效，我们可以使用Vue中的Class绑定（` v-bind:class `）。我们需要判断`upvoted/downvote`的真/假，然后判断是否需要添加`disabled`的class。

```html
<!-- 如果 upvoted === true, 添加 disabled class -->
<i class="glyphicon glyphicon-chevron-up" @click="upvote" :class="{disabled: upvoted}"></i>
<!-- 如果 the downvoted === true, 添加 disabled class -->
<i class="glyphicon glyphicon-chevron-down" @click="downvote" :class="{disabled: downvoted}"></i>
```

现在我们的类似Reddit的文章管理系统已经基本完成了。

# 总结

现在我们已经了解了Vue关于组件的基础知识：通过`props`传递数据、通过`<template>`标签创建模板。然后我们还利用所学知识编写了一个类似Reddit的文章管理系统。

用几点需要注意的是：

- 记住要申明想在子组件中获得的数据（`props: ['post'`）
- 使用`props`绑定数据，通过`v-bind`绑定数据，或者缩写(`:post="post"`)
- 为了确保组件的独立，我们需要把`data`对象放在函数的返回值中。
- 别忘了Vue根实例，我们还地需要在根实例中显示子组件呢。



\#EOF\#

