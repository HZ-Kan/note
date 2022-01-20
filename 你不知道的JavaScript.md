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

### 闭包

#### 定义

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

#### 闭包的作用

1. 在 foo() 执行后，通常会期待 foo() 的整个内部作用域都被销毁，因为我们知道引擎有垃圾回收器用来释放不再使用的内存空间。由于看上去 foo() 的内容不会再被使用，所以很自然地会考虑对其进行回收。
   	而闭包的“神奇”之处正是可以阻止这件事情的发生。事实上内部作用域依然存在，因此没有被回收。
2. 闭包使得函数可以继续访问定义时的词法作用域。
3. 用来创建模块。

#### 常见的闭包

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

#### 闭包的使用

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

### 模块

#### 模块模式需要具备两个必要条件

1. 必须有外部的封闭函数，该函数必须至少被调用一次（每次调用都会创建一个新的模块实例）。
2. 封闭函数必须返回至少一个内部函数，这样内部函数才能在私有作用域中形成闭包，并且可以访问或者修改私有的状态。

#### 以前的模块机制

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

#### es6的模块机制

- import 可以将一个模块中的一个或多个API 导入到当前作用域中，并分别绑定在一个变量上。

- module 会将整个模块的API 导入并绑定到一个变量上。

- export 会将当前模块的一个标识符（变量、函数）导出为公共API。

# 第二部分 this和对象原型

## 第1章、关于this

### 首先

​	`this` 关键字是 JavaScript 中最复杂的机制之一。它是一个很特别的关键字，**被自动定义在所有函数的作用域中**。

看一段代码：

```js
function identify() { 
	return this.name.toUpperCase();
}
function speak() { 
    var greeting = "Hello, I'm " + identify.call( this ); 
    console.log( greeting );
}
var me = { 
    name: "Kyle"
};
var you = { 
    name: "Reader"
};

identify.call( me ); // KYLE 
identify.call( you ); // READER

speak.call( me ); // Hello, 我是 KYLE
speak.call( you ); // Hello, 我是 READER

// call可以用于修改this的指向

// 上面代码其实就等同于你在函数中声明一个形参用来接收传递过来的数据
function identify(context) { 
    return context.name.toUpperCase();
}
function speak(context) { 
    var greeting = "Hello, I'm " + identify( context ); 
    console.log( greeting );
}

identify( you ); // READER
speak( me ); //hello, 我是 KYLE
```

### 那么为什么要使用this呢？

​	随着你的使用模式越来越复杂，**显式**传递上下文对象会让代码变得越来越混乱，使用 this 则不会这样，this 提供了一种更优雅的方式来**隐式**“传递”一个对象引用，因此可以将API 设计得更加简洁并且易于复用。

​	具体当我们介绍对象和原型时，你就会明白函数可以自动引用合适的上下文对象有多重要。

### this的误解

#### 1.指向自身

​	人们很容易把 this 理解成指向函数自身。然而并不是。

​	我尝试通过下面一个例子来论证。

```js
function foo(num) {
	console.log( "foo: " + num );
    // 记录 foo 被调用的次数 
    this.count++;
} 
foo.count = 0;
window.count = 0;
var i;
for (i=0; i<10; i++) { 
    if (i > 5) {
        foo( i );
    }
} 
// foo 被调用了多少次？
console.log( foo.count ); // 如果this指向的函数本身的话，那么这里应该会输出4，然而这里会输出0。
console.log( window.count ); // 再来看这个输出，输出是4。很显然函数foo里的this指向的是window，而不是foo本身。原因在后面将会说到。
```

​	如果你想让函数里的this指向函数本身的话，可以使用 call，apply，bind 函数来改变this的指向。

​	这里以call举例：

```
function foo(num) {
	console.log( "foo: " + num );
    this.count++;
} 
foo.count = 0;
var i;
for (i=0; i<10; i++) { 
    if (i > 5) {
        foo.call(foo, i); // 改变foo中this的指向，指向foo
    }
} 
console.log( foo.count ); // 输出4
```

