---
layout:     post
title:      "千里之堤：VueJs基础[译]"
subtitle:   "VueJs: The Basics"
date:       2016-04-27
author:     "J3n5en"
http-header-img: "http://ww4.sinaimg.cn/large/6bf00bd8gw1f4nufb4hduj20nm0bk3yl.jpg"
tags:
    - VueJs
    - front-end
    - Javascript
---

> 原文：[VueJs: The Basics](https://coligo.io/vuejs-the-basics/)
>
> PS：非直译，根据译者的理解来翻译。



# 简介:

VueJs 是一个**灵活**、**容易上手**的微型 __Javascript库__，开发者可以利用VueJs __非常简洁__、__快速地__ 开发出交互式的Web应用。VueJs和Angular, React, Ember, Polymer, and Riot等其他一些框架/库同中存异。但是相比下VueJs __更简洁__、__更灵活__、__性能更高__。如果你感兴趣可以去[VueJs的官网](http://vuejs.org)了解更多。

这篇文章将会通过一些实例帮助你了解VueJs的一些基础概念。在接下里的文章里我们才会讲解和利用VueJs更先进的特性去编写可拓展的Web应用。

# 利用 `new Vue()` 创建一个Vue实例

我们创建一个基本的HTML页面，并把VueJS引入进去（我们可以使用NPM、Bower安装VueJs，或者直接引入VueJs文件（或使用CDN））。为了方便演示这里我们直接引入CDN文件。

```html
<!DOCTYPE html>
<html>
  <head>
    <title>VueJs Tutorial - coligo</title>
  </head>
  <body>
  <script src="http://cdnjs.cloudflare.com/ajax/libs/vue/1.0.21/vue.js"></script>
  </body>
</html>
```

**提示：在开发时请用开发版本，遇到常见错误它会给出友好的警告。**

现在我们可以创建一个 `DIV` 标签和一个Vue实例，并把他们绑定起来。当我们创建一个Vue实例的时候应当使用构造函数`Vue()` 并对此为实例提供挂载元素。你可以把这个挂载元素理解为VueJs的边界，VueJs只会接管这个元素的内部。我们可以在`Vue()` 的选项`el` 中为实例提供挂载元素。值可以是 CSS 选择符，或实际 HTML 元素，或返回 HTML 元素的函数。例如：

```html
<!DOCTYPE html>
<html>
  <head>
    <title>VueJs Tutorial - coligo</title>
  </head>
  <body>

  <div id="vue-instance">
    <!-- VueJs将会挂载在这个DOM元素上  -->
  </div>

  <script src="http://cdnjs.cloudflare.com/ajax/libs/vue/1.0.21/vue.js"></script>

  <script>
    // 在Vue实例中绑定DIV的ID
    var vm = new Vue({
      el: '#vue-instance'
    });
  </script>

  </body>
</html>
```

正如你上面看到的，每一个Vue实例通过 `Vue()` 创建，并通过实例的选项`el` 提供挂载点。在这个例子中，我们通过 `Vue()` 创建了一个Vue实例，并通过CSS选择起` #vue-instance` 提供Vue的提供挂载点。一个 Vue 实例其实正是一个 [MVVM 模式](https://en.wikipedia.org/wiki/Model_View_ViewModel)中所描述的 ViewModel - 因此在文档中经常会使用`vm` 这个变量名。接下来，我们将在这个实例下演示Vue的基础功能。

# 双向数据绑定—— `v-model`

为了说明双向数据绑定的特点，我们将会通过`v-model`指令创建一个简单的数据绑定页面

```html
...
  <div id="vue-instance">
    <!-- VueJs将会挂载在这个DOM元素上  -->
    Enter a greeting: <input type="text" v-model="greeting">
  </div>

  <script src="http://cdnjs.cloudflare.com/ajax/libs/vue/1.0.21/vue.js"></script>

  <script>
    // 在Vue实例中绑定DIV的ID
    var vm = new Vue({
      el: '#vue-instance',
      data: {
      greeting: 'Hello VueJs!'
    }
    });
  </script>
...
```

现在，任何时候用户触发`input`的输入事件，`greeting`的值都会随之改变；如果`greeting`的值发生改变，文本输入框的值也会随之改变。通过`v-model` 实现输入框与数据的同步更新，这也就是双向数据绑定的基础概念了。

为了证明输入框和数据间的双向数据绑定，我们可以通过`{{ $data }}`导出Vue实例中的数据导出到页面中，其中`$data` 是Vue中的特殊属性，代表Vue中的数据，而两个大括号则表示要求Vue把括号中的属性替换成相应的值。

```html
...
  <div id="vue-instance">
    <!-- VueJs将会挂载在这个DOM元素上  -->
    Enter a greeting: <input type="text" v-model="greeting">
    <pre>{{ $data | json }}</pre>
  </div>

  <script src="http://cdnjs.cloudflare.com/ajax/libs/vue/1.0.21/vue.js"></script>

  <script>
    // 在Vue实例中绑定DIV的ID
    var vm = new Vue({
      el: '#vue-instance',
      data: {
      greeting: 'Hello VueJs!'
    }
    });
  </script>
...
```
去试试出来的效果吧！哈哈。

其中`{% raw %}{{ $data | json }}{% endraw%}` 中的`| json` 就只是一个辅助（或者说是Vue中的过滤器），他的功能是以一种更顺眼的方式（json）打印我们所需要的`$data` 对象，其实他调用`JSON.stringify() `覆盖我们所需要的值。

当然我们也可以通过双大括号直接打印`data` 中的 `greeting` ，例如：

```html
...
  <div id="vue-instance">
    <!-- VueJs将会挂载在这个DOM元素上  -->
    Enter a greeting: <input type="text" v-model="greeting">
    <p>{{ greeting }}</p>
  </div>
...
```

正如你所见，我们能在Vue中轻而易举地实现双向数据绑定，根本不需要写任何处理事件的Js代码，要做的只是给`input` 通过`v-model`绑定数据。

# 事件处理 —— `v-on`

在VueJs中，我们使用`v-on`指令监听DOM事件和分配事件处理器。你可以在Vue实例中创建一个给点击事件处理器，并绑定在一个事件监听器上。

在这个案例里，我们在Vue中的`methods`选择里创建一个叫`sayHello`的方法，`sayHello`可以弹出一个包含用户输入值的警告框(`alert`) ，并使用`v-on:click` 指令把`sayHello` 绑定在一个`button`按钮上。

```html
<div id="vue-instance">
  Enter your name: <input type="text" v-model="name">
  <button v-on:click="sayHello">Hey there!</button>
</div>

<script src="http://cdnjs.cloudflare.com/ajax/libs/vue/1.0.21/vue.js"></script>

<script>
  var vm = new Vue({
    el: '#vue-instance',
    data: {
      name: ''
    },
    methods: {
      sayHello: function(){
        alert('Hey there, ' + this.name);
      }
    }
  });
</script>
```

当然`v-on`不仅仅可以接受`click`事件，他还可以接受普通的JavaScript事件如 `v-on:mouseover`, `v-on:keydown`, `v-on:submit`, `v-on:keypress`, 等等... 甚至是你自己所定义的事件。或者你会发现在开发中经常使用`v-on`会有点麻烦，为此VueJs提供了一种缩写语法——通过`@*`代替`v-on:*`：例如：

```html
<button v-on:click="sayHello">Hey there!</button>
```

缩写语法：

```html
<button @click="sayHello">Hey there!</button>
```

# 条件渲染 —— `v-if` & `v-show`

当我们希望已登录用户，显示一个欢迎页面，而未登录用户显示的是登录表单。这种场景我们可以使用  `v-if`  & `v-show`指令很简单地实现出来。

利用我们在上一节`v-on`的知识，我们将创建一个模拟登录的方法，当用户点击登录或注销按钮时，切换`isloggedin`的值。

当`isloggedin`的值发生改变时，Vue会重新判断条件，并正确渲染所需的部分。

```html
...

<body>

<div id="vue-instance">
  <div v-if="isLoggedIn">
    Welcome to coligo!
    <button @click="login" type="submit">Logout</button>
  </div>
  <div v-else>
    <input type="text" placeholder="username">
    <input type="password" placeholder="password">
    <button @click="login" type="submit">Login</button>
  </div>
</div>
<script src="http://cdnjs.cloudflare.com/ajax/libs/vue/1.0.21/vue.js"></script>

<script>
  var vm = new Vue({
    el: '#vue-instance',
    data: {
      isLoggedIn: false
    },
    methods:{
      login: function(){
        // 'this' 指向实例vm
        this.isLoggedIn = !this.isLoggedIn;
      }
    }
  });
</script>

</body>

...
```

`v-if` 和 `v-show`的用法和作用基本相同，不同的是有 `v-show` 的元素会始终渲染并保持在 DOM 中。`v-show` 是简单的切换元素的 CSS 属性 `display`。一般来说，`v-if`进行切换时更耗资源，而`v-show`则是在初始化时更耗资源。所以，当一个元素需要经常切换时建议使用` v-show`,很少切换时则推荐用`v-if`。

# 列表渲染 —— `v-for`

假设我们在做一个在线商店，我们希望所有的库存都展现出来。`v-for` 利用类似` item in items`的语法帮助我们循环输出数组中的内容。

这里我们模拟库存数据，并用`v-for`指令输出每一项的名称及其价格到页面中。

```html
...

<body>

<div id="vue-instance">
  <ul>
    <li v-for="item in items">
      {{ item.name }} - ${{ item.price }}
    </li>
  </ul>
</div>

<script src="http://cdnjs.cloudflare.com/ajax/libs/vue/1.0.21/vue.js"></script>

<script>
var vm = new Vue({
  el: '#vue-instance',
  data: {
    items: [
      {name: 'MacBook Air', price: 1000},
      {name: 'MacBook Pro', price: 1800},
      {name: 'Lenovo W530', price: 1400},
      {name: 'Acer Aspire One', price: 300}
    ]
  }
});
</script>

</body>

...
```

在实际的开发中，我们只需要把请求接口得到的数据替换掉`items`的数据就能实现这个功能了。有时候我们希望获得循环中的索引值，这里我们有两个方法去实现:

一、使用`$index`变量:

```html
<div id="vue-instance">
  <ul>
    <li v-for="item in inventory">
      {{ $index }} - {{ item.name }} - ${{ item.price }}
    </li>
  </ul>
</div>
```

二、使用别名：

```html
<div id="vue-instance">
  <ul>
    <li v-for="(index, item) in inventory">
      {{ index }} - {{ item.name }} - ${{ item.price }}
    </li>
  </ul>
</div>
```
#  计算属性（Computed）

当其中一个变量依赖于一个或多个变量时，计算属性就显得非常有用了。

这里用我们需要用户输入一个整数，他会自动返回他的两倍，通常我们都使用事件监听器等来实现这个功能。为了了解计算属性，我们这里就使用他。

```html
<body>

<div id="vue-instance">
    <input type="number" v-model="x">
    result: {{ doubleX }}
</div>

<script src="http://cdnjs.cloudflare.com/ajax/libs/vue/1.0.21/vue.js"></script>

<script>
var vm = new Vue({
  el: '#vue-instance',
  data: {
    x: 1
  },
    computed: {
        doubleX: function(){
            return this.x*2;
        }
    }
});
</script>

</body>
```

计算属性对象是一组返回一个值的函数，我们可以和`methods`一样在Vue实例中定义`computed`。

# 总结：

1. 创建Vue实例 =>  new Vue()
2. 双向数据绑定 => `v-model`
3. 输出数据到页面 => 双大括号`{% raw %}{{}}{% endraw%}`
4. 列表渲染 => `v-for` 指令
5. 条件渲染 => `v-if` & `v-show`
6. 事件监听 = > `v-on:*` 或者缩写`@*`
7. 定义：`data` 、 `methods` 、`computed` 属性

现在你已经掌握了VueJs的基本知识了，希望你继续关注我博客关于VueJs的文章，并坚持练习！

\#EOF\#
