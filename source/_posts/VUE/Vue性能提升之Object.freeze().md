---
title: Vue性能提升之Object.freeze()
categories: ['VUE']
tags: ['VUE']
---

## 序

在 Vue 的文档中介绍数据绑定和响应时，特意标注了对于经过 Object.freeze() 方法的对象无法进行更新响应。因此，特意去查了 Object.freeze() 方法的具体含义。

## 含义

Object.freeze() 方法用于冻结对象，禁止对于该对象的属性进行修改（由于`数组本质也是对象`，因此该方法可以对数组使用）。在 Mozilla MDN 中是如下介绍的：

> 可以冻结一个对象。一个被冻结的对象再也不能被修改；冻结了一个对象则不能向这个对象添加新的属性，不能删除已有属性，不能修改该对象已有属性的可枚举性、可配置性、可写性，以及不能修改已有属性的值。此外，冻结一个对象后该对象的原型也不能被修改

该方法的返回值是其参数本身。

需要注意的是以下两点

1.  Object.freeze() 和 const 变量声明不同，也不承担 const 的功能。

    const 和 Object.freeze() 完全不同

-   const 的行为像 let。它们唯一的区别是， const 定义了一个无法重新分配的变量。 通过 const 声明的变量是具有块级作用域的，而不是像 var 声明的变量具有函数作用域。
-   Object.freeze() 接受一个对象作为参数，并返回一个相同的不可变的对象。这就意味着我们不能添加，删除或更改对象的任何属性。
-   const 和 Object.freeze() 并不同，const 是防止变量重新分配，而 Object.freeze() 是使对象具有不可变性。

以下代码是正确的：