#### 2.指向它的作用域

​	第二种常见的误解是，this 指向函数的作用域。*这个问题有点复杂，因为在某种情况下它是正确的，但是在其他情况下它却是错误的。*（这句话爷也不懂，书里也没提到正确的情况）

​	this 在任何情况下都不指向函数的词法作用域。作用域“对象”无法通过 JavaScript 代码访问，它存在于 JavaScript 引擎内部。

思考这样一段代码：

这里试图使用 this 来隐式引用函数的词法作用域。

```js
function foo() { 
    var a = 2; 
    this.bar(); // 这里的this并不是指向函数的作用域，所以下面的this不会拿到foo中的a。
}
function bar() {
    console.log( this.a ); // 试图访问foo函数作用域中的a，报错。
}
foo(); // ReferenceError: a is not defined
```

### this到底是什么

​	`this` 的绑定和函数声明的位置没有任何关系，**只取决于函数的调用方式**。

​	当一个函数被调用时，会创建一个活动记录（有时候也称为执行上下文）。这个记录会包含函数在哪里被调用（调用栈）、函数的调用方法、传入的参数等信息。this 就是记录的其中一个属性，会在函数执行的过程中用到。

**总之，this非常重要！**

## 第2章、this全面解析

### 调用位置

​	调用位置就是函数在代码中被调用的位置（而不是声明的位置）。

​	下面我们来看看到底什么是调用栈和调用位置：

```js
function baz() { 
    // 当前调用栈是：baz 
    // 因此，当前调用位置是全局作用域
	console.log( "baz" );
    bar(); // <-- bar 的调用位置
}
function bar() {
    // 当前调用栈是 baz -> bar 
    // 因此，当前调用位置在 baz 中
    console.log( "bar" );
    foo(); // <-- foo 的调用位置
}
function foo() { 
    // 当前调用栈是 baz -> bar -> foo
    // 因此，当前调用位置在 bar 中
    console.log( "foo" );
}
baz(); // <-- baz 的调用位置
```

### 绑定规则

#### 1.默认规则

​	最常用的函数调用类型：独立函数调用。

```js
function foo() {
	console.log( this.a );
}
var a = 2;
foo(); // 2
// 这里的foo()其实是window.foo()（浏览器中），函数调用时应用了 this 的默认绑定，绑定的就是全局对象。
```

​	**注：如果使用严格模式（strict mode），那么全局对象将无法使用默认绑定，因此 this 会绑定到 undefined：**对于默认绑定来说，决定 this 绑定对象的并不是调用位置是否处于严格模式，而是函数体是否处于严格模式。如果函数体处于严格模式，this 会被绑定到 undefined，否则this 会被绑定到全局对象。

```js
function foo() {
    "use strict";
    console.log( this.a );
}
var a = 2;
foo(); // TypeError: this is undefined

// 另外，要注意使用严格模式的位置
function foo() { 
    console.log( this.a );
} 
var a = 2;
(function(){
    "use strict";
    foo(); // 2
})();

// 尽量不要这样混合使用严格模式与非严格模式，了解即可
```

#### 2.隐式绑定

​	另一条需要考虑的规则是调用位置是否有上下文对象，或者说是否被某个对象拥有或者包含。

```js
// 举个栗子
// 隐式绑定规则会把函数调用中的 this 绑定到这个上下文对象
function foo() {
    console.log( this.a );
}
var obj = { 
    a: 2, 
    foo: foo
};
obj.foo(); // 2
```

​	对象属性引用链中只有最顶层或者说最后一层会影响调用位置。举例来说：

```js
function foo() { 
     console.log( this.a );
}
var obj2 = {
    a: 42, 
    foo: foo
};
var obj1 = { 
    a: 2,
    obj2: obj2
};
obj1.obj2.foo(); // 42
```

