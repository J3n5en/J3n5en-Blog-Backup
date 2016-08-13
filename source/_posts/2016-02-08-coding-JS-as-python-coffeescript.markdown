---
layout:     post
title:      "像写Python/Ruby一样写Javascript——coffeescript "
subtitle:   "Javascripe，coffeescript"
date:       2016-02-08
author:     "J3n5en"
tags:
    - Javascript
    - front-end
    - coffeescript
---



作为一名Pyer，我非常python的简洁语法。也希望在写前端JS是能用到类似他的语法。。。。于是碰到了coffeescript。coffeescript能让我们编写更少的代码，使得语言特性更强。其实最爽的是写js让我感觉我在写python，不用再管那些麻烦的符号。。哇哈哈哈。。。。。

那么壮士，让我们干了这杯热 ~~ 翔 ~~  咖啡吧。。

![image](/img/post-img/703d37d8-cdb0-11e5-9e0d-e1cd222bc4ea.png)

我们先来看看官方给的demo

左边是 CoffeeScript, 右边是编译后输出的 JavaScript. 

![image](/img/post-img/84624a64-cdb0-11e5-9d74-42a0f18e0640.png)

代码量骤减啊，有木有！！其实他也大概讲到了一些常用的要点了。赋值，条件语句，函数，数组，对象，等。。。下面我就分开来仔细说说把。

---

CoffeeScript的单行注释是#，多行注释是###, 

# 声明

``` 
#编译前
array = [1, 2, 3, 4, 5, 6]
obj = 
  name: 'xxx'
  age: 10

//编译后    
var array, obj;

array = [1, 2, 3, 4, 5, 6];

obj = {
  name: 'xxx',
  age: 10
};
```

CoffeeScript里面的代码块都不需要花括号{}来包裹，而是通过换行和缩进来控制的。很优雅，right？ 

# 函数

函数声明

用一组可选的圆括号包裹的参数, 一个箭头, 一个函数体来定义函数

``` 
#编译前
square = (x) ->
  x * x

//编译后
var square;

square = function(x) {
  return x * x;
};

#没参数  包括参数的括号可要可不要
square = ->  # or square =() ->
  x * x

##多参数
square = (x, y) ->
  x * y

# 默认参数值  如果有多个参数的话，必填参数在前，默认参数在后！
square = (x, y = 2) ->
  x * y
```

## if,else,else if,unless

``` 
#编译前
if a
  a()
else if b
  b()
else
  c()

//编译后
if (a) {
  a();
} else if (b) {
  b();
} else {
  c();
}
```

什么?不够简洁??那么来试试if/unless后置写法吧.

``` 
#编译前
a() if a
b() unless a

//编译后
if (a) {
  a();
}

if (!a) {
  b();
}
```

不够后置写法只能单操作呢,,,,也就是不能判断后运行几条操作.

还有三目运算呢....

``` 
#编译前
date = if a then b else c

//编译后    
date = a ? b : c;
```

可读性好强把,

## 循环和推倒式

对象的遍历

对象的遍历只要拿到key和value就可以做爱做的事了。嘿嘿

先看操作前置写法：

``` 
#编译前
obj =
  name: 'xxx'
  age: 10

console.log key + ':' + value for key,value of obj


#编译后
var key, obj, value;

obj = {
  name: 'xxx',
  age: 10
};

for (key in obj) {
  value = obj[key];
  console.log(key + ':' + value);
}
```

多操作还是得这样写，注意缩进

``` 
#编译前
obj =
  name: 'xxx'
  age: 10

for key,value of obj
  console.log key + ':' + value
  alert key + ':' + value


#编译后
var key, obj, value;

obj = {
  name: 'xxx',
  age: 10
};

for (key in obj) {
  value = obj[key];
  console.log(key + ':' + value);
  alert(key + ':' + value);
}
```

如果你希望仅迭代在当前对象中定义的属性，通过hasOwnProperty检查并避免属性是继承来的，可以这样来写：

``` 
#编译前
obj =
  name: 'xxx'
  age: 10

for own key,value of obj
  console.log key + ':' + value


#编译后
var key, obj, value,
  __hasProp = {}.hasOwnProperty;

obj = {
  name: 'xxx',
  age: 10
};

for (key in obj) {
  if (!__hasProp.call(obj, key)) continue;
  value = obj[key];
  console.log(key + ':' + value);
}
```

### 注意：别搞错关键字了

``` 
for item in array
for key of obj
```

### 推导式

所谓的推导式其实就是在遍历数组进行操作的同时，将操作后的结果生成一个新的数组。注意啊，这里仅仅是操作数组，对象可不行。

看例子：将每个数组的每个元素进行+1

``` 
#编译前
array = [1, 2, 3, 4]

addOne = (item)->
  return item + 1

newArray = (addOne item for item in array)


#编译后
var addOne, array, item, newArray;

array = [1, 2, 3, 4];

addOne = function(item) {
  return item + 1;
};

newArray = (function() {
  var _i, _len, _results;
  _results = [];
  for (_i = 0, _len = array.length; _i < _len; _i++) {
    item = array[_i];
    _results.push(addOne(item));
  }
  return _results;
})();
```

推导式的代码就一行，但是编译到JavaScript，大家可以看到节省了大量的代码，而且从CoffeeScript代码上，我们一眼就看出了代码功能。

