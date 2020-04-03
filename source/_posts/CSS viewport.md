---
title: CSS viewport
date: 2020-03-05 15:20:55
tags:
  - CSS
  - viewport
categories: CSS
cover_img: https://tva1.sinaimg.cn/large/00831rSTgy1gdgevvdqcpj30u0190hdv.jpg
feature_img: https://tva1.sinaimg.cn/large/00831rSTgy1gdgevvdqcpj30u0190hdv.jpg
---

#viewport

## 页面适配的标签viewport这个标签是什么作用？meta几个属性都是什么含义？user-scanl=no不生效时，用js怎么控制达到禁止缩放的效果？

### viewport meta的背景

浏览器的 [viewport](https://developer.mozilla.org/en-US/docs/Glossary/viewport) 是可以看到Web内容的窗口区域，通常与渲染出的页面的大小不同，这种情况下，浏览器会提供滚动条以滚动访问所有内容。

窄屏幕设备（如移动设备）在一个虚拟窗口或视口中渲染页面，这个窗口或视口通常比屏幕宽；然后缩小渲染的结果，以便在一屏内显示所有内容。然后用户可以移动、缩放以查看页面的不同区域。例如，如果移动屏幕的宽度为640px，则可能会用980px的虚拟视口渲染页面，然后缩小页面以适应640px的窗口大小。

这样做是因为许多页面没有做移动端优化，在小窗口渲染时会乱掉（或看起来乱）。所以，这种虚拟视口是一种让未做移动端优化的网站在窄屏设备上看起来更好的办法。

但是，对于用媒体查询针对窄屏幕做了优化的页面，这种机制不大好 - 比如如果虚拟视口宽 980px，那么在 640px 或 480px 或更小宽度要起作用的媒体查询就不会触发了，浪费了这些响应式设计。

为了缓解这个问题，Apple 在 Safari iOS 中引入了“viewport meta 标签”，**让Web开发人员控制视口的大小和比例。**很多其他移动浏览器现在也支持此标签，但它不属于 Web 标准。

### viewport meta 标签的概念

移动设备上的viewport就是设备的屏幕上能用来显示我们的网页的那一块区域，在具体一点，就是浏览器上(也可能是一个app中的webview)用来显示网页的那部分区域，但viewport又不局限于浏览器可视区域的大小，它可能比浏览器的可视区域要大，也可能比浏览器的可视区域要小。在默认情况下，一般来讲，移动设备上的viewport都是要大于浏览器可视区域的，这是因为考虑到移动设备的分辨率相对于桌面电脑来说都比较小，所以为了能在移动设备上正常显示那些传统的为桌面浏览器设计的网站，移动设备上的浏览器都会把自己默认的viewport设为980px或1024px（也可能是其它值，这个是由设备自己决定的），但带来的后果就是浏览器会出现横向滚动条，因为浏览器可视区域的宽度是比这个默认的viewport的宽度要小的。下图列出了一些设备上浏览器的默认viewport的宽度。

### **css中的1px并不等于设备的1px**

 在css中我们一般使用px作为单位，在桌面浏览器中css的1个像素往往都是对应着电脑屏幕的1个物理像素，但在移动设备上，在早先的移动设备中，屏幕像素密度都比较低，如iphone3，它的分辨率为320x480，在iphone3上，一个css像素确实是等于一个屏幕物理像素的。后来随着技术的发展，移动设备的屏幕像素密度越来越高，从iphone4开始，苹果公司便推出了所谓的Retina屏，分辨率提高了一倍，变成640x960，但屏幕尺寸却没变化，这就意味着同样大小的屏幕上，像素却多了一倍，这时，一个css像素是等于两个物理像素的。其他品牌的移动设备也是这个道理。例如安卓设备根据屏幕像素密度可分为ldpi、mdpi、hdpi、xhdpi等不同的等级，分辨率也是五花八门，安卓设备上的一个css像素相当于多少个屏幕物理像素，也因设备的不同而不同，没有一个定论。

 还有一个因素也会引起css中px的变化，那就是用户缩放。例如，当用户把页面放大一倍，那么css中1px所代表的物理像素也会增加一倍；反之把页面缩小一倍，css中1px所代表的物理像素也会减少一倍。

window对象有一个devicePixelRatio属性，它的官方的定义为：设备物理像素和设备独立像素的比例，也就是 devicePixelRatio = 物理像素 / 独立像素。

例如，在Retina屏的iphone上，devicePixelRatio的值为2，也就是说1个css像素相当于2个物理像素。

### 三个viewport

这些浏览器就决定默认情况下把viewport设为一个较宽的值，比如980px，这样的话即使是那些为桌面设计的网站也能在移动浏览器上正常显示了。ppk把这个浏览器默认的viewport叫做 ***layout viewport**。****layout viewport*** 的宽度是大于浏览器可视区域的宽度的，所以我们还需要一个viewport来代表 浏览器可视区域的大小，ppk把这个viewport叫做 **visual viewport**。visual viewport的宽度可以通过window.innerWidth 来获取，但在Android 2, Oprea mini 和 UC 8中无法正确获取。

![img](https://tva1.sinaimg.cn/large/006y8mN6ly1g6j5puat7yj30dt0ai0t9.jpg)

![img](https://tva1.sinaimg.cn/large/006y8mN6ly1g6j5qah7plj30dw0ahjrw.jpg)

 完美适配指的是，首先不需要用户缩放和横向滚动条就能正常的查看网站的所有内容；第二，显示的文字的大小是合适；不只是文字，其他元素像图片什么的也是这个道理ppk把这个viewport叫做 ***ideal viewport***，也就是第三个viewport——移动设备的理想viewport。

 ideal viewport并没有一个固定的尺寸，不同的设备拥有有不同的ideal viewport。所有的iphone的ideal viewport宽度都是320px，无论它的屏幕宽度是320还是640，也就是说，在iphone中，css中的320px就代表iphone屏幕的宽度。

![img](https://tva1.sinaimg.cn/large/006y8mN6ly1g6j5rjwmr1j307i05wq2s.jpg)![img](https://tva1.sinaimg.cn/large/006y8mN6ly1g6j5s6pthkj307p05w3yc.jpg)

但是安卓设备就比较复杂了，有320px的，有360px的，有384px的等等，关于不同的设备ideal viewport的宽度都为多少，可以到[http://viewportsizes.com](http://viewportsizes.com/)去查看一下，里面收集了众多设备的理想宽度。

### **利用meta标签对viewport进行控制**

 移动设备默认的viewport是layout viewport，也就是那个比屏幕要宽的viewport，但在进行移动设备网站的开发时，我们需要的是ideal viewport。

我们在开发移动设备的网站时，最常见的的一个动作就是把下面这个东西复制到我们的head标签中：

```html
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=0">
```

该meta标签的作用是让当前viewport的宽度等于设备的宽度，同时不允许用户手动缩放。

meta viewport 标签首先是由苹果公司在其safari浏览器中引入的，目的就是解决移动设备的viewport问题。后来安卓以及各大浏览器厂商也都纷纷效仿，引入对meta viewport的支持，事实也证明这个东西还是非常有用的。

#### meta viewport 有6个属性(暂且把content中的那些东西称为一个个属性和值)，如下：

| 属性          | 作用                                                         |
| ------------- | ------------------------------------------------------------ |
| width         | 设置***layout viewport***  的宽度，为一个正整数，或字符串"device-width" |
| initial-scale | 设置页面的初始缩放值，为一个数字，可以带小数（缩放是相对于 ideal viewport来进行缩放的） |
| minimum-scale | 允许用户的最小缩放值，为一个数字，可以带小数                 |
| maximum-scale | 允许用户的最大缩放值，为一个数字，可以带小数                 |
| height        | 设置***layout viewport***  的高度，这个属性对我们并不重要，很少使用 |
| user-scalable | 是否允许用户进行缩放，值为"no"或"yes", no 代表不允许，yes代表允许 |

- `width`**设为`device-width` 特殊值，代表缩放为 100% 时以 CSS 像素计量的屏幕宽度**。（相应的也有`height`及`device-height`属性，可能对包含基于视口高度调整大小及位置的元素的页面有用。）

  注意的是，在iphone和ipad上，无论是竖屏还是横屏，宽度都是竖屏时ideal viewport的宽度。

- 对于设置了初始或最大缩放的页面，width属性实际上变成了**最小**视口宽度。比如，如果你的布局需要至少500像素的宽度，那么你可以使用以下标记。当屏幕宽度大于500像素时，浏览器会扩展视口（而不是放大页面）来适应屏幕：

- ```html
  <meta name="viewport" content="width=500, initial-scale=1">
  ```

  当遇到这种情况时，浏览器会取它们两个中较大的那个值。例如，当width=400，ideal viewport的宽度为320时，取的是400；当width=400， ideal viewport的宽度为480时，取的是ideal viewport的宽度。

- 在安卓中还支持  target-densitydpi  这个私有属性，它表示目标设备的密度等级，作用是决定css中的1px代表多少物理像素

  target-densitydpi：值可以为一个数值或 high-dpi 、 medium-dpi、 low-dpi、 device-dpi 这几个字符串中的一个

  特别说明的是，当 target-densitydpi=device-dpi 时， css中的1px会等于物理像素中的1px。

  因为这个属性只有安卓支持，并且安卓已经决定要废弃~~target-densitydpi~~  这个属性了，所以这个属性我们要避免进行使用  

### 关于缩放

缩放是相对于ideal viewport来缩放的，缩放值越大，当前viewport的宽度就会越小，反之亦然。例如在iphone中，ideal viewport的宽度是320px，如果我们设置 initial-scale=2 ，此时viewport的宽度会变为只有160px了，这也好理解，放大了一倍嘛，就是原来1px的东西变成2px了，但是1px变为2px并不是把原来的320px变为640px了，而是在实际宽度不变的情况下，1px变得跟原来的2px的长度一样了，所以放大2倍后原来需要320px才能填满的宽度现在只需要160px就做到了。因此，我们可以得出一个公式：

```json
visual viewport宽度 = ideal viewport宽度  / 当前缩放值

当前缩放值 = ideal viewport宽度  / visual viewport宽度
```

根据测试，我们可以在iphone和ipad上得到一个结论，就是无论你给layout viewpor设置的宽度是多少，而又没有指定初始的缩放值的话，那么iphone和ipad会自动计算initial-scale这个值，以保证当前layout viewport的宽度在缩放后就是浏览器可视区域的宽度，也就是说不会出现横向滚动条。

当前缩放值 = ideal viewport宽度  / visual viewport宽度，我们可以得出：

​      当前缩放值 = 320 / 980

也就是当前的initial-scale默认值应该是 0.33这样子。当你指定了initial-scale的值后，这个默认值就不起作用了。

**在iphone和ipad上，无论你给viewport设的宽的是多少，如果没有指定默认的缩放值，则iphone和ipad会自动计算这个缩放值，以达到当前页面不会出现横向滚动条(或者说viewport的宽度就是屏幕的宽度)的目的。**

### js事件监听阻止用户缩放

移动端web缩放有两种：

**1.双击缩放；**

**2.双指手势缩放。**

- 禁止双击缩放

```javascript
window.οnlοad=function () {         
	document.addEventListener('touchstart',function (event) {
    if(event.touches.length>1){
      event.preventDefault();
    }
  })  
  var lastTouchEnd=0;  
  document.addEventListener('touchend',function (event) {  
    var now=(new Date()).getTime();  
    if(now-lastTouchEnd<=300){  
      event.preventDefault();  
    }  
    lastTouchEnd=now;  
  },false)  
}
```

- 完美方案

```html
<script>
  window.onload=function () {
    document.addEventListener('touchstart',function (event) {
      if(event.touches.length>1){
        event.preventDefault();
      }
    });
    var lastTouchEnd=0;
    document.addEventListener('touchend',function (event) {
      var now=(new Date()).getTime();
      if(now-lastTouchEnd<=300){
        event.preventDefault();
      }
      lastTouchEnd=now;
    },false);
    //依然存在bug，双指同时放下放大禁止了，但一指先放下，另一指在放下滑动放大依然不管用
    document.addEventListener('gesturestart', function (event) {
      event.preventDefault();
    });
  }
</script>
```