​	隐式丢失：一个最常见的 this 绑定问题就是被隐式绑定的函数会丢失绑定对象，也就是说它会应用默认绑定。

```js
function foo() { 
    console.log( this.a );
}
var obj = { 
    a: 2, 
    foo: foo
};
var bar = obj.foo; // 函数别名！ 
var a = "oops, global"; // a 是全局对象的属性
bar(); // "oops, global"
```

#### 3.显示绑定

​	通过call(..) 和 apply(..) 来实现显示的this绑定。他们的第一个参数是一个对象，也就是this将要绑定的对象。

```js
function foo() { 
    console.log( this.a );
}
var obj = { 
    a:2
};
foo.call( obj ); // 2

// 如果你传入了一个原始值（字符串类型、布尔类型或者数字类型）来当作 this 的绑定对 象，这个原始值会被转换成它的对象形式（也就是 new String(..)、new Boolean(..) 或者new Number(..)）。这通常被称为“装箱”。
```

​	从 this 绑定的角度来说，call(..) 和 apply(..) 是一样的，它们的区别体现 在其他的参数上。call可以添加多个参数，而apply则是要传递一个数组。 这里不多做介绍，但请注意，在你调用它们的时候会执行一次你所要绑定this的那个函数。

​	可惜的是，显式绑定仍然无法解决我们之前提出的丢失绑定问题。

##### 硬绑定

​	显式绑定的一个变种可以解决这个问题。 

思考下面的代码：

```js
function foo() { 
    console.log( this.a );
}
var obj = { 
    a:2
};
var bar = function() {
    foo.call( obj );
};
bar(); // 2 
setTimeout( bar, 100 ); // 2
```

​	我们来看看这个变种到底是怎样工作的。我们创建了函数 bar()，并在它的内部手动调用了foo.call(obj)，因此强制把 foo 的 this 绑定到了 obj。无论之后如何调用函数 bar，它总会自动在 obj 上调用 foo。这种绑定是一种显式的强制绑定，因此我们称之为**硬绑定**。

硬绑定的典型应用场景就是创建一个包裹函数，传入所有的参数并返回接收到的所有值：

```js
function foo(something) {
    console.log( this.a, something );
    return this.a + something;
}
var obj = {
    a:2
};
var bar = function() { 
    return foo.apply( obj, arguments );
};
var b = bar( 3 ); // 2 3
console.log( b ); // 5
```

另一种使用方法是创建一个 i 可以重复使用的辅助函数：

```js
function foo(something) {
    console.log( this.a, something );
    return this.a + something;
}
// 简单的辅助绑定函数
function bind(fn, obj) { 
    return function() {
        return fn.apply( obj, arguments );
    };
}
var obj = {
    a:2
};
var bar = bind( foo, obj );
var b = bar( 3 ); // 2 3
console.log( b ); // 5
```

​	由于硬绑定是一种非常常用的模式，所以在 ES5 中提供了内置的方法 **Function.prototype. bind**，它的用法如下：

```js
function foo(something) {
    console.log( this.a, something );
    return this.a + something;
}
var obj = { 
    a:2
};
var bar = foo.bind( obj );
var b = bar( 3 ); // 2 3
console.log( b ); // 5

// bind(..) 会返回一个硬编码的新函数，它会把参数设置为 this 的上下文并调用原始函数。
// bind与之前的call，apply的区别就是在绑定时不会调用函数，而且他可以返回绑定了this的一个函数。
```

​	**看到这里bind的用法，不得不鼓一手掌，想到之前面试被问到call，apply，bind的区别，当时只是背的八股文，根本不懂原来bind其实是由call与apply实现的，把他们放在一起不太恰当。又想起之前看的一个博客说，面试让手写bind的实现，今天读到这里真是醍醐灌顶。这里真的是有必要花时间去认真阅读一下！！orz**

##### API调用的“上下文”

