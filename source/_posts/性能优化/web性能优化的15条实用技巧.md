---
title: web性能优化的15条实用技巧
categories: ["性能优化"]
tags: ["工程化", "性能优化"]
---

> javascript 在浏览器中运行的性能，可以认为是开发者所面临的最严重的可用性问题。这个问题因为 javascript 的阻塞性而变得复杂，事实上，多数浏览器使用单一进程来处理用户界面和 js 脚本执行，所以同一时刻只能做一件事。js 执行过程耗时越久，浏览器等待响应的时间越长。

加载和执行

#### 一. 提高加载性能

> 1.IE8,FF,3.5,Safari 4 和 Chrome 都允许并行下载 js 文件，当 script 下载资源时不会阻塞其他 script 的下载。但是 js 下载仍然会阻塞其他资源的下载，如图片。尽管脚本下载不会互相影响，但页面仍然必须等待所有 js 代码下载并执行完才能继续。因此仍然存在脚本阻塞问题. 推荐将所有 js 文件放在 body 标签底部以减少对整个页面的影响。

> **2. 减少页面外链脚本文件的数量将会提高页面性能：**
>
> http 请求会带来额外的开销，因此下载单个 300k 的文件将比下载 10 个 30k 的文件效率更高。

> **3. 动态脚本加载技术：**
>
> 无论何时启动下载，文件的下载和执行都不会阻塞页面其他进程。

```javascript
function laodScript(url, callback) {
  var script = document.createElement("script");
  script.type = "text/javascript";

  if (script.readyState) {
    script.onreadystatechange = function () {
      if (script.readyState == "loaded" || script.readyState == "complete") {
        script.onreadystatechange = null;
        callback();
      }
    };
  } else {
    script.onload = function () {
      callback();
    };
  }
  script.src = url;
  document.getElementsByTagName("head")[0].appendChild(script);
}

// 使用
loadScript("./a.js", function () {
  loadScript("./b.js", function () {
    loadScript("./c.js", function () {
      console.log("加载完成");
    });
  });
});
```

> 4. 无阻塞加载类库——LABjs, 使用方法如下：

```php
<script src="lab.js"></script>
// 链式调用时文件逐个下载，.wait()用来指定文件下载并执行完毕后所调用的函数
$LAB.script('./a.js')
    .script('./b.js')
    .wait(function(){
        App.init();
})

// 为了保证执行顺序，可以这么做,此时a必定在b前执行
$LAB.script('./a.js').wait()
    .script('./b.js')
    .wait(function(){
        App.init();
})
```

#### 二. 数据存取与 JS 性能

> 1\. 在 js 中，数据存储的位置会对代码整体性能产生重大影响。数据存储共有 4 种方式：字面量，变量，数组项，对象成员。他们有着各自的性能特点。

> 2\. 访问字面量和局部变量的速度最快，相反，访问数组和对象相对较慢

> 3\. 由于局部变量存在于作用域链的起始位置，因此访问局部变量的比访问跨域作用变量更快

> 4\. 嵌套的对象成员会明显影响性能，应尽量避免

> 5\. 属性和方法在原型链位置越深，访问他的速度越慢

> 6\. 通常我们可以把需要多次使用的对象成员，数组元素，跨域变量保存在局部变量中来改善 js 性能

#### 三. DOM 编程

1\. 访问 DOM 会影响浏览器性能，修改 DOM 则更耗费性能，因为他会导致浏览器重新计算页面的几何变化。**&lt; 通常的做法是减少访问 DOM 的次数，把运算尽量留在 JS 这一端。**

> 注：如过在一个对性能要求比较高的操作中更新一段 HTML，推荐使用 innerHTML，因为它在绝大多数浏览器中运行的都很快。但对于大多数日常操作而言，并没有太大区别，所以你更应该根据可读性，稳定性，团队习惯，代码风格来综合决定使用 innerHTML 还是 createElement()

2. HTML 集合优化

> HTML 集合包含了 DOM 节点引用的类数组对象，一直与文档保持连接，每次你需要最新的信息时，都会重复执行查询操作，哪怕只是获取集合里元素的个数。

① 优化一——集合转数组 collToArr

```javascript
function collToArr(coll) {
  for (var i = 0, a = [], len = coll.length; i < len; i++) {
    a.push(coll[i]);
  }
  return a;
}
```

② 缓存集合 length

