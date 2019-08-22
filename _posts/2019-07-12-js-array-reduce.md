---
layout:     post
title:      深入分析JS内置对象(二)
subtitle:   JavaScript标准库内置对象分析之Array的reduce()
date:       2019-07-12
author:     Aro
header-img: img/post-bg-js-version.jpg
catalog: true
tags:
    - JS
    - 前端
---

## `Array.prototype.reduce()`

`reduce()` 方法对数组中的每个元素执行一个由您提供的`reducer`函数(升序执行)，将其结果汇总为单个返回值。注意，这个升序指的是按照字典序。

```js
const array1 = [1, 2, 3, 4];
const reducer = (accumulator, currentValue) => accumulator + currentValue;

// 1 + 2 + 3 + 4
console.log(array1.reduce(reducer));
// expected output: 10

// 5 + 1 + 2 + 3 + 4
console.log(array1.reduce(reducer, 5));
// expected output: 15
```

`reducer`函数接收4个参数:

- Accumulator (acc) (累计器)
- Current Value (cur) (当前值)
- Current Index (idx) (当前索引)
- Source Array (src) (源数组)

您的 `reducer` 函数的返回值分配给累计器Accumulator，该返回值在数组的每个迭代中被记住，并最后成为最终的单个结果值。

>arr.reduce(callback(accumulator, currentValue[, index[, array]])[, initialValue])


#### 参数

`callback`执行数组中每个值的函数，包含四个参数：

- `accumulator` 累计器累计回调的返回值; 它是上一次调用回调时返回的累积值，或initialValue（见于下方）。
- `currentValue` 数组中正在处理的元素。
- `currentIndex` 数组中正在处理的当前元素的索引。 如果提供了initialValue，则起始索引号为0，否则为1。
- `array` 调用reduce()的数组

`initialValue` 作为第一次调用 `callback` 函数时的第一个参数的值。 如果没有提供初始值，则将使用数组中的第一个元素。 在没有初始值的空数组上调用 `reduce` 将报错。


#### 返回值

函数累计处理的结果

## 描述

`reduce`为数组中的每一个元素依次执行`callback`函数，不包括数组中被删除或从未被赋值的元素，接受四个参数：

- accumulator 累计器
- currentValue 当前值
- currentIndex 当前索引
- array 数组

回调函数第一次执行时，accumulator 和currentValue的取值有两种情况：如果调用reduce()时提供了initialValue，accumulator取值为initialValue，currentValue取数组中的第一个值；`如果没有提供 initialValue，那么accumulator取数组中的第一个值（索引为0的值），currentValue取数组中的第二个值（索引为1的值）`。

>注意：如果没有提供initialValue，reduce 会从索引1的地方开始执行 callback 方法，跳过第一个索引，此时因为没有initialValue，所以accumulator取数组的第一个值（索引为0的值）做初始值。如果提供initialValue，从索引0开始。

如果数组为空且没有提供initialValue，会抛出TypeError 。如果数组仅有一个元素（无论位置如何）并且没有提供initialValue， 或者有提供initialValue但是数组为空，那么此唯一值将被返回并且callback不会被执行。

提供初始值通常更安全，正如下面的例子，如果没有提供initialValue，则可能有三种输出：

```js
var maxCallback = ( acc, cur ) => Math.max( acc.x, cur.x );
var maxCallback2 = ( max, cur ) => Math.max( max, cur );

// reduce() 没有初始值
[ { x: 22 }, { x: 42 } ].reduce( maxCallback ); // 42
[ { x: 22 }            ].reduce( maxCallback ); // { x: 22 } 
[                      ].reduce( maxCallback ); // TypeError
```


#### `reduce()`如何运行

假如运行下段`reduce()`代码：

```js
[0, 1, 2, 3, 4].reduce(function(accumulator, currentValue, currentIndex, array){
  return accumulator + currentValue;
});
```

callback 被调用四次，每次调用的参数和返回值如下表：

callback | accumulator | currentValue | currentIndex | array | return value
- | - | - | - | - | -
first call | `0` | `1` | `1` | `[0, 1, 2, 3, 4]` | `1`
second call | `1` | `2` | `2` | `[0, 1, 2, 3, 4]` | `3`
third call | `3` | `3` | `3` | `[0, 1, 2, 3, 4]` | `6`
fourth call | `6` | `4` | `4` | `[0, 1, 2, 3, 4]` | `10`

由`reduce`返回的值将是最后一次回调返回值`10`。

您还可以提供Arrow Function 来代替完整的函数。 下面的代码将产生与上面的代码中相同的输出：

```js
[0, 1, 2, 3, 4].reduce((prev, curr) => prev + curr );
```


如果你打算提供一个初始值作为reduce()方法的第二个参数，以下是运行过程及结果：