当然了，实际上推导式不可能就这样简单：遍历所有元素进行操作。有时候得进行一些过滤。CoffeeScript里面也提供了这些功能，看例子：

``` 
#编译前
array = [1, 2, 3, 4]

addOne = (item)->
  return item + 1

newArray = (addOne item for item,i in array)
newArray1 = (addOne item for item,i in array when i isnt 0) #过滤掉第一个元素
newArray2 = (addOne item for item,i in array when item > 3) #过滤掉小于4的元素
newArray3 = (addOne item for item,i in array by 2) #迭代的跨度


#编译后
var addOne, array, i, item, newArray, newArray1, newArray2, newArray3;

array = [1, 2, 3, 4];

addOne = function(item) {
  return item + 1;
};

newArray = (function() {
  var _i, _len, _results;
  _results = [];
  for (i = _i = 0, _len = array.length; _i < _len; i = ++_i) {
    item = array[i];
    _results.push(addOne(item));
  }
  return _results;
})();

newArray1 = (function() {
  var _i, _len, _results;
  _results = [];
  for (i = _i = 0, _len = array.length; _i < _len; i = ++_i) {
    item = array[i];
    if (i !== 0) {
      _results.push(addOne(item));
    }
  }
  return _results;
})();

newArray2 = (function() {
  var _i, _len, _results;
  _results = [];
  for (i = _i = 0, _len = array.length; _i < _len; i = ++_i) {
    item = array[i];
    if (item > 3) {
      _results.push(addOne(item));
    }
  }
  return _results;
})();

newArray3 = (function() {
  var _i, _len, _results;
  _results = [];
  for (i = _i = 0, _len = array.length; _i < _len; i = _i += 2) {
    item = array[i];
    _results.push(addOne(item));
  }
  return _results;
})();
```

### 相信大家也发现了，推导式都是写在一行的，如果要使用推导式，得将你的操作封装成一个函数供调用。

### 切片

CoffeeScript构建的思想，借鉴了很多Python和Ruby的。比如现在所说的切片功能。

切片其实就是对数组的截断，插入和删除操作。说白了就是用JavaScript的数组slice和splice函数操作数组。还是先简单说明一下这两函数吧。注意啊，这里讲的JavaScript。

slice(start,end)

-   功能：数组截取

-   参数： 

    -   start：开始位置
    -   end：结束位置

-   返回值：新的数组

      -注意：数组本身不发生变化

      -第一个参数是截取开始位置（数组下标是从0开始），第二个参数是结束位置，但是不包括结束位置。

      -如果只有一个参数，从开始位置截取到剩下所有数据

      -开始位置和结束位置也可以传递负数，负数表示倒着数。例如-3是指倒数第三个元素

      ```
      
      var array = [1, 2, 3, 4, 5];
      ```

var newArray = array.slice(1); //newArray: [2,3,4,5]

var newArray1 = array.slice(0, 3); //newArray1: [1,2,3]

var newArray2 = array.slice(0, -1); //newArray2: [1,2,3,4]

var newArray3 = array.slice(-3, -2); //newArray3: [3]

``` 
###splice(start,len,data...)

- 功能：数组插入数据、删除数据
- 参数： 
    - start：开始位置
    - len：截断的个数
    - data：插入的数据
- 返回值：截断的数据，是数组
-注意：操作的是数组本身
如果只有一个参数，从开始位置截取剩下所有数据
```

var array = [1, 2, 3, 4, 5]

var newArray = array.splice(1);//array=[1] newArray=[2,3,4,5]

var newArray1 = array.splice(0, 2);//array=[3,4,5] newArray1=[1,2]

var newArray2 = array.splice(0, 1, 6, 7);//array=[6,7,2,3,4,5] newArray2=[1]

``` 
好了，回到CoffeeScript，看看所谓的切片。
```

# 编译前

array = [1, 2, 3, 4]

newArray = array[0...2] #newArray=[1,2]

newArray1 = array[0..2] #newArray1=[1,2,3]

newArray2 = array[..] #newArray2=[1,2,3,4]

newArray3 = array[-3...-1] #newArray3=[2,3]



# 编译后

var array, newArray, newArray1, newArray2, newArray3;

array = [1, 2, 3, 4];

newArray = array.slice(0, 2);    

newArray1 = array.slice(0, 3);    

newArray2 = array.slice(0);    

newArray3 = array.slice(-3, -1);

``` 
## 注意：... 不包括结束位置的元素，.. 包括结束位置的元素
CoffeeScript里面是这样操作数组的。
```

# 编译前

array = [1, 2, 3, 4]

array[0...1] = [5] #array=[5,2,3,4]

array[0..1] = [5] #array=[5,3,4]



# 编译后

var array, _ref, _ref1;

array = [1, 2, 3, 4];

[].splice.apply(array, [0, 1].concat(_ref = [5])), _ref;

[].splice.apply(array, [0, 2].concat(_ref1 = [5])), _ref1;

```

其实就是把两个下标之间的元素替换成新的数据。同样的，... 不包括结束位置的元素，.. 包括结束位置的元素

## 运算符

![image](/img/post-img/35d1a54a-cdb3-11e5-944a-6bebb8ddc9e3.png)

---

好了~ 

coffeescript的知识点大概就这些了....漏了再补吧..哈哈哈....

happy hacking!
```