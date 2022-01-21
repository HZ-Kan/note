

# 写在前面

​	仅用于学习

# 日志

​	创建         2021/12/10

# 参考

[官网](https://www.sass.hk/guide/)

[Scss教程](https://www.jianshu.com/p/a99764ff3c41)  

[Sass用法指南](https://www.ruanyifeng.com/blog/2012/06/sass.html)																																							

# Scss

## 安装和使用

npm全局安装

```
npm i sass -g
```

直接运行sass，可以将编译后的css输出到控制台中

```
sass test.scss
```

编译scss文件到指定css文件

```
sass test.scss test.css
```

监听文件或者目录

```
sass --watch input.scss:output.css
```

```
sass --watch test1:test2
```

编译风格

```
# nested：嵌套缩进的css代码，默认值。
# expanded：没有缩进的、扩展的css代码。
# compact：简洁格式的css代码。
# compressed：压缩后的css代码。

# 生产环境当中，一般使用最后一个选项
　sass --style compressed test.sass test.css
```

## 简介

## 语法

### 基本

- ### 变量

  变量用来存储需要在CSS中复用的信息，例如：颜色、字体、像素...

  SASS通过$符号声明变量。

  ```scss
  $primary-color: violet;
  body{
  	background-color: $primary-color;
  }
  ```

- ### 操作符

  - 算术运算符，例如+、-、*、/、%。当然也可以使用CSS3新增的计算函数`calc()`，在进行数值计算时推荐`calc`，但对于 `字符串的拼接` 或者 ` 颜色值运算 `，则可以使用。

    ```scss
    body {
        width: 10px / 100px * 100%;
    }
    ```

    编译后输出：

    ```css
    body {
      width: 10%;
    }
    ```

  - 相等运算 `==` 或 `!=`

  - 关系运算 `<, >, <=, >=`
  - 布尔运算 `and` `or` 以及 `not`

- ### 插值语句 

  `#{}` 类似于js中的 `${}`

- ### 嵌套

  以嵌套的形式使用css，过度的使用嵌套也会导致样式难以维护。

  ```scss
  ul{
  	li{
  		color: violet;
  	}
  }
  ```

- ### 父选择器

  Sass使用 `&` 代表嵌套规则外层的父选择器

  ```scss
  body{
      width: 1px;
      &:hover { width: 2px; }
  }
  ```

  编译后输出：

  ```css
  body {
    width: 1px;
  }
  body:hover {
    width: 2px;
  }
  ```

- ### 属性嵌套

  ​	有些 CSS 属性遵循相同的命名空间 (namespace)，比如 `font-family, font-size, font-weight` 都以 `font` 作为属性的命名空间。为了便于管理这样的属性，同时也为了避免了重复输入，Sass 允许将属性嵌套在命名空间中，例如：

  ```scss
  body {
      font: {
        family: fantasy;
        size: 30em;
        weight: bold;
      }
  }
  ```

  编译后输出：

  ```css
  body {
    font-family: fantasy;
    font-size: 30em;
    font-weight: bold;
  }
  ```

- ### 引入

  `@import`

  ​	SASS能够将代码分割为多个片段，并以underscore风格的下划线作为其命名前缀（_partial.scss），SASS会通过这些下划线来辨别哪些文件是SASS片段，并且不让片段内容直接生成为CSS文件，从而只是在使用`@import`指令的位置被导入。

  ​	CSS原生的`@import`会通过额外的HTTP请求获取引入的样式片段，而SASS的`@import`则会直接将这些引入的片段合并至当前CSS文件，并且不会产生新的HTTP请求。下面例子中的代码，将会在`base.scss`文件当中引入`_reset.scss`片断。

  ```scss
  // _reset.scss
  html, body, ul, ol {
    margin:  0;
    padding: 0;
  }
  
  // base.scss
  @import 'reset'; //引入的位置会影响编译后的文件
  body {
    font: 100% Helvetica, sans-serif;
    background-color: #efefef;
  }
  ```

  编译后输出：

  ```css
  html, body, ul, ol {
    margin: 0;
    padding: 0;
  }
  
  body {
    font: 100% Helvetica, sans-serif;
    background-color: #efefef;
  }
  ```

- ### 混合（mixin）

  `@mixin`

  ​	混合（Mixin）用来分组那些需要在页面中复用的CSS声明，开发人员可以通过向Mixin传递变量参数来让代码更加灵活，该特性在添加浏览器兼容性前缀的时候非常有用，SASS目前使用`@mixin`指令来进行混合操作。

  ```scss
  @mixin border-radius($radius) {
            border-radius: $radius;
        -ms-border-radius: $radius;
       -moz-border-radius: $radius;
    -webkit-border-radius: $radius;
  }
  
  .box {
    @include border-radius(10px);
  }
  ```

  上面的代码建立了一个名为border-radius的Mixin，并传递了一个变量$radius作为参数，然后在后续代码中通过`@include` border-radius(10px)使用该Mixin，最终编译的结果如下：

  ```css
  .box {
    border-radius: 10px;
    -ms-border-radius: 10px;
    -moz-border-radius: 10px;
    -webkit-border-radius: 10px; 
  }
  ```

- ### 继承

  ​	继承是SASS中非常重要的一个特性，可以通过`@extend`指令在选择器之间复用CSS属性，并且不会产生冗余的代码，下面例子将会通过SASS提供的继承机制建立一系列样式：

  ```scss
  // 这段代码不会被输出到最终生成的CSS文件，因为它没有被任何代码所继承。
  %other-styles {
      display: flex;
      flex-wrap: wrap;
  }
  
  // 下面代码会正常输出到生成的CSS文件，因为它被其接下来的代码所继承。
  %message-common {
      border: 1px solid #ccc;
      padding: 10px;
      color: #333;
  }
  
  .message {
      @extend %message-common;
  }
  
  .success {
      @extend .message; // !!
      border-color: green;
  }
  
  .error {
      @extend %message-common;
      border-color: red;
  }
  
  .warning {
      @extend %message-common;
      border-color: yellow;
  }
  ```

  上面代码将.message中的CSS属性应用到了.success、.error、.warning上面，魔法将会发生在最终生成的CSS当中。这种方式能够避免在HTML元素上书写多个class选择器，最终生成的CSS样式是下面这样的：

  ```scss
  .message, .success, .error, .warning {
    border: 1px solid #ccc;
    padding: 10px;
    color: #333; 
  }
  
  .success {
    border-color: green; 
  }
  
  .error {
    border-color: red; 
  }
  
  .warning {
    border-color: yellow; 
  }
  ```

### 高级

- ### 数据类型

  Sass支持6种数据类型。

  - 数字：`1, 2, 13, 10px`
  - 字符串：有引号字符串与无引号字符串，`"foo", 'bar', baz`
  - 颜色：`blue, #04a3f9, rgba(255,0,0,0.5)`
  - 布尔型：`true, false`
  - 空值：`null`
  - 数组 (list)：用空格或逗号作分隔符，`1.5em 1em 0 2em, Helvetica, Arial, sans-serif`
  - maps： 相当于 JavaScript 的 object，`(key1: value1, key2: value2)`

- ### 条件语句

  `if`，`@if`，`@else if`，`@else`

  `@if`可以用来判断，表达式返回值不是false或者null时，条件成立，输出{}中的代码。

  ```scss
  body{
      @if 1 + 1 == 2 { width: 100px; }
      @else if 1 + 1 < 2 { width: 200px; }
      @else { width: 300px; }
  }
  ```

  编译之后：

  ```css
  body {
    width: 100px;
  }
  ```

- ### 循环语句

  `@for`、`@while`、`@each`

  **for循环：**

  ​	这里使用的时from...to...，不会包含结束（左闭右开区间）。对应的有from...through...，两边都会包含。同时开始与结束以及` $i `必须是整数。

  ```scss
  @for $i from 1 to 10 {
      .item#{$i} {
          width: 12px * $i;
      }
  }
  ```

  编译后输出：

  ```css
  .item1 {
    width: 12px;
  }
  
  .item2 {
    width: 24px;
  }
  ```

  **while循环：**

  ​	`	@while` 指令用法与普通编程语言中的while相同。但是很少会用到。

  ```scss
  $i: 6;
  @while $i > 0 {
    .item-#{$i} { width: 2em * $i; }
    $i: $i - 2;
  }
  ```

  编译后输出：

  ```css
  .item-6 {
    width: 12em;
  }
  
  .item-4 {
    width: 8em;
  }
  
  .item-2 {
    width: 4em;
  }
  ```

  **@each：**

  ​	`@each` 指令的格式是 `$var in <list>`, `$var` 是变量名，而 `<list>` 是一连串的值，也就是值列表。类似于js，python中的 for in。

  ```scss
  $list: puma, sea-slug, egret, salamander;
  @each $animal in $list{
      .#{$animal}-icon {
        background-image: url('/images/#{$animal}.png');
      }
  }
  ```

  编译后输出;

  ```css
  .puma-icon {
    background-image: url("/images/puma.png");
  }
  
  .sea-slug-icon {
    background-image: url("/images/sea-slug.png");
  }
  
  .egret-icon {
    background-image: url("/images/egret.png");
  }
  
  .salamander-icon {
    background-image: url("/images/salamander.png");
  }
  ```

- ### 自定义函数

  SASS允许用户编写自己的函数。

  `@function`，`@return`

  ```scss
  @function fn($n) {
      @return $n * 2;
  }
  
  body {
      width: fn(5px);
  }
  ```

  编译后输出：

  ```css
  body {
    width: 10px;
  }
  ```