```js
[0, 1, 2, 3, 4].reduce((accumulator, currentValue, currentIndex, array) => { return accumulator + currentValue; }, 10 );
```

callback | accumulator | currentValue | currentIndex | array | return value
- | - | - | - | - | -
first call | `10` | `0` | `0` | `[0, 1, 2, 3, 4]` | `10`
second call | `10` | `1` | `1` | `[0, 1, 2, 3, 4]` | `11`
third call | `11` | `2` | `2` | `[0, 1, 2, 3, 4]` | `13`
fourth call | `13` | `3` | `3` | `[0, 1, 2, 3, 4]` | `16`
fifth call | `16` | `4` | `4` | `[0, 1, 2, 3, 4]` | `20`

这种情况下`reduce()`返回的值是`20`。


## 例子🌰


#### 数组里所有值的和

```js
var sum = [0, 1, 2, 3].reduce(function (accumulator, currentValue) {
  return accumulator + currentValue;
}, 0);
// 和为 6
```

你也可以写成箭头函数的形式：

```js
var total = [ 0, 1, 2, 3 ].reduce(
  ( acc, cur ) => acc + cur,
  0
);
```


#### 累加对象数组里的值

要累加对象数组中包含的值，必须提供初始值，以便各个item正确通过你的函数。

```js
var initialValue = 0;
var sum = [{x: 1}, {x:2}, {x:3}].reduce(function (accumulator, currentValue) {
    return accumulator + currentValue.x;
},initialValue)

console.log(sum) // logs 6
```

你也可以写成箭头函数的形式：

```js
var initialValue = 0;
var sum = [{x: 1}, {x:2}, {x:3}].reduce(
    (accumulator, currentValue) => accumulator + currentValue.x
    ,initialValue
);

console.log(sum) // logs 6
```


#### 将二维数组转化为一维

```js
var flattened = [[0, 1], [2, 3], [4, 5]].reduce(
  function(a, b) {
    return a.concat(b);
  },
  []
);
// flattened is [0, 1, 2, 3, 4, 5]
```

你也可以写成箭头函数的形式：

```js
var flattened = [[0, 1], [2, 3], [4, 5]].reduce(
 ( acc, cur ) => acc.concat(cur),
 []
);
```


#### 计算数组中每个元素出现的次数

```js
var names = ['Alice', 'Bob', 'Tiff', 'Bruce', 'Alice'];

var countedNames = names.reduce(function (allNames, name) { 
  if (name in allNames) {
    allNames[name]++;
  }
  else {
    allNames[name] = 1;
  }
  return allNames;
}, {});
// countedNames is:
// { 'Alice': 2, 'Bob': 1, 'Tiff': 1, 'Bruce': 1 }
```


#### 按属性对object分类

```js
var people = [
  { name: 'Alice', age: 21 },
  { name: 'Max', age: 20 },
  { name: 'Jane', age: 20 }
];

function groupBy(objectArray, property) {
  return objectArray.reduce(function (acc, obj) {
    var key = obj[property];
    if (!acc[key]) {
      acc[key] = [];
    }
    acc[key].push(obj);
    return acc;
  }, {});
}

var groupedPeople = groupBy(people, 'age');
// groupedPeople is:
// { 
//   20: [
//     { name: 'Max', age: 20 }, 
//     { name: 'Jane', age: 20 }
//   ], 
//   21: [{ name: 'Alice', age: 21 }] 
// }
```


#### 使用扩展运算符和initialValue绑定包含在对象数组中的数组

```js
// friends - 对象数组
// where object field "books" - list of favorite books 
var friends = [{
  name: 'Anna',
  books: ['Bible', 'Harry Potter'],
  age: 21
}, {
  name: 'Bob',
  books: ['War and peace', 'Romeo and Juliet'],
  age: 26
}, {
  name: 'Alice',
  books: ['The Lord of the Rings', 'The Shining'],
  age: 18
}];

// allbooks - list which will contain all friends' books +  
// additional list contained in initialValue
var allbooks = friends.reduce(function(prev, curr) {
  return [...prev, ...curr.books];
}, []);

// allbooks = [
//   'Bible', 'Harry Potter', 'War and peace', 
//   'Romeo and Juliet', 'The Lord of the Rings',
//   'The Shining'
// ]
```


#### 数组去重

```js
let arr = [1,2,1,11,2,3,5,4,5,3,4,4,4,4,-2,-3];
let result = arr.sort().reduce((init, current) => {
    if(init.length === 0 || init[init.length-1] !== current) {
        init.push(current);
    }
    return init;
}, []);
console.log(result); //[-2, -3, 1, 11, 2, 3, 4, 5]
```


## 参考链接

<a href="https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/Reduce" target="_blank">https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/Reduce</a>