③ 访问集合元素时使用局部变量（即将重复的集合访问缓存到局部变量中，用局部变量来操作）

3. 遍历 DOM

① 使用只返回元素节点的 API 遍历 DOM, 因为这些 API 的执行效率比自己实现的效率更高：

| 属性名 | 被替代属性 |
| children | childNodes |
| childElementCount | childNodes.length |
| firstElementChild | firstChild |
| lastElementChild | lastChild |
| nextElementSibling | nextSibling |
| previousElementSibling | previousSibling |

② 选择器 API——querySelectorAll()

> querySelectorAll() 方法使用 css 选择器作为参数并返回一个 NodeList——包含着匹配节点的类数组对象，该方法不会返回 HTML 集合，因此返回的节点不会对应实时文档结构，着也避免了 HTML 集合引起的性能问题。

```javascript
let arr = document.querySelectorAll("div.warning, div.notice > p");
```

4\. 重绘和重排

> 浏览器在下载完页面的所有组件——html,js,css, 图片等之后，会解析并生成两个内部数据结构—— DOM 树，渲染树. 一旦 DOM 树和渲染树构建完成，浏览器就开始绘制页面元素（paint）.

① 重排发生的条件：

添加或删除可见的 DOM 元素位置变化 元素尺寸改变 内容改变 页面渲染器初始化 浏览器窗口尺寸变化 出现滚动条时会触发整个页面的重排

重排必定重绘

5\. 渲染树变化的排列和刷新

> 大多数浏览器通过队列化修改并批量执行来优化重排过程，然而获取布局信息的操作会导致队列强制刷新。

    `offsetTop,offsetWidth...``scrollTop,scrollHeight...``clientTop,clientHeight...``getComputedStyle()`

> 一些优化建议：将设置样式的操作和获取样式的操作分开：

```javascript
// 设置样式
body.style.color = "red";
body.style.fontSize = "24px";
// 读取样式
let color = body.style.color;
let fontSize = body.style.fontSize;
```

另外，获取计算属性的兼容写法：

```javascript
function getComputedStyle(el){
    var computed = (document.body.currentStyle ? el.currentStyle : document.defaultView.getComputedStyle(el,'');
    return computed
}
```

6\. 最小化重绘和重排

> ①. 批量改变样式

```swift
/* 使用cssText */
el.style.cssText = 'border-left: 1px; border-right: 2px; padding: 20px';
```

> ②. 批量修改 dom 的优化方案——使元素脱离文档流 - 对其应用多重改变 - 把元素带回文档

```kotlin
function appendDataToEl(option){
    var targetEl = option.target || document.body,
        createEl,
        data = option.data || [];

    var targetEl_display = targetEl.style.display;
    targetEl.style.display = 'none';


    var fragment = document.createDocumentFragment();

    for(var i=0, max = data.length; i< max; i++){
        createEl = document.createElement(option.createEl);
        for(var item in data[i]){
            if(item.toString() === 'text'){
                createEl.appendChild(document.createTextNode(data[i][item]));
                continue;
            }
            if(item.toString() === 'html'){
                createEl.innerHTML = item,data[i][item];
                continue;
            }
            createEl.setAttribute(item,data[i][item]);
        }

        fragment.appendChild(createEl);
    }

    targetEl.appendChild(fragment);

    targetEl.style.display = targetEl_display;
}

// 使用
var wrap = document.querySelectorAll('.wrap')[0];
var data = [
    {name: 'xujaing',text: '选景', title: 'xuanfij'},
    {name: 'xujaing',text: '选景', title: 'xuanfij'},
    {name: 'xujaing',text: '选景', title: 'xuanfij'}
];

appendDataToEl({
    target: wrap,
    createEl: 'div',
    data: data
});
```

> 上面的优化方法使用了文档片段:  当我们把文档片段插入到节点中时，实际上被添加的只是该片段的子节点，而不是片段本身。可以使得 dom 操作更有效率。

> ③. 缓存布局信息

```javascript
//缓存布局信息
let current = el.offsetLeft;
current++;
el.style.left = current + "px";
if (current > 300) {
  stop();
}
```

> ④. 慎用: hover

如果有大量元素使用: hover, 那么会降低相应速度，CPU 升高

> ⑤. 使用事件委托（通过事件冒泡实现）来减少事件处理器的数量，减少内存和处理时间