​	第三方库的许多函数，以及 JavaScript 语言和宿主环境中许多新的内置函数，都提供了一 个可选的参数，通常被称为“上下文”（context），其作用和 bind(..) 一样，确保你的回调 函数使用指定的 this。
举例来说：

```js
function foo(el) {
	console.log( el, this.id );
}
var obj = { 
    id: "awesome"
};
// 调用 foo(..) 时把 this 绑定到 obj 
[1, 2, 3].forEach( foo, obj );
// 1 awesome 2 awesome 3 awesome
```

​	这些函数实际上就是通过 call(..) 或者 apply(..) 实现了显式绑定，这样你可以少些一些 代码。

#### new绑定

​	绝大多数开发者都认为 JavaScript 中 new 的机制也和那些语言一样。然而，JavaScript 中 new 的机制实际上和面向类的语言完全不同。

​	首先我们重新定义一下 JavaScript 中的“构造函数”。在 JavaScript 中，构造函数只是一些 使用 new 操作符时被调用的函数。它们并不会属于某个类，也不会实例化一个类。实际上，它们甚至都不能说是一种特殊的函数类型，它们只是被 new 操作符调用的普通函数而已。

​	使用 new 来调用函数，或者说发生构造函数调用时，会自动执行下面的操作。 

```
 1. 创建（或者说构造）一个全新的对象。
 2. 这个新对象会被执行 [[ 原型 ]] 连接。
 3. 这个新对象会绑定到函数调用的 this。
 4. 如果函数没有返回其他对象，那么 new 表达式中的函数调用会自动返回这个新对象。
```

思考下面的代码：

```js
function foo(a) {
	this.a = a;
}
var bar = new foo(2);
console.log( bar.a ); // 2

//	使用 new 来调用 foo(..) 时，我们会构造一个新对象并把它绑定到 foo(..) 调用中的 this 上。我们称之为 new 绑定。
```

### 优先级

​	对于上面的绑定规则是设有优先级的。

​	举几个例子：

**默认最低不做比较。**

**显示与隐式的比较**

```js
// 显示 > 隐式
function foo() {
    console.log( this.a );
}
var obj1 = {
    a: 2,
    foo: foo
};
var obj2 = { 
    a: 3,
    foo: foo
};
obj1.foo(); // 2
obj2.foo(); // 3

obj1.foo.call( obj2 ); // 3
obj2.foo.call( obj1 ); // 2
```

**new与隐式比较**

```js
// new > 隐式
function foo(something) {
    this.a = something;
}
var obj1 = {
    foo: foo
}; 
obj1.foo( 2 ); 
console.log( obj1.a ); // 2

var bar = new obj1.foo( 4 );
console.log( obj1.a ); // 2
console.log( bar.a ); // 4
```

**new与显示比较**

**注：new 和 call/apply 无法一起使用**，因此无法通过 new foo.call(obj1) 来直接进行测试。但是我们可以使用硬绑定来测试它俩的优先级。

```js
// new > 显示
function foo(something) { 
    this.a = something;
} 
ar obj1 = {};

var bar = foo.bind( obj1 );
bar( 2 );
console.log( obj1.a ); // 2

var baz = new bar(3); 
console.log( obj1.a ); // 2
console.log( baz.a ); // 3
```

​	为什么要在 new 中使用硬绑定函数呢？直接使用普通函数不是更简单吗？ 

​	之所以要在 new 中使用硬绑定函数，主要目的是预先设置函数的一些参数，这样在使用 new 进行初始化时就可以只传入其余的参数。bind(..) 的功能之一就是可以把除了第一个 参数（第一个参数用于绑定 this）之外的其他参数都传给下层的函数（这种技术称为“部分应用”，是“**柯里化**”的一种）。

​	举例来说：

```js
function foo(p1,p2) {
	this.val = p1 + p2;
}
// 之所以使用 null 是因为在本例中我们并不关心硬绑定的 this 是什么 
// 反正使用 new 时 this 会被修改 
var bar = foo.bind( null, "p1" );
var baz = new bar( "p2" );
baz.val; // p1p2
```

