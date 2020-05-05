---
layout: post
title: 【坚果商城实战系列学习】第 1-2 课：ES6 简单入门
category: wechat
tags: [wechat]
keywords: 小程序,小程序云开发,微信小程序
excerpt: 第 1-2 课：ES6 简单入门
---

# 第 1-2 课：ES6 简单入门

本章的内容来自阮一峰的 [ ECMAScript  6  入门](http://es6.ruanyifeng.com/)选择部分在小程序开发中所会用到的知识点，大家需要学习更多ES6 知识可以参考我给的链接。

## 1 var 、 let 、 const
ES2015 ( ES6 ) 新增加了两个重要的  JavaScript  关键字:  **let**  和  **const** 。
let  声明的变量在  let 命令所在的代码块内有效。
const 声明一个只读的常量，一旦声明，常量的值就不能改变。
在 ES6 之前，JavaScript 只有两种作用域： **全局变量** 与 **函数内的局部变量**。

### 1.var
`var` 声明，无论它们出现在何处，都会在执行任何代码之前进行处理。 这称为提升，将在下面进一步讨论。
用 var 声明的变量的范围是它的当前执行上下文，它是封闭函数，或者对于在任何函数外部声明的变量，是全局的。 如果重新声明 JavaScript 变量，它将不会丢失其值。
在执行赋值时，为未声明的变量赋值会隐式地将其创建为全局变量（它将成为全局对象的属性）。 声明和未声明变量之间的差异是：
声明的变量在声明它们的执行上下文中受到约束。 未声明的变量总是全局的。
```
function test() {
  a = 10; // 在严格模式下引发引用错误
  var b = 20;
}
test();
console.log(a) // 10
var a = 100;  // 变量提升
console.log(a); // 100
console.log(b); // /引发引用错误：b不是在a之外定义的。
```
不是每个人都能很好的按照规范来做, 于是 ES6 推出了  let / const 来解决 `var 声明`的弊端 。因此使用 `var` ，建议始终在变量作用域的顶部(全局代码的顶部和函数代码的顶部)声明变量，这样就可以清楚地知道哪些变量是函数作用域(本地)，哪些是在作用域链上解析的。

###  2.let

ES6 新增了 `let` 命令，用来声明变量。它的用法类似于 `var` ，但是所声明的变量，只在 `let` 命令所在的代码块内有效。 
```
{
  let a = 10;
  var b = 1;
}

a // ReferenceError: a is not defined.
b // 1
```
上面代码在代码块之中，分别用 `let` 和 `var` 声明了两个变量。然后在代码块之外调用这两个变量，结果 `let` 声明的变量报错，`var` 声明的变量返回了正确的值。这表明，`let` 声明的变量只在它所在的代码块有效。 
`for` 循环的计数器，就很合适使用 `let` 命令。 

```
for (let i = 0; i < 10; i++) {
  // ...
}
console.log(i); // ReferenceError: i is not defined
```
上面代码中，计数器 `i` 只在 `for` 循环体内有效，在循环体外引用就会报错。 
另外，`for` 循环还有一个特别之处，就是设置循环变量的那部分是一个父作用域，而循环体内部是一个单独的子作用域。 
```
for (let i = 0; i < 3; i++) {
  let i = 'abc';
  console.log(i);
}
// abc
// abc
// abc
```

上面代码正确运行，输出了 3 次  `abc` 。这表明函数内部的变量 `i` 与循环变量 `i` 不在同一个作用域，有各自单独的作用域。 

### 1.3  不存在变量提升

`var` 命令会发生“变量提升”现象，即变量可以在声明之前使用，值为 `undefined` 。这种现象多多少少是有些奇怪的，按照一般的逻辑，变量应该在声明语句之后才可以使用。
为了纠正这种现象， `let` 命令改变了语法行为，它所声明的变量一定要在声明后使用，否则报错。
```
// var 的情况
console.log(foo); // 输出undefined
var foo = 2;

// let 的情况
console.log(bar); // 报错ReferenceError
let bar = 2;
```

上面代码中，变量 `foo` 用 `var` 命令声明，会发生变量提升，即脚本开始运行时，变量 `foo`已经存在了，但是没有值，所以会输出 `undefined` 。变量 `bar` 用 `let` 命令声明，不会发生变量提升。这表示在声明它之前，变量 `bar` 是不存在的，这时如果用到它，就会抛出一个错误。

### 1.4  暂时性死区

只要块级作用域内存在 `let` 命令，它所声明的变量就“绑定”（ binding ）这个区域，不再受外部的影响。
```
var tmp = 123;

if (true) {
  tmp = 'abc'; // ReferenceError
  let tmp;
}
```
上面代码中，存在全局变量 `tmp` ，但是块级作用域内 `let` 又声明了一个局部变量 `tmp` ，导致后者绑定这个块级作用域，所以在 `let` 声明变量前，对 `tmp` 赋值会报错。

ES6 明确规定，如果区块中存在 `let ` 和  `const` 命令，这个区块对这些命令声明的变量，从一开始就形成了封闭作用域。凡是在声明之前就使用这些变量，就会报错。

### 1.5  不允许重复声明

`let` 不允许在相同作用域内，重复声明同一个变量。
```
// 报错
function func() {
  let a = 10;
  var a = 1;
}

// 报错
function func() {
  let a = 10;
  let a = 1;
}
```
因此，不能在函数内部重新声明参数。
##2 const 命令

### 2.1 基本用法

`const` 声明一个只读的常量。一旦声明，常量的值就不能改变。

```
const PI = 3.1415;

PI // 3.1415

PI = 3;// TypeError: Assignment to constant variable.

```

上面代码表明改变常量的值会报错。

`const` 声明的变量不得改变值，这意味着，`const` 一旦声明变量，就必须立即初始化，不能留到以后赋值。

```
const foo;// SyntaxError: Missing initializer in const declaration
```

上面代码表示，对于 `const` 来说，只声明不赋值，就会报错。

`const `的作用域与 `let` 命令相同：只在声明所在的块级作用域内有效。

```
if (true) {
  const MAX = 5;
}

MAX // Uncaught ReferenceError: MAX is not defined
```

`const` 命令声明的常量也是不提升，同样存在暂时性死区，只能在声明的位置后面使用。

## 3 箭头函数

### 3.1 基本用法

ES6 允许使用“箭头”（`=>`）定义函数。

```
 var f=(a,b)=>{ return a+b}
 console.log(f(10,20));  //30
```

如果箭头函数不需要参数或需要多个参数，就使用一个圆括号代表参数部分。

```
var f = () => 5;
// 等同于
var f = function () { return 5 };

var sum = (a, b) => a + b;
// 等同于
var sum = function(a, b) {
  return a + b;
};
```

如果箭头函数的代码块部分多于一条语句，就要使用大括号将它们括起来，并且使用`return`语句返回。

### 3.2 使用注意点

箭头函数有几个使用注意点。

（1）函数体内的 `this` 对象，就是定义时所在的对象，而不是使用时所在的对象。

（2）不可以当作构造函数，也就是说，不可以使用 `new` 命令，否则会抛出一个错误。

（3）不可以使用 `arguments` 对象，该对象在函数体内不存在。如果要用，可以用 rest 参数代替。

（4）不可以使用 `yield` 命令，因此箭头函数不能用作 Generator 函数。

上面四点中，第一点尤其值得注意。`this` 对象的指向是可变的，但是在箭头函数中，它是固定的。

```
function foo() {
  setTimeout(() => {
    console.log('id:', this.id);
  }, 100);
}

var id = 10;

foo.call({ id: 20 });
```

## 4 数组的扩展

### 4.1 扩展运算符

扩展运算符（ spread ）是三个点（`...`）。它好比 rest 参数的逆运算，将一个数组转为用逗号分隔的参数序列。

```
let data = [1, 2, 3]
console.log(...data)
// 1 2 3
```

该运算符主要用于函数调用。

```
function add(x, y) {
  return x + y;
}

const data = [10, 20];
console.log(add(...data)) // 30
```

## 5 class类

ES6 提供了更接近传统语言的写法，引入了 Class（类）这个概念，作为对象的模板。通过 `class` 关键字，可以定义类。

基本上，ES6 的 `class` 可以看作只是一个语法糖，它的绝大部分功能，ES5 都可以做到，新的 `class` 写法只是让对象原型的写法更加清晰、更像面向对象编程的语法而已。上面的代码用 ES6 的 `class` 改写，就是下面这样。

```
class Point {
  constructor(a, b) {
    this.a = a;
    this.b = b;
  }

  toString() {
    return '(' + this.a + ', ' + this.b + ')';
  }
}
```

上面代码定义了一个“类”，可以看到里面有一个 `constructor` 方法，这就是构造方法，而 `this` 关键字则代表实例对象。也就是说，ES5 的构造函数 `Point`，对应 ES6 的 `Point` 类的构造方法。 

### 5.1 constructor

可以看到里面有一个 constructor 方法，这就是构造方法，而 this 关键字则代表实例对象。这里面通常保存着类基本数据类型

### 5.2 定义类方法

我们可以直接在类中添加自己的方法，前面不需要加上 function 这个关键字，另外，为了使类更加的符合大众的写法，去掉了逗号分隔，我们不需要在方法之间使用逗号进行分隔。

### 5.3 类的继承

Class 可以通过 `extends` 关键字实现继承，这比 ES5 的通过修改原型链实现继承，要清晰和方便很多。 

```
 class A {
       constructor(name) {
           this.name = name;
       }
       say() {
           console.log(`name is${this.name}`);
       }
   }
   class B extends A {
       constructor(name, age) {
           super(name)
           this.age = age
       }
       say() {
           console.log(`name :${this.name}  age:${this.age}`);
       }
   }
   var a = new A('test')
   var b = new B('test', 1)
   a.say()
   b.say()
```

注意：使用 extends 关键字来实现继承在子类中的构造器 constructor 中，必须要显式调用父类的 super 方法，如果不调用，则 this 不可用。

## 6 Promise 对象

 基本用法

ES6 规定，`Promise` 对象是一个构造函数，用来生成 `Promise` 实例。

下面代码创造了一个 `Promise `实例。

```
const promise = new Promise((resolve, reject) => {
  let rand = Math.random();
  if(rand<0.6){
     resolve("success");
  } else {
    reject("error");
  }
});
```

`Promise` 构造函数接受一个函数作为参数，该函数的两个参数分别是 `resolve` 和 `reject`  。它们是两个函数，由  JavaScript 引擎提供，不用自己部署。

`resolve` 函数的作用是，将 `Promise`  对象的状态从“未完成”变为“成功”（即从 pending 变为 resolved ），在异步操作成功时调用，并将异步操作的结果，作为参数传递出去；`reject` 函数的作用是，将 `Promise` 对象的状态从“未完成”变为“失败”（即从  pending  变为  rejected ），在异步操作失败时调用，并将异步操作报出的错误，作为参数传递出去。
`Promise` 实例生成以后，可以用 `then` 方法分别指定 `resolved` 状态和 `rejected` 状态的回调函数。
```
promise.then((res) =>{
  // success
}, (error)=> {
  // fail
});
```
`then` 方法可以接受两个回调函数作为参数。第一个回调函数是 `Promise` 对象的状态变为`resolved` 时调用，第二个回调函数是 `Promise` 对象的状态变为 `rejected` 时调用。其中，第二个函数是可选的，不一定要提供。这两个函数都接受 `Promise` 对象传出的值作为参数。
下面是一个 `Promise` 对象的简单例子。
```
function timeout(ms) {
  return new Promise((resolve, reject) => {
    setTimeout(resolve, ms, 'done');
  });
}

timeout(100).then((value) => {
  console.log(value);
});
```
上面代码中，`timeout` 方法返回一个 `Promise` 实例，表示一段时间以后才会发生的结果。过了指定的时间（ `ms` 参数）以后，`Promise` 实例的状态变为 `resolved` ，就会触发 `then` 方法绑定的回调函数。

Promise  新建后就会立即执行。
```
let promise = new Promise((resolve, reject)=> {
  console.log('Promise...');
  resolve();
});

promise.then(() => {
  console.log('resolved...');
});

console.log('this is promise!');

// Promise...
// this is promise!!
// resolved...
```
上面代码中，Promise 新建后立即执行，所以首先输出的是 `Promise` 。然后， `then` 方法指定的回调函数，将在当前脚本所有同步任务执行完才会执行，所以 `resolved` 最后输出。

## 代码示例

本文示例代码访问下面查看仓库：

- github： [https://github.com/mtcarpenter/nux-shop](https://github.com/mtcarpenter/nux-shop)
- gitee :  [https://gitee.com/mtcarpenter/nux-shop](https://gitee.com/mtcarpenter/nux-shop)