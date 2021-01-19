---
title: 一个关于image访问图片跨域的问题
---



<a name="2e2d5a8d"></a>
## 一、背景

<br />项目中遇到一个问题，同一个图片在 dom 节点中使用了'img' 标签来加载，同时由于项目使用了 ThreeJS 3D 渲染引擎，在加载纹理时使用了 TextureLoader 来加载了同一张图片，而由于图片是在阿里云服务器上的，所以最后报出了如下错误，意思是在访问图片时出现了跨域问题：<br />
<br />![](https://user-gold-cdn.xitu.io/2019/3/13/169753fa27382a47?imageView2/0/w/1280/h/960/format/webp/ignore-error/1#align=left&display=inline&height=107&margin=%5Bobject%20Object%5D&originHeight=107&originWidth=1240&status=done&style=none&width=1240)<br />

<a name="e7b6a419"></a>
## 二、问题梳理


<a name="6669edd9"></a>
### 2.1 关于图片的加载

<br />图片是来自于阿里云服务器的，和本地 localhost 必然存在跨域问题。通过 dom 节点的'img' 标签来直接访问是没有问题，因为浏览器本身不会有跨域问题。问题出在通过 TextureLoader 来加载图片时出现了跨域问题。查看了 TextureLoader 的源码，发现其进一步使用了 ImageLoader 来加载图片，加载图片的代码大致如下：<br />

```bash
crossOrigin: 'anonymous',
......
var image = document.createElementNS( 'http://www.w3.org/1999/xhtml', 'img' );
......
if ( url.substr( 0, 5 ) !== 'data:' ) {
	if ( this.crossOrigin !== undefined )
	    image.crossOrigin = this.crossOrigin;
}
......
image.src = url;
```

<br />这段代码所描述的大致思路是：<br />

1. 通过 JS 代码，创建一个 img 的 dom element，然后使用这个 element 来加载图片。
2. 默认情况下，设置了 crossOrigin 的跨域属性为'anonymous'。


<br />所以，问题的关键在于，同一张图片，先用'img' 标签去加载了，然后再在 JS 代码中，创建一个'img' 并且设置了 crossOrigin 的跨域属性为'anonymous'，那么在 JS 中创建的'img' 就会出现访问图片而产生跨域的问题。<br />

<a name="9af35fcc"></a>
### 2.2 关于 crossOrigin

<br />关于 crossOrigin，我们看看 MDN 的解释。<br />

> ![](https://user-gold-cdn.xitu.io/2019/3/13/169753fa275153e0?imageView2/0/w/1280/h/960/format/webp/ignore-error/1#align=left&display=inline&height=798&margin=%5Bobject%20Object%5D&originHeight=798&originWidth=1240&status=done&style=none&width=1240)


<br />这段话，用我自己的理解来解释一下：<br />

1. 加了 crossorigin 属性，则表明图片就一定会按照 CORS 来请求图片。而通过 CORS 请求到的图片可以再次被复用到 canvas 上进行绘制。换言之，如果不加 crossorigin 属性的话，那么图片是不能再次被复用到 canvas 上去的。
2. 可以设置的值有 anonymous 以及 use-credentials，2 个 value 的作用都是设置通过 CORS 来请求图片，区别在于 use-credentials 是加了证书的 CORS。
3. 如果默认用户不进行任何设置，那么就不会发起 CORS 请求。但如果设置了除 anonymous 和 use-credentials 以外的其他值，包括空字串在内，默认会当作 anonymous 来处理。



<a name="e36988b7"></a>
### 2.3 问题总结

<br />通过前面 2 点的梳理，我们得出如下结论：<br />

1. 通过'img' 加载的图片，浏览器默认情况下会将其缓存起来。
2. 当我们从 JS 的代码中创建的'img' 再去访问同一个图片时，浏览器就不会再发起新的请求，而是直接访问缓存的图片。但是由于 JS 中的'img' 设置了 crossorigin，也就意味着它将要以 CORS 的方式请求，但缓存中的图片显然不是的，所以浏览器直接就拒绝了。连网络请求都没有发起。
3. 在 Chrome 的调试器中，在 network 面板中，我们勾选了 disable cache 选项，验证了问题确实如第 2 点所述，浏览器这时发起了请求并且 JS 的'img' 也能正常请求到图片。



<a name="2c6feff8"></a>
## 三、解决问题

<br />前面通过勾选 disable cache 来避免浏览器使用缓存图片而解决了问题，但实际用户不会这样使用啊。根据前面的梳理，'img' 不跨域请求，而 JS 中的'img' 跨域请求，所以不能访问缓存，那么是不是可以将 JS 中的'img' 也设置成不跨域呢，于是将 JS 中的'img' 的 crossorigin 设置为 undefine，结果图片是可以加载了，但又得到如下错误。<br />
<br />![](https://user-gold-cdn.xitu.io/2019/3/13/169753fa27647a7f?imageView2/0/w/1280/h/960/format/webp/ignore-error/1#align=left&display=inline&height=316&margin=%5Bobject%20Object%5D&originHeight=316&originWidth=1240&status=done&style=none&width=1240)<br />
<br />这段错误的意思是，这一个来自于 CORS 的图片，是不可以再次被复用到 canvas 上去的。这就验证了关于 crossorigin 中的第 1 点。<br />
<br />既然'img' 和 JS 中的'img' 都不加 crossorigin 不能解决 canvas 重用的问题，那么在两边同时都加上 crossorigin 呢？果然，在'img' 中和 JS 中的'img' 都加上 crossorigin = "anonymous"，图片可以正常加了，同时也可以被复用到'canvas' 上去了。<br />
<br />另外，需要注意的 2 个小问题是：<br />

1. 服务器必须加上字段，否则，客户端设置了也是没用的。



> Access-Control-Allow-Origin: *



2. 如果是已经出了问题，你才看到这篇文章，或者才去想到这么解决。那么要记得先清理一下游览器所缓存的图片。否则你就会发现，有的图片可以访问，而有的不可以。那是因为缓存中之前存储了未 CORS 的图片。



<a name="2d325051"></a>
## 四、总结

<br />前面说了一框，只是想把这个过程完整的记录下来。整个问题的总结是：<br />

1. 同一张图片或者同一个地址，同时被'img' 所访问，而随后后又会被如 JS 中去访问。而图片存储的地址是跨域的，那么就可能因为缓存问题而导致 JS 中的访问出现跨域问题。
2. 解决的办法是让'img' 标签和 JS 中的访问都走跨域访问的方式，这样既可以解决跨域访问的问题，也可以解决跨域图片在 canvas 中的复用。


<br />最后，感谢你能读到并读完此文章，如果分析的过程中存在错误或者疑问都欢迎留言讨论。如果我的分享能够帮助到你，还请记得帮忙点个赞吧，谢谢。<br />
<br />
<br />[安装掘金浏览器插件](https://juejin.cn/extension/?utm_source=juejin.cn&utm_medium=post&utm_campaign=extension_promotion)<br />
<br />打开新标签页发现好内容，掘金、GitHub、Dribbble、ProductHunt 等站点内容轻松获取。快来安装掘金浏览器插件获取高质量内容吧！<br />[https://juejin.cn/post/6844903795726483463](https://juejin.cn/post/6844903795726483463)
