---
title: 关于JS对象键值隐性转换导致的复合数组求长度出现的问题
date: 2021-05-02 15:29:33
tags:
---
# JS对象键值的隐性转换

> **今天在求一种类似[{string:bool},{string:bool}]的数组长度时， 直接用Array.length没有用。**
>
> **在外网上看到有解释，希望能对大家有用**
>
> **问题是这样的**
>
> **在往这个数组添加新元素的时候，我用的是 list['string'] = bool, 而不是 list.push({'string':bool})**
> 
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210324173038800.JPG#pic_center)


```javascript
//上图的结果分别是这些东西  我们可以看到 这个object array 虽然看着里面有对象  但是长度却是0
console.log(array,array.length,Object.prototype.apply(array))
```

我们把问题复现：
分别尝试以取键赋值 和 数组push的方法分别实现 一个字典包对象
我们能看到 这两种方法不只是长度不一样 结果也是不一样的

```js
var a = []
a['a'] = 1
a['b'] = 2

var b = []
b.push({'a':1})
b.push({'b':2})

console.log(a.length, b.length); //0, 2
console.log(a,b); //[ a: 1, b: 2 ], [ { a: 1 }, { b: 2 } ]
```

可能这个例子不够清晰：再举一个 让大家明白JS作为弱语言发生的隐性转换

```js
var a = []
a['1'] = 1
a['2'] = 2

var b = []
b.push({'1':1})
b.push({'2':2})

console.log(a.length, b.length); // 3 2
console.log(a,b); // [ <1 empty item>, 1, 2 ] [ { '1': 1 }, { '2': 2 } ]
```

JS会隐性地在读取对象地时候将对象地字符串 键 转换为js理解的类型

所以说

```js
a['1'] 实际上等价于 a[1]
```

所以说以 1 为键时 即时取得时候是字符串形式，JS也会自动补全a[0]为empty

所以说也能解释如果是a【‘a'】的形式 

```
var a = []
a['a'] = 1
a['b'] = 2
```

外网的解释是

JS将'a'隐性转换成了对象a，所以只是给a的a属性添加了值，而并没有增加数组的实际长度

如果想遍历这个含有对象地数组 或者看里面的长度 最好还是用类数组的遍历方法


```js
Object.keys(a) //或者
Object.entries(a) //不是很好用 因为如果数组是对象 会出现每个索引为键  而值才是你要的每个字典 所以要再进行一次遍历
//推荐用keys 再从数组中取

```
英文好的可以看看https://stackoverflow.com/questions/2528680/javascript-array-length-incorrect-on-array-of-objects
