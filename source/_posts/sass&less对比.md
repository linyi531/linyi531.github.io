---
title: sass&less对比
date: 2019-10-11 14:09:00
tags:
  - CSS
  - SASS
  - LESS
categories: CSS
cover_img: https://tva1.sinaimg.cn/large/006y8mN6ly1g864pxp85lj30u0140b29.jpg
feature_img: https://tva1.sinaimg.cn/large/006y8mN6ly1g864pxp85lj30u0140b29.jpg
---

# sass&less

##为什么要使用 CSS 预处理器？

作为前端开发人员，大家都知道，Js 中可以自定义变量，而 CSS 仅仅是一个标记语言，不是编程语言，因此不可以自定义变量，不可以引用等等。

**CSS 有具体以下几个缺点：**

语法不够强大，比如无法嵌套书写，导致模块化开发中需要书写很多重复的选择器；

没有变量和合理的样式复用机制，使得逻辑上相关的属性值必须以字面量的形式重复输出，导致难以维护。

这就导致了我们在工作中无端增加了许多工作量。而使用 CSS 预处理器，提供 CSS 缺失的样式层复用机制、减少冗余代码，提高样式代码的可维护性。大大提高了我们的开发效率。

但是，CSS 预处理器也不是万金油，CSS 的好处在于简便、随时随地被使用和调试。预编译 CSS 步骤的加入，让我们开发工作流中多了一个环节，调试也变得更麻烦了。更大的问题在于，预编译很容易造成后代选择器的滥用。

所以我们在实际项目中衡量预编译方案时，还是得想想，比起带来的额外维护开销，CSS 预处理器有没有解决更大的麻烦。

## 主要区别：

首先 sass 和 less 都是 css 的预编译处理语言，他们引入了 mixins，参数，嵌套规则，运算，颜色，名字空间，作用域，JavaScript 赋值等 加快了 css 开发效率,当然这两者都可以配合 gulp 和 grunt 等前端构建工具使用

sass 和 less**主要区别:在于实现方式 less 是基于 JavaScript 的在客户端处理 所以安装的时候用 npm，sass 是基于 ruby 所以在服务器处理。**

## SASS 介绍

Sass 是 Ruby 开发者为前端开发提供的处理 CSS 的工具。它为 CSS 提供更动态的设定方式，允许编译、变量、函数……总之，使 CSS 更动态，也更像一门真正的可编程语言。

1. Sass 是基于 Ruby 开发的，所以开发环境首先需要安装 Ruby。
2. 浏览器中无法编译 Sass，所以要先编译好 css 文件，再交给浏览器。Sass**不能**在浏览器环境中直接运行。

##Less 介绍

Less 是晚些产生的语言，基于 JS 进行开发，在 Node 中进行编译。所以使用时不需要安装其他语言，不过要记得**先导入 less 文件，然后导入 less.js**。提供 CDN 地址在这里：

```html
<script src="https://cdn.bootcss.com/less.js/3.0.4/less.js"></script>
<script src="https://cdn.bootcss.com/less.js/3.0.4/less.min.js"></script>
```

当然 Less 也提供服务器端的编译功能。

## Stylus 介绍

