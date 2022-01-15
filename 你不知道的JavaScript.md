---
author: [美］Kyle Simpson
title: 《你不知道的JavaScript》学习笔记
date: 2022-01-11
---

# 第一部分  作用域和闭包

## 第1章、作用域是什么？

### 是什么？

​	**作用域**是一套规则，用于确定在何处以及如何查找变量 （变量）。也就是说用来明确变量的作用范围。

### LHS、RHS

​	“L”，“R”分别代表左侧和右侧，当变量出现在赋值操作的左侧时进行`LHS` 查询，出现在右侧时进行`RHS` 查询。

​	你也可以将 RHS 理解成 retrieve his source value（取到它的源值）。

​	总之，如果查找的目的是对变量进行赋值，那么就会使用 `LHS` 查询；如果目的是获取变量的值，就会使用 `RHS` 查询。

例如：

```js
function foo(a) {
    var b = a; 
    return a + b;
}
var c = foo( 2 );
```

​	这段代码将会进行3次 `LHS` 查询，4次 `RHS` 查询。

​	当执行这段代码时，首先是对于foo的 `RHS` 查询，查找到一个函数并使用，这里有一个隐式赋值a = 2，这里会进行一次 `LHS` 查询，然后执行函数内部的语句，对a的两次 `RHS` ，对b的一次 `LHS` 一次 `RHS` ，最后是返回值后对c的一次 `LHS` 查询。

### var a = 1，JS引擎会干什么？

1. 遇到 var a，编译器会询问作用域是否已经有一个该名称的变量存在于同一个作用域的 集合中。如果是，编译器会忽略该声明，继续进行编译；否则它会要求作用域在当前作 用域的集合中声明一个新的变量，并命名为 a。
2. 接下来编译器会为引擎生成运行时所需的代码，这些代码被用来处理 a = 2 这个赋值操作。引擎运行时会首先询问作用域，在当前的作用域集合中是否存在一个叫作 a 的 变量。如果是，引擎就会使用这个变量；如果否，引擎会继续查找该变量（去上级作用域中查找），如果没找到则会异常。

### 异常

**ReferenceError（引用错误）：**

​	当对一个变量进行RHS查询时，找不到所需变量，引擎就会抛出一个ReferenceError（引用错误）的异常。

​	而对于LHS查询，如果在顶层（全局作用域）中也无法找到目标变量，全局作用域中就会自动创建一个具有该名称的变量，并将其返回给引擎使用，前提是程序运行在非 ‘严格模式’ 下。

​	ES5引入了 ‘严格模式’ ，其中一个规范就是禁止自动或隐式地创建全局变量。这时，再进行LHS时就会像RHS查询一样抛出一个ReferenceError（引用错误）的异常。

**TypeError（类型错误）：**

​	如果 RHS 查询找到了一个变量，但是你尝试对这个变量的值进行不合理的操作。

​	比如试图对一个非函数类型的值进行函数调用，或着引用 null 或 undefined 类型的值中的属性，那么引擎会抛出另外一种类型的异常，叫作 TypeError。

**总结**

​	ReferenceError 同作用域判别失败相关，而 TypeError 则代表作用域判别成功了，但是对结果的操作是非法或不合理的。

## 第2章、词法作用域

​	作用域共有两种主要的工作模型。第一种是最为普遍的，被大多数编程语言所采用的`词法作用域`，我们会对这种作用域进行深入讨论。另外一种叫作`动态作用域`，仍有一些编程语言在使用（比如Bash 脚本、Perl 中的一些模式等）。

```js
var a = 2;

function foo() {
  console.log(a); // 会输出2还是3？
}

function bar() {
  var a = 3;
  foo();
}

bar();

/*
	这里只是简单举例！！
	如果这里使用的是词法作用域，那么foo函数的上级作用域就是全局作用域，那么会输出2。
	而如果这里是动态作用域，它并不关心函数和作用域是如何声明以及在何处声明的，只关心它们从何处调用。因此这里会输出3
*/
```

[词法作用域 VS 动态作用域](https://www.jianshu.com/p/70b38c7ab69c)

### 词法作用域是什么？

​	简单地说，词法作用域就是定义在词法阶段的作用域。换句话说，词法作用域是由你在写代码时将变量和块作用域写在哪里来决定的，因此当词法分析器处理代码时会保持作用域不变（大部分情况下是这样的）。

### 欺骗词法

​	**首先，欺骗词法作用域会导致性能 下降。**

​	有两种欺骗作用域的机制：

- eval

  ​	JavaScript 中的 eval(..) 函数可以接受一个字符串为参数，并将其中的内容视为好像在书 写时就存在于程序中这个位置的代码。换句话说，可以在你写的代码中用程序生成代码并运行，就好像代码是写在那个位置的一样。
  
  *比如：*
  
  ```js
  function fn(str){
      eval(str)
      console.log(b);
  }
  var b = 1;
  fn('var b = 2;');
  
  // 对于这段代码来说，正常情况下应该输出1，但是由于eval的特性，会修改fn的作用域，在fn中重新声明了一个变量b，这样会遮蔽上级中的b，因此结果会输出2。
  ```
  
- with

  ​	with 通常被当作重复引用同一个对象中的多个属性的快捷方式，可以不需要重复引用对象本身。

  ​	with 块可以将一个对象处理为词法作用域。这里不做深究。

### 性能

​	JavaScript 引擎会在编译阶段进行数项的性能优化。其中有些优化依赖于能够根据代码的词法进行静态分析，并预先确定所有变量和函数的定义位置，才能在执行过程中快速找到标识符。

​	最悲观的情况是如果出现了 eval(..) 或 with，所有的优化可能都是无意义的，因此最简单的做法就是完全不做任何优化。

​	如果代码中大量使用 eval(..) 或 with，那么运行起来一定会变得非常慢。

​	在`严格模式`下，with 被完全禁止，而在保留核心功能的前提下，间接或非安全地使用eval(..) 也被禁止了。因此，不要使用它们。

## 第3章、函数作用域和块作用域

### 函数作用域

​	函数是 JavaScript 中最常见的作用域单元。函数作用域的含义是指，属于这个函数的全部变量都可以在整个函数的范围内使用及复 用（事实上在嵌套的作用域中也可以使用）

​	本质上，声明在一个函数内部的变量或函数会在所处的作用域中“隐藏”起来，这是有意为之的良好软件的设计原则。

### 最小特权原则

​	也叫最小授权或最小暴露原则。这个原则是指在软件设计中，应该最小限度地暴露必要内容，而将其他内容都“隐藏”起来，比如某个模块或对象的API 设计。即让一些变量或函数是私有的。

### 匿名函数的缺点

1. 匿名函数在栈追踪中不会显示出有意义的函数名，使得调试很困难。
2. 如果没有函数名，当函数需要引用自身时只能使用已经过期的 arguments.callee 引用， 比如在递归中。另一个函数需要引用自身的例子，是在事件触发后事件监听器需要解绑 自身。
3. 匿名函数省略了对于代码可读性 / 可理解性很重要的函数名。一个描述性的名称可以让代码不言自明。

始终给函数表达式命名是一个最佳实践。

### 立即执行函数

​	IIFE（Immediately Invoked Function Expression），代表立即执行函数表达式 ；

​	**(function foo(){ .. })()**  或者 **(function foo(){ .. }())**

​	这两种形式在功能上是一致的。选择哪个全凭个人喜好。

### 块作用域

- with：用 with 从对象中创建出的作用域
- try/catch： ES3 规范中规定 try/catch 的 catch 分句会创建一个块作用域
- let
- const

### let

​	ES6 引入了新的 let 关键字，在这之前的var是没有块级作用域的。

​	在声明中的任意位置都可以使用 { .. } 括号来为 let 创建一个用于绑定的块。

​	*比如：*

```js
{ // 创建了一个块级作用域
	let i = 1;
}
console.log(i) // ReferenceError
```

### 1. 垃圾收集

​	块作用域可以告诉引擎块内的变量可以回收。

​	内部的实现原理，也就是闭包的机制会在第 5 章详细解释。

### 2. let循环

```js
for (let i=0; i<10; i++) { 
	console.log( i );
}
console.log( i ); // ReferenceError

// 由于let没有变量提升（后续会讲），所以在for外面打印i时会报错；
```

```js
// 而且let所产生的块级作用域会确保使用的是上一个循环迭代结束时的值。
// 看下面两个对比

 for(var i = 1; i<=3; i++){
     setTimeout(() => { 
         console.log(i);
     },1000)
 }
// 4,4,4

 for(let i = 1; i<=3; i++){
     setTimeout(() => { 
         console.log(i);
     },1000)
 }
// 1,2,3
```

### const

​	const同样也是ES6引入的，与了let的区别就是会生命常量。当你试图去改变它的值的时候会报错。

## 第4章、提升

### js提升

​	js中代码都是分为两部，编译+执行。

​	包括变量和函数在内的所有声明都会在任何代码被执行前首先被处理，这就是**提升**。

​	当然，**只有声明本身会被提升，而赋值或其他运行逻辑会留在原地**。如果提升改变了代码执行的顺序，会造成非常严重的破坏。另外值得注意的是，**每个作用域都会进行提升操作，提升到作用域的最顶端**。

​	考虑如下代码：

```js
console.log(a);
var a = 1;

// 该段代码会输出undefined，因为在运行时a的声明会提升到代码的最开始并执行，因此上面代码等同于如下代码：
var a;
console.log(a);
a = 1;

// 这样就显然易见了。
```

​	再考虑如下代码：

```js
a = 1;
var a;
console.log(a);

/* 
	你可能认为会输出undefined，但事实上这里会输出1。
	会觉得报错的的原因可以这样分析，这段js代码顺序执行，在执行a = 1时由于处在非严格模式下，a未声明会给a在全局作用域（当前作用域）下声明一个a用于存放1，然后当继续执行到var a时，对a（同一个a，具体请参照第1章的知识）进行重新赋值，被赋予默认值undefined，因此将会打印出undefined。
	正确的思路：首先因为变量提升，var a将会最先执行，然后再执行a = 1，这里其实用的就是第二行声明的a变量，最终打印1，就等同于如下代码。
	var a;
	a = 1;
	console.log(a);
*/
```

​	再看下面代码：

```js
foo(); 
function foo() { 
	console.log( a ); // undefined 
	var a = 2;
}

// 这里不再赘述，等同于如下代码。
function foo() {
    var a;
    console.log(a);
    a = 2;
}
foo();

// 但是如果将上述代码稍作修改
foo(); // 不是 ReferenceError, 而是 TypeError! 
var foo = function bar() {
	...
};
    
// 我试图用代码去解释上面的报错。
var foo;
foo();
foo = function() {
    var bar = ...self...
	...
};
// 显然，试图对一个非函数类型的值（undefined）进行函数调用，那么将会抛出 TypeError 异常。 
// 其实这页再一次印证了‘只有声明本身会被提升，而赋值或其他运行逻辑会留在原地’，这句话。
// 同时也要记住，即使是具名的函数表达式，名称标识符在赋值之前也无法在所在作用域中。比如：
    
foo(); // TypeError 
bar(); // ReferenceError
var foo = function bar() { 
	...
};
```

### 提升顺序

​	函数声明和变量声明都会被提升。但是一个值得注意的细节是**函数会首先被提升**，然后才是变量。

​	考虑以下代码：

```js
foo(); // 1 
var foo;
function foo() { 
	console.log( 1 );
}
foo = function() { 
    console.log( 2 );
};

//会输出 1 而不是 2 ！这个代码片段会被引擎理解为如下形式：
function foo() { 
    console.log( 1 );
}
foo(); // 1
foo = function() {
    console.log( 2 );
};
// 注意，var foo 尽管出现在 function foo()... 的声明之前，但它是重复的声明（因此被忽略了），因为函数声明会被提升到普通变量之前。

// 尽管重复的 var 声明会被忽略掉，但出现在后面的函数声明还是可以覆盖前面的函数声明。
foo(); // 3
function foo() { 
    console.log( 1 );
}
var foo = function() { 
    console.log( 2 );
};
function foo() { 
    console.log( 3 );
}
```

### 最后

​	**尽量避免重复声明！！！**

## 第5章、作用域闭包

## 闭包

### 定义

​	【本书】：当函数可以记住并访问所在的词法作用域时，就产生了闭包，即使函数是在当前词法作用域之外执行。

​	【[MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Closures)】：一个函数和对其周围状态（**lexical environment，词法环境**）的引用捆绑在一起（或者说函数被引用包围），这样的组合就是**闭包**（**closure**）。

```js
function foo() { 
    var a = 2;
    function bar() { // 这就是一个闭包
    	console.log( a ); // 使用了foo作用域中的a
    } 
    return bar; 
}
var baz = foo();
baz(); // 这就是闭包的效果。
```

### 闭包的作用

1. 在 foo() 执行后，通常会期待 foo() 的整个内部作用域都被销毁，因为我们知道引擎有垃圾回收器用来释放不再使用的内存空间。由于看上去 foo() 的内容不会再被使用，所以很自然地会考虑对其进行回收。
   	而闭包的“神奇”之处正是可以阻止这件事情的发生。事实上内部作用域依然存在，因此没有被回收。
2. 闭包使得函数可以继续访问定义时的词法作用域。
3. 用来创建模块。

### 常见的闭包

​	在定时器、事件监听器、 Ajax 请求、跨窗口通信、Web Workers 或者任何其他的异步（或者同步）任务中，只要使用了`回调函数`，实际上就是在使用闭包！比如：

```js
function wait(message) {
    setTimeout( function timer() {
        console.log( message );
	}, 1000 );
}
wait( "Hello, closure!" );

// 将一个内部函数（名为 timer）传递给 setTimeout(..)。timer具有涵盖 wait(..) 作用域的闭包，因此还保有对变量 message 的引用。
```

### 闭包的使用

举例：

```js
for (let i=1; i<=5; i++) { 
	setTimeout( function timer() { 
		console.log( i );
	}, i*1000 );
}

// for 循环头部的 let 声明还会有一个特殊的行为。这个行为指出变量在循环过程中不止被声明一次，每次迭代都会声明。随后的每个迭代都会使用上一个迭代结束时的值来初始化这个变量。

// 块作用域 + 闭包
```

## 模块

### 模块模式需要具备两个必要条件

1. 必须有外部的封闭函数，该函数必须至少被调用一次（每次调用都会创建一个新的模块实例）。
2. 封闭函数必须返回至少一个内部函数，这样内部函数才能在私有作用域中形成闭包，并且可以访问或者修改私有的状态。

### 以前的模块机制

```js
var MyModules = (function Manager() { 
    var modules = {};
	function define(name, deps, impl) {
        for (var i=0; i<deps.length; i++) { 
            deps[i] = modules[deps[i]];
		}
        modules[name] = impl.apply( impl, deps );
	}
	function get(name) { 
        return modules[name];
    }
	return { 
        define: define,
        get: get
	};
})();

// 下面展示了如何使用它来定义模块： 
MyModules.define( "bar", [], function() { 
    function hello(who) { 
        return "Let me introduce: " + who;
    }
	return { 
        hello: hello
    };
});
MyModules.define( "foo", ["bar"], function(bar) { 
    var hungry = "hippo";
	function awesome() { 
        console.log( bar.hello( hungry ).toUpperCase());
    }
    return { 
        awesome: awesome
    }; 
});

var bar = MyModules.get( "bar" );
var foo = MyModules.get( "foo" );

console.log( bar.hello( "hippo" )); // Let me introduce: hippo
foo.awesome(); // LET ME INTRODUCE: HIPPO
```

### es6的模块机制

- import 可以将一个模块中的一个或多个API 导入到当前作用域中，并分别绑定在一个变量上。

- module 会将整个模块的API 导入并绑定到一个变量上。

- export 会将当前模块的一个标识符（变量、函数）导出为公共API。

