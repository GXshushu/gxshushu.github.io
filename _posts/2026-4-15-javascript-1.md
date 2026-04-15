---
title: JavaScript 学习笔记 - var, let, 名字空间与解构赋值
date: 2026-04-15 14:19:00 +0800
categories: [JavaScript]
tags: [JavaScript 学习]
pin: false
author: C9xx

toc: true
comments: true
typora-root-url: ../../gxshushu.github.io
math: false
mermaid: true

---

[廖雪峰的官方网站 - JavaScript - 变量作用域与结构赋值](https://liaoxuefeng.com/books/javascript/function/scope/index.html)

## var
var的作用域和Python有点相似，分为函数作用域和全局作用域，几乎和Python下的local范围和module范围相对应。

表现为：
- 嵌套函数内部的变量引用从里向外查找，直到找到变量为止。
- 如果在嵌套最里层函数内部没有声明变量，类似于自动有个nonlocal或global声明。
- JavaScript的函数定义有个特点，它会先扫描整个函数体的语句，把所有用var申明的变量“提升”到函数顶部，但赋值在后面，这可能会出现问题。
- 在函数内部定义变量时，请严格遵守“在函数内部首先申明所有变量”这一规则，最常见的做法是用一个var申明函数内部用到的所有变量。

```JavaScript
function foo() {
    var
        x = 1, // x初始化为1
        y = x + 1, // y初始化为2
        z, i; // z和i为undefined
    // 其他语句:
    for (i=0; i<100; i++) {
        ...
    }
}
```
## let
如果不需要兼容低版本浏览器，完全可以用let代替var来申明变量。
同样，let变量可以在全局作用域声明，也就是不在任何函数内定义。实际上，JavaScript默认有一个全局对象window，全局作用域的变量实际上被绑定到window的一个属性：

```JavaScript
let course = 'Learn JavaScript';
console.log(course); // 'Learn JavaScript'
console.log(window.course); // undefined
```

顶层函数的定义也是一种全局变量，也是绑定为windows对象的一个属性。
```JavaScript
// 方式一
function foo() {}

// 方式二
var foo = function () {}
```

alert参数也是windows对象的一个属性。

```JavaScript
alert('hello world');
```

JavaScript实际上只有一个全局作用域。任何变量（函数也视为变量），如果没有在当前函数作用域中找到，就会继续往上查找，最后如果在全局作用域中也没有找到，则报ReferenceError错误。

另外，let还可以和其他常用的后端语言一样定义块级作用域。

```JavaScript
function foo() {
    let sum = 0;
    for (let i=0; i<100; i++) {
        sum += i;
    }
    // SyntaxError:
    i += 1;
}
```

# 名字空间
全局变量都会绑定到windows下面，如果有多个js文件，可能会有命名冲突。为了避免命名冲突，可以使用名字空间来隔离变量。

```JavaScript
// 唯一的全局变量MYAPP:
let MYAPP = {};

// 其他变量:
MYAPP.name = 'myapp';
MYAPP.version = 1.0;

// 其他函数:
MYAPP.foo = function () {
    return 'foo';
};
```
只要保证别的JS文件没有MYAPP全局变量，我们大量的变量都绑定到这个MYAPP上，就可以避免命名冲突。

# 常量(const)
常量用`const PI = 3.14;`定义

# 解构赋值
从ES6开始，JavaScript引入了解构赋值，可以同时对一组变量进行赋值。`let [x, y, z] = ['hello', 'JavaScript', 'ES6'];`
```JavaScript
let [x, [y, z]] = ['hello', ['JavaScript', 'ES6']];
x; // 'hello'
y; // 'JavaScript'
z; // 'ES6'

let [, , z] = ['hello', 'JavaScript', 'ES6']; // 忽略前两个元素，只对z赋值第三个元素
z; // 'ES6'
```
对一个对象进行解构赋值时，同样可以直接对嵌套的对象属性进行赋值，只要保证对应的层次是一致的：
```JavaScript
let person = {
    name: '小明',
    age: 20,
    gender: 'male',
    passport: 'G-12345678',
    school: 'No.4 middle school',
    address: {
        city: 'Beijing',
        street: 'No.1 Road',
        zipcode: '100001'
    }
};
let {name, address: {city, zip}} = person;
name; // '小明'
city; // 'Beijing'
zip; // undefined, 因为属性名是zipcode而不是zip
// 注意: address不是变量，而是为了让city和zip获得嵌套的address对象的属性:
address; // Uncaught ReferenceError: address is not defined
```  
使用解构赋值对对象属性进行赋值时，如果对应的属性不存在，变量将被赋值为undefined，这和引用一个不存在的属性获得undefined是一致的。如果要使用的变量名和属性名不一致

```JavaScript
let person = {
    name: '小明',
    age: 20,
    gender: 'male',
    passport: 'G-12345678',
    school: 'No.4 middle school'
};

// 把passport属性赋值给变量id:
let {name, passport:id} = person;
name; // '小明'
id; // 'G-12345678'
// 注意: passport不是变量，而是为了让变量id获得passport属性:
passport; // Uncaught ReferenceError: passport is not defined
```

解构赋值还可以使用默认值，这样就避免了不存在的属性返回undefined的问题
```JavaScript
let person = {
    name: '小明',
    age: 20,
    gender: 'male',
    passport: 'G-12345678'
};

// 如果person对象没有single属性，默认赋值为true:
let {name, single=true} = person;
name; // '小明'
single; // true

```

解构赋值的使用场景
- 交换值
```JavaScript
let x=1, y=2;
[x, y] = [y, x]
```

- 快速获取当前页面的域名和路径
```JavaScript
let {hostname:domain, pathname:path} = location;
```

- 如果一个函数接收一个对象作为参数，那么，可以使用解构直接把对象的属性绑定到变量中。例如，下面的函数可以快速创建一个Date对象：
```JavaScript
function buildDate({year, month, day, hour=0, minute=0, second=0}) {
    return new Date(`${year}-${month}-${day} ${hour}:${minute}:${second}`);
}

buildDate({ year: 2017, month: 1, day: 1 });
// Sun Jan 01 2017 00:00:00 GMT+0800 (CST)

buildDate({ year: 2017, month: 1, day: 1, hour: 20, minute: 15 });
// Sun Jan 01 2017 20:15:00 GMT+0800 (CST)
```