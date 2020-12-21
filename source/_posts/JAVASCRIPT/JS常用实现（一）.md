---
title: JS常用实现（一）
---

<a name="r31KD"></a>
## debounce（防抖）
**防抖可以让多个顺序调用的事件合成一个，防止抖动，防止正常需要一次执行就可以的事件执行了多次。在用户停止触发事件的时候，才进行执行事件。**<br />
<br />例如在搜索引擎输入框中输入文字时，每输入一个键盘字符，搜索框会实时将输入值通过请求发送到后台。将每个用户输入的字符都发送至后台，会导致请求过于频繁，造成资源的浪费。<br />
<br />同时用户的体验也不佳，每输入一个字符都发送请求，对于服务器的性能要求很高，因为如果请求耗时太长，就无法做到每敲一个字符，就马上出来关联的输入提示内容，需要很优秀的服务器性能来配合实现。所以一般来说，这样的输入框都需要用防抖来进行处理。除非数据检索或者代码执行耗时足够短，可以不做处理。<br />
<br />同样适用的场景有浏览器的大小调整resize监听，滚动条滚动scroll监听，按钮点击click事件等。<br />![image.png](https://cdn.nlark.com/yuque/0/2020/png/203222/1608514863170-60b7772a-9745-4620-838f-856332a2dd15.png#align=left&display=inline&height=401&margin=%5Bobject%20Object%5D&name=image.png&originHeight=802&originWidth=1702&size=140476&status=done&style=none&width=851)
```javascript
/**
 * 防抖
 * @param func 处理函数
 * @param wait 等待时长，毫秒
 */
const debounce = (func, wait) => {
  let timeout = null;
  return () => {
  	clearTimeout(timeout);
    timeout = setTimeout(func, wait);
  }
}
```
```javascript
/** 测试用例 */
const testDebounce = (e) => { console.log(e) }
document.addEventListener('scroll', debounce(testDebounce, 300));
```
这里为什么要return一个函数，因为timeout变量存在于debounce局部作用域内，正常调用后timeout变量会被内存回收。返回一个函数，引用了timeout变量，形成了闭包，所以在返回的函数内部可以读取到局部timeout变量。<br />

<a name="eZuDD"></a>
## throttle（节流）
**节流，也可以理解为限流。让事件在每个间隔的时间里，只执行一次。和防抖不同，节流保证了在x毫秒的事件内，必然触发一次事件。**<br />
<br />例如地铁上下班时间限流，每隔5分钟才能进站一波人，这就是限流。防抖和节流，作用区别不大，在应用场景上有一定的重合，关键在于是否需要在一段时间内必须触发一次事件。<br />

```javascript
/**
 * 节流，结合了防抖的例子
 * @param func 处理函数
 * @param wait 等待时长，毫秒
 * @param mustRun 间隔时长，毫秒
 */
const throttle = (func, wait, mustRun) => {
  let timeout = null;
  let startTime = new Date();
  return (...args) => {
  	const curTime = new Date();
    clearTimeout(timeout);
    if (curTime - startTime >= mustRun) {
    	func.apply(this, args);
      startTime = curTime;
    } else {
      timeout = setTimeout(func, wait);
    }
  }
}
```
```javascript
/**
 * 节流，不使用定时器。previous变量不会被销毁，所以根据节流的特性可以这样直接调用。
 * @param func 处理函数
 * @param mustRun 间隔时长，毫秒
 */
const throttle = (func, mustRun) => {
  let previous = 0;
  return (...args) => {
    const curTime = new Date();
  	if (curTime - previous >= mustRun) {
    	func.apply(this, args);
      previous = curTime;
    }
  }
}
```
```javascript
/**
 * 节流，使用定时器
 * @param func 处理函数
 * @param wait 间隔时长，毫秒
 */
const throttle = (func, mustRun) => {
  let timeout = null;
  return (...args) => {
    if (timeout) {
    	return;
    }
    timeout = setTimeout(() => {
    	func.apply(this, args);
      timeout = null;
    }, mustRun);
  }
}
```
注意
```javascript
/** 测试用例 */
const test = (e) => { console.log(e) }
document.addEventListener('scroll', throttle(test, 300));
```


<a name="QRWVH"></a>
## new（new运算符）
**`new` 运算符创建一个用户定义的对象类型的实例或具有构造函数的内置对象的实例。**<br />语法 new _constructor_[([_arguments_])]<br />
<br />new是创建一个新实例，新实例肯定是一个对象。<br />
<br />New操作符实际上经历以下4个步骤：

1. 创建一个新对象
1. 将构造函数的作用域赋给新对象（this指向新对象）
1. 执行构造函数中的代码（为新对象添加属性）
1. 返回新对象，如果是引用类型，返回这个引用类型对象，否则返回新创建对象


<br />new 主要来新建实例，要理解new首先来对JS原型链有一定的认识。JS原型prototype在父类里是protoype，在实例里是[[prototype]]，可以通过__proto__访问。因此创建一个新对象，同时将新对象的__proto__设置为父类的prototype，实现继承父类。<br />
<br />例如:
```javascript
class A {};
const a = new A();
a.__proto__ === A.prototype; // true
```


```javascript
/**
 * 模拟实现new
 * @param parentObject 父类
 * @param args 父类构造函数参数
 */
const newObject = (parentObject, ...args) => {
  // 绑定原型
  const newObj= Object.create(parentObject.prototype); // 使用create方法设置新对象__proto__
  // 调用构造函数
  const result = parentObject.apply(newObj, args);
  // 返回
  return result instanceof Object ? result : newObj;
}
```


<a name="r93wO"></a>
## deepClone (深拷贝)
网上的深拷贝代码一般都有些问题的，因为对于一些特殊的对象没有进行处理，但是一般也不会出现bug，简单的深拷贝有时也能实现功能。<br />
<br />**实现一 JSON.stringify && JSON.parse**<br />通过JSON的两个方法使对象重新构造成新的对象实现深拷贝，它的问题在于会丢弃对象的constructor，同时必须保证处理的对象为能够被json数据结构表示。

不推荐这种实现，如果要用，需要配合错误捕获方法来使用。
```javascript
const a = {};
const b = JSON.parse(JSON.stringify(a));
```

<br />**实现二 递归**<br />存在问题，支持的特殊对象不是很多，但是还算优雅，解决了循环引用的问题，遇到已经引用过的对象，不再重复循环一遍。
```javascript
const deepClone = (obj, cache = new WeakMap()) => {
  if (!obj instanceof Object) return obj;
  if (cache.get(obj)) return cache.get(obj); // 防止循环引用
  if (obj instanceof Function) {
    return (...arg) => { obj.apply(this, args) }
  } // 支持函数
  if (obj instanceof Date) return new Date(obj); // 支持日期
  if (obj instanceof RegExp) return new RegExp(obj.source, obj.flags); // 支持正则对象

  const res = Array.isArray(obj) ? [] : {};
	cache.set(obj, res);   // 缓存 copy 的对象，用于处理循环引用的情况

  Object.keys(obj).forEach((key) => {
    if (obj[key] instanceof Object) {
      res[key] = deepClone(obj[key], cache);
    } else {
      res[key] = obj[key];
    }
  });

  return res;
}
```


