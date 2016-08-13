---
layout:     post
title:      "《ECMAScript 6入门》笔记"
date:       2016-03-16
author:     "J3n5en"
tags:
    - front-end
    - ES6
    - Javascript
---

# let & const

---
## let命令

let类似于var，都用来声明变量。不过let只在它所在的代码块中起作用。

### 常用场景：

for循环的计数器。

```javascript
for(let i = 0;i<arr.length;i++)
```

### 注意：

-  不存在变量提升（变量要先声明再使用）
-  暂时性死区（本质就是，只要一进入当前作用域，所要使用的变量就已经存在了，但是不可获取，只有等到声明变量的那一行代码出现，才可以获取和使用该变量。）
-  ES6允许块级作用域的任意嵌套.外层作用域无法读取内层作用域的变量。但是内层作用域可以读取外层作用域的变量。

## const 命令

const用来声明常量。一旦声明常量的值就不能改变。也就是说一旦用const声明就要立即初始化，不能在后面再进行赋值。

## let与const类比

const的作用域与let命令相同：只在声明所在的块级作用域内有效。const命令声明的常量也是不提升，同样存在暂时性死区，只能在声明的位置后面使用。const声明的常量，也与let一样不可重复声明。对于复合类型的变量，变量名不指向数据，而是指向数据所在的地址。const命令只是保证变量名指向的地址不变，并不保证该地址的数据不变，所以将一个对象声明为常量必须非常小心。

```javascript
const a = [];
a.push("Hello"); // 可执行
a.length = 0;    // 可执行
a = ["J3n5en"];    // 报错
```

## 跨模块常量
const声明的常量只在当前代码块有效。如果想设置跨模块的常量，可以采用下面的写法。
```javascript
// constants.js 模块
export const A = 1;
export const B = 3;
export const C = 4;

// test1.js 模块
import * as constants from './constants';
console.log(constants.A); // 1
console.log(constants.B); // 3

// test2.js 模块
import {A, B} from './constants';
console.log(A); // 1
console.log(B); // 3
```

# 变量的解构赋值

---

## 基本用法

ES6允许按照一定模式，从数组和对象中提取值，对变量进行赋值，这被称为解构（Destructuring）。

```javascript
var a = 1;
var b = 2;
var c = 3;
```

ES6允许写成下面这样。

```javascript
var [a, b, c] = [1, 2, 3];
```

解构赋值允许指定默认值。

```javascript
var [foo = true] = [];
foo // true

[x, y = 'b'] = ['a'] // x='a', y='b'
[x, y = 'b'] = ['a', undefined] // x='a', y='b'
```

解构不仅可以用于数组，还可以用于对象.对象的解构与数组有一个重要的不同。**数组的元素是按次序排列的，变量的取值由它的位置决定；而对象的属性没有次序，变量必须与属性同名**，才能取到正确的值。
```javascript
var { bar, foo } = { foo: "aaa", bar: "bbb" };
foo // "aaa"
bar // "bbb"

var { baz } = { foo: "aaa", bar: "bbb" };
baz // undefined
```
如果变量名与属性名不一致，必须写成下面这样。

```javascript
var { foo: baz } = { foo: "aaa", bar: "bbb" };
baz // "aaa"
let obj = { first: 'hello', last: 'world' };
let { first: f, last: l } = obj;
f // 'hello'
l // 'world'
```

和数组一样，解构也可以用于嵌套结构的对象。对象的解构也可以指定默认值。

字符串也可以解构赋值。这是因为此时，字符串被转换成了一个类似数组的对象。

```javascript
const [a, b, c, d, e] = 'hello';
a // "h"
b // "e"
c // "l"
d // "l"
e // "o"
```

数值和布尔值的解构赋值时，如果等号右边是数值和布尔值，则会先转为对象。

```javascript
let {toString: s} = 123;
s === Number.prototype.toString // true

let {toString: s} = true;
s === Boolean.prototype.toString // true
```

### 函数参数的解构赋值

函数的参数也可以使用解构赋值。

```javascript
function add([x, y]){
  return x + y;
}

add([1, 2]) // 3
```

## 使用场景

- 交换变量的值 `[x, y] = [y, x];`-855**
- 从函数返回多个值
- 函数参数的定义
- 提取JSON数据 !! 
- 遍历Map结构




>   Happy Hacking !

\# EOF \#