**总结**

```
new绑定 > 显示绑定 > 隐式绑定 > 默认绑定
```

### 绑定总结

现在我们可以根据优先级来判断函数在某个调用位置应用的是哪条规则。可以按照**下面的顺序**来进行判断：

```js
// 1. 函数是否在 new 中调用（new 绑定）？如果是的话 this 绑定的是新创建的对象。 var bar = new foo()
// 2. 函数是否通过 call、apply（显式绑定）或者硬绑定调用？如果是的话，this 绑定的是 指定的对象。 var bar = foo.call(obj2)
// 3. 函数是否在某个上下文对象中调用（隐式绑定）？如果是的话，this 绑定的是那个上 下文对象。 var bar = obj1.foo()
// 4. 如果都不是的话，使用默认绑定。如果在严格模式下，就绑定到 undefined，否则绑定到 全局对象。var bar = foo()
```

### 绑定例外

#### 1.被忽略的this

​	如果你把 null 或者 undefined 作为 this 的绑定对象传入 call、apply 或者 bind，这些值 在调用时会被忽略，实际应用的是默认绑定规则：

```js
function foo() { 
    console.log( this.a );
} 
var a = 2;
foo.call( null ); // 2
```

​	那么什么情况下你会传入 null 呢？ 一种非常常见的做法是使用 apply(..) 来“展开”一个数组，并当作参数传入一个函数。 类似地，bind(..) 可以对参数进行`柯里化`（预先设置一些参数），这种方法有时非常有用：

```js
function foo(a,b) {
    console.log( "a:" + a + ", b:" + b );
}

// 把数组“展开”成参数
foo.apply( null, [2, 3] ); // a:2, b:3

// 使用 bind(..) 进行柯里化
var bar = foo.bind( null, 2 );
bar( 3 ); // a:2, b:3
```

​	这两种方法都需要传入一个参数当作 this 的绑定对象。如果函数并不关心 this 的话，你仍然需要传入一个`占位值`，这时 null 可能是一个不错的选择，就像代码所示的那样。但在 ES6 中，可以用 ... 操作符代替 apply(..) 来“展 开”数组，foo(...[1,2]) 和 foo(1,2) 是一样的，这样可以避免不必要的 this 绑定。可惜，在 ES6 中没有柯里化的相关语法，因此还是需要使用bind(..)。

**更安全的this**

​	一种“更安全”的做法是传入一个特殊的对象，把 this 绑定到这个对象不会对你的程序产生任何副作用（包括它不会像null一样进行`默认绑定`）。就像网络（以及军队）一样，我们可以创建一个“DMZ”（demilitarized zone，非军事区）对象。比如：

```js
function foo(a,b) {
    console.log( "a:" + a + ", b:" + b );
}
// 我们的 DMZ 空对象 
// Object.create(null) 和 {} 很像，但是并不会创建 Object.prototype 这个委托（后面会讲），所以它比 {}“更空”
var ø = Object.create( null );

// 把数组展开成参数 
foo.apply( ø, [2, 3] ); // a:2, b:3

// 使用 bind(..) 进行柯里化 
var bar = foo.bind( ø, 2 );
bar( 3 ); // a:2, b:3
```

#### 2.间接引用	

​	另一个需要注意的是，你有可能（有意或者无意地）创建一个函数的“间接引用”，在这 种情况下，调用这个函数会应用默认绑定规则。
间接引用最容易在**赋值时**发生：

```js
function foo() {
    console.log( this.a );
}
var a = 2; 
var o = {
    a: 3,
    foo: foo 
}; 
var p = { a: 4 };
o.foo(); // 3
console.log(p.foo = o.foo);
(p.foo = o.foo)(); // 2
```

​	赋值表达式 p.foo = o.foo 的返回值是目标函数的引用，因此调用位置是 foo() 而不是 p.foo() 或者 o.foo()。根据我们之前说过的，这里会应用默认绑定。

