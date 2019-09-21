---
title: 字符编码：Unicode与Javascript
date: 2019-04-06 19:08:55
tags:
  - javascript
categories: javascript
cover_img: https://tva1.sinaimg.cn/large/006y8mN6ly1g77hcau3fij31940u01kx.jpg
feature_img: https://tva1.sinaimg.cn/large/006y8mN6ly1g77hcau3fij31940u01kx.jpg
---

# 字符编码：Unicode 与 Javascript

## 1.**ASCII 码**

我们知道，计算机内部，所有信息最终都是一个二进制值。每一个二进制位（bit）有`0`和`1`两种状态，因此八个二进制位就可以组合出 256 种状态，这被称为一个字节（byte）。也就是说，一个字节一共可以用来表示 256 种不同的状态，每一个状态对应一个符号，就是 256 个符号，从`00000000`到`11111111`。

上个世纪 60 年代，美国制定了一套字符编码，对英语字符与二进制位之间的关系，做了统一规定。这被称为 ASCII 码，一直沿用至今。

ASCII 码一共规定了 128 个字符的编码，比如空格`SPACE`是 32（二进制`00100000`），大写的字母`A`是 65（二进制`01000001`）。这 128 个符号（包括 32 个不能打印出来的控制符号），只占用了一个字节的后面 7 位，最前面的一位统一规定为`0`。

 <!-- more -->

## 2.非 ASCII 编码

英语用 128 个符号编码就够了，但是用来表示其他语言，128 个符号是不够的。比如，在法语中，字母上方有注音符号，它就无法用 ASCII 码表示。于是，一些欧洲国家就决定，利用字节中闲置的最高位编入新的符号。比如，法语中的`é`的编码为 130（二进制`10000010`）。这样一来，这些欧洲国家使用的编码体系，可以表示最多 256 个符号。

这里就又出现了新的问题。不同的国家有不同的字母，因此，哪怕它们都使用 256 个符号的编码方式，代表的字母却不一样。比如，130 在法语编码中代表了`é`，在希伯来语编码中却代表了字母`Gimel` (`ג`)，在俄语编码中又会代表另一个符号。但是不管怎样，所有这些编码方式中，0--127 表示的符号是一样的，不一样的只是 128--255 的这一段。

至于亚洲国家的文字，使用的符号就更多了，汉字就多达 10 万左右。一个字节只能表示 256 种符号，肯定是不够的，就必须使用多个字节表达一个符号。比如，简体中文常见的编码方式是 GB2312，使用两个字节表示一个汉字，所以理论上最多可以表示 256 x 256 = 65536 个符号。

中文编码的问题需要专文讨论，此处不涉及。这里只指出，虽然都是用多个字节表示一个符号，但是 GB 类的汉字编码与后文的 Unicode 和 UTF-8 是毫无关系的。

## 3.Unicode

Unicode 源于一个很简单的想法：将全世界所有的字符包含在一个集合里，计算机只要支持这一个字符集，就能显示所有的字符，再也不会有乱码了。

