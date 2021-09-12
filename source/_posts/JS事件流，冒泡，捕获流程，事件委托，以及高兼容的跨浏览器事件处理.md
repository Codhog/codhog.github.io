---
title: JS事件流，冒泡，捕获流程，事件委托，以及高兼容的跨浏览器事件处理
date: 2021-04-03 10:19:23
tags:
---
# JS事件流，冒泡，捕获流程，事件委托，以及高兼容的跨浏览器事件处理

先说结论：选择捕获或冒泡完全取决于你想要的元素事件启动顺序。

### DOM 事件流

JS是事件驱动的(Event-oriented)语言，DOM规范提供了点击(onclick)，加载(onload)，鼠标悬停(onmouseover)

DOM2 Events 规范规定事件流分为3 个阶段：**事件捕获、到达目标和事件冒泡**。

这是怎么回事呢，因为微软IE和Netscape的大佬在开发浏览器的时候分别开会，认为表单(事件)处理可以从服务器放到浏览器上，确认元素的具体层级，肯定要层层定位，这个分别开会后诞生了很有意义的结果，虽然思路是相似的，可结果却有点相反。IE觉得要从子元素找到父元素（事件冒泡），而Netscape则相信从父元素找到子元素(事件捕获)。

W3C规范则认为两个功能都要被浏览器提供给开发者

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210326172703317.JPG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21haW1hZGRhZGFv,size_16,color_FFFFFF,t_70#pic_center)


根据DOM2 Events 规范规定，捕获阶段是不会提取到div元素的，而此规范也认为**到达目标**和**事件冒泡**是同一阶段。

但现实是，现代浏览器会提供多种方案让我们在捕获阶段获取元素

IE10及以下不能在捕获阶段提取元素，IE11、Chrome 、Firefox、Safari等浏览器则可在两个阶段获取元素

### DOM0事件处理 

内联模式：它的js和html不分离、耦合性高、维护困难所以不推荐使用

```HTML
<input type="button" value="点击" onlcick="alert('click on btn.')"></input>
```

普通传统模式：被最多浏览器支持的方法 包括DOM2，

下图表示通过DOM0绑定一个元素，和解除绑定

值得注意的是此时btn.this作用于指向的是Event.target，与IE的**attachEvent(**)**相反**，IE的this指向window

DOM0的事件处理程序会在元素的作用域中运行，而attachEvent()为全局作用域

```js
let btn = document.getElementById("myBtn");
btn.onclick = function() {
	console.log(this.id); // "myBtn"
};
// 移除事件处理操作
btn.onclick = null;
```

可是DOM0只能对一个元素绑定一个事件处理程序，且只能**事件冒泡**型选元素，也就引开了下一个话题

### DOM2事件处理

DOM2相比DOM多了addEventListener()和removeEventListener(), 它们接收3 个参数：事件名、事件处理函
数和一个布尔值，true在捕获阶段调用事件处理程序，false在冒泡阶段调用事件处理程序

addEventListener()最大的优势就是可以为同一个事件添加多个事件处理程序

大多数情况下，事件处理程序会被添加到事件流的冒泡阶段，主要原因是跨浏览器兼容性好

要注意的是 addEventListener()中的匿名函数不能被移除

```js
let btn = document.getElementById("myBtn");
btn.addEventListener("click", () => {
console.log(this.id);
}, false);

btn.removeEventListener("click", function() { // 无法移除
console.log(this.id);
}, false);
```

匿名函数导致传给removeEventListener()的事件处理函数必须与传给addEventListener()
的不是同一个。

```js
let btn = document.getElementById("myBtn");
let handler = function() {
console.log(this.id);
};
btn.addEventListener("click", handler, false);

btn.removeEventListener("click", handler, false); // 移除成功
```

### 跨浏览器事件处理

```js
var EventUtil = {
    addHandler: function (element, type, handler) {
        if (element.addEventListener) {
            element.addEventListener(type, handler, false);
        } else if (element.attachEvent) {
            element.attachEvent("on" + type, handler);
        } else {
            element["on" + type] = handler;
        }
    },
    removeHandler: function (element, type, handler) {
        if (element.removeEventListener) {
            element.removeEventListener(type, handler, false);
        } else if (element.detachEvent) {
            element.detachEvent("on" + type, handler);
        } else {
            element["on" + type] = null;
        }
    }
};
```

先检测是否存在DOM2 方式。如果有DOM2 方式，就使用该方式，传入事件类型和事件处理函数，以及表示冒泡阶段的第三个参数false。否则，如果存在IE 方式，则使用该方式。注意这时候必须在事件类型前加上"on"，才能保证在IE8 及更早版本中有效

要确保事件处理代码具有最大兼容性，只需要让代码在**冒泡阶段**运行即可。

### 事件委托

事件委托 - 是一种神奇的方法，我们不需要监听任何子元素，而是监听一个父元素的所有子元素是否发生我们指定的事件。

这样可以解决经常被替换的元素无法被绑定事件，比如表格中的button需要删除和添加，但他们本质的点击事件都是一样的。

或者说是父元素生成后，子元素还未生成，却要给未来生成的子元素绑定事件，这些情况就需要事件委托。

JQuery中的delegate 就能很好的解决这类情况

```js
$("div").delegate("button","click",function(){
  $("p").slideToggle();
});
```

JS

```html
<ul id="parent-list">
  <li id="post-1">Item 1</li>
  <li id="post-2">Item 2</li>
  <li id="post-3">Item 3</li>
  <li id="post-4">Item 4</li>
  <li id="post-5">Item 5</li>
  <li id="post-6">Item 6</li>
</ul>
<script>
// 选中父元素并监听包括父元素和其中的所有点击事件
document.getElementById("parent-list").addEventListener("click", function(e) {
  // e.target 就是被点击的事件
  // 检查是否为li元素
  if(e.target && e.target.nodeName == "LI") {
    // 成功的监听到了父元素内的所有子元素
    console.log("List item ", e.target.id.replace("post-"), " was clicked!");
   }
});
</script>
```

如果说我们不用事件委托，为每个子元素单独绑定事件函数，那么内存就会被大幅使用，因为每个绑定事件都需要事件解除，而DOM0级的事件是无法被消除掉的。

事件委托利用了事件冒泡的机制，因此也只能发生在事件冒泡阶段

注意：如果父元素离子元素太远了（其中有过多嵌套）事件冒泡会经过大量的父元素导致占用大量内存
