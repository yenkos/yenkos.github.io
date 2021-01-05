---
title: 再次梳理AMD、CMD、CommonJS、ES6 Module的区别
categories: ['工程化']
tags: ['工程化']
---

<a name="df368884"></a>
### 前言

<br />回想起之前的一次面试，第一轮面试官问我 AMD 和 CMD 的区别，我只回答说 AMD 是提前加载，CMD 是按需加载。第二轮面试官又问了我 CommonJS 和 ES6 Module 的区别，emmm...，我大致回答说新的比旧的好~~，虽然面试官并没有说什么，不过显然这样的答案并不是有助于面试、有助于自己的技术积累的。<br />
<br />所以有必要进行一次梳理，以便更清晰地了解它们的特点及差异。<br />

<a name="AMD"></a>
### AMD

<br />AMD 一开始是 CommonJS 规范中的一个草案，全称是 Asynchronous Module Definition，即异步模块加载机制。后来由该草案的作者以 RequireJS 实现了 AMD 规范，所以一般说 AMD 也是指 RequireJS。<br />

<a name="4f511765"></a>
#### RequireJS 的基本用法

<br />通过`define`来定义一个模块，使用`require`可以导入定义的模块。<br />

```javascript
//a.js
//define可以传入三个参数，分别是字符串-模块名、数组-依赖模块、函数-回调函数
define(function(){
    return 1;
})

// b.js
//数组中声明需要加载的模块，可以是模块名、js文件路径
require(['a'], function(a){
    console.log(a);// 1
});
```


<a name="296d8495"></a>
#### RequireJS 的特点

<br />对于依赖的模块，AMD 推崇**依赖前置，提前执行**。也就是说，在`define`方法里传入的依赖模块 (数组)，会在一开始就下载并执行。<br />

<a name="CMD"></a>
### CMD

<br />CMD 是 SeaJS 在推广过程中生产的对模块定义的规范，在 Web 浏览器端的模块加载器中，SeaJS 与 RequireJS 并称，SeaJS 作者为阿里的玉伯。<br />

<a name="cf31777a"></a>
#### SeaJS 的基本用法


```javascript
//a.js
/*
* define 接受 factory 参数，factory 可以是一个函数，也可以是一个对象或字符串，
* factory 为对象、字符串时，表示模块的接口就是该对象、字符串。
* define 也可以接受两个以上参数。字符串 id 表示模块标识，数组 deps 是模块依赖.
*/
define(function(require, exports, module) {
  var $ = require('jquery');

  exports.setColor = function() {
    $('body').css('color','#333');
  };
});

//b.js
//数组中声明需要加载的模块，可以是模块名、js文件路径
seajs.use(['a'], function(a) {
  $('#el').click(a.setColor);
});
```


<a name="7316a2bf"></a>
#### SeaJS 的特点

<br />对于依赖的模块，CMD 推崇**依赖就近，延迟执行**。也就是说，只有到`require`时依赖模块才执行。<br />

<a name="CommonJS"></a>
### CommonJS

<br />CommonJS 规范为 CommonJS 小组所提出，目的是弥补 JavaScript 在服务器端缺少模块化机制，NodeJS、webpack 都是基于该规范来实现的。<br />

<a name="b5a8f379"></a>
#### CommonJS 的基本用法


```javascript
//a.js
module.exports = function () {
  console.log("hello world")
}

//b.js
var a = require('./a');

a();//"hello world"

//或者

//a2.js
exports.num = 1;
exports.obj = {xx: 2};

//b2.js
var a2 = require('./a2');

console.log(a2);//{ num: 1, obj: { xx: 2 } }
```


<a name="758110de"></a>
#### CommonJS 的特点


- 所有代码都运行在模块作用域，不会污染全局作用域；
- 模块是同步加载的，即只有加载完成，才能执行后面的操作；
- 模块在首次执行后就会缓存，再次加载只返回缓存结果，如果想要再次执行，可清除缓存；
- CommonJS 输出是值的拷贝 (即，`require`返回的值是被输出的值的拷贝，模块内部的变化也不会影响这个值)。



<a name="6387b9d5"></a>
### ES6 Module

<br />ES6 Module 是 ES6 中规定的模块体系，相比上面提到的规范， ES6 Module 有更多的优势，有望成为浏览器和服务器通用的模块解决方案。<br />

<a name="7bbecdbb"></a>
#### ES6 Module 的基本用法


```javascript
//a.js
var name = 'lin';
var age = 13;
var job = 'ninja';

export { name, age, job};

//b.js
import { name, age, job} from './a.js';

console.log(name, age, job);// lin 13 ninja

//或者

//a2.js
export default function () {
  console.log('default ');
}

//b2.js
import customName from './a2.js';
customName(); // 'default'
```


<a name="db1982dc"></a>
#### ES6 Module 的特点 (对比 CommonJS)


- CommonJS 模块是运行时加载，ES6 Module 是编译时输出接口；
- CommonJS 加载的是整个模块，将所有的接口全部加载进来，ES6 Module 可以单独加载其中的某个接口；
- CommonJS 输出是值的拷贝，ES6 Module 输出的是值的引用，被输出模块的内部的改变会影响引用的改变；
- CommonJS `this`指向当前模块，ES6 Module `this`指向`undefined`;


<br />目前浏览器对 ES6 Module 兼容还不太好，我们平时在 webpack 中使用的`export`/`import`，会被打包为`exports`/`require`。<br />

<a name="15141066"></a>
### 写在后面

<br />这里比较宽泛地把 JavaScript 中的几大模块化规范列举出来，希望借此对 JavaScript 模块化有大致的认识，而未对细节展开具体分析，感兴趣的可以自行探索。<br />
<br />[https://juejin.cn/post/6844903983987834888](https://juejin.cn/post/6844903983987834888)