```javascript
function delegation(e, selector, callback) {
  e = e || window.event;
  var target = e.target || e.srcElement;

  if (
    target.nodeName !== selector ||
    target.className !== selector ||
    target.id !== selector
  ) {
    return;
  }

  if (typeof e.preventDefault === "function") {
    e.preventDefault();
    e.stopPropagation();
  } else {
    e.returnValue = false;
    e.cancelBubble = true;
  }

  callback();
}
```

#### 四. 算法和流程控制

> 1\. 循环中减少属性查找并反转 (可以提升 50%-60% 的性能)

```javascript
// for 循环
for(var i=item.length; i--){
    process(item[i]);
}
// while循环
var j = item.length;
while(j--){
    process(item[i]);
}
```

> 2\. 使用 Duff 装置来优化循环（该方法在后面的文章中会详细介绍）

> 3\. 基于函数的迭代（比基于循环的迭代慢）

```php
items.forEach(function(value,index,array){
    process(value);
})
```

> 4\. 通常情况下 switch 总比 if-else 快，但是不是最佳方案

#### 五. 字符串和正则表达式

> 1\. 除了 IE 外，其他浏览器会尝试为表达式左侧的字符串分配更多的内存，然后简单的将第二个字符串拷贝到他的末尾，如果在一个循环中，基础字符串位于最左侧，就可以避免重复拷贝一个逐渐变大的基础字符串。2. 使用\[\\s\\S]来匹配任意字符串 3. 去除尾部空白的常用做法：

```javascript
if (!String.prototype.trim) {
  String.prototype.trim = function () {
    return this.replace(/^\s+/, "").replace(/\s\s*$/, "");
  };
}
```

#### 六. 快速响应的用户界面

> 1\. 浏览器的 UI 线程：用于执行 javascript 和更新用户界面的进程。

> 2\. 在 windows 系统中定时器分辨率为 15 毫秒，因此设置小于 15 毫秒将会使 IE 锁定，延时的最小值建议为 25ms.

> 3\. 用延时数组分割耗时任务：

```javascript
function multistep(steps, args, callback) {
  var tasks = steps.concat();

  setTimeout(function () {
    var task = tasks.shift();
    task.apply(null, args || []);

    if (tasks.length > 0) {
      setTimeout(arguments.callee, 25);
    } else {
      callback();
    }
  }, 25);
}
```

> 4\. 记录代码运行时间批处理任务：

```javascript
function timeProcessArray(items, process, callback) {
  var todo = item.concat();

  setTimeout(function () {
    var start = +new Date();

    do {
      process(todo.shift());
    } while (todo.length > 0 && +new Date() - start < 50);

    if (todo.length > 0) {
      setTimeout(arguments.callee, 25);
    } else {
      callback(items);
    }
  }, 25);
}
```

> 5\. 使用 Web Worker：它引入了一个接口，能使代码运行且不占用浏览器 UI 线程的时间。一个 Worker 由如下部分组成：
>
> ① 一个 navigator 对象，包括 appName,appVersion,user Agent 和 platform.
>
> ② 一个 location 对象，只读。
>
> ③ 一个 self 对象，指向全局 worker 对象
>
> ④ 一个 importScripts() 方法，用来加载 worker 所用到的外部 js 文件
>
> ⑤ 所有的 ECMAScript 对象。如 object,Array,Date 等
>
> ⑥ XMLHttpRequest 构造器
>
> ⑦ setTimeout()，setInterval()
>
> ⑧ 一个 close() 方法，它能立刻停止 worker 运行
>
> **应用场景**
>
> 1. 编码 / 解码大字符串
>
> 2.  复杂数学运算（包括图像，视屏处理）
>
> 3.  大数组排序

```php

var worker = new Worker('code.js');

worker.onmessage = function(event){
    console.log(event.data);
}

worker.postMessage('hello');




importScripts('a.js','b.js');

self.onmessage = function(event){
    self.postMessage('hello' + event.data);
}
```

#### 七. ajax 优化

1\. 向服务器请求数据的五种方式：

> ① XMLHttpRequest
>
> ② Dynamic script tag insertion 动态脚本注入
>
> ③ iframes
>
> ④ Comet（基于 http 长连接的服务端推送技术）
>
> ⑤ Multipart XHR（允许客户端只用一个 http 请求就可以从服务器向客户端传送多个资源）

2\. 单纯向服务端发送数据（beacons 方法）——信标

