---
layout:     post
title:      深入分析JS内置对象(一)
subtitle:   JavaScript标准库内置对象分析之Function的apply()
date:       2019-07-01
author:     Aro
header-img: img/post-bg-js-version.jpg
catalog: true
tags:
    - JS
    - 前端
---


## `Function.prototype.apply()`

`apply()` 方法调用一个具有给定 `this` 值的函数，以及作为一个数组（或类似数组对象）提供的参数。

>注意： `call()` 方法的作用和 `apply()` 方法类似，区别就是 `call()` 方法接受的是参数列表，而 `apply()` 方法接受的是一个参数数组。

**举几个简单例子来说一说 `apply()` 的使用:**

#### 求数组的最大值

```js
var numbers = [5, 6, 2, 3, 7];

var max = Math.max.apply(null, numbers);

console.log(max);
// expected output: 7

var min = Math.min.apply(null, numbers);

console.log(min);
// expected output: 2
```

#### 自己实现一个效果

```js
var obj = {
    name : 'add'
}

function func(firstName, lastName){
    console.log(firstName + ' ' + this.name + ' ' + lastName);
}

func.apply(obj, ['A', 'B']);    // A add B
```

可以看到， `obj` 是作为函数上下文的对象，函数 `func` 中 `this` 指向了 `obj` 这个对象。参数 A 和 B 是放在数组中传入 `func` 函数，分别对应 `func` 参数的列表元素。


#### 对比 `call()`

call 方法第一个参数也是作为函数上下文的对象，但是后面传入的是一个参数列表，而不是单个数组。

```js
var obj = {
    name: 'add'
}

function func(firstName, lastName) {
    console.log(firstName + ' ' + this.name + ' ' + lastName);
}

func.call(obj, 'C', 'D');       // C add D
```

对比 `apply` 我们可以看到区别，C 和 D 是作为单独的参数传给 `func` 函数，而不是放到数组中。

对于什么时候该用什么方法，其实不用纠结。如果你的参数本来就存在一个数组中，那自然就用 `apply`，如果参数比较散乱相互之间没什么关联，就用 `call`。


## 语法

>func.apply(thisArg, [argsArray])

#### 参数

**thisArg:**
可选的。在 `func` 函数运行时使用的 `this` 值。请注意， `this`可能不是该方法看到的实际值：如果这个函数处于非严格模式下，则指定为 `null` 或 `undefined` 时会自动替换为指向全局对象，原始值会被包装。

**argsArray:**
可选的。一个数组或者类数组对象，其中的数组元素将作为单独的参数传给 `func` 函数。如果该参数的值为 `null` 或`undefined` ，则表示不需要传入任何参数。

#### 返回值

调用有指定 `this` 值和参数的函数的结果。


## 描述

在调用一个存在的函数时，你可以为其指定一个 `this` 对象。 `this` 指当前对象，也就是正在调用这个函数的对象。 使用 `apply`， 你可以只写一次这个方法然后在另一个对象中继承它，而不用在新对象中重复写该方法。

`apply()` 与 `call()` 非常相似，不同之处在于提供参数的方式。 `apply` 使用参数数组而不是一组参数列表。 `apply` 可以使用数组字面量 `（array literal）` ，如  `fun.apply(this, ['eat', 'bananas'])` ，或数组对象， 如  `fun.apply(this, new Array('eat', 'bananas'))`。

你也可以使用 `arguments` 对象作为 `argsArray` 参数。 `arguments` 是一个函数的局部变量。 它可以被用作被调用对象的所有未指定的参数。 这样，你在使用apply函数的时候就不需要知道被调用对象的所有参数。 你可以使用arguments来把所有的参数传递给被调用对象。 被调用对象接下来就负责处理这些参数。

从 ECMAScript 第5版开始，可以使用任何种类的类数组对象，就是说只要有一个 length 属性和(0..length-1)范围的整数属性。例如现在可以使用 NodeList 或一个自己定义的类似 {'length': 2, '0': 'eat', '1': 'bananas'} 形式的对象。


