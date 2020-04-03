---
title: Html标签默认属性样式及浏览器默认样式
date: 2020-03-14 14:20:32
tags:
  - HTML
categories: HTML
cover_img: https://tva1.sinaimg.cn/large/00831rSTgy1gdgjsg25qwj30u01401kx.jpg
feature_img: https://tva1.sinaimg.cn/large/00831rSTgy1gdgjsg25qwj30u01401kx.jpg
---

# html标签默认属性样式及浏览器默认样式

## 具有默认样式的html标签

| html     | address | blockquote |
| -------- | ------- | ---------- |
| body     | dd      | div        |
| dl       | dt      | fieldset   |
| form     | frame   | frameset   |
| h1       | h2      | H3         |
| h4       | h5      | h6         |
| noframes | ol      | p          |
| ul       | center  | dir        |
| hr       | menu    | pre        |

以上列表元素默认状态下以块状显示，未显示的将以内联元素显示，该列表针对HTML4版本，部分元素在XHTML1中将废弃

```css
li {display: list-item }`/*默认以列表显示*/`
head {display: none }/*默认不显示*/
table {display: table }/*默认为表格显示*/
tr {display: table-row }/*默认为表格行显示*/
thead {display: table-header-group }/*默认为表格头部分组显示*/
tbody {display: table-row-group }/*默认为表格行分组显示*/
tfoot {display: table-footer-group }/*默认为表格底部分组显示*/
col {display: table-column }/*默认为表格列显示*/
colgroup {display: table-column-group }/*默认为表格列分组显示*/
td, th { display:table-cell; }/*默认为单元格显示*/
caption {display: table-caption }/*默认为表格标题显示*/
th {font-weight: bolder; text-align: center }/*默认为表格标题显示，呈现加粗居中状态*/
caption {text-align: center }/*默认为表格标题显示，呈现居中状态*/
body {margin: 8px; line-height: 1.12 }
h1 {font-size: 2em; margin: .67em 0 }
h2 {font-size: 1.5em; margin: .75em 0 }
h3 {font-size: 1.17em; margin: .83em 0 }
h4, p,blockquote, ul, fieldset, form, ol, dl, dir, menu { margin: 1.12em 0 }
h5 {font-size: .83em; margin: 1.5em 0 }
h6 {font-size: .75em; margin: 1.67em 0 }
h1, h2,h3, h4, h5, h6, b,strong { font-weight: bolder }
blockquote{ margin-left: 40px; margin-right: 40px }
i, cite,em,var, address { font-style: italic }
pre, tt,code, kbd, samp { font-family: monospace }
pre {white-space: pre }
button,textarea, input, object, select { display:inline-block; }
big {font-size: 1.17em }
small,sub, sup { font-size: .83em }
sub {vertical-align: sub }/*定义sub元素默认为下标显示*/
sup {vertical-align: super }/*定义sub元素默认为上标显示*/
table {border-spacing: 2px; }
thead,tbody, tfoot { vertical-align: middle }/*定义表头、主体表、表脚元素默认为垂直对齐*/
td, th {vertical-align: inherit }/*定义单元格、列标题默认为垂直对齐默认为继承*/
s, strike,del { text-decoration: line-through }/*定义这些元素默认为删除线显示*/
hr {border: 1px inset }/*定义分割线默认为1px宽的3D凹边效果*/
ol, ul,dir, menu, dd { margin-left: 40px }
ol {list-style-type: decimal }
ol ul, ulol, ul ul, ol ol { margin-top: 0; margin-bottom: 0 }
u, ins {text-decoration: underline }
br:before{ content: ""A" }/*定义换行元素的伪对象内容样式*/
```

```css
:before,:after { white-space: pre-line }/*定义伪对象空格字符的默认样式*/
center {text-align: center }
abbr,acronym { font-variant: small-caps; letter-spacing: 0.1em }
:link,:visited { text-decoration: underline }
:focus {outline: thin dotted invert }
/* Beginbidirectionality settings (do not change) */
BDO[DIR="ltr"]{ direction: ltr; unicode-bidi: bidi-override }/*定义BDO元素当其属性为DIR="ltr"时的默认文本读写显示顺序*/
BDO[DIR="rtl"]{ direction: rtl; unicode-bidi: bidi-override }/*定义BDO元素当其属性为DIR="rtl"时的默认文本读写显示顺序*/
*[DIR="ltr"]{ direction: ltr; unicode-bidi: embed }/*定义任何元素当其属性为DIR="ltr"时的默认文本读写显示顺序*/
*[DIR="rtl"]{ direction: rtl; unicode-bidi: embed }/*定义任何元素当其属性为DIR="rtl"时的默认文本读写显示顺序*/
@mediaprint { /*定义标题和列表默认的打印样式*/
h1 {page-break-before: always }
h1, h2,h3, h4, h5, h6 { page-break-after: avoid }
ul, ol, dl{ page-break-before: avoid }
}
```

## **浏览器默认样式**

###页边距 

* IE默认为10px，通过body的margin属性设置 
* FF默认为8px，通过body的padding属性设置 

要清除页边距一定要清除这两个属性值 

```css
body {
  margin:0;
  padding:0;
}
```

### 段间距 

* IE默认为19px，通过p的margin-top属性设置 
* FF默认为1.12em，通过p的margin-bottom属性设 

p默认为块状显示，要清除段间距，一般可以设置 

```css
p {
  margin-top:0;
  margin-bottom:0;
}
```

### 标题样式 

h1~h6默认加粗显示：font-weight:bold;。 
默认大小请参上表 

```css
h1{font-size:xx-large;}
h2{font-size:x-large;}
h3{font-size:large;}
h4{font-size:medium;}
h5{font-size:small;}
h6{font-size:x-small;}
```

各大浏览器默认字体大小为16px，即等于medium，h1~h6元素默认以块状显示字体显示为粗体， 
要清除标题样式，一般可以设置 

```css
hx {
  font-weight:normal;
  font-size:value;
}
```

### 列表样式 

* IE默认为40px，通过ul、ol的margin属性设置 
* FF默认为40px，通过ul、ol的padding属性设置 

dl无缩进，但起内部的说明元素dd默认缩进40px，而名称元素dt没有缩进。 
要清除列表样式，一般可以设置 

```css
ul, ol, dd{
  list-style-type:none;/*清楚列表样式符*/
  margin-left:0;/*清楚IE左缩进*/
  padding-left:0;/*清楚非IE左缩进*/
}
```

### 元素居中 

* IE默认为text-align:center; 
* FF默认为margin-left:auto;margin-right:auto; 

### 超链接样式 

a 样式默认带有下划线，显示颜色为蓝色，被访问过的超链接变紫色，要清除链接样式，一般可以设置 
代码如下:

```css
a {
  text-decoration:none;
  color:#colorname;
}
```

### 鼠标样式 

* IE默认为cursor:hand; 
* FF默认为cursor:pointer;。该声明在IE中也有效 

### 图片链接样式 

* IE默认为紫色2px的边框线 
* FF默认为蓝色2px的边框线 

要清除图片链接样式，一般可以设置

```css
img {
  border:0;
}
```

