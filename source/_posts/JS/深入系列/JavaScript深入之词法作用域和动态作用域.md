# JavaScript深入之词法作用域和动态作用域



## 作用域


作用域是指程序源代码中定义变量的区域。


作用域规定了如何查找变量，也就是确定当前执行代码对变量的访问权限。


JavaScript 采用词法作用域 (lexical scoping)，也就是静态作用域。


## 静态作用域与动态作用域


因为 JavaScript 采用的是词法作用域，函数的作用域在函数定义的时候就决定了。


而与词法作用域相对的是动态作用域，函数的作用域是在函数调用的时候才决定的。


让我们认真看个例子就能明白之间的区别：


```javascript
var value = 1;

function foo() {
    console.log(value);
}

function bar() {
    var value = 2;
    foo();
}

bar();

// 结果是 ???
```


假设 JavaScript 采用静态作用域，让我们分析下执行过程：


执行 foo 函数，先从 foo 函数内部查找是否有局部变量 value，如果没有，就根据书写的位置，查找上面一层的代码，也就是 value 等于 1，所以结果会打印 1。


假设 JavaScript 采用动态作用域，让我们分析下执行过程：


执行 foo 函数，依然是从 foo 函数内部查找是否有局部变量 value。如果没有，就从调用函数的作用域，也就是 bar 函数内部查找 value 变量，所以结果会打印 2。


前面我们已经说了，JavaScript 采用的是静态作用域，所以这个例子的结果是 1。


## 动态作用域


也许你会好奇什么语言是动态作用域？


bash 就是动态作用域，不信的话，把下面的脚本存成例如 scope.bash，然后进入相应的目录，用命令行执行 `bash ./scope.bash`，看看打印的值是多少。


```shell
value=1
function foo () {
    echo $value;
}
function bar () {
    local value=2;
    foo;
}
bar
```


这个文件也可以在 [Github 博客仓库](https://github.com/mqyqingfeng/Blog/blob/master/demos/scope/scope.bash)中找到。


## 思考题


最后，让我们看一个《JavaScript 权威指南》中的例子：


```javascript
var scope = "global scope";
function checkscope(){
    var scope = "local scope";
    function f(){
        return scope;
    }
    return f();
}
checkscope();
```


```javascript
var scope = "global scope";
function checkscope(){
    var scope = "local scope";
    function f(){
        return scope;
    }
    return f;
}
checkscope()();
```


猜猜两段代码各自的执行结果是多少？


这里直接告诉大家结果，两段代码都会打印：`local scope`。


原因也很简单，因为 JavaScript 采用的是词法作用域，函数的作用域基于函数创建的位置。


而引用《JavaScript 权威指南》的回答就是：


JavaScript 函数的执行用到了作用域链，这个作用域链是在函数定义的时候创建的。嵌套的函数 f() 定义在这个作用域链里，其中的变量 scope 一定是局部变量，不管何时何地执行函数 f()，这种绑定在执行 f() 时依然有效。


但是在这里真正想让大家思考的是：


虽然两段代码执行的结果一样，但是两段代码究竟有哪些不同呢？


如果要回答这个问题，就要牵涉到很多的内容，词法作用域只是其中的一小部分，让我们期待下一篇文章————《JavaScript 深入之执行上下文栈》。


## 