Unicode 当然是一个很大的集合，现在的规模可以容纳 100 多万个符号。每个符号的编码都不一样，比如，`U+0639`表示阿拉伯字母`Ain`，`U+0041`表示英语的大写字母`A`，`U+4E25`表示汉字`严`。具体的符号对应表，可以查询[unicode.org](http://www.unicode.org/)，或者专门的[汉字对应表](http://www.chi2ko.com/tool/CJK.htm)。

### 码点

**它从 0 开始，为每个符号指定一个编号，这叫做"码点"（code point）。**比如，码点 0 的符号就是 null（表示所有二进制位都是 0）。

```javascript
U+0000 = null
```

上式中，U+表示紧跟在后面的十六进制数是 Unicode 的码点。

目前，Unicode 的最新版本是 7.0 版，一共收入了 109449 个符号，其中的中日韩文字为 74500 个。可以近似认为，全世界现有的符号当中，三分之二以上来自东亚文字。比如，中文"好"的码点是十六进制的 597D。

### 分区（基本平面&&辅助平面）

这么多符号，Unicode 不是一次性定义的，而是分区定义。每个区可以存放 65536 个（216）字符，称为一个平面（plane）。目前，一共有 17 个（25）平面，也就是说，整个 Unicode 字符集的大小现在是 221。

最前面的 65536 个字符位，称为基本平面（缩写 BMP），它的码点范围是从 0 一直到 216-1，写成 16 进制就是从 U+0000 到 U+FFFF。所有最常见的字符都放在这个平面，这是 Unicode 最先定义和公布的一个平面。

剩下的字符都放在辅助平面（缩写 SMP），码点范围从 U+010000 一直到 U+10FFFF。

![](https://tva1.sinaimg.cn/large/006y8mN6ly1g6iy7fmupoj30ug0fedin.jpg)

### 问题

需要注意的是，Unicode 只是一个符号集，它只规定了符号的二进制代码，却没有规定这个二进制代码应该如何存储。

比如，汉字`严`的 Unicode 是十六进制数`4E25`，转换成二进制数足足有 15 位（`100111000100101`），也就是说，这个符号的表示至少需要 2 个字节。表示其他更大的符号，可能需要 3 个字节或者 4 个字节，甚至更多。

- 如何才能区别 Unicode 和 ASCII ？计算机怎么知道三个字节表示一个符号，而不是分别表示三个符号呢？
- 我们已经知道，英文字母只用一个字节表示就够了，如果 Unicode 统一规定，每个符号用三个或四个字节表示，那么每个英文字母前都必然有二到三个字节是`0`，这对于存储来说是极大的浪费，文本文件的大小会因此大出二三倍，这是无法接受的。

## 4.UTF-32 与 UTF-8

### UTF-32

**最直观的编码方法是，每个码点使用四个字节表示，字节内容一一对应码点。这种编码方法就叫做 UTF-32。**比如，码点 0 就用四个字节的 0 表示，码点 597D 就在前面加两个字节的 0。

```javascript
U+0000 = 0x0000 0000

U+597D = 0x0000 597D
```

![](https://tva1.sinaimg.cn/large/006y8mN6ly1g6iy7hhpnmj30tc0g8412.jpg)

- #### UTF-32 的优点

  转换规则简单直观，查找效率高。

- #### 缺点

  浪费空间，同样内容的英语文本，它会比 ASCII 编码大四倍。这个缺点很致命，导致实际上没有人使用这种编码方法，HTML 5 标准就明文规定，网页不得编码成 UTF-32。

### UTF-8

**UTF-8 是一种变长的编码方法，字符长度从 1 个字节到 4 个字节不等。**越是常用的字符，字节越短，最前面的 128 个字符，只使用 1 个字节表示，与 ASCII 码完全相同。

由于 UTF-8 这种节省空间的特性，导致它成为互联网上最常见的网页编码。

- #### UTF-8 的编码规则很简单，只有二条：

1）对于单字节的符号，字节的第一位设为`0`，后面 7 位为这个符号的 Unicode 码。因此对于英语字母，UTF-8 编码和 ASCII 码是相同的。

2）对于`n`字节的符号（`n > 1`），第一个字节的前`n`位都设为`1`，第`n + 1`位设为`0`，后面字节的前两位一律设为`10`。剩下的没有提及的二进制位，全部为这个符号的 Unicode 码。

- #### 编码规则：

```javascript
Unicode符号范围      |        UTF-8编码方式
(十六进制)        	 |              （二进制）
----------------------+---------------------------------------------
0000 0000-0000 007F | 0xxxxxxx
0000 0080-0000 07FF | 110xxxxx 10xxxxxx
0000 0800-0000 FFFF | 1110xxxx 10xxxxxx 10xxxxxx
0001 0000-0010 FFFF | 11110xxx 10xxxxxx 10xxxxxx 10xxxxxx
```

跟据上表，解读 UTF-8 编码非常简单。如果一个字节的第一位是`0`，则这个字节单独就是一个字符；如果第一位是`1`，则连续有多少个`1`，就表示当前字符占用多少个字节。

下面，还是以汉字`严`为例，演示如何实现 UTF-8 编码。

`严`的 Unicode 是`4E25`（`100111000100101`），根据上表，可以发现`4E25`处在第三行的范围内（`0000 0800 - 0000 FFFF`），因此`严`的 UTF-8 编码需要三个字节，即格式是`1110xxxx 10xxxxxx 10xxxxxx`。然后，从`严`的最后一个二进制位开始，依次从后向前填入格式中的`x`，多出的位补`0`。这样就得到了，`严`的 UTF-8 编码是`11100100 10111000 10100101`，转换成十六进制就是`E4B8A5`。

## 5.UTF-16

UTF-16 编码介于 UTF-32 与 UTF-8 之间，同时结合了定长和变长两种编码方法的特点。

它的编码规则很简单：基本平面的字符占用 2 个字节，辅助平面的字符占用 4 个字节。**也就是说，UTF-16 的编码长度要么是 2 个字节（U+0000 到 U+FFFF），要么是 4 个字节（U+010000 到 U+10FFFF）。**

![](https://tva1.sinaimg.cn/large/006y8mN6ly1g6iy7e7xfaj30q40e40v3.jpg)

于是就有一个问题，当我们遇到两个字节，怎么看出它本身是一个字符，还是需要跟其他两个字节放在一起解读？

在基本平面内，从 U+D800 到 U+DFFF 是一个空段，即这些码点不对应任何字符。因此，这个空段可以用来映射辅助平面的字符。

具体来说，辅助平面的字符位共有 220 个，也就是说，对应这些字符至少需要 20 个二进制位。UTF-16 将这 20 位拆成两半，前 10 位映射在 U+D800 到 U+DBFF（空间大小 210），称为高位（H），后 10 位映射在 U+DC00 到 U+DFFF（空间大小 210），称为低位（L）。这意味着，一个辅助平面的字符，被拆成两个基本平面的字符表示。

- #### 编码规则

![](https://tva1.sinaimg.cn/large/006y8mN6ly1g6iy7gmgarj30w20g2myq.jpg)

**所以，当我们遇到两个字节，发现它的码点在 U+D800 到 U+DBFF 之间，就可以断定，紧跟在后面的两个字节的码点，应该在 U+DC00 到 U+DFFF 之间，这四个字节必须放在一起解读。**

- #### Unicode 码点与 UTF-16 转码

首先区分这是基本平面字符，还是辅助平面字符。如果是前者，直接将码点转为对应的十六进制形式，长度为两字节。

```javascript
U+597D = 0x597D
```

如果是辅助平面字符，Unicode 3.0 版给出了转码公式。

```javascript
H = Math.floor((c - 0x10000) / 0x400) + 0xd800;

L = ((c - 0x10000) % 0x400) + 0xdc00;
```

## 6.JavaScript 用的是 UCS-2

JavaScript 语言采用 Unicode 字符集，但是只支持一种编码方法。**UCS-2！**

_互联网还没出现的年代，曾经有两个团队，不约而同想搞统一字符集。一个是 1988 年成立的 Unicode 团队，另一个是 1989 年成立的 UCS 团队。等到他们发现了对方的存在，很快就达成一致：世界上不需要两套统一字符集 1991 年 10 月，两个团队决定合并字符集。也就是说，从今以后只发布一套字符集，就是 Unicode，并且修订此前发布的字符集，UCS 的码点将与 Unicode 完全一致。_

**两者的关系简单说，就是 UTF-16 取代了 UCS-2，或者说 UCS-2 整合进了 UTF-16。**

**在 JavaScript 语言出现的时候，还没有 UTF-16 编码。**

**由于 JavaScript 只能处理 UCS-2 编码，造成所有字符在这门语言中都是 2 个字节，如果是 4 个字节的字符，会当作两个双字节的字符处理。**JavaScript 的字符函数都受到这一点的影响，无法返回正确结果。
![](https://tva1.sinaimg.cn/large/006y8mN6ly1g6iy7g1rsaj30xi0bogn7.jpg)

以字符![](https://tva1.sinaimg.cn/large/006y8mN6ly1g6iy7epbosj301c01g741.jpg)为例，它的 UTF-16 编码是 4 个字节的 0xD834 DF06。问题就来了，4 个字节的编码不属于 UCS-2，JavaScript 不认识，只会把它看作单独的两个字符 U+D834 和 U+DF06。前面说过，这两个码点是空的，所以 JavaScript 会认为![](https://tva1.sinaimg.cn/large/006y8mN6ly1g6iy7epbosj301c01g741.jpg)是两个空字符组成的字符串！

![](https://tva1.sinaimg.cn/large/006y8mN6ly1g6iy7gyspej30gk0go0tp.jpg)

上面代码表示，JavaScript 认为字符![](https://tva1.sinaimg.cn/large/006y8mN6ly1g6iy7epbosj301c01g741.jpg)的长度是 2，取到的第一个字符是空字符，取到的第一个字符的码点是 0xDB34。这些结果都不正确！

解决这个问题，必须对码点做一个判断，然后手动调整。下面是正确的遍历字符串的写法。

```javascript
while (++index < length) {
  // ...
  if (charCode >= 0xd800 && charCode <= 0xdbff) {
    output.push(character + string.charAt(++index));
  } else {
    output.push(character);
  }
}
```

上面代码表示，遍历字符串的时候，必须对码点做一个判断，只要落在 0xD800 到 0xDBFF 的区间，就要连同后面 2 个字节一起读取。

类似的问题存在于所有的 JavaScript 字符操作函数。

- String.prototype.replace()
- String.prototype.substring()
- String.prototype.slice()
- ...

上面的函数都只对 2 字节的码点有效。要正确处理 4 字节的码点，就必须逐一部署自己的版本，判断一下当前字符的码点范围。

## 7.ES6

ECMAScript 6（简称 ES6），大幅增强了 Unicode 支持，基本上解决了这个问题。

**（1）正确识别字符**

ES6 可以自动识别 4 字节的码点。因此，遍历字符串就简单多了。

```javascript
for (let s of string) {
  // ...
}
```

但是，为了保持兼容，length 属性还是原来的行为方式。为了得到字符串的正确长度，可以用下面的方式。

```javascript
Array.from(string).length;
```

**（2）码点表示法**

JavaScript 允许直接用码点表示 Unicode 字符，写法是"反斜杠+u+码点"。

```javascript
"好" === "\u597D"; // true
```

但是，这种表示法对 4 字节的码点无效。ES6 修正了这个问题，只要将码点放在大括号内，就能正确识别。

![](https://tva1.sinaimg.cn/large/006y8mN6ly1g6iy7f113bj30xo08mgm0.jpg)

**（3）字符串处理函数**

ES6 新增了几个专门处理 4 字节码点的函数。

- **String.fromCodePoint()**：从 Unicode 码点返回对应字符
- **String.prototype.codePointAt()**：从字符返回对应的码点
- **String.prototype.at()**：返回字符串给定位置的字符

**（4）正则表达式**

ES6 提供了 u 修饰符，对正则表达式添加 4 字节码点的支持。
![](https://tva1.sinaimg.cn/large/006y8mN6ly1g6iy7emd6cj30xu08qt95.jpg)

**（5）Unicode 正规化**

有些字符除了字母以外，还有[附加符号](http://zh.wikipedia.org/wiki/附加符号)。比如，汉语拼音的 Ǒ，字母上面的声调就是附加符号。对于许多欧洲语言来说，声调符号是非常重要的。

Unicode 提供了两种表示方法。一种是带附加符号的单个字符，即一个码点表示一个字符，比如 Ǒ 的码点是 U+01D1；另一种是将附加符号单独作为一个码点，与主体字符复合显示，即两个码点表示一个字符，比如 Ǒ 可以写成 O（U+004F） + ˇ（U+030C）。

```javascript
// 方法一
"\u01D1";
// 'Ǒ'

// 方法二
"\u004F\u030C";
// 'Ǒ'
```

这两种表示方法，视觉和语义都完全一样，理应作为等同情况处理。但是，JavaScript 无法辨别。

```javascript
"\u01D1" === "\u004F\u030C";
//false
```

ES6 提供了 normalize 方法，允许["Unicode 正规化"](http://zh.wikipedia.org/wiki/Unicode正規化)，即将两种方法转为同样的序列。

```javascript
"\u01D1".normalize() === "\u004F\u030C".normalize();
// true
```

## 8.总结截取含有四字节字符的字符串不会出现乱码的方法

```javascript
let nickname = "非拉🍒非拉";
nickname.length; // 6
```

### Array.from 方法

`Array.from`这个方法能够将类数组转换为真实的数组，比如`NodeList`, `argument`等，同样，也包括字符串。

```javascript
Array.from(nickname); // ["非", "拉", "🍒", "非", "拉"]
nickname.split(""); // ["非", "拉", "�", "�", "非", "拉"]
```

使用 Array.from 把 nickname 转换后，可以看到转换成一个真实的数组了，樱桃字符占了数组中的一个位置，然后按照数组中的方法截取再进行拼接即可，而使用 split 方法拆分，则还是乱码：

```javascript
function truncated(str, num) {
  return Array.from(str)
    .slice(0, num)
    .join("");
}
truncated(nickname, 3); // 非拉🍒
```

### codePointAt()方法

在 ES6 之前， JS 的字符串以 16 位字符编码(UTF-16)为基础。每个 16 位序列(相当于 2 个字节)是一个编码单元(code unit)，可简称为码元，用于表示一个字符。字符串所有的属性与方法(如 length 属性与 charAt() 方法等)都是基于 16 位序列。

比如 length 方法、nickname[2]、split 方法等操作，都会产生异常。为此在 ES6 中，加强了对 Unicode 的支持，并且扩展了字符串对象。

对于 Unicode 码点大于 0xFFFF 的字符，是使用 4 个字节进行存储。ES6 提供了`codePointAt`方法，能够正确处理 4 个字节储存的字符，返回一个字符的码点。

```javascript
// 获取樱桃的码点
"🍒".codePointAt(0).toString(16); // 1f352

// 输出码点对应的字符
("\u{1f352}"); // 🍒
```

请注意： 在之前 Unicode 编码，均在`[\u000-\uFFFF]`之间，因此可以使用类似`\u0047`这样的编码；但是现在码点超过`\uFFFF`的界限，若再这样使用，则获取不到对应的字符。因此在 ES6 中，码点的字符放在中括号内，类似上面的格式（所有的码点均可以使用这种格式）：

```javascript
"\u{1f352}"; // 🍒
"\u{47}"; // G
"\u{0047}"; // G
```

那么就容易了：判断需要截取的位置是否正好是 4 字节的字符，如果是则延长一位截取，否则正常截取：

```javascript
function truncated(str, num) {
  let index = Array.from(str)[num - 1].codePointAt(0) > 0xffff ? num + 1 : num;
  return str.slice(0, index);
}
truncated(nickname, 3); // 非拉🍒
```

### for-of

`for-in`方法是遍历 key 值，`for-of`是遍历 value 值：

```javascript
let arr = ["a", "b", "c"];
for (let k in arr) {
  console.log(k); // 0 1 2
}

for (let v of arr) {
  console.log(v); // a b c
}

for (let v of nickname) {
  console.log(v); // 非 拉 🍒 非 拉
}
```

因此利用这个功能，我们也能进行截取：

```javascript
function truncated(str, num) {
  let s = "";
  for (let v of nickname) {
    s += v;
    num--;
    if (num <= 0) {
      break;
    }
  }
  return s;
}
truncated(nickname, 3);
```

### 正确输出字符串的字符个数：

```javascript
function getLen(str) {
  var len = str.length;
  for (var i = 0; i < len; i++) {
    var charCode = str.charCodeAt(i);
    if (charCode >= 0xd800 && charCode <= 0xdbff) {
      len--;
      i++;
    }
  }
  return len;
}
```

```javascript
Array.from(str).length;
```

## 9.string.length

### string.length 返回的是什么 字符的个数还是字节数？为什么会与实际长度不一样？编码的部分详细说

string.length():
返回字符串的长度（以字节为单位）。是符合字符串内容的实际字节数，不一定等于其容量。

string.size()和 string.length()是同义词，并返回完全相同的值。

string.max_size()：
返回字符串的最大大小，返回字符串可以达到的最大长度。

string.resize():
string.resize(n)：把字符串的长度设置为 n 个字符
如果 n 小于当前字符串长度 ，则只截取前 n 个字符，删除超出第 n 个字符的字符。
如果大于，则在末端插入尽可能多的字符来扩展当前内容，以达到大小 n。 如果指定 c，则新元素将初始化为 c 的副本，否则为值初始化字符（空字符）。

string.capacity()：
返回已分配存储的大小。当前为字符串分配的存储空间的大小，以字节表示。

此容量不一定等于字符串长度。 它可以相等或更大，额外的空间允许对象在将新字符添加到字符串时优化其操作。
