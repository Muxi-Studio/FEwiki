说到移动端，不得不提适配问题，大大小小的移动设备不但让做Android和ios的难过，因为设备大小和浏览器的差异，现在也让前端开始头疼了，不过，遇到问题我们总归要去寻求解决的方案。

因为现如今市面上移动设备的分辨率大小不同，显然咱们常用的px单位在这个时候就有些不太灵光了，为此，我们需要找到一个合理的适配方案。

## 布局方案

### 百分比布局

我们能够想到的最简单的方案就是百分比布局了。各级元素的宽度都设成百分比以实现在水平方向上适应各种屏幕宽度的显示效果，我们先来看一个demo。

<a href="http://codepen.io/no1024/pen/mOPNvd" alt="demo-1"><img src="http://res.cloudinary.com/da4uixfcu/image/upload/v1479347646/p4r6suuzcykfev1aiapx.gif"/></a>

我们看到通过宽度设置百分比我们很容易的让页面中的元素实现了宽度自适应，那么高度怎么办呢？通常在开发过程中我们需要实际设置高度的情况比较少，限定高度值也不是一种合理的实现。在实际开发过程中，我们一般让高度由文字或者图片（img标签的高度会随宽度的值自动调整）撑开，当然也有例外的情况，比如我们使用背景图片代替img标签时，就需要我们手动设置高度的值了，如果设置不合理，图片显示的宽高比就会发生改变，改变浏览器的大小，我们会看到demo中图片显示的效果：

<a href="http://codepen.io/no1024/pen/ObNKbK" alt="demo-2"><img src="http://res.cloudinary.com/da4uixfcu/image/upload/v1479345803/ajzdppvfeh9rzlokyhl1.gif"/></a>

这个时候我们要解决的问题可以抽象为如何保证*在页面缩放过程中元素的宽高比不变*，为此，我们想*让元素的宽高都基于一个基准值计算*，这样实现可以保证元素的宽高比例始终是一个固定值。刚好我们有这么一个属性可以使用：padding，我想一定不止我一个人被这个属性的值为百分比时的表现坑过，不同于我们常见的宽度和高度的计算方式，*padding值为百分比时其计算值始终是基于父元素宽度的*，比如有下面的CSS样式：
```
.class {
	width: 200px;
	height: 100px;
	padding: 5%;
}
```

浏览器渲染时采用的样式是这样的：

```
.class {
	width: 200px;
	height: 100px;
	padding-top: 10px;
	padding-right: 10px;
	padding-bottom: 10px;
	padding-left: 10px;
}
```

下面我们要利用它来实现一种高度自适应的效果，我们如果将父元素的宽度设为百分比，然后再给元素设置height设为0，width和padding-top为百分比，元素的高度就会被padding撑起来，并且高度的计算值是以宽度为基准的，这完美的满足了我们的需求。

<a href="http://codepen.io/no1024/pen/LbNKWW" alt="demo-3"><img src="http://res.cloudinary.com/da4uixfcu/image/upload/v1479345933/j0nsodmh4rxwpx2vuiwq.gif"/></a>

