---
title: 事件循环和同步异步
categories: ['Javascript']
tags: ['Javascript', 'html', 'nodejs']
---

某日，群里有人发了一张这样的图，问输出结果为什么和预期的不一样，有几个人在讨论，都说不出个为什么。这一看就是一道面试题，考察了js的同步异步和事件循环，当然工作中类似的场景也非常多，这道题对于理解同类的问题会有一定的帮助。这里就这道题，解析一下它是如何执行的。在继续看下面内容之前，可以把你的答案先记下来。<br />
<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/203222/1611107142638-dac4e3de-0a74-4f1e-85ae-15abec94f72a.png#align=left&display=inline&height=244&margin=%5Bobject%20Object%5D&name=image.png&originHeight=488&originWidth=800&size=197788&status=done&style=none&width=400)<br />

```javascript
async function async1() {
  console.log('async1 start')
  await async2()
  console.log('async1 end')
}

async function async2() {
  console.log('async2')
}

console.log('script start')
setTimeout(function() {
  console.log('setTimeout')
}, 0)

async1();

new Promise( function( resolve ) {
 console.log('promise1')
 resolve();
} ).then( function() {
 console.log('promise2')
} )

console.log('script end')
```

<br />

<a name="54j49"></a>
### async
要理解这道题，首先要理解es2017 async 语法。

- async 函数返回promise对象，后面可以直接接then。
- 语法规定，async 的 await 命令后面，可以接promise对象或者原始类型的值（但是会转成立即resolve的promise对象。Promise.resolve()）。



<a name="0iiVM"></a>
#### async的基本用法
async函数返回一个 Promise 对象，可以使用then方法添加回调函数。当函数执行的时候，一旦遇到await就会先返回，等到异步操作完成，再接着执行函数体内后面的语句。<br />
<br />下面是一个例子。函数前面的async关键字，表明该函数内部有异步操作。调用该函数时，会立即返回一个Promise对象。
```javascript
async function getStockPriceByName(name) {
  const symbol = await getStockSymbol(name);
  const stockPrice = await getStockPrice(symbol);
  return stockPrice;
}

getStockPriceByName('goog').then(function (result) {
  console.log(result);
});
```

<br />举个栗子，async3() 返回了promise对象
```javascript
// 预期输出
// res
// undefined
async function async3() {
  console.log('res');
}
async3().then((res) => { console.log(res); })



// 类似async1里面的操作，但是这里没有await关键字，所以不会阻塞
// console.log('async1 start')
// await async2()
// console.log('async1 end')
console.log(1);
Promise.resolve();
console.log(2);
// 输出
// 1
// 2
// undefined
```


<a name="HRKtx"></a>
### event loops
<a name="XtV7n"></a>
#### event loops是什么
Javascript是单线程的语言，既然是单线程的，在某个特定的时刻只有特定的代码能够被执行，并阻塞其它的代码。 而浏览器是事件驱动的（Event driven），浏览器中很多行为是异步（Asynchronized）的，会创建事件并放入执行队列中，按照一定的顺序进行执行。

<a name="uhAwT"></a>
#### event loops的任务类型

- 宏任务
   - setTimeout
   - setInterval
   - requestAnimationFrame
   - 解析HTML
   - 执行主线程JS代码
   - 修改URL
   - 页面加载
   - 用户交互
   - 。。。
- 微任务
   - Promise
   - MutationObserver
   - process.nextTick （nodejs）
   - queueMicrotask

<br />
<a name="rCKxJ"></a>
#### event loops的执行顺序（浏览器）

1. 第一步，检查macrotask队列，运行最前面的任务，如果队列为空，前往第二步。
1. 第二步，检查microtask队列，一直运行该队列任务直到该队列为空。
1. 第三步，执行渲染ui。渲染后回到第一步。



<a name="YtfiF"></a>
### 执行过程解析
理解基本async语法和event loops机制后，可以写出执行过程如下<br />

- 定义了async函数 async1
- 定义了async函数 async2
- 输出 script start
- 定义了一个setTimeout，该异步操作属于宏任务，推入macrotask队列
- 执行到了async1，进入async1
- 输出 async1 start
- 执行到了await语句，先执行async2()，输出 async2， async2()返回了一个promise对象，这个promise被推入microtask队列，await会阻塞当前执行函数体后续的语句，等待这个异步promise完成。
- 代码跳出async1继续执行
- 进入new Promise同步执行部分，输出 promise1
- 执行resolve，异步回调推入推入microtask队列
- 输出 script end
- 从microtask队列取出第一个异步任务执行，这个promise对象的resolve返回值为 undefined
- 继续执行async1里await后面的语句，输出 async1 end
- 从microtask队列取出第二个异步任务执行，输出 promise2
- 当前macrotask执行完毕
- 取出下一个macrotask执行，输出 setTimeout


<br />
<br />最终输出结果

1. script start
1. async1 start
1. async2
1. promise1
1. script end
1. async1 end
1. promise2
1. setTimeout


<br />

