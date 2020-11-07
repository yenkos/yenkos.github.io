# CSS 变量及其应用

<a name="lkSvl"></a>
## 基本介绍
**自定义属性**（有时候也被称作**CSS变量**或者**级联变量**）是由CSS作者定义的，它包含的值可以在整个文档中重复使用。由自定义属性标记设定值（比如： **`--main-color: black;`**），由[var() ](https://developer.mozilla.org/zh-CN/docs/Web/CSS/var)函数来获取值（比如： `color: **var(--main-color)**;`）<br />
<br />复杂的网站都会有大量的CSS代码，通常也会有许多重复的值。举个例子，同样一个颜色值可能在成千上百个地方被使用到，如果这个值发生了变化，需要全局搜索并且一个一个替换（很麻烦哎～）。自定义属性在某个地方存储一个值，然后在其他许多地方引用它。另一个好处是语义化的标识。比如，`--main-text-color` 会比 `#00ff00` 更易理解，尤其是这个颜色值在其他上下文中也被使用到。自定义属性受级联的约束，并从其父级继承其值。<br />

<a name="v644F"></a>
### 与sass less 变量的区别

- sass命名是$color，less命名是@color，css命名是--color。
- 读取css变量，需要使用val()方法，sass和less可以直接使用。
- css最大优势在于不需要编译，在运行时可以随时修改，同时应用到上下文。缺点是不兼容ie浏览器。

![image.png](https://cdn.nlark.com/yuque/0/2020/png/203222/1604713577345-d881bc3c-8817-44e3-b9db-ca8e8a8aae55.png#align=left&display=inline&height=357&margin=%5Bobject%20Object%5D&name=image.png&originHeight=984&originWidth=2062&size=180395&status=done&style=none&width=749)<br />

<a name="3cWdV"></a>
### 基本用法
声明一个自定义属性，属性名需要以两个减号（`--`）开始，属性值则可以是任何有效的CSS值。和其他属性一样，自定义属性也是写在规则集之内的，如下：
```javascript
element {
  --main-bg-color: brown;
}
```
注意，规则集所指定的选择器定义了自定义属性的可见作用域。通常的最佳实践是定义在根伪类 [`:root`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:root) 下，这样就可以在HTML文档的任何地方访问到它了：
```javascript
:root {
  --main-bg-color: brown;
}
```
然而这条规则不是绝对的，如果有理由去限制你的自定义属性，那么就应该限制。<br />**<br />**注意：**自定义属性名是大小写敏感的，`--my-color` 和 `--My-color` 会被认为是两个不同的自定义属性。<br />如前所述，使用一个局部变量时用 [`var()`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/var()) 函数包裹以表示一个合法的属性值：
```javascript
element {
  background-color: var(--main-bg-color);
}
```
<a name="1w34a"></a>
### 
<a name="xrm9i"></a>
### 使用自定义属性的第一步
我们从这个简单的CSS代码开始，它将相同的颜色应用在了不同class的元素上：
```css
.one {
  color: white;
  background-color: brown;
  margin: 10px;
  width: 50px;
  height: 50px;
  display: inline-block;
}
.two {
  color: white;
  background-color: black;
  margin: 10px;
  width: 150px;
  height: 70px;
  display: inline-block;
}
.three {
  color: white;
  background-color: brown;
  margin: 10px;
  width: 75px;
}
.four {
  color: white;
  background-color: brown;
  margin: 10px;
  width: 100px;
}
.five {
  background-color: brown;
}
```
应用在如下HTML上：
```html
<div>
  <div class="one">1:</div>
  <div class="two">2: Text <span class="five">5 - more text</span></div>
  <input class="three">
  <textarea class="four">4: Lorem Ipsum</textarea>
</div>
```
其呈现是：<br />注意到在CSS代码中的重复：背景色 `brown` 被多处设置。对于一些CSS声明，是可以在级联关系更高的位置设置，通过CSS继承自然地解决这个重复的问题。但在一般项目中，是不可能通过这样的方式去解决。通过在 [`:root`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/:root) 伪类上设置自定义属性，然后在整个文档需要的地方使用，可以减少这样的重复性：
```css
:root {
  --main-bg-color: brown;
}
.one {
  color: white;
  background-color: var(--main-bg-color);
  margin: 10px;
  width: 50px;
  height: 50px;
  display: inline-block;
}
.two {
  color: white;
  background-color: black;
  margin: 10px;
  width: 150px;
  height: 70px;
  display: inline-block;
}
.three {
  color: white;
  background-color: var(--main-bg-color);
  margin: 10px;
  width: 75px;
}
.four {
  color: white;
  background-color: var(--main-bg-color);
  margin: 10px;
  width: 100px;
}
.five {
  background-color: var(--main-bg-color);
}
```
这里呈现的结果和前面的例子是一致的，但允许对所需属性值进行一个规范的声明。
<a name="340DF"></a>
### 
<a name="jFKal"></a>
### 自定义属性的继承性
自定义属性会继承。这意味着如果在一个给定的元素上，没有为这个自定义属性设置值，在其父元素上的值会被使用。看这一段HTML：
```html
<div class="one">
  <div class="two">
    <div class="three"></div>
    <div class="four"></div>
  </div>
</div>
```
配套的CSS：
```css
.two {
  --test: 10px;
}
.three {
  --test: 2em;
}
```
在这个情况下， `var(--test)` 的结果分别是：

- 对于元素 `class="two"` ：`10px`
- 对于元素 `class="three"` ：`2em`
- 对于元素 `class="four"` ：`10px` （继承自父属性）
- 对于元素 `class="one"` ：_非法值_，会变成自定义属性的默认值

注意，这些是自定义属性，并不是你在其他编程语言中遇到的实际的变量。这些值仅当需要的时候才会计算，而并不会按其他规则进行保存。比如，你不能为元素设置一个属性，然后让它从兄弟或旁支子孙规则上获取值。属性仅用于匹配当前选择器及其子孙，这和通常的CSS是一样的。
<a name="u6pZb"></a>
### 
<a name="q4uZ6"></a>
### 自定义属性备用值
用 [`var()`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/var()) 函数可以定义多个**备用值**(fallback value)，当给定值未定义时将会用备用值替换。这对于 [Custom Elements](https://wiki.developer.mozilla.org/en-US/docs/Web/Web_Components/Using_custom_elements) 和 [Shadow DOM](https://wiki.developer.mozilla.org/en-US/docs/Web/Web_Components/Using_shadow_DOM) 都很有用。<br />**备用值并不是用于实现浏览器兼容性的。**如果浏览器不支持CSS自定义属性，备用值也没什么用。**它仅对支持CSS自定义属性的浏览器提供了一个备份机制**，该机制仅当给定值未定义或是无效值的时候生效。<br />函数的第一个参数是[自定义属性](https://www.w3.org/TR/css-variables/#custom-property)的名称。如果提供了第二个参数，则表示备用值，当[自定义属性](https://www.w3.org/TR/css-variables/#custom-property)值无效时生效。第二个参数可以嵌套，但是不能继续平铺展开下去了，例如：
```css
.two {
  color: var(--my-var, red); /* Red if --my-var is not defined */
}
.three {
  background-color: var(--my-var, var(--my-background, pink)); /* pink if --my-var and --my-background are not defined */
}
.three {
  background-color: var(--my-var, --my-background, pink); /* Invalid: "--my-background, pink" */
}
```
第二个例子展示了如何处理一个以上的 fallback。该技术可能会导致性能问题，因为它花了更多的时间在处理这些变量上。<br />**注意：**像[自定义属性](https://www.w3.org/TR/css-variables/#custom-property)这些 fallback 语法允许使用逗号。比如 `var(--foo, red, blue)` 定义了一个 `red, blue` 的备用值——从第一个逗号到最后的全部内容，都会被作为备用值的一部分。
<a name="UzlaA"></a>
### 
<a name="s76k9"></a>
### 有效性和值
传统的CSS概念里，有效性和属性是绑定的，这对自定义属性来说并不适用。当自定义属性值被解析，浏览器不知道它们什么时候会被使用，所以必须认为这些值都是_有效的_。<br />不幸的是，即便这些值是有效的，但当通过 `var()` 函数调用时，它在特定上下文环境下也可能不会奏效。属性和自定义变量会导致无效的CSS语句，这引入了一个新的概念：_计算时有效性_。
<a name="JIIwG"></a>
### 
<a name="Hzeml"></a>
### 无效变量会导致什么？
当浏览器遇到无效的 `var()` 时，会使用继承值或初始值代替。<br />考虑如下代码：<br />

```html
<p>This paragraph is initial black.</p>
```


```css
:root { --text-color: 16px; } 
p { color: blue; } 
p { color: var(--text-color); }
```
毫不意外，浏览器将 `--text-color` 的值替换给了 `var(--text-color)`，但是 `16px` 并不是 [`color`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/color) 的合法属性值。代换之后，该属性不会产生任何作用。浏览器会执行如下两个步骤：

1. 检查属性 color 是否为继承属性。是，但是 `<p>` 没有任何父元素定义了 color 属性。转到下一步。
1. 将该值设置为它的**默认初始值**，比如 black。Result


<br />段落颜色并不是蓝色，因为无效代换导致了它被替换成了默认初始值的黑色。如果你直接写n `color: 16px` 的话，则会导致语法错误，而前面的定义则会生效（段落显示为蓝色）。<br />**注意：**当CSS属性-值对中存在语法错误，该行则会被忽略。然而如果自定义属性的值无效，它并不会被忽略，从而会导致该值被覆盖为默认值。
<a name="1o2YB"></a>
### 
<a name="euKww"></a>
### JavaScript 中的值
在 JavaScript 中获取或者修改 CSS  变量和操作普通 CSS 属性是一样的：
```javascript
// 获取一个 Dom 节点上的 CSS 变量
element.style.getPropertyValue("--my-var");
// 获取任意 Dom 节点上的 CSS 变量
getComputedStyle(element).getPropertyValue("--my-var");
// 修改一个 Dom 节点上的 CSS 变量
element.style.setProperty("--my-var", jsVar + 4);
```


<a name="UToIR"></a>
## 应用：全局样式，定义主题，快速切换。
<a name="grCsn"></a>
### 实现黑夜模式
深色模式为目前网络发展的一大趋势，可以看到大量的网站为了提高网站的体验都添加了深色模式。深色模式在光线不足的情况下看起来不会那么刺眼，能够很好的保护我们的眼睛。<br />在这边文章中主要讲如何使用CSS和JS实现深色模式和浅色模式的任意切换
<a name="3vp2n"></a>
### 
<a name="LRNvJ"></a>
### 分析需求
假设有这么一个页面，我们需要自由切换深色模式和浅色模式。那么就需要在不同模式使用不同的css，这里可以通过两种方式一种是直接引入不同的css文件，另外一种通过更改css变量值的方式进行更改样式，下面是浅色模式的截图
<a name="cHbVZ"></a>
### 
<a name="zwOc9"></a>
### 具体实现
首先定义浅色模式的变量名和变量值
```css
:root {
  --primary-bg: #eee;
  --primary-fg: #000;
  --secondary-bg: #ddd;
  --secondary-fg: #555;
  --primary-btn-bg: #000;
  --primary-btn-fg: #fff;
  --secondary-btn-bg: #ff0000;
  --secondary-btn-fg: #ffff00;
}
复制代码
```
当切换场景的时候需要更改css变量的值，更改如下:
```css
:root {
  --primary-bg: #282c35;
  --primary-fg: #fff;
  --secondary-bg: #1e2129;
  --secondary-fg: #aaa;
  --primary-btn-bg: #ddd;
  --primary-btn-fg: #222;
  --secondary-btn-bg: #780404;
  --secondary-btn-fg: #baba6a;
}
复制代码
```
可以看到当切换到深色模式的时候，变量使用了更加暗的颜色，从而实现深色模式
<a name="RnZkn"></a>
### 
<a name="CVrOE"></a>
### 更改css
如何切换到暗模式有多种解决方法，在这里我们使用媒体查询，`prefers-color-scheme`这个媒体查询能够获取到用户的系统是否切换到了深色主题，具体如下:
```css
@media (prefers-color-scheme: dark) {
  :root {
    --primary-bg: #282c35;
    --primary-fg: #fff;
    --secondary-bg: #1e2129;
    --secondary-fg: #aaa;
    --primary-btn-bg: #ddd;
    --primary-btn-fg: #222;
    --secondary-btn-bg: #780404;
    --secondary-btn-fg: #baba6a;
    --image-opacity: 0.85;
  }
}
复制代码
```
如果希望用户可以通过选择系统的设置来切换浅色模式还是深色模式，那么上面这种方式就足够了。浏览的网站能够通过系统设置选择不同的样式<br />但是上面这种方式存在一个问题，就是用户希望这个页面的模式不要跟随系统配置的更改而更改。用户可以主动更改网站的模式，那么上面这种方式就不合适了
<a name="Yln4Y"></a>
### 
<a name="P8NYk"></a>
### 手动选择模式
思路就是通过控制js来给元素添加不同的class，不同的class拥有不同的样式。首先添加在html中添加一个按钮用于切换不同的模式
```html
<button id="toggle-button">toggle</button>
<script>
  const toggleButton = document.querySelector('#toggle-button')
</script>
复制代码
```
然后需要地方存储用户的偏好设置，这里使用localStorage来存储用户的选择。<br />然后给按钮添加事件用于切换主题，下面是具体的代码
```javascript
const toggleButton = document.querySelector('#toggle-button')
  
  toggleButton.addEventListener('click', (e) => {
    darkMode = localStorage.getItem('theme');
    if (darkMode === 'dark') {
      disableDarkMode();
    } else {
      enableDarkMode();
    }
  });
  function enableDarkMode() {
    localStorage.setItem('theme', 'dark');
  }
  function disableDarkMode() {
    localStorage.setItem('theme', 'light');
  }
复制代码
```
现在我们就可以存储这个用户的偏好设置。然后不同的主题下给body元素添加不同的`class`，具体如下
```javascript
function enableDarkMode() {
   document.body.classList.add("dark-mode")
    localStorage.setItem('theme', 'dark');
  }
  function disableDarkMode() {
    document.body.classList.remove("dark-mode")
    localStorage.setItem('theme', 'light');
  }
复制代码
```
和媒体查询一样，在`dark-mode`的情况下更改css变量的属性值，具体如下:
```javascript
.dark-mode {
  --primary-bg: #282c35;
  --primary-fg: #fff;
  --secondary-bg: #1e2129;
  --secondary-fg: #aaa;
  --primary-btn-bg: #ddd;
  --primary-btn-fg: #222;
  --secondary-btn-bg: #780404;
  --secondary-btn-fg: #baba6a;
  --image-opacity: 0.85;
}
复制代码
```
同时在进入这个页面的时候需要获取到用户的偏好设置，从localStorage中读取
```javascript
let darkMode = localStorage.getItem("theme")
if (darkMode === "dark") enableDarkMode()
复制代码
```
这次就可以在页面刷新以后仍然拿到用户的偏好设置。<br />
<br />