然而对于不使用背景图又要求宽高比固定不变的元素，这似乎并没有什么鸟🐦用，我们都知道元素的padding里面是不能放置元素的（正常[文档流](https://www.w3.org/TR/CSS2/visuren.html#normal-flow)的情况下），这时我们想到一个属性：position:absolute,我们可以让子元素绝对定位，这样它就可以出现在父元素的padding位置了。我们来看demo：

<a href="http://codepen.io/no1024/pen/bBeaGP" alt="demo-4"><img src="http://res.cloudinary.com/da4uixfcu/image/upload/v1479345975/xufjbjweo5iuci6ikrid.gif"/></a>

绝对定位让元素脱离文档流，我们可以把它定位在父元素的padding里面，然后让它的宽度和高度都基于父元素的宽度调整，完美的实现了自适应。

下面我们来看一下有哪些网站是使用了百分比布局：


[今日头条](http://m.toutiao.com/?W2atIF=1)

对于单列元素宽度百分之百，高度由内容撑开。对于多列元素，元素宽度设百分比，元素大小随父元素宽度自动调整。

[京东](http://m.jd.com/)

直接将元素设为百分比，高度由内容撑开，图片只设置宽度，高度自动调整。

[天猫超市](https://chaoshi.m.tmall.com/)

天猫使用了百分比布局结合下文想要谈到的flex布局的方式，对于图片等一些需要保持宽高比例不变的元素，它采用padding-top加绝对定位的实现。

### media queries

在HTML4和CSS2中充许你使用“media”来指定特定的媒体类型，如屏幕（screen）和打印（print）等设备的样式表。CSS3中的Media Queries增加了更多的媒体查询，同时你可以对于特定的设备类型、屏幕尺寸检查媒体是否符合某些条件，如果媒体符合相应的条件，那么就会调用对应的样式表。这个属性成为一堆成为CSS3最被人看好的一个属性之一——你可以让一个网站同时适配移动端和pc端。

我们将要讨论的是media query在移动布局中的应用，移动布局如果使用media query来为不同屏幕尺寸下的元素设置不同的大小以让其自适应是不现实的，国内目前安卓的主流机型有480x800, 480x854, 540x960, 720x1280, 800x1280 这五种，非主流机型还包括：240x320, 320x480, 640x960 这三种。iOS主流机型主要为 320x480, 640x960, 640x1136, 1024x768, 2048x1536五种，这么多的屏幕使用media query来适配那开发的工作量是很大的。media query在移动布局中一般用来设置字体和其他需要定制的元素尺寸。

例如：

```
.className {
	width: 200px;
}

@media screen and (max-width: 1024px){
   .className {
     	width: 400px;
   }
}
@media only screen and (max-width: 320px) { body{ font-size: 12px; }}
@media only screen and (max-width: 480px) { body{ font-size: 14px; }}
@media only screen and (max-width: 640px) { body{ font-size: 16px; }}
@media only screen and (max-width: 1024px) { body{ font-size: 18px; }}
```
在移动布局中常用媒体查询来设置几个断点以调整字体的大小。

### flexbox布局

CSS3 弹性盒子(Flexible Box 或 Flexbox)提供了一种全新的布局方式，弹性盒子中的子元素可以在各个方向上进行布局，并且能以弹性尺寸来适应显示空间。在定义方面来说，弹性布局是指通过调整其内元素的宽高，从而在任何显示设备上实现对可用显示空间最佳填充的能力。弹性容器扩展其内元素来填充可用空间，或将其收缩来避免溢出。更多关于flexbox的知识可以查看[使用 CSS 弹性盒子](https://developer.mozilla.org/zh-CN/docs/Web/CSS/CSS_Flexible_Box_Layout/Using_CSS_flexible_boxes)和[一个完整的Flexbox指南](http://www.w3cplus.com/css3/a-guide-to-flexbox-new.html).

下面我们来看一个demo：

<a href="http://codepen.io/no1024/pen/eBzVwo" alt="demo-5"><img src="http://res.cloudinary.com/da4uixfcu/image/upload/v1479346020/dwtlu9u1nvqw4npy5fsy.gif"/></a>

调整浏览器窗口我们看到元素的宽度可以自适应，我们可以给子元素的设置不同的伸缩比例就可以控制元素水平方向所占空间的大小。但是使用这种方案元素的高度我们必须回到使用百分比设置或者由内容撑开的思路上去，遇到前面谈到的使用背景图片代替img标签造成无法使用高度撑开父元素时，这种方案不能解决这个问题。当然在不使用背景图片代替img标签或者元素高度可以由内容撑开的时候，flexbox布局还是很有用处的。

应用到了flex布局的网站有：

[天猫](https://www.tmall.com/)

我们看到天猫的首页使用flex使元素在水平方向上均匀排列，实现水平方向上的自适应布局，高度方面直接由子元素撑开。

### rem布局

提到单位大家首先会想到的是em，px，pt这类的词语，rem也是这样一个长度单位。对于rem来说它可以成为做移动端的响应式适配的利器。

我们来看下兼容性：

![rem兼容性](http://res.cloudinary.com/da4uixfcu/image/upload/v1479347727/ssvs1orku8p5lmgokued.png)


我们看到所有主流浏览器都兼容了，可以放心大胆的使用。

rem进行屏幕适配

在讲rem屏幕适配之前，先来看一下什么是rem：

  这个单位代表相对于根元素的 font-size 大小（例如 font-size <html> 元素）。当用在 font-size 在根元素，它代表了它的初始值。

上面的解释来着mdn，也就是说rem是相对于根元素html计算的，如果根元素是12px，那么1rem的计算值就是12px,我们把整个页面中所有元素的长度单位都设置为rem，不同屏幕尺寸下调整根元素字体的大小句可以实现元素自适应了。当让对于字体和边框这样的属性，建议不使用rem，我们不希望在大屏幕中字体过大，当边框这样较小的值，如果再计算为rem，浏览器会去四舍五入的计算值，最终会对显示效果会有影响。

rem基准值计算

由于我们所写出的页面是要在不同的屏幕大小设备上运行的，所以我们在写样式的时候必须要先以一个确定的屏幕来作为参考，这个就由我们拿到的视觉稿来定假如我们拿到的视觉稿是以iphone6的屏幕为基准设计的iPhone6的屏幕大小是375px。

```
rem = (375 / 10)px
```

我们一般讲设备的宽度除以10，为什么是10，我们想尽量取一个不大又不小的整数，当然你也可以去其他值或者选择不除，只要保证页面的元素尺寸相对于这个值转换就OK。
根元素字体大小设置为37.5以后，可以保证浏览器的显示效果和设计图相同，当我们换一种屏幕尺寸，我们只需要改变html的font-size值，页面中元素的尺寸就会随html的font-size值调整。

动态设置html的font-size

现在关键问题来了，我们该如何通过不同的屏幕去动态设置html的font-size呢，最简单的方式是利用javascript来动态设置。我们可以利用js动态算出当前屏幕所适配的font-size即：

```
document.getElementsByTagName('html')[0].style.fontSize = window.innerWidth / 10 + 'px';
```

我们可以通过设置不同的html基础值来达到在不同页面适配的目的，当然在使用js来设置时，需要绑定页面的resize事件来达到变化时更新html的font-size。

<a href="http://codepen.io/no1024/pen/NbrYrB" alt="demo-6"><img src="http://res.cloudinary.com/da4uixfcu/image/upload/v1479346167/sdwt1ujvqz7mryo6g6np.gif"/></a>

使用rem布局的网站：

[淘宝](https://m.taobao.com/#index)

淘宝首页中除字体以外的其他长度单位都使用了rem.

[手机网易网](http://3g.163.com/touch/all?nav=2&version=v_standard)

手机网易网使用了百分比加rem的布局方式，页面中的字体大小也使用rem来实现。

### rem适配进阶

高清显示屏下图片变模糊

乔帮主发布的“iPhone4”和"new iPad"以及“Macbook Pro”中使用的Retina(视网膜)技术让我们走进了视网膜的Web时代。所谓视网膜的高清屏，就是改变原来一个物理像素显示一个逻辑像素的方式，一个物理像素现在可以显示多个逻辑像素，市面上的高清屏有1.5倍屏、2倍屏、3倍屏之分，一个物理像素分别对应显示9/4、4、9个逻辑像素，在这种高清屏上，普通显示屏上显示的比较清晰的图片会变得模糊。原因：浏览器在渲染是会将图片尺寸渲染为逻辑像素，然后在放大到物理像素的大小。假设我没有一个100px\*100px的图片，在普通屏幕上浏览器会正常渲染为100px\*100px,然而在高清屏中会渲染为100px\*100px的逻辑像素，这个时候在iPhone的2陪屏中对应的物理像素是50px\*50px,然后浏览器会再将图片放大到100px*100px，矢量图在放大的过程中就会变得模糊。更多高清屏原理可以查看[走向视网膜（Retina）的Web时代](http://www.w3cplus.com/css/towards-retina-web.html)

我们首先要知道设备像素比(device pixel ratio)dpr这个概念，dpr的计算方式为：屏幕的逻辑像素/物理像素。一般我们获取到的视觉稿大部分是iphone6的，所以我们看到的尺寸一般是双倍大小的。设计给的稿子双倍的原因是iphone6这种屏幕属于高清屏，也即是设备像素比(device pixel ratio)dpr比较大，所以显示的像素较为清晰。一般手机的dpr是1，iphone4，iphone5这种高清屏是2，iphone6s plus这种高清屏是3，可以通过js的window.devicePixelRatio获取到当前设备的dpr，拿到了dpr之后，我们就可以在viewport meta头里，取消让浏览器自动缩放页面，而自己去设置viewport的content例如（这里之所以要设置viewport是因为我们要实现border1px的效果，我给border设置了1px，在scale的影响下，高清屏中就会显示成0.5px的效果）。

```
meta.setAttribute('content', 'initial-scale=' + 1/dpr + ', maximum-scale=' + 1/dpr + ', minimum-scale=' + 1/dpr + ', user-scalable=no');
```

这样实现的原理是：当使用高清屏幕时，例如在2倍屏幕上，为了保证显示效果，我们可以在给元素设置尺寸是将其放大为两倍，使用2陪尺寸大小的图片，整个页面的显示效果就是设计图的两倍大小，然后再设置viewport使页面缩小为原来的1/2,这样页面就显示为高清效果了。

在iphone6下的例子：

我们使用动态设置viewport，在iphone6下，scale会被设置成1/2即0.5.对于高清屏，我们需要将尺寸

放大为dpr的倍数。

例如下面的这段代码：

```
.selector {
    height: 32px;
}
```

在一倍屏下这是没有问题的，但是如果是两倍屏幕，就应该设为：

```
.selector {
    height: 64px;
}
```

相应3倍屏幕就应该为：

```
selector {
    height: 96px;
}
```

这种适配是一件比较麻烦的事情，淘宝前端为我们提供了一个解决方案。[使用Flexible实现手淘H5页面的终端适配](https://github.com/amfe/article/issues/17)，它使用rem和viewport完美的解决了移动端元素大小自适应和高清屏幕的适配问题。

## 总结：

前面谈到的适配方案没有优劣之分，具体如何选用还是要看业务的需求。比如我们有一个像腾讯新闻这样的资讯展示型网站，使用百分比布局就可以实现。对于天猫、京东、淘宝这类展示型多图片网站，涉及到图片的宽高比例问题，使用百分比结合padding+绝对定位、或者百分比结合flex也是不错的选择，当然我觉得采用淘宝这样使用rem布局的方式会更加简单，如果要适配高清屏那么建议你使用它的适配工具[lib-flexible](https://github.com/amfe/lib-flexible)可以简单快速的满足业务需求。我们在实际开发过程中也会面临各种各样的业务场景，具体选用还要根据是否多图片、是否适配高清屏等业务要求来搭配布局方式作出最好的选择。





