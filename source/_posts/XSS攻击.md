---
title: XSS攻击
date: 2019-06-13 18:29:55
tags:
  - javascript
categories: javascript
cover_img: https://tva1.sinaimg.cn/large/006y8mN6ly1g77hppoib4j30u01907td.jpg
feature_img: https://tva1.sinaimg.cn/large/006y8mN6ly1g77hppoib4j30u01907td.jpg
---

# XSS 攻击

## 1.什么是 XSS

Cross-Site Scripting（跨站脚本攻击）简称 XSS，是一种代码注入攻击。攻击者通过在目标网站上注入恶意脚本，使之在用户的浏览器上运行。利用这些恶意脚本，攻击者可获取用户的敏感信息如 Cookie、SessionID 等，进而危害数据安全。

为了和 CSS 区分，这里把攻击的第一个字母改成了 X，于是叫做 XSS。

XSS 的本质是：恶意代码未经过滤，与网站正常的代码混在一起；浏览器无法分辨哪些脚本是可信的，导致恶意脚本被执行。

  <!-- more -->

而由于直接在用户的终端执行，恶意代码能够直接获取用户的信息，或者利用这些信息冒充用户向网站发起攻击者定义的请求。

在处理输入时，以下内容都不可信：

- 来自用户的 UGC 信息
- 来自第三方的链接
- URL 参数
- POST 参数
- Referer （可能来自不可信的来源）
- Cookie （可能来自其他子域注入）

## 2.XSS 分类

根据攻击的来源，XSS 攻击可分为存储型、反射型和 DOM 型三种。