#### 3.软绑定

​	之前我们已经看到过，硬绑定这种方式可以把 this 强制绑定到指定的对象（除了使用 new 时），防止函数调用应用默认绑定规则。问题在于，硬绑定会大大降低函数的灵活性，使用硬绑定之后就无法使用隐式绑定或者显式绑定来修改 this。

​	 如果可以给默认绑定指定一个全局对象和 undefined 以外的值，那就可以实现和硬绑定相 同的效果，同时保留隐式绑定或者显式绑定修改 this 的能力。 可以通过一种被称为`软绑定`的方法来实现我们想要的效果：

```js
if (!Function.prototype.softBind) { 
    Function.prototype.softBind = function(obj) { 
        var fn = this; // 捕获所有 curried 参数
        var curried = [].slice.call( arguments, 1 ); 
        var bound = function() { 
            return fn.apply( (!this || this === (window || global)) ? obj : this
curried.concat.apply( curried, arguments ) );
    };
        bound.prototype = Object.create( fn.prototype ); 
        return bound;
}; }
```

​	除了软绑定之外，softBind(..) 的其他原理和 ES5 内置的 bind(..) 类似。它会对指定的函数进行封装，首先检查调用时的 this，如果 this 绑定到全局对象或者 undefined，那就把 指定的默认对象 obj 绑定到 this，否则不会修改 this。此外，这段代码还支持可选的柯里化（详情请查看之前和 bind(..) 相关的介绍）。

下面我们看看 softBind 是否实现了软绑定功能：

```js
function foo() { 
	console.log("name: " + this.name);
}
var obj = { name: "obj" },
    obj2 = { name: "obj2" },
    obj3 = { name: "obj3" };
var fooOBJ = foo.softBind( obj );
fooOBJ(); // name: obj
obj2.foo = foo.softBind(obj);
obj2.foo(); // name: obj2 <---- 看！！！
fooOBJ.call( obj3 ); // name: obj3 <---- 看！
setTimeout( obj2.foo, 10 ); // name: obj <---- 应用了软绑定
```

可以看到，软绑定版本的 foo() 可以手动将 this 绑定到 obj2 或者 obj3 上，但如果应用默 认绑定，则会将 this 绑定到 obj。

**注：js标准中暂未添加软绑定（2022-01-20）**

### this词法

​	ES6 新增了一种无法使用上述绑定规则的特殊函数类型：箭头函数。它的绑定规则是根据外层（函数或者全局）作用域来决定的。也就是箭头函数的上下文。























































































# 附录

## 唯一一种可以从匿名函数对象内部引用自身的方法

​	arguments. callee（已被弃用）尽量使用具名函数，这样可以直接拿到函数名（函数本身）。

## js创建空对象

​	Object.create(null) 和 {} 很像，但是并不会创建 Object. prototype 这个委托，所以它比 {}“更空”：

## 柯里化



## js中的4种函数调用模式：

### 1 函数调用

包括函数名()和匿名函数调用,this指向window

```javascript
 function getSum() {
    console.log(this) //window
 }
 getSum()
 
 (function() {
    console.log(this) //window
 })()
 
 var getSum=function() {
    console.log(this) //window
 }
 getSum()
```

### 2 方法调用

对象.方法名(),this指向对象

```javascript
var objList = {
   name: 'methods',
   getSum: function() {
     console.log(this) //objList对象
   }
}
objList.getSum()
```

### 3 构造器调用

new 构造函数名(),this指向构造函数

```javascript
function Person() {
  console.log(this); //指向构造函数Person
}
var personOne = new Person();
```

### 4 间接调用

利用call和apply来实现,this就是call和apply对应的第一个参数,如果不传值或者第一个值为null,undefined时this指向window

```javascript
function foo() {
   console.log(this);
}
foo.apply('我是apply改变的this值');//我是apply改变的this值
foo.call('我是call改变的this值');//我是call改变的this值
```