>需要注意：Chrome 14 以及 Internet Explorer 9 仍然不接受类数组对象。如果传入类数组对象，它们会抛出异常。


## 再举几个例子🌰

#### 用 `apply` 将数组添加到另一个数组

我们可以使用 `push` 将元素追加到数组中。并且，因为push接受可变数量的参数，我们也可以一次推送多个元素。但是，如果我们传递一个数组来推送，它实际上会将该数组作为单个元素添加，而不是单独添加元素，因此我们最终得到一个数组内的数组。如果那不是我们想要的怎么办？在这种情况下， `concat` 确实具有我们想要的行为，但它实际上并不附加到现有数组，而是创建并返回一个新数组。 但是我们想要附加到我们现有的阵列......那么现在呢？ 写一个循环？当然不是吗？

apply来帮你！

```js
var array = ['a', 'b'];
var array2 = [0, 1, 2];
array.push.apply(array, array2);
console.info(array); // ["a", "b", 0, 1, 2]
```

#### 使用 `apply` 和内置函数

聪明的apply用法允许你在某些本来需要写成遍历数组变量的任务中使用内建的函数。在接下里的例子中我们会使用Math.max/Math.min来找出一个数组中的最大/最小值。

```js
/* 找出数组中最大/小的数字 */
var numbers = [5, 6, 2, 3, 7];

/* 应用(apply) Math.min/Math.max 内置函数完成 */
var max = Math.max.apply(null, numbers); /* 基本等同于 Math.max(numbers[0], ...) 或 Math.max(5, 6, ..) */
var min = Math.min.apply(null, numbers);

/* 代码对比： 用简单循环完成 */
max = -Infinity, min = +Infinity;

for (var i = 0; i < numbers.length; i++) {
  if (numbers[i] > max)
    max = numbers[i];
  if (numbers[i] < min) 
    min = numbers[i];
}

/* 代码对比es6使用数组的扩展 */
var max_2 = Math.max(...numbers);

console.log(max_2);
// expected output: 7
```

但是当心：如果用上面的方式调用apply，会有超出JavaScript引擎的参数长度限制的风险。当你对一个方法传入非常多的参数（比如一万个）时，就非常有可能会导致越界问题, 这个临界值是根据不同的 JavaScript 引擎而定的（JavaScript 核心中已经做了硬编码  参数个数限制在65536），因为这个限制（实际上也是任何用到超大栈空间的行为的自然表现）是未指定的. 有些引擎会抛出异常。更糟糕的是其他引擎会直接限制传入到方法的参数个数，导致参数丢失。举个例子：如果某个引擎限制了方法参数最多为4个（实际真正的参数个数限制当然要高得多了, 这里只是打个比方）, 上面的代码中, 真正通过 apply传到目标方法中的参数为 5, 6, 2, 3 而不是完整的数组。

如果你的参数数组可能非常大，那么推荐使用下面这种策略来处理：将参数数组切块后循环传入目标方法：

```js
function minOfArray(arr) {
  var min = Infinity;
  var QUANTUM = 32768;

  for (var i = 0, len = arr.length; i < len; i += QUANTUM) {
    var submin = Math.min.apply(null, arr.slice(i, Math.min(i + QUANTUM, len)));
    min = Math.min(submin, min);
  }

  return min;
}

var min = minOfArray([5, 6, 2, 3, 7]);
```

#### 使用 `apply` 来链接构造器

你可以使用apply来链接一个对象构造器，类似于Java。在接下来的例子中我们会创建一个全局 `Function` 对象的construct方法 ，来使你能够在构造器中使用一个类数组对象而非参数列表。

```js
Function.prototype.construct = function (aArgs) {
  var oNew = Object.create(this.prototype);
  this.apply(oNew, aArgs);
  return oNew;
};
```

#### 改变 `this` 的指向

```js
var obj = {
    name: 'aro'
}

function func() {
    console.log(this.name);
}

func.call(obj);       // aro
```