![](https://tva1.sinaimg.cn/large/006y8mN6ly1g6j5yj76xtj31by0ecgm2.jpg)

#### 存储型 XSS

存储型 XSS 的攻击步骤：

1. 攻击者将恶意代码提交到目标网站的数据库中。
2. 用户打开目标网站时，网站服务端将恶意代码从数据库取出，拼接在 HTML 中返回给浏览器。
3. 用户浏览器接收到响应后解析执行，混在其中的恶意代码也被执行。
4. 恶意代码窃取用户数据并发送到攻击者的网站，或者冒充用户的行为，调用目标网站接口执行攻击者指定的操作。

这种攻击常见于带有用户保存数据的网站功能，如论坛发帖、商品评论、用户私信等。

#### 反射型 XSS

反射型 XSS 的攻击步骤：

1. 攻击者构造出特殊的 URL，其中包含恶意代码。
2. 用户打开带有恶意代码的 URL 时，网站服务端将恶意代码从 URL 中取出，拼接在 HTML 中返回给浏览器。
3. 用户浏览器接收到响应后解析执行，混在其中的恶意代码也被执行。
4. 恶意代码窃取用户数据并发送到攻击者的网站，或者冒充用户的行为，调用目标网站接口执行攻击者指定的操作。

反射型 XSS 跟存储型 XSS 的区别是：存储型 XSS 的恶意代码存在数据库里，反射型 XSS 的恶意代码存在 URL 里。

反射型 XSS 漏洞常见于通过 URL 传递参数的功能，如网站搜索、跳转等。

由于需要用户主动打开恶意的 URL 才能生效，攻击者往往会结合多种手段诱导用户点击。

POST 的内容也可以触发反射型 XSS，只不过其触发条件比较苛刻（需要构造表单提交页面，并引导用户点击），所以非常少见。

#### DOM 型 XSS

DOM 型 XSS 的攻击步骤：

1. 攻击者构造出特殊的 URL，其中包含恶意代码。
2. 用户打开带有恶意代码的 URL。
3. 用户浏览器接收到响应后解析执行，前端 JavaScript 取出 URL 中的恶意代码并执行。
4. 恶意代码窃取用户数据并发送到攻击者的网站，或者冒充用户的行为，调用目标网站接口执行攻击者指定的操作。

DOM 型 XSS 跟前两种 XSS 的区别：DOM 型 XSS 攻击中，取出和执行恶意代码由浏览器端完成，属于前端 JavaScript 自身的安全漏洞，而其他两种 XSS 都属于服务端的安全漏洞。

## 3.XSS 的预防

XSS 攻击有两大要素：

1. 攻击者提交恶意代码。
2. 浏览器执行恶意代码。

- ### 输入过滤

  输入侧过滤能够在某些情况下解决特定的 XSS 问题，但会引入很大的不确定性和乱码问题。

  对于明确的输入类型，例如数字、URL、电话号码、邮件地址等等内容，进行输入过滤还是必要的。

  输入过滤并非完全可靠，我们就要通过“防止浏览器执行恶意代码”来防范 XSS。这部分分为两类：

  - 防止 HTML 中出现注入。
  - 防止 JavaScript 执行时，执行恶意代码。

- ### 预防存储型和反射型 XSS 攻击

  预防这两种漏洞，有两种常见做法：

  - 改成纯前端渲染，把代码和数据分隔开。

    纯前端渲染的过程：

    1. 浏览器先加载一个静态 HTML，此 HTML 中不包含任何跟业务相关的数据。
    2. 然后浏览器执行 HTML 中的 JavaScript。
    3. JavaScript 通过 Ajax 加载业务数据，调用 DOM API 更新到页面上。

    在纯前端渲染中，我们会明确的告诉浏览器：下面要设置的内容是文本（`.innerText`），还是属性（`.setAttribute`），还是样式（`.style`）等等。浏览器不会被轻易的被欺骗，执行预期外的代码了。

    但纯前端渲染还需注意避免 DOM 型 XSS 漏洞

  - 对 HTML 做充分转义。

    **转义应该在输出 HTML 时进行，而不是在提交用户输入时。**

    常用的模板引擎，如 doT.js、ejs、FreeMarker 等，对于 HTML 转义通常只有一个规则，就是把 `& < > " ' /` 这几个字符转义掉，确实能起到一定的 XSS 防护作用，但并不完善：

  ![](https://tva1.sinaimg.cn/large/006y8mN6ly1g6j5yiazcnj31by0giaac.jpg)

- ### 预防 DOM 型 XSS 攻击

  1. 在使用 `.innerHTML`、`.outerHTML`、`document.write()` 时要特别小心，不要把不可信的数据作为 HTML 插到页面上，而应尽量使用 `.textContent`、`.setAttribute()` 等。
  2. 如果用 Vue/React 技术栈，并且不使用 `v-html`/`dangerouslySetInnerHTML` 功能，就在前端 render 阶段避免 `innerHTML`、`outerHTML` 的 XSS 隐患。
  3. DOM 中的内联事件监听器，如 `location`、`onclick`、`onerror`、`onload`、`onmouseover` 等，`<a>` 标签的 `href` 属性，JavaScript 的 `eval()`、`setTimeout()`、`setInterval()` 等，都能把字符串作为代码运行。如果不可信的数据拼接到字符串中传递给这些 API，很容易产生安全隐患，请务必避免。

## 4.其他 XSS 防范措施

### Content Security Policy

严格的 CSP 在 XSS 的防范中可以起到以下的作用：

- 禁止加载外域代码，防止复杂的攻击逻辑。
- 禁止外域提交，网站被攻击后，用户的数据不会泄露到外域。
- 禁止内联脚本执行（规则较严格，目前发现 GitHub 使用）。
- 禁止未授权的脚本执行（新特性，Google Map 移动版在使用）。
- 合理使用上报可以及时发现 XSS，利于尽快修复问题。

### 输入内容长度控制

对于不受信任的输入，都应该限定一个合理的长度。虽然无法完全防止 XSS 发生，但可以增加 XSS 攻击的难度。

### 其他安全措施

- HTTP-only Cookie: 禁止 JavaScript 读取某些敏感 Cookie，攻击者完成 XSS 注入后也无法窃取此 Cookie。
- 验证码：防止脚本冒充用户提交危险操作。

## 5.XSS 漏洞总结

### 漏洞总结

- 在 HTML 中内嵌的文本中，恶意内容以 script 标签形成注入。
- 在内联的 JavaScript 中，拼接的数据突破了原本的限制（字符串，变量，方法名等）。
- 在标签属性中，恶意内容包含引号，从而突破属性值的限制，注入其他属性或者标签。
- 在标签的 href、src 等属性中，包含 `javascript:` 等可执行代码。
- 在 onload、onerror、onclick 等事件中，注入不受控制代码。
- 在 style 属性和标签中，包含类似 `background-image:url("javascript:...");` 的代码（新版本浏览器已经可以防范）。
- 在 style 属性和标签中，包含类似 `expression(...)` 的 CSS 表达式代码（新版本浏览器已经可以防范）。

总之，如果开发者没有将用户输入的文本进行合适的过滤，就贸然插入到 HTML 中，这很容易造成注入漏洞。攻击者可以利用漏洞，构造出恶意的代码指令，进而利用恶意代码危害数据安全。

- 内联 JSON 也是不安全的：
  - 当 JSON 中包含 `U+2028` 或 `U+2029` 这两个字符时，不能作为 JavaScript 的字面量使用，否则会抛出语法错误。
  - 当 JSON 中包含字符串 `</script>` 时，当前的 script 标签将会被闭合，后面的字符串内容浏览器会按照 HTML 进行解析；通过增加下一个 `<script>` 标签等方法就可以完成注入。

## 6.减少漏洞产生的原则

- **利用模板引擎**
  开启模板引擎自带的 HTML 转义功能。例如：
  在 ejs 中，尽量使用 `<%= data %>` 而不是 `<%- data %>`；
  ![](https://tva1.sinaimg.cn/large/006y8mN6ly1g6j6vu6iq2j30ns01gjr9.jpg)
  在 FreeMarker 中，确保引擎版本高于 2.3.24，并且选择正确的 `freemarker.core.OutputFormat`。
- **避免内联事件**
  尽量不要使用 `onLoad="onload('{{data}}')"`、`onClick="go('{{action}}')"` 这种拼接内联事件的写法。在 JavaScript 中通过 `.addEventlistener()` 事件绑定会更安全。
- **避免拼接 HTML**
  前端采用拼接 HTML 的方法比较危险，如果框架允许，使用 `createElement`、`setAttribute` 之类的方法实现。或者采用比较成熟的渲染框架，如 Vue/React 等。
- **时刻保持警惕**
  在插入位置为 DOM 属性、链接等位置时，要打起精神，严加防范。
- **增加攻击难度，降低攻击后果**
  通过 CSP、输入长度配置、接口安全措施等方法，增加攻击的难度，降低攻击的后果。
- **主动检测和发现**
  可使用 XSS 攻击字符串和自动扫描工具寻找潜在的 XSS 漏洞。

## 7.vue react 框架对 xss 的防护有做什么？

- VUE

  - 尽量使用插值表达式(双花括号)，它会把要显示的内容转为字符串。
  - - 如果使用`v-html`，要保证来自服务端的渲染数据都是安全的。
  - 在使用第三方 UI 组件库的的时候，要检查一下它们渲染页面的方式，是否使用了`v-html`

- REACT

  XSS 防御措施就是对任何用户输入的信息进行处理，只允许合法值，其它值一概过滤掉

  1. 所有的用户输入都需要经过 HTML 实体编码，React 已经做了，它会在运行时动态创建 DOM 节点然后填入文本内容 (也可以强制设置 HTML 内容，不过这样比较危险)
  2. 序列化某些状态并且传给客户端的时候，也进行 HTML 实体编码，可以使用 Yahoo 的 Serialize JavaScript 中的 serialize 方法替换 JSON.stringify 方法，Serialize JavaScript 中的方法会自动将 HTML 和 JavaScript 代码进行转码，GitHub 访问地址 : https://github.com/yahoo/serialize-javascript

  默认情况下，React DOM 在重新渲染页面时将所有进行转码，官方宣称在 React 应用中确保不会注入任何没显式编写的数据，所有的数据在页面渲染之前都会被转换成字符串，这防止 XSS 进攻

## 8.xss 攻击后果？

恶意代码未经过滤，与网站正常的代码混在一起；浏览器无法分辨哪些脚本是可信的，导致恶意脚本被执行。

而由于直接在用户的终端执行，恶意代码能够直接获取用户的信息，或者利用这些信息冒充用户向网站发起攻击者定义的请求。

## 9.需要转义的字符？

- `escapeHTML()` 按照如下规则进行转义：

![](https://tva1.sinaimg.cn/large/006y8mN6ly1g6j5yh9feqj30ne0gqglm.jpg)

- 连 `javascript:` 这样的字符串如果出现在特定的位置也会引发 XSS 攻击。

  `%20javascript:alert('XSS')` 经过 URL 解析后变成 `javascript:alert('XSS')`，这个字符串以空格开头。这样攻击者可以绕过后端的关键词规则，又成功的完成了注入。

- 插入 JSON 的地方不能使用 `escapeHTML()`，因为转义 `"` 后，JSON 格式会被破坏。

  要实现一个 `escapeEmbedJSON()` 函数，对内联 JSON 进行转义。

![](https://tva1.sinaimg.cn/large/006y8mN6ly1g6j5ygpakdj31bk0csq2y.jpg)
