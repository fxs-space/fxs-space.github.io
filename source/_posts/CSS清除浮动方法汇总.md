---
title: CSS清除浮动方法汇总
date: 2017-07-06 19:42:39
categories: 前端
tags: CSS
---

## CSS清除浮动方法汇总

##### 在清除浮动前我们要了解两个重要的定义：

- **浮动的定义**：使元素脱离文档流，按照指定方向发生移动，遇到父级边界或者相邻的浮动元素停了下来。

- **高度塌陷**：浮动元素父元素高度自适应（父元素不写高度时，子元素写了浮动后，父元素会发生高度塌陷）

知道浮动和为什么要清除浮动之后我们可以开始学习如何清除浮动了，这时候我们就需要用到清除浮动的属性`clear`：

- clear:left | right | both | none | inherit：元素的某个方向上不能有浮动元素
- clear:both：在左右两侧均不允许浮动元素。

<!--more-->

##### 具体清楚浮动的方法主要一下几种：

##### 1、clear清除浮动（添加空div法）

```
在浮动元素下方添加空div,并给该元素写css样式：   {clear:both;height:0;overflow:hidden;}
```

##### 2、方法：给浮动元素父级设置高度

```
我们知道了高度塌陷是应为给浮动元素的父级高度是自适应导致的，那么我们给它的设置适当的高度就可以解决这个问题了。

缺点：在浮动元素高度不确定的时候不适用
```

##### 3、方法：以浮制浮（父级同时浮动）

```
何谓“以浮制浮”呢？就是让 浮动元素的父级也浮动 。

缺点：需要给每个浮动元素父级添加浮动，浮动多了容易出现问题。
```

##### 4、方法：父级设置成inline-block

```
缺点：父级的margin左右auto失效，无法使用margin: 0 auto;居中了
```

##### 5、 br 清浮动

```
<div class="box">
    <div class="top"></div>
    <br clear="both" />
</div>
```

br 标签自带clear属性，将它设置成both其实和添加空div原理是一样的。

问题：不符合工作中：结构、样式、行为，三者分离的要求。


##### 6、给父级添加overflow:hidden 清浮动方法；

```
问题：需要配合 宽度 或者 zoom 兼容IE6 IE7；

overflow: hidden;
*zoom: 1;
```

##### 7、万能清除法 after伪类 清浮动（现在主流方法，推荐使用）

```
选择符:after{
    content:".";
    clear:both;
    display:block;
    height:0;
    overflow:hidden;
    visibility:hidden;
}
```

同时为了兼容 IE6，7 同样需要配合zoom使用例如：

```
.clear:after{content:'';display:block;clear:both;height:0;overflow:hidden;visibility:hidden;}

.clear{zoom:1;}
```


需要注意的东西：

```
after伪类： 元素内部末尾添加内容；
    :after{content"添加的内容";} IE6，7下不兼容

zoom 缩放
    a、触发 IE下 haslayout，使元素根据自身内容计算宽高；
    b、FF 不支持。
```


[尊重原创，感谢原创分享](http://blog.csdn.net/promiseCao/article/details/52771856)