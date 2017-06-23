### 一些基本概念
**viewport 视窗**

在桌面浏览器中，viewport就是浏览器窗口的宽度高度。但在移动端设备上，假如没有viewport，移动端会采用桌面版的屏幕宽度，然后适应屏幕进行缩放。设置viewport可以控制页面的宽度并且在不同的设备上进行缩放。
（A viewport controls how a webpage is displayed on a mobile device. Without a viewport, mobile devices will render the page at a typical desktop screen width, scaled to fit the screen. Setting a viewport gives control over the page's width and scaling on different devices.）

移动端提供了两个viewport，虚拟的viewport：visual viewport和布局的viewport：layout viewport。[Stack Overflow上有深入的讲解。](http://stackoverflow.com/questions/6333927/difference-between-visual-viewport-and-layout-viewport)

**物理像素 & 设备独立像素 & CSS像素**

**物理像素**又被称为设备像素，他是显示设备中一个最微小的物理部件。每个像素可以根据操作系统设置自己的颜色和亮度。

**设备独立像素**也称为密度无关像素，可以认为是计算机坐标系统中的一个点，这个点代表一个可以由程序使用的虚拟像素(比如说CSS像素)，然后由相关系统转换为物理像素。

**CSS设置的像素值(px)** 属于普通像素点，或者是标准像素点。CSS像素是一个抽像的单位，主要使用在浏览器上，用来精确度量Web页面上的内容。一般情况之下，CSS像素称为与设备无关的像素(device-independent pixel)，简称DIPs。

**devicePixelRatio 设备像素比**

设备像素比简称为dpr，其定义了物理像素和设备独立像素的对应关系。它的值可以按下面的公式计算得到：
> 设备像素比 ＝ 物理像素 / 设备独立像素

**高清屏和普通屏幕**

高清屏和普通屏来做对比就是普通屏幕的1个像素点就是1个物理像素点，而高清屏的1个像素点是4个物理像素点。
通过计算 devicePixelRatio 的值，可以区分普通显示屏和高清显示器，当devicePixelRatio值等于1时（也就是最小值），那么它是普通显示屏，当devicePixelRatio值大于1(通常是1.5、2.0)，那么它就是高清显示屏。
比如iPhone6的devicePixelRatio为2，所以是高清显示屏。iPhone6s plus这种高清屏dpr是3。

**REM**

REM就是相对于根元素<html>的font-size来做计算。因为网页<html>的默认字体大小是 16px，所以
 > 1rem=16px ，10rem=160px


-----
### Retina屏幕1px产生的问题
因为viewport的设置和屏幕物理分辨率是按比例而不是相同的，<meta>标签里实际上是设置了ideal viewport的宽度，而不同手机的ideal viewport宽度是不一样的。移动端window对象有devicePixelRatio属性，CSS里写1px的边框在devicePixelRatio = 2的Retina屏下会显示成2px。

![1px的效果](http://upload-images.jianshu.io/upload_images/4938344-40592fda54568184.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![1px被显示成2px](http://upload-images.jianshu.io/upload_images/4938344-72d1355fcd358aa6.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

图片来自[Retina屏的移动设备如何实现真正1px的线？](http://jinlong.github.io/2015/05/24/css-retina-hairlines/)

-----

### 如何实现Retina屏幕上1px的效果
##### 方法一 Viewport + REM

通过viewport + REM的方式来兼容,（[淘宝移动端的方法](https://github.com/amfe/lib-flexible)）使用JS动态获取屏幕的设备像素比devicePixelRatio，然后动态设置viewport。

devicePixelRatio=2的时候，控制viewport的initial-scale值为0.5进行缩放
`<meta name="viewport" content="initial-scale=0.5, user-scalable=no"/>`

devicePixelRatio=3的时候，
`<meta name="viewport" content="initial-scale=0.333333, user-scalable=no"/>`

Android下不支持initial-scale，虽然这不适用于安卓, 但它里面的这一段代码可以用来做对安卓机的部署.
```
    if (!dpr && !scale) {
    var isAndroid = win.navigator.appVersion.match(/android/gi);
    var isIPhone = win.navigator.appVersion.match(/iphone/gi);
    var devicePixelRatio = win.devicePixelRatio;
    if (isIPhone) {
        // iOS下，对于2和3的屏，用2倍的方案，其余的用1倍方案
        if (devicePixelRatio >= 3 && (!dpr || dpr >= 3)) {                
            dpr = 3;
        } else if (devicePixelRatio >= 2 && (!dpr || dpr >= 2)){
            dpr = 2;
        } else {
            dpr = 1;
        }
    } else {
        // 其他设备下（比如安卓），仍旧使用1倍的方案
        dpr = 1;
    }
    scale = 1 / dpr;
}
```


对于安卓机做检测，动态加载CSS

```
var link = document.createElement('link');
link.setAttribute("rel","stylesheet");
link.setAttribute("type","text/css");
link.setAttribute("href",".......Android.css");
document.querySelector('head').appendChild(link);
```

这个方案结合了viewport和rem，所以使用的时候要考虑到REM布局。另外，REM布局下字体的单位仍建议使用px，还有出现1px像素线的地方，也仍旧使用border-width:1px;而不是border-width:.1rem;

##### 方法二 Media Query
根据屏幕的设备像素比来加载不同图片可以使用CSS的Media Query来解决，它对应devicePixelRatio有几个查询值-webkit-device-pixel-ratio，-webkit-min-device-pixel-ratio和 -webkit-max-device-pixel-ratio，但是这样我们就要切图片的一倍图和二倍图。

```
.img{ /* 普通显示屏(设备像素比例小于等于1.3)使用一倍图 */
    background-image: url(img_1x.png);
}
@media screen and (-webkit-min-device-pixel-ratio:1.5){
.img{/* 高清显示屏(设备像素比例大于等于1.5)使用二倍图  */
    background-image: url(img_2x.png);
  }
}
```
据说IOS8以上CSS可以用小数点，理论上可以这样写。

```
.border {
 border: 1px solid #ffffff;
}
@media screen and (-webkit-min-device-pixel-ratio: 2) {
    .border { border: 0.5px solid #ffffff; }
}
@media screen and (-webkit-min-device-pixel-ratio: 3) {
    .border { border: 0.333333px solid #ffffff; }
}
```

但是由于安卓与低版本IOS不适用，所以不推荐这种写法。

##### 方法三 box-shadow
利用CSS对阴影处理的方式实现
`-webkit-box-shadow:0 1px 1px -1px rgba(0, 0, 0, 0.5);`

优点是基本所有场景都能满足，包含圆角的button，单条，多条线。

缺点是颜色不好处理， 黑色 rgba(0,0,0,1) 是最浓的情况，而且有阴影出现。

##### 方法四 background-image背景渐变实现
通过CSS修改image，设置图片50%有颜色，50%透明，实现1px的效果。

 `linear-gradient`属性一个表示颜色线性渐变的 image，语法是`linear-gradient([ [ [ <angle>| to[top | bottom] || [left | right] ],]? <color-stop>[, <color-stop>]+);`详细讲解看[MDN文档](https://developer.mozilla.org/en-US/docs/Web/CSS/linear-gradient)
 
```
.border {
      background-image: linear-gradient(180deg, red, red 50%, transparent 50%),
      linear-gradient(270deg, red, red 50%, transparent 50%),
      linear-gradient(0deg, red, red 50%, transparent 50%),
      linear-gradient(90deg, red, red 50%, transparent 50%);
      background-size: 100% 1px, 1px 100% ,100% 1px, 1px 100%;
      background-repeat: no-repeat;
      background-position: top, right top,  bottom, left top;
      padding: 10px;
  }
```
这个方法代码量大，而且无法实现边框的圆角效果。

##### 方法五 border-image
![6X6的图片](http://upload-images.jianshu.io/upload_images/4938344-0320e1b1f4cde23b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
图片可以是gif、png、base 64

```
.border{
    border-width: 1px;
    border-image: url(border.png) 2 repeat;
}
```
缺点
- 想要实现圆角效果的话要放大修改图片
- 边框颜色不方便修改
- 边框存在多种颜色的时候麻烦

##### 方法六 transform: scale(0.5)+伪元素:before, :after
原理去掉把原先元素的 border ，然后利用 :before 或者 :after 重做 border ，并 transform 的 scale 属性将元素大小缩小为设置的值的一半，将原先的元素相对定位，新写的 border 设置绝对定位。

使用样式的时候，要结合 JS 代码，判断是否 Retina 屏
```
if(window.devicePixelRatio && devicePixelRatio >= 2){
    document.querySelector('.box').className += 'box1';
}
```
看一下demo代码：
```
.box {
    border: none;
    position: relative;
    z-index: 3;
}
.box1 :before {
    content: '';
    position: absolute;
    top: 0;
    left: 0;
    -webkit-box-sizing: border-box;
    box-sizing: border-box;
    width: 200%;
    height: 200%;
    -webkit-transform: scale(0.5);
    transform: scale(0.5);
    color: #0b2029;
    border-radius: 8px;
    border: 2px solid #737373;
    font-size: 0;
    -webkit-transform-origin: 0 0;
    transform-origin: 0 0;
    z-index: -1;
}
```

:before和:after 是用来给指定的元素的内容前面或后面插入新的内容。给`:before`添加了属性 `content`并设置为空（对于伪元素 :before 和 :after 而言，属性 content 是必须设置的否则伪元素不会生效），然后给它设置我需要的样式的两倍大小，再设置` transform: scale(0.5);`就实现了1px的效果。像这里的demo代码，我的box里面有input，所以要设置`index`，否则input就会被覆盖。

注意`<input type="button">`没有:before, :after伪元素。

ps：起初，伪元素的前缀使用的是单冒号语法，但为了和伪类(pseudo-classes)”区分开，在CSS3的规范里，伪元素的语法被修改成使用双冒号，成为::before & ::after ，但是因为IE8只支持单冒号的语法，所以如果你想兼容IE8，保险的做法是使用单冒号。

对于大型的项目，推荐使用[手淘的flexible方案](http://www.w3cplus.com/mobile/lib-flexible-for-html5-layout.html)，小型的页面用 transform: scale(0.5)+伪元素:before, :after较为方便。