```javascript
// 唯一缺点是接收到的响应类型是有限的
var url = '/req.php';
var params = ['step=2','time=123'];

(new Image()).src = url + '?' + params.join('&');

// 如果向监听服务端发送回的数据，可以在onload中实现
var beacon = new Image();
beacon.src = ...;
beacon.onload = function(){
    ...
}
beacon.onerror = function(){
    ...
}
```

3.ajax 性能的一些建议

> 缓存数据
>
> 1\. 在服务端设置 Expires 头信息确保浏览器缓存多久响应（必须 GET 请求）
>
> 2\. 客户端把获取到的信息缓存到本地，避免再次请求

#### 八. 编程实践

> 1\. 避免重复工作

```javascript
// 1.延迟加载
var a = (x, y) => {
  if (x > 4) {
    a = 0;
  } else {
    a = 1;
  }
};
// 需要使用时调用
a();

// 2.条件预加载（适用于函数马上执行并频繁操作的场景）
var b = a > 0 ? "4" : "0";
```

> 2\. 使用 Object/Array 字面量

> 3\. 多用原生方法

#### 九. 构建与部署高性能的 js 应用

> 1.js 的 http 压缩 当 web 浏览器请求一个资源时，它通常会发送一个 Accept-Encoding HTTP 头来告诉 Web 服务器它支持那种编码转换类型。这个信息主要用来压缩文档以获取更快的下载，从而改善用户体验。Accept-Encoding 可用的值包括：gzip,compress,deflate,identity. 如果 Web 服务器在请求中看到这些信息头，他会选择最合适的编码方式，并通过 Content-Encoding HTTP 头通知 WEB 浏览器它的决定。

> 2\. 使用 H5 离线缓存

> 3\. 使用内容分发网络 CDN

> 4\. 对页面进行性能分析

```javascript
// 检测代码运行时间
var Timer = {
  _data: {},
  start: function (key) {
    Timer._data[key] = new Date();
  },
  stop: function (key) {
    var time = Timer._data[key];
    if (time) {
      Timer._data[key] = new Date() - time;
    }
    console.log(Timer._data[key]);
    return Timer._data[key];
  },
};
```

#### 十. 浏览器缓存

> 1\. 添加 Expires 头

> 2\. 使用 cache-control cache-ontrol 详解   浏览器缓存机制

#### 十一. 压缩组件

> 1.web 客户端可以通过 http 请求中的 Accept-Encoding 头来表示对压缩的支持

```javascript

Accept-Encoding: gzip

Content-Encoding: gzip
```

> 2\. 压缩能将响应的数据量减少将近 70%，因此可考虑对 html, 脚本，样式，图片进行压缩

#### 十二. 白屏现象的原因

> 浏览器（如 IE）在样式表没有完全下载完成之前不会呈现页面，导致页面白屏。如果样式表放在页面底部，那么浏览器会花费更长的时间下载样式表，因此会出现白屏，所以最好把样式表放在 head 内。白屏是浏览器对 “无样式闪烁” 的修缮。如果浏览器不采用 “白屏” 机制，将页面内容逐步显示（如 Firefox），则后加载的样式表将导致页面重绘重排，将会承担页面闪烁的风险。

#### 十三. css 表达式使用一次性表达式 (但最好避免 css 表达式)

> 使用 css 表达式时执行函数重写自身

```javascript
// css
p{
    background-color: expression(altBgcolor(this))
}
// js
function altBgcolor(el){
    el.style.backgroundColor = (new Date()).getHours() % 2 ? "#fff" : "#06c";
}
```

#### 十四. 减少 DNS 查找

DNS 缓存和 TTL

    1.DNS查找可以被缓存起来以提高性能：DNS信息会留在操作系统的DNS缓存中（Microsoft Windows上的“DNS Client服务”,之后对该主机名的请求无需进行过多的查找2.TTL(time to live): 该值告诉客户端可以对记录缓存多久。建议将TTL值设置为一天// 客户端收到DNS记录的平均TTL只有最大TTL值的一半因为DNS解析器返回的时间是其记录的TTL的剩余时间，对于给定的主机名，每次执行DNS查找时接收的TTL值都会变化3.通过使用Keep-Alive和较少的域名来减少DNS查找4.一般建议将页面组件分别放到至少2个，但不要超过4个主机名下复制代码

#### 十五. 避免重定向

这块需要前后端共同配合，对页面路由进行统一规范。

### 最后

