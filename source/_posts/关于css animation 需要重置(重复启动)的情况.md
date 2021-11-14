---
title: 关于css animation 需要重置(重复启动)的情况
date: 2021-04-22 17:21:53
tags:
    - my CSDN blog
    - disscussion
---
# 关于css animation 需要重置(重复启动)的情况

CSS animation基础设置就是只跑一次，如果需要一个特点的结束状态（比如说结束后在新的xy位置 或者大小变化后不还原），那需要用的是transition 这篇文章不讨论这个但建议理解一下animation 和transition之间的区别。



因为CSS animation的基础设置就是只跑一次，通过触发事件添加animation类到元素上的方法（有点绕，请看第一段代码），会导致只有第一次添加类触发该 animation但是后期的反复操作会无效，要注意的是 在这个情况下 这个animation类 (transform-active) 自从第一次被添加后 始终在这个元素内 即使动画结束

```js
    $("#fullscreen-button").click(function () {
        $('#fullscreen-button').addClass('position-animatable')
               
        $('#fullscreen-button').addClass('transform-active')
        if($('#fullscreen-button').hasClass('new-position')){
            $('#fullscreen-button').removeClass('new-position')

        }
        else{
            $('#fullscreen-button').addClass('new-position')

        }
    });  
// 这是基本考虑下写的代码
```



于是我首先想到的是在点击事件中添加removeClass，将该animation类在点击事件函数中执行后移除，下次执行不就没有了么，但这样是不行的，jquery不会按行判断，而是直接把这两个抵消掉。(在stackoverflow上听别人说的..)



然后我又看到  据说 通过修改元素的 **样式**会导致渲染树重新渲染(repaint) 那样animation类 (transform-active) 就会被重置，还有一个办法是给removeClass加上settimeout 时间设置为0也会导致重绘。

我试了一下 都不行..

```js
    $("#fullscreen-button").click(function () {
        $('#fullscreen-button').addClass('position-animatable')
        setTimeout(function () {
            $('#fullscreen-button').removeClass('transform-active')
        }, 700)
        $('#fullscreen-button').addClass('transform-active')
        if($('#fullscreen-button').hasClass('new-position')){
            $('#fullscreen-button').removeClass('new-position')

        }
        else{
            $('#fullscreen-button').addClass('new-position')

        }
    });
```

然后我还是老实的给setTimeout 加上个时间，成了， 就是效果不对劲， 把延迟时间改成animation-duration的时间， 效果符合预期。

这是篇好文 https://blog.csdn.net/culi4814/article/details/108377840
