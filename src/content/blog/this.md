---
title: 'this+apply+bind+call'
description: 'this指向及apply、bind、call的手写'
pubDate: 'Sept 13 2023'
heroImage: '../../assets/images/te.jpg'
category: 'Basic'
tags: ['JavaScript']
---

## this

### 是什么,各自有什么情况

> 本质上是个对象，谁调用就指谁，所以使用call/bind/apply时通过this获取调用的对象

- 全局对象中的this指向全局，严格模式下是undefined
- 函数的this在函数调用时已经确定，fun()，其中fun是调用者，如果fun被其他对象拥有，比如window.fun()，那么this指向拥有者window(最后一个调用的)，否则按第一种情况判断
- 对象的this调用由于{}不会形成单独的作用域所以指的是全局对象(注意：区分作用域【定义时确立】和this指向【执行时且可以被改变】)
- apply/bind/call等用于灵活改变this指向，第一个参数为this的真正指向对象
- new构造函数this指向实例出来的对象
- 箭头函数this指向同外层作用域，且指向函数**定义**时的this而非执行时(关联知识点：箭头函数跟普通函数的区别)

有两种情况容易发生`隐式丢失问题`：

- 使用另一个变量来给函数取别名
- 将函数作为参数传递时会被隐式赋值，回调函数丢失this绑定，普通传参及setTimeout均为此种情况

情况一：变量别名导致隐式丢失

```javascript
function foo () {
  console.log(a); // 这里的a表示的是作用域下的a
  console.log(this.a); // 这里的a是this对象中的a属性
}
// 注意这里的foo函数不能使用箭头函数,不然直接指向定义时外层的this即window
var obj = { a: 1, foo }
var a = 2; 
obj.foo(); // obj调用的foo,this指向obj,打印1
var foo2 = obj.foo;
foo2(); // 隐式丢失,this指向的是window
```

![img](https://moon345.oss-cn-hangzhou.aliyuncs.com/img/599584-391af3aad043c028.png)

### 关于为什么存在this

> JavaScript中的`this`关键字用于指向当前正在执行的函数的上下文对象。它的存在是为了提供对当前上下文的访问权限和操作。

[【建议👍】再来40道this面试题酸爽继续(1.2w字用手整理)](https://juejin.cn/post/6844904083707396109)

## 关于call/bind/apply

- 使用`.call()`或者`.apply()`的函数是会直接执行的，即obj1.fun.call(obj2)等价于obj1.fun()，this为obj2
- `bind()`是创建一个新的函数，需要手动调用才会执行，返回一个bound函数
- `.call()`和`.apply()`用法基本类似，不过`call`接收若干个参数，而`apply`接收的是一个数组
- 如果`call、apply、bind`接收到的第一个参数是空或者`null、undefined`的话，则会忽略这个参数。
- :star:`forEach、map、filter`函数的第二个参数也是能显式绑定`this`的

### call

call() 方法在使用一个指定的 this 值和若干个指定的参数值的前提下调用某个函数或方法。

[非常好理解](https://juejin.cn/post/6844903476477034510)

> 1. 将函数设为对象的属性
> 2. 执行该函数
> 3. 删除该函数

```javascript
Function.prototype.myCall = function (context, ...args) {
  // 获取调用call的函数
  var fn = this;
  var result;

  // 如果没有传递上下文，则默认为全局对象（非严格模式下）
  context = context || window;

  // 为上下文对象创建一个唯一的属性，用于保存要调用的函数
  var uniqueProp = Symbol('call');
  context[uniqueProp] = fn;

  // 执行函数
  result = context[uniqueProp](...args);

  // 删除临时创建的函数属性
  delete context[uniqueProp];

  return result;
};
```

### apply

```javascript
Function.prototype.myApply = function(context, argsArray) {
  // 保存原始函数
  var fn = this;
  var result;
  
  // 如果没有传递上下文对象，则默认为全局对象（非严格模式下）
  context = context || window;
  
  // 为上下文对象创建一个唯一的属性，用于保存要调用的函数
  var uniqueProp = Symbol('apply');
  context[uniqueProp] = fn;
  
  // 执行函数
  if (argsArray) {
    // 使用展开操作符 `...` 将参数数组传递给函数
    result = context[uniqueProp](...argsArray);
  } else {
    result = context[uniqueProp]();
  }
  
  // 删除临时创建的函数属性
  delete context[uniqueProp];
  
  return result;
};
```

### bind

```javascript
// 第一版 修改this指向，合并参数
Function.prototype.bindFn = function bind(thisArg){
  if(typeof this !== 'function'){
      throw new TypeError(this + 'must be a function');
  }
  // 存储函数本身
  var self = this;
  // 去除thisArg的其他参数 转成数组
  var args = [].slice.call(arguments, 1);
  var bound = function(){
      // bind返回的函数 的参数转成数组
      var boundArgs = [].slice.call(arguments);
      // apply修改this指向，把两个函数的参数合并传给self函数，并执行self函数，返回执行结果
      // 内外arguments不同
      return self.apply(thisArg, args.concat(boundArgs));
  }
  return bound;
}
// 测试
var obj = {
  name: '若川',
};
function original(a, b){
  console.log(this.name);
  console.log([a, b]);
}
var bound = original.bindFn(obj, 1);
bound(2); // '若川', [1, 2]
```

### ES3写法

```javascript
var args = ['1', 'dierge'];
var fnStr = 'context.fn(';
for (var i = 0; i < args.length; i++) {
  fnStr += i == args.length - 1 ? args[i] : args[i] + ',';
}
fnStr += ')';
console.log(fnStr); // context.fn(1,dierge);
```