欢迎一起探索打造高性能的 web 应用，在公众号《趣谈前端》加入前端大家庭，和我们一起讨论吧！

### 汇总系列推荐

- [](http://mp.weixin.qq.com/s?__biz=MzU2Mzk1NzkwOA==&mid=2247483927&idx=1&sn=c05490e8c46e5cfba96f4f734c9e4751&chksm=fc531beccb2492fa36e4fe47471ff2f43fd533782fa3254292b029dbb1b383651b11163f54f2&scene=21#wechat_redirect)[](http://mp.weixin.qq.com/s?__biz=MzU2Mzk1NzkwOA==&mid=2247483927&idx=1&sn=c05490e8c46e5cfba96f4f734c9e4751&chksm=fc531beccb2492fa36e4fe47471ff2f43fd533782fa3254292b029dbb1b383651b11163f54f2&scene=21#wechat_redirect)[前端必不可少的 Git 使用技巧](http://mp.weixin.qq.com/s?__biz=MzU2Mzk1NzkwOA==&mid=2247483879&idx=1&sn=650a0301276ffbb3946150bdff981cb8&chksm=fc53181ccb24910a62a08a8635a81c29c2a0c446cc05cf51cdef0ce11e006ca3570e5a858e32&scene=21#wechat_redirect)
- [《javascript 高级程序设计》核心知识总结](http://mp.weixin.qq.com/s?__biz=MzU2Mzk1NzkwOA==&mid=2247483921&idx=1&sn=1566ed50bba3619cffdda36c535a4e33&chksm=fc531beacb2492fce461de9f9fdfd52ecd2e107c4c4395d6b579b4841298130434e9dfdf105a&scene=21#wechat_redirect)
- [前端开发中不可忽视的知识点汇总（一）](http://mp.weixin.qq.com/s?__biz=MzU2Mzk1NzkwOA==&mid=2247483879&idx=1&sn=650a0301276ffbb3946150bdff981cb8&chksm=fc53181ccb24910a62a08a8635a81c29c2a0c446cc05cf51cdef0ce11e006ca3570e5a858e32&scene=21#wechat_redirect)
- [css3 实战汇总（附源码）](http://mp.weixin.qq.com/s?__biz=MzU2Mzk1NzkwOA==&mid=2247483869&idx=1&sn=be680c5ac7f3e3ad1ddf60daf1569800&chksm=fc531826cb249130f8874c8db57fd6276cd7e0ca308b9c17efe1e2792370a4f1907978c4ad70&scene=21#wechat_redirect)
- [让你瞬间提高工作效率的常用 js 函数汇总 (持续更新)](http://mp.weixin.qq.com/s?__biz=MzU2Mzk1NzkwOA==&mid=2247483716&idx=1&sn=5a60c4280fef8ca2ec6dece43c4f772b&chksm=fc5318bfcb2491a91308880bfac2efa8b4df91e102a2423ed5c677e26fde389a35ef01151260&scene=21#wechat_redirect)

欢迎关注下方公众号，获取更多前端知识精粹和学习社群：

回复  **学习路径**，将获取笔者多年从业经验的前端学习路径的思维导图

回复 **lodash**，将获得本人亲自翻译的 lodash API 中文思维导图

趣谈前端

Vue、React、小程序、Node

![](https://mmbiz.qpic.cn/mmbiz_jpg/dFTfMt0114icqICY55gwgsAIS4EQyl2GnZFYOQ7DJYMnwGUpVSak6sibib9n3hAjQQKy5Mofpun9PXQY0lTspniaOQ/640?wx_fmt=jpeg)

**前端**算法 | 性能 | 架构 | 安全
[https://mp.weixin.qq.com/s?\_\_biz=MzU2Mzk1NzkwOA%3D%3D&chksm=fc531be6cb2492f04475bac0fecbd1a9f9781ca67f35bd30c320964b24a8cbc2ca3d7bbd5345&idx=1&mid=2247483933&scene=21&sn=c2729ef1fd4a28f4707bb923a5ffae79#wechat_redirect](https://mp.weixin.qq.com/s?__biz=MzU2Mzk1NzkwOA%3D%3D&chksm=fc531be6cb2492f04475bac0fecbd1a9f9781ca67f35bd30c320964b24a8cbc2ca3d7bbd5345&idx=1&mid=2247483933&scene=21&sn=c2729ef1fd4a28f4707bb923a5ffae79#wechat_redirect)
