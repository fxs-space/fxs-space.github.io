---
title: exports、module.exports 和 export、export default 之间的关系与区别
date: 2017-10-18 20:05:09
categories: 模块化
tags: 模块化
---

## exports、module.exports 和 export、export default 之间的关系与区别

#### 前言

开始之前，先理理require、export、module.exports等的使用范围。

```
require: node 和 ES6 都支持的引入
export / import : 只有 ES6 支持的导出引入
module.exports / exports: 只有 node 支持的导出
```

#### node模块
`Node`里面的模块系统遵循的是[CommonJS](http://javascript.ruanyifeng.com/nodejs/module.html)规范。

<!--more-->

##### 那问题又来了，什么是`CommonJS`规范呢？

由于`js`以前比较混乱，各写各的代码，没有一个模块的概念，而这个规范出来其实就是对模块的一个定义。

==CommonJS定义的模块分为: 模块标识(module)、模块定义(exports) 、模块引用(require)==

##### 先解释 `exports` 和 `module.exports`

在一个node执行一个文件时，会给这个文件内生成一个 `exports` 和 `module` 对象，
而module又有一个exports属性。他们之间的关系如下图，都指向一块{}内存区域。

```
exports = module.exports = {};
```

![内存结构示意图](exports、module.exports 和 export、export default 之间的关系与区别/内存结构示意图.png)

那下面我们来看看代码的吧。

```
//utils.js
let a = 100;

console.log(module.exports); //能打印出结果为：{}
console.log(exports); //能打印出结果为：{}

exports.a = 200; //这里辛苦劳作帮 module.exports 的内容给改成 {a : 200}

exports = '指向其他内存区'; //这里把exports的指向指走

//test.js

var a = require('/utils');
console.log(a) // 打印为 {a : 200}
```
从上面可以看出，其实`require`导出的内容是`module.exports`的指向的内存块内容，并不是`exports`的。

简而言之，区分他们之间的区别就是 exports 只是 module.exports的引用，辅助后者添加内容用的。

用白话讲就是，`exports`只辅助`module.exports`操作内存中的数据，辛辛苦苦各种操作数据完，累得要死，结果到最后真正被`require`出去的内容还是`module.exports`的，真是好苦逼啊。

其实大家用内存块的概念去理解，就会很清楚了。

然后呢，为了避免糊涂，尽量都用 `module.exports` 导出，然后用`require`导入。

#### ES中的模块导出导入
说实话，在es中的模块，就非常清晰了。不过也有一些细节的东西需要搞清楚。
比如 `export` 和 `export default`，还有 导入的时候，`import a from ..`,`import {a} from ..`，总之也有点乱，那么下面我们就开始把它们捋清楚吧。

##### export 和 export default
首先我们讲这两个导出，下面我们讲讲它们的区别

1. export与export default均可用于导出常量、函数、文件、模块等
2. 在一个文件或模块中，export、import可以有多个，export default仅有一个
3. 通过export方式导出，在导入时要加{ }，export default则不需要
4. export能直接导出变量表达式，export default不行。

下面咱们看看代码去验证一下

##### testEs6Export.js

```
'use strict'
//导出变量
export const a = '100';

 //导出方法
export const dogSay = function(){
    console.log('wang wang');
}

 //导出方法第二种
function catSay(){
   console.log('miao miao');
}
export { catSay };

//export default导出
const m = 100;
export default m;
//export defult const m = 100;// 这里不能写这种格式。
```

##### index.js

```
//index.js
'use strict'
var express = require('express');
var router = express.Router();

import { dogSay, catSay } from './testEs6Export'; //导出了 export 方法
import m from './testEs6Export';  //导出了 export default

import * as testModule from './testEs6Export';//as 集合成对象导出

/* GET home page. */
router.get('/', function(req, res, next) {
  dogSay();
  catSay();
  console.log(m);
  testModule.dogSay();
  console.log(testModule.m); // undefined , 因为  as 导出是 把 零散的 export 聚集在一起作为一个对象，而export default 是导出为 default属性。
  console.log(testModule.default); // 100
  res.send('恭喜你，成功验证');
});

module.exports = router;
```
从上面可以看出，确实感觉 `ES6` 的模块系统非常灵活的。

#### 参考文献
[老树新芽，在ES6下使用Express](https://link.jianshu.com/?t=https://link.juejin.im?target=https%3A%2F%2Fsegmentfault.com%2Fa%2F1190000006707756)

[exports 和 module.exports 的区别](https://link.jianshu.com/?t=https://link.juejin.im?target=https%3A%2F%2Fcnodejs.org%2Ftopic%2F5231a630101e574521e45ef8)

[module.exports与exports,export与export default之间的关系](https://link.jianshu.com/?t=https://link.juejin.im?target=http%3A%2F%2Fwww.cnblogs.com%2Ffayin%2Fp%2F6831071.html)


[尊重原创，感谢原创分享](https://www.jianshu.com/p/076f2af97db0)