![](https://user-gold-cdn.xitu.io/2019/8/23/16cbdce0faf57e69?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

2.  Object.freeze() 是 “浅冻结”，以下代码是生效的:

![](https://user-gold-cdn.xitu.io/2019/8/23/16cbdd063e5f978d?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

## 实例

常规用法

![](https://user-gold-cdn.xitu.io/2019/8/23/16cbdd0a2ef901f8?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

明显看到，a 的 prop 属性未被改变，即使重新赋值了。

## 延伸

"深冻结"

要完全冻结具有嵌套属性的对象，您可以编写自己的库或使用已有的库来冻结对象，如[Deepfreeze](https://github.com/substack/deep-freeze)或[immutable-js](https://github.com/immutable-js/immutable-js)

```bash
// 深冻结函数.
function deepFreeze(obj) {

  // 取回定义在obj上的属性名
  var propNames = Object.getOwnPropertyNames(obj);

  // 在冻结自身之前冻结属性
  propNames.forEach(function(name) {
    var prop = obj[name];

    // 如果prop是个对象，冻结它
    if (typeof prop == 'object' && prop !== null)
      deepFreeze(prop);
  });

  // 冻结自身(no-op if already frozen)
  return Object.freeze(obj);
}
```

其实就是个简单的递归方法。但是涉及到一个很重要，但是在写业务逻辑的时候很少用的知识点 `Object.getOwnPropertyNames(obj)` 。我们都知道在 JS 的 Object 中存在原型链属性，通过这个方法可以获取所有的非原型链属性。

## 利用`Object.freeze()`提升性能

除了组件上的优化，我们还可以对 vue 的依赖改造入手。初始化时，vue 会对 data 做 getter、setter 改造，在现代浏览器里，这个过程实际上挺快的，但仍然有优化空间。

`Object.freeze()` 可以冻结一个对象，冻结之后不能向这个对象添加新的属性，不能修改其已有属性的值，不能删除已有属性，以及不能修改该对象已有属性的可枚举性、可配置性、可写性。该方法返回被冻结的对象。

当你把一个普通的 JavaScript 对象传给 Vue 实例的  `data`  选项，Vue 将遍历此对象所有的属性，并使用  [Object.defineProperty](https://link.juejin.im/?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fen-US%2Fdocs%2FWeb%2FJavaScript%2FReference%2FGlobal_Objects%2FObject%2FdefineProperty)  把这些属性全部转为 getter/setter，这些 getter/setter 对用户来说是不可见的，但是在内部它们让 Vue 追踪依赖，在属性被访问和修改时通知变化。

但 Vue 在遇到像 `Object.freeze()` 这样被设置为不可配置之后的对象属性时，不会为对象加上 setter getter 等数据劫持的方法。[参考 Vue 源码](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Fvuejs%2Fvue%2Fblob%2Fv2.5.17%2Fsrc%2Fcore%2Fobserver%2Findex.js%3F1535281657346%23L134)

**Vue observer 源码**

![](https://user-gold-cdn.xitu.io/2019/8/23/16cbdd104a327a4e?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

## 性能提升效果对比

在基于 Vue 的一个 [big table benchmark](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Fvuejs%2Fvue%2Fblob%2Fv2.5.17%2Fbenchmarks%2Fbig-table%2Findex.html%3F1535282017690) 里，可以看到在渲染一个一个 1000 x 10 的表格的时候，开启`Object.freeze()` 前后重新渲染的对比。

**big table benchmark**

![](https://user-gold-cdn.xitu.io/2019/8/23/16cbdd25cff1f24f?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

**开启优化之前**

![](https://user-gold-cdn.xitu.io/2019/8/23/16cbdd277b5f3257?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

**开启优化之后**

![](https://user-gold-cdn.xitu.io/2019/8/23/16cbdd310b5a6289?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

在这个例子里，**使用了 `Object.freeze()`比不使用快了 4 倍**

## 为什么`Object.freeze()` 的性能会更好

**不使用`Object.freeze()` 的 CPU 开销**

![](https://user-gold-cdn.xitu.io/2019/8/23/16cbdd5d9c431850?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

**使用 `Object.freeze()`的 CPU 开销**

![](https://user-gold-cdn.xitu.io/2019/8/23/16cbdd6000b42411?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

对比可以看出，使用了 `Object.freeze()` 之后，减少了 observer 的开销。

## `Object.freeze()`应用场景

由于 `Object.freeze()`会把对象冻结，所以比较适合展示类的场景，如果你的数据属性需要改变，可以重新替换成一个新的 `Object.freeze()`的对象。

## Javascript 对象解冻

修改 React props React 生成的对象是不能修改 props 的, 但实践中遇到需要修改 props 的情况. 如果直接修改, js 代码将报错, 原因是 props 对象被冻结了, 可以用 Object.isFrozen() 来检测, 其结果是 true. 说明该对象的属性是只读的.

那么, 有方法将 props 对象解冻, 从而进行修改吗?

事实上, 在 javascript 中, 对象冻结后, 没有办法再解冻, 只能通过克隆一个具有相同属性的新对象, 通过修改新对象的属性来达到目的.

可以这样:

```bash
ES6: Object.assign({}, frozenObject);
lodash: _.assign({}, frozenObject);
```

来看实际代码:

```bash
function modifyProps(component) {
  let condictioin = this.props.condictioin,
    newComponent = Object.assign({}, component),
    newProps = Object.assign({}, component.props)

  if (condictioin) {
    if (condictioin.add) newProps.add = true
    if (condictioin.del) newProps.del = true
  }
  newComponent.props = newProps

  return newComponent
}
```

**锁定对象的方法**

-   Object.preventExtensions()

no new properties or methods can be added to the project 对象不可扩展, 即不可以新增属性或方法, 但可以修改 / 删除

-   Object.seal()

same as prevent extension, plus prevents existing properties and methods from being deleted 在上面的基础上，对象属性不可删除, 但可以修改

-   Object.freeze()

same as seal, plus prevent existing properties and methods from being modified 在上面的基础上，对象所有属性只读, 不可修改

以上三个方法分别可用 Object.isExtensible(), Object.isSealed(), Object.isFrozen() 来检测

## Object.freeze( ) 阻止 Vue 无法实现 响应式系统

当一个 Vue 实例被创建时，它向 Vue 的响应式系统中加入了其 data 对象中能找到的所有的属性。当这些属性的值发生改变时，视图将会产生 “响应”，即匹配更新为新的值。但是如果使用 Object.freeze()，这会阻止修改现有的属性，也意味着响应系统无法再追踪变化。

具体使用办法举例：

```bash
<template>
  <div>
     <p>freeze后会改变吗
        {{obj.foo}}
     </p>
      <!-- 两个都不能修改？？为什么？第二个理论上应该是可以修改的-->
      <button @click="change">点我确认</button>
  </div>
</template>

<script>
var obj = {
  foo: '不会变'
}
Object.freeze(obj)
export default {
  name: 'index',
  data () {
    return {
      obj: obj
    }
  },
  methods: {
    change () {
      this.obj.foo = '改变'
    }
  }
}
</script>
```

运行后：

![](https://user-gold-cdn.xitu.io/2019/8/23/16cbdd64393f9d5f?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

从报错可以看出只读属性 foo 不能进行修改，Object.freeze() 冻结的是值，你仍然可以将变量的引用替换掉, 将上述代码更改为：

```bash
<button @click="change">点我确认</button>

change () {
      this.obj = {
        foo: '会改变'
      }
    }
```

Object.freeze() 是 ES5 新增的特性，可以冻结一个对象，冻结指的是不能向这个对象添加新的属性，不能修改其已有属性的值，不能删除已有属性，以及不能修改该对象已有属性的可枚举性、可配置性、可写性。防止对象被修改。 如果你有一个巨大的数组或 Object，并且确信数据不会修改，使用 Object.freeze() 可以让性能大幅提升。

## 实践心得和技巧

Object.freeze() 是 ES5 新增的特性，可以冻结一个对象，防止对象被修改。

vue 1.0.18 + 对其提供了支持，对于 data 或 vuex 里使用 freeze 冻结了的对象，vue 不会做 getter 和 setter 的转换。

如果你有一个巨大的数组或 Object，并且确信数据不会修改，使用 Object.freeze() 可以让性能大幅提升。在我的实际开发中，这种提升大约有 5~10 倍，倍数随着数据量递增。

并且，Object.freeze() 冻结的是值，你仍然可以将变量的引用替换掉。举个例子：

```bash
<p v-for="item in list">{{ item.value }}</p>
```

```bash
new Vue({
    data: {
        // vue不会对list里的object做getter、setter绑定
        list: Object.freeze([
            { value: 1 },
            { value: 2 }
        ])
    },
    created () {
        // 界面不会有响应
        this.list[0].value = 100;

        // 下面两种做法，界面都会响应
        this.list = [
            { value: 100 },
            { value: 200 }
        ];
        this.list = Object.freeze([
            { value: 100 },
            { value: 200 }
        ]);
    }
})
```

vue 的文档没有写上这个特性，但这是个非常实用的做法，对于纯展示的大数据，都可以使用 Object.freeze 提升性能。
 [https://juejin.cn/post/6844903922469961741](https://juejin.cn/post/6844903922469961741)