我们知道，`call` 方法的第一个参数是作为函数上下文的对象，这里把 `obj` 作为参数传给了 `func`，此时函数里的 `this` 便指向了 `obj` 对象。此处 `func` 函数里其实相当于

```js
function func() {
    console.log(obj.name);
}
```

#### 借用别的对象的方法

```js
var Person1  = function () {
    this.name = 'aro';
}
var Person2 = function () {
    this.getname = function () {
        console.log(this.name);
    }
    Person1.call(this);
}
var person = new Person2();
person.getname();       // aro
```

从上面我们看到，Person2 实例化出来的对象 person 通过 getname 方法拿到了 Person1 中的 name。因为在 Person2 中，Person1.call(this) 的作用就是使用 Person1 对象代替 this 对象，那么 Person2 就有了 Person1 中的所有属性和方法了，相当于 Person2 继承了 Person1 的属性和方法。


#### 调用函数

apply、call 方法都会使函数立即执行，因此它们也可以用来调用函数。

```js
function func() {
    console.log('aro');
}
func.call();            // aro
```


## `call` 和 `bind` 的区别

在 EcmaScript5 中扩展了叫 bind 的方法，在低版本的 IE 中不兼容。它和 call 很相似，接受的参数有两部分，第一个参数是是作为函数上下文的对象，第二部分参数是个列表，可以接受多个参数。
它们之间的区别有以下两点。

#### `bind` 发返回值是函数

```js
var obj = {
    name: 'aro'
}

function func() {
    console.log(this.name);
}

var func1 = func.bind(obj);
func1();                        // aro
```

`bind` 方法不会立即执行，而是返回一个改变了上下文 `this` 后的函数。而原函数 `func` 中的 `this` 并没有被改变，依旧指向全局对象 `window`。


#### 参数的使用

```js
function func(a, b, c) {
    console.log(a, b, c);
}
var func1 = func.bind(null,'aro');

func('A', 'B', 'C');            // A B C
func1('A', 'B', 'C');           // aro A B
func1('B', 'C');                // aro B C
func.call(null, 'aro');      // aro undefined undefined
```

`call` 是把第二个及以后的参数作为 `func` 方法的实参传进去，而 `func1` 方法的实参则是在 `bind` 中参数的基础上再往后排。

#### 为低版本浏览器实现一个bind(Polyfill)

```js
if (!Function.prototype.bind) {
  Function.prototype.bind = function(oThis) {
    if (typeof this !== 'function') {
      // closest thing possible to the ECMAScript 5
      // internal IsCallable function
      throw new TypeError('Function.prototype.bind - what is trying to be bound is not callable');
    }

    var aArgs   = Array.prototype.slice.call(arguments, 1),
        fToBind = this,
        fNOP    = function() {},
        fBound  = function() {
          // this instanceof fBound === true时,说明返回的fBound被当做new的构造函数调用
          return fToBind.apply(this instanceof fBound
                 ? this
                 : oThis,
                 // 获取调用时(fBound)的传参.bind 返回的函数入参往往是这么传递的
                 aArgs.concat(Array.prototype.slice.call(arguments)));
        };

    // 维护原型关系
    if (this.prototype) {
      // Function.prototype doesn't have a prototype property
      fNOP.prototype = this.prototype; 
    }
    // 下行的代码使fBound.prototype是fNOP的实例,因此
    // 返回的fBound若作为new的构造函数,new生成的新对象作为this传入fBound,新对象的__proto__就是fNOP的实例
    fBound.prototype = new fNOP();

    return fBound;
  };
}
```

上述算法和实际的实现算法还有许多其他的不同 （尽管可能还有其他不同之处，却没有那个必要去穷尽）


## 参考链接

- <a href="https://github.com/lin-xin/blog/issues/7" target="_blank">https://github.com/lin-xin/blog/issues/7</a>

<a href="https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/apply" target="_blank">https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/apply</a>