[Stylus](https://link.juejin.im?target=https%3A%2F%2Flink.zhihu.com%2F%3Ftarget%3Dhttp%3A%2F%2Flearnboost.github.io%2Fstylus%2F)相对前两者较新，可以看官方文档介绍的功能。

1.来自 NodeJS 社区，所以和 NodeJS 走得很近，与 JavaScript 联系非常紧密。还有专门 JavaScript API：[learnboost.github.io/stylus/docs…](https://link.juejin.im?target=https%3A%2F%2Flink.zhihu.com%2F%3Ftarget%3Dhttp%3A%2F%2Flearnboost.github.io%2Fstylus%2Fdocs%2Fjs.html)

2.支持 Ruby 之类等等框架

3.更多更强大的支持和功能

以 stylus 的角度来说,stylus 更加注重对 javascript( [learnboost.github.io/stylus/docs…](https://link.juejin.im?target=https%3A%2F%2Flink.zhihu.com%2F%3Ftarget%3Dhttp%3A%2F%2Flearnboost.github.io%2Fstylus%2Fdocs%2Fjs.html) ) 的利用。 当用户觉得写 stylus 函数遇到了过于复杂或者无法测试，stylus 语法不支持等需求时， 也可以直接用 js 来实现这个函数并且在 stylus 中调用。

其次从编译器源码上来看：stylus 应该比 less.js 更有条理， 感觉如果未来社区添加功能的话，stylus 的源码添加起功能来会更简单，同样，目前 stylus 的功能也更复杂。

stylus 和 sass 另一个区别在于 sass 本身会建议，以下划线(\_) 打头的文件在静态资源打包的时候不会被编译成 .css 文件【只是作为一种 import 存在】。而 stylus 没有这方面的规范。

同时写过 stylus 和 sass， 就语法简洁来看， stylus 在这方面占了很大的便宜。

## [三者对比](https://cloud.tencent.com/developer/article/1092653)

- ### 变量

Sass: \$var
Less: @var
两种语言都会有作用域的问题。一个变量只能在它被定义的代码块中使用。重复定义的变量会报错。

Stylus 对变量名没有任何限定，你可以是 \$ 开始，也可以是任意的字符，而且与变量值之间可以用冒号、空格隔开，需要注意的是 Stylus (0.22.4) 将会编译 @ 开始的变量，但其对应的值并不会赋予该变量，换句话说，在 Stylus 的变量名不要用 @ 开头。

- ### 运算赋值：

只要保持单位统一或可相互转换，就可以进行运算，包括颜色在内：
Sass:

```scss
p {
  cursor: e + -resize;
}
// 编译为
// p {
//   cursor: e-resize;
// }

body {
  margin: (14px/2);
  top: 50px + 100px;
  right: $var * 10%;
}
```

Less:

```less
@base: 5%;
@filler: @base * 2;
@other: @base + @filler;

color: #888 / 4;
background-color: @base-color + #111;
height: 100% / 2 + @filler;

@var: 1px + 5;

width: (@var + 5) * 2;

border: (@width * 2) solid black;
```

- ### 嵌套

Sass 和 Less 均允许元素嵌套。如果父子选择器均用逗号分开，那么编译时会按结合律拆开编译。
Sass 和 Less 指代上层元素均使用`&`符号。

- ### 继承

Sass 中，写好的选择器进行集成，需要`@extend`关键字。（sytlus 与 sass 相同）
Less 中，直接写入即可：`.be-extend-class;`

- ### Mixin

Sass 中，需要进行 Mixin 操作的选择器需要`@mixin`关键字，选择器后可以传入变量和默认值。

```scss
@mixin left($value: 10px) padding: $value;
```

使用时使用`@include`关键字，并可以更新变量：

```scss
@include left @include left(20px);
```

Less 中 Mixin 和继承感觉更相似，选择器在书写时就留好了变量，直接继承或更新变量即可：

```less
.be-extend-class(@width: 10px) {
  padding: @width;
}
// 使用
.be-extend-class;
.be-extend-class(20px);
```

sass：在 sass 定义 Mixins 和 less、stylus 有所不同，在声明 Mixins 时需要使用“@mixin”,然后后面紧跟 Mixins 的名，他也可以定义参数，同时可以给这个参数设置一个默认值，但参数名是使用“\$”符号开始，而且和参数值之间需要使用冒号（：）分开。另外在 sass 中调用 Mixins 需要使用“@include”，然后在其后紧跟你要调用的 Mixins 名。

less：less 中声明 Mixins 和 CSS 定义样式非常类似，可以将 Mixins 看成是一个选择器，当然 Mixins 也可以设置参数，并给参数设置默认值。不过设置参数的变量名是使用“@”开始，同样参数和默认参数值之间需要使用冒号（：）分开。

stylus：stylus 和前两者也略有不同，他可以不使用任何符号，就是直接定义 Mixins 名，然后在定义参数和默认值之间用等号（=）来连接。

- ### 注释

两种语言相同：多行注释格式可保留，单行注释格式会在编译时被删除。

```scss
/* 会被保留的注释格式 */
// 不保存的注释格式
```

- ### 颜色运算：

CSS 预处理器提供一系列**颜色函数**帮助生成主题系列颜色：
Sass：

```scss
lighten(#cc3, 10%) // #d6d65c
darken(#cc3, 10%) // #a3a329
grayscale(#cc3) // #808080 灰度
complement(#cc3) // #33c
```

Less：

```less
lighten(@color, 10%);     // return a color which is 10% *lighter* than @color
darken(@color, 10%);      // return a color which is 10% *darker* than @color

saturate(@color, 10%);    // return a color 10% *more* saturated than @color
desaturate(@color, 10%);  // return a color 10% *less* saturated than @color

fadein(@color, 10%);      // return a color 10% *less* transparent than @color
fadeout(@color, 10%);     // return a color 10% *more* transparent than @color
fade(@color, 50%);        // return @color with 50% transparency

spin(@color, 10);         // return a color with a 10 degree larger in hue than @color
spin(@color, -10);        // return a color with a 10 degree smaller hue than @color

mix(@color1, @color2);    // return a mix of @color1 and @color2

```

stylus：

```stylus
lighten(color, 10%); /* 返回的颜色在'color'基础上变亮10% */
darken(color, 10%);  /* 返回的颜色在'color'基础上变暗10% */
saturate(color, 10%);   /* 返回的颜色在'color'基础上饱和度增加10% */
desaturate(color, 10%); /* 返回的颜色在'color'基础上饱和度降低10% */

```

- ### 插入文件

两种语言相同，使用@import 关键字引入。注意后缀名，可以直接导入 css 文件。后缀名为 css 的文件不会被预处理器处理。

```scss
@import "path/filename.scss";

@import "lib.less";
@import "lib.css";
```

- ### ==高级语法：==

#### SASS

在 Sass 中，需要用 Sass 自己的一套语言编程：

1. 条件 if-else

```scss
@if lightness($color) > 30% {
  background-color: #000;
} @else {
  background-color: #fff;
}
```

2. 循环

for:

```scss
@for $i from 1 to 10 {
  .border-#{$i} {
    border: #{$i}px solid blue;
  }
}
```

while:

```scss
$i: 6;
@while $i > 0 {
  .item-#{$i} {
    width: 2em * $i;
  }
  $i: $i - 2;
}
```

each:

```scss
@each $member in a, b, c, d {
  .#{$member} {
    background-image: url("/image/#{$member}.jpg");
  }
}
```

3. 自定义函数

需要`@function`、`@return`关键字。

```scss
@function double($n) {
  @return $n * 2;
}

#sidebar {
  width: double(5px);
}
```

#### Less

Less 是使用 JS 作为编译环境的，所以它支持 JS 语法。

1. 字符串插值

```less
@base-url: "http://assets.fnord.com";
background-image: url("@{base-url}/images/bg.png");
```

2. 用反引号使用 JS 语法：

```less
@var: ` "hello" .toUpperCase() + "!" `;
```

3. 直接访问 JS 环境

```less
@height: `document.body.clientHeight`;
```
