做移动开发的时候，有时会遇到使用很多图片的需求，然而**过多图片会导致很多的http请求**，带来很差的用户体验，为了解决这个问题，我们目前解决的办法是将图片合并成雪碧图，然后设置雪碧图为容器背景图。

## 关于雪碧图

参考：[雪碧图](https://developer.mozilla.org/zh-CN/docs/Web/Guide/CSS/CSS_Image_Sprites)

CSS雪碧图即CSS Sprite，也有人叫它CSS精灵，是一种CSS图像合并技术，该方法是将小图标和背景图像合并到一张图片上，然后利用css的背景定位`backgroun-position`和`background-size`来控制显示需要显示的图片部分。为了**减少http请求数量，加速网页内容显示**，很多网站的导航栏图标、登录框图片等，使用的并不是`<img>`标签，而是CSS Sprite雪碧图。

![sprite合成图片-1](http://oev2d4dz7.bkt.clouddn.com/sprite_2.png)

## 制作雪碧图

首先需要使用雪碧图制作工具来生成雪碧图，gulp、grunt、webpack都有相关工具。关键是要得到每个子图片在整张雪碧图中的**高度、宽度、偏移量以及整张雪碧图的宽度和高度**，然后再对这些值做计算**转换**使其可以适应页面。

下面我将以`webpack`+`webpack-spritesmith`为例来讲解如何制作雪碧图：

[webpack-spritesmith](https://github.com/mixtur/webpack-spritesmith)是一个在[spritesmith](https://github.com/Ensighten/spritesmith)的基础上制作的一个工具，关于如何配合webpack使用可以阅读相关文档，也可以参考[webpack自动雪碧图生成](http://kyon-df.com/2016/03/16/webpack_auto_sprites/)。我在这里给出一种配置方式:

webpack.config.js：

```javascript
var SpritesmithPlugin = require('webpack-spritesmith')
new SpritesmithPlugin({
    /*合成之前对源图片文件的配置*/
    src: {
        /*子图片存放的路径（相对于更路径）*/
        cwd: './static/img',
        /*规定后缀名为png的图片合并成雪碧图*/
        glob: '*.png'
    },
    /*对合成之后产生文件的配置*/
    target: {
        /*雪碧图存放的路径（相对于更路径）*/
        image: './src/assets/sprite.png',
        /*合成生成的scss文件存放的路径（相对于更路径）*/
        css: './static/style/_sprite.scss'
    },
    apiOptions: {
        /*合成生成的scss中雪碧图的路径，将用于background-image的url*/
        cssImageRef: '../assets/sprite.png'
    },
    spritesmithOptions: {
        /*合成之后雪碧图中子图片的排列方式，top-down为从上往下排列*/
        algorithm: 'top-down'
    }
})
```

做好相关配置后，运行`webpack-spritesmith`会生成一份`_sprite.scss`文件(文件名与配置有关)。我们可以看到在这份文件中自动生成了合并之前的每个单份图片的**尺寸、偏移量、名称、路径**。同时还贴心的给我们写好了`mixin`,我们在使用的时候只需要引入这份scss文件，然后引入mixin就可以引入图片了。代码示例如下,我在代码注释中也对重要的部分做了说明：

```scss
/*
SCSS variables are information about icon's compiled state, stored under its original file name

.icon-home {
    width: $icon-home-width;
}

The large array-like variables contain all information about a single icon
$icon-home: x y offset_x offset_y width height total_width total_height image_path;

At the bottom of this section, we provide information about the spritesheet itself
$spritesheet: width height image $spritesheet-sprites;
*/

$img-1-name: 'img_1';    /*子图片名（去掉了后缀名），在引入子图时作为参数传入mixin*/
$img-1-x: 0px;    /*子图片img-1，在x轴上的坐标*/
$img-1-y: 0px;    /*子图片img-1，在y轴上的坐标*/
/*在引入子图片img-1，在x轴上的偏移量，将用于background-position*/
$img-1-offset-x: 0px;    
/*在引入子图片img-1，在y轴上的偏移量，将用于background-position*/
$img-1-offset-y: 0px;    
$img-1-width: 1444px;    /*子图片img-1的宽度*/
$img-1-height: 893px;    /*子图片img-1的高度*/
$img-1-total-width: 1920px;    /*整张雪碧图的宽度*/
$img-1-total-height: 3009px;    /*整张雪碧图的高度*/
/*雪碧图的路径，用于background-image的url*/
$img-1-image: '../assets/sprite.png';
/*定义一个数据地图（可以理解为数组）,方便mixin中对上面的各个值引用,要使用每个子图的偏移量、高度、宽度等数据时直接使用nth(img-1-name, n)*/ 
$img-1: (0px, 0px, 0px, 0px, 1444px, 893px, 1920px, 3009px, '../assets/sprite.png', 'img_1', );
$img-2-name: 'img_2';
$img-2-x: 0px;
$img-2-y: 893px;
$img-2-offset-x: 0px;
$img-2-offset-y: -893px;
$img-2-width: 1719px;
$img-2-height: 1037px;
$img-2-total-width: 1920px;
$img-2-total-height: 3009px;
$img-2-image: '../assets/sprite.png';
$img-2: (0px, 893px, 0px, -893px, 1719px, 1037px, 1920px, 3009px, '../assets/sprite.png', 'img_2', );
$img-3-name: 'img_3';
$img-3-x: 0px;
$img-3-y: 1930px;
$img-3-offset-x: 0px;
$img-3-offset-y: -1930px;
$img-3-width: 1920px;
$img-3-height: 1079px;
$img-3-total-width: 1920px;
$img-3-total-height: 3009px;
$img-3-image: '../assets/sprite.png';
$img-3: (0px, 1930px, 0px, -1930px, 1920px, 1079px, 1920px, 3009px, '../assets/sprite.png', 'img_3', );
$spritesheet-width: 1920px;
$spritesheet-height: 3009px;
$spritesheet-image: '../assets/sprite.png';
$spritesheet-sprites: ($img-1, $img-2, $img-3, );
$spritesheet: (1920px, 3009px, '../assets/sprite.png', $spritesheet-sprites, );
/*
The provided mixins are intended to be used with the array-like variables

.icon-home {
    @include sprite-width($icon-home);
}

.icon-email {
    @include sprite($icon-email);
}

Example usage in HTML:

`display: block` sprite:
<div class="icon-home"></div>

To change `display` (e.g. `display: inline-block;`), we suggest using a common CSS class:

// CSS
.icon {
    display: inline-block;
}

// HTML
<i class="icon icon-home"></i>
*/

/*
定义一个使用雪碧图的宽度值的width属性
 */
@mixin sprite-width($sprite) {
    width: nth($sprite, 5);
}

/*
定义一个使用雪碧图的高度值的height属性
 */
@mixin sprite-height($sprite) {
    height: nth($sprite, 6);
}

/*
使用子图偏移量来设置background-position的值
 */
@mixin sprite-position($sprite) {
    $sprite-offset-x: nth($sprite, 3);
    $sprite-offset-y: nth($sprite, 4);
    background-position: $sprite-offset-x  $sprite-offset-y;
}

/*
定义雪碧图的路径
 */
@mixin sprite-image($sprite) {
    $sprite-image: nth($sprite, 9);
    background-image: url(#{$sprite-image});
}

/*
把上面的代码块导入到名为sprite的mixin中，使用时导入这个代码块就同时配置了背景图片的路径、显示位置、宽度、高度
*/
@mixin sprite($sprite) {
    /*定义雪碧图的路径*/
    @include sprite-image($sprite);
    //使用background-position来让雪碧图只显示某个特定部分*/
    @include sprite-position($sprite);
    //定义使用雪碧图的容器的宽度*/
    @include sprite-width($sprite);
    //定义使用雪碧图的容器的高*/
    @include sprite-height($sprite);
}

/*
The `sprites` mixin generates identical output to the CSS template
but can be overridden inside of SCSS

@include sprites($spritesheet-sprites);
*/

/*
使用这个mixin传入多个参数可以循环引入多张子图
如：
$sprites:($img_1,$img_2,$img_3)
body {
    @include sprites($sprites)
}
效果等同与：
body {
    .img_1 {
        @include sprite($img_1)
    }
    .img_2 {
        @include sprite($img_2)
    }
    .img_2 {
        @include sprite($img_2)
    }
}
 */

@mixin sprites($sprites) {
    @each $sprite in $sprites {
        $sprite-name: nth($sprite, 10);
        .#{$sprite-name} {
            @include sprite($sprite);
        }
    }
}
```

为了在线演示，将背景图片的url改成了在线地址.

如果在某个地方需要使用雪碧图作为背景，我们只要引入这份SCSS文件，然后再使用它的地方导入mixin就可以了。例如有如下的HTML结构需要使用雪碧图作为背景：

```
/*这个div需要使用img_1.png作为背景图片.*/
<div class="icon_img_1"></div>
```

我们只需要在这样写SCSS代码：

```
/*假设雪碧图生成的SCSS文件的相对路径是_sprite.scss.
/*引入雪碧图生成的SCSS文件*/
@import url('./scss/_sprite.scss')

/*导入mixin*/
.icon_img_1 {
    /*
    其他定义类名为icon_img_1的div样式的SCSS代码
     */
    
    /*参数为以美元符号开头的去掉后缀的子图片文件名*/
    @include sprite($img_1)
}
```

## 使用background-position调整雪碧图

首先介绍一下使用background-position调整雪碧图位置的原理，非小白可以直接看下一部分。

我们要引入雪碧图张的一张子图作为容器的背景图，我们必须控制只有这个子图显示在容器里，CSS设置背景图的众多属性中有一个属性background-position是设置背景图在容器里面显示位置的,我们称这个值为偏移量，我们只要使用这个属性控制背景图在容器里面的显示位置，刚好使子图出现在容器中，而雪碧图的其它部分刚好不显示。

![sprite合成图片](http://oev2d4dz7.bkt.clouddn.com/sprite_3.png)

如果在引入雪碧图过程中不要求自适应的话，那么整张雪碧图和以雪碧图为背景的容器的大小都是固定的，此时background-position值也是固定的数值，`background-position: xpos ypos`是数值时，第一个值是水平位置，第二个值是垂直位置，参照点是父元素的左上角 0 0.

如果要求自适应,那么整张雪碧图和以雪碧图为背景的容器的大小都是不固定的.此时背景图片的大小要用`background-size`设置，`background-position`的值也应该是一个比例，这篇文章主要就是讨论这种情况下怎么设置这两个值，我们将在文中讨论.

## 无自适应效果的雪碧图

现在我们**不做任何修改**，将雪碧图引入：

引入子图img_1：[点击查看代码效果](http://codepen.io/no1024/pen/mAKvrA)

引入子图img_2：[点击查看代码效果](http://codepen.io/no1024/pen/jrpAaR/)

我们现在已经成功引入雪碧图作为背景图片了，这在图片大小是固定的情况下是没有任何问题的。但是如果要求背景图片大小可以随着浏览器窗口大小不同而能够自适应，我们调整浏览器窗口大小，我们发现图片宽高并不能自适应屏幕的宽高.

## 容器大小自适应

要实现随着浏览器宽度或者高度调整，我们要将雪碧图容器的宽度或者高度设置为百分比，然后再设置背景图片填满容器。这个百分比的计算方式为：

```
width:设计图中图片的的宽度/图片所在页面的宽度
height:设计图中图片的的高度/图片所在页面的高度
```

下面我们修改一下代码：

把：

```scss
@mixin sprite-width($sprite) {
    width: nth($sprite, 5);
}

@mixin sprite-height($sprite) {
    height: nth($sprite, 6);
}
```

改为：

```scss
/*把容器的宽高改成百分比，这个百分比应该是按照图片占设计图的宽高的百分比，假设改为90%*/
@mixin sprite-width($sprite) {
    width: 90%;
}

@mixin sprite-height($sprite) {
    height: 90%;
}
```

![sprite合成图片-3](http://oev2d4dz7.bkt.clouddn.com/sprite_10.jpg)

引入子图img_1：[点击查看代码效果](http://codepen.io/no1024/pen/BLOoqP)

![sprite合成图片-3](http://oev2d4dz7.bkt.clouddn.com/sprite_9.png)

引入子图img_2：[点击查看代码效果](http://codepen.io/no1024/pen/ORoyaw)

现在调整屏幕宽高，我们发现容器的大小自适应屏幕的宽高了，但是图片还没有自适应。

## 图片大小自适应

容器大小自适应了，但是为什么背景图片的大小不能自适应呢？因为我们给`background-size`设了定值，所以即使容器大小改变，背景图片的大小始终是一个定值。我们要让背景图片大小跟着容器大小一起调整，显然我们需要使用`background-size`给图片设置一个**相对于容器大小的百分比**。但是这个百分比是多少呢？我们要先明确`background-size`的计算方式，当`background-size`的值是**百分比**时，`background-size`的计算值（浏览器渲染是使用的值）是`百分比*容器宽度或高度的计算值`，也就是：

```css
background-size的设置值：
background-size: x%  y%;

background-size的计算值：
background-size: x%*容器的宽度  y%*容器的高度;
```

所以背景图片的展示的时候会被调整为以`width:x%*父容器的宽度`,`height:y%*父容器的高度`的图片;

下面我们看看这个百分比究竟应该设为多少。我们要引入一个子图到一个容器中，那么这个子图要填满容器，子图就应该被设置成容器大小，也就是子图的背景大小应该设置为`background-size：100% 100%；`，然而`background-size`会作用到整张雪碧图上，如下：

我们添加一个`background-size`的mixin,设置`background-size`值为100%:

```scss
@mixin sprite-size($sprite) {
    background-size: 100%  100%;
}
```

![sprite合成图片-3](http://oev2d4dz7.bkt.clouddn.com/sprite_6.png)

引入子图img_1：[点击查看效果](http://codepen.io/no1024/pen/mAGZgx)

我们看到`background-size`作用在了整张雪碧图上而不是作用在子图上，通过为背景图片设置`backgroun-size:100% 100%;`后**整张雪碧图**刚好填满了容器。我们要的只是**其中的某一个子图来填充容器**，那么我们要思考当这个值是多少的时候**整张雪碧图中的子图**刚好是容器的大小。我们知道，生成雪碧图后子图和整张雪碧图都是固定大小的，那么**子图的宽高和整张雪碧图的宽高比例是固定的**，所以我们应当设置整张雪碧图的宽高到`(整张雪碧图的宽高/子图的宽高)*100%`，此时子图的宽高刚好等于容器的宽高。即backgroun-size`的计算方式应该为：

```css
background-size: (雪碧图的宽度/子图的宽度)*100% (雪碧图的高度/子图的高度)*100%;
```

在实际应用中，一般会根据浏览器的宽度或者高度来做自适应，如果两个方向上的值都设置会造成图片的拉伸或者压缩，所以background-size只用设置x或者y方向上的值。在下面的例子中我们将讲解如何设置y方向上的值，让x方向上自适应，这样可以保证图片原始的宽高比例。所以我们将:

```scss
@mixin sprite-size($sprite) {
    background-size: 100%  100%;
}
```

改为：

```scss
@mixin sprite-size($sprite,$img_height) {
    $sprite-total-height: nth($sprite, 8);
    background-size: auto  ($sprite-total-height / $img_height) * 100%;
}
```

对应的我们也要修改引入mixin的方式：

```scss
@mixin sprite($sprite,$img_height) {
    /*增加参数$img_height*/
    @include sprite-size($sprite,$img_height);
    @include sprite-image($sprite);
    @include sprite-width($sprite);
    @include sprite-height($sprite);
}
.box {
    /*增加参数$img_height*/
    @include sprite($img-1,893px);
}
```
![sprite合成图片-3](http://oev2d4dz7.bkt.clouddn.com/sprite_8.png)

引入子图img_1：[点击查看代码效果](http://codepen.io/no1024/pen/xEakxq)

![sprite合成图片-4](http://oev2d4dz7.bkt.clouddn.com/sprite_4.jpg)

引入子图img_2：[点击查看代码效果](http://codepen.io/no1024/pen/bwxEGk)

不妨调节浏览器的高度，你会发现子图img\_1的高度完美自适应了，但是子图img\_2并不能正常引入.

## 图片偏移量自适应（background-position）

在引入img_2,我们看到效果很糟糕，这是因为针对不同的屏幕高度，在上面的讨论中我们的得到到的代码会将**雪碧图放大到了不同的倍数**，然而我们代码中的`background-position`依然是一个**定值**,所以我们现在要重新的计算雪碧图的偏移量`background-position`值.既然雪碧图的大小是一个比例，那么**background-position这个值也应该是一个比例**，这个比例怎么计算呢？我们要先明确当其值为百分比时它的实际值的计算方式：

```css
background-postion:x y;

x：{容器(container)的宽度—背景图片的宽度}*x百分比，超出的部分隐藏。
y：{容器(container)的高度—背景图片的高度}*y百分比，超出的部分隐藏。
```

也是就是说我们需要设置的这个百分比的计算方式应该是：

```css
background-postion:x y;

x百分比：background-postion-x实际值/{容器(container)的宽度—背景图片的宽度}*100%。
y百分比：background-postion-y实际值/{容器(container)的高度—背景图片的高度}*100%。
```

所以求`background-position`的偏移量的代码块：

由：

```scss
@mixin sprite-position($sprite) {
    $sprite-offset-x: nth($sprite, 3);
    $sprite-offset-y: nth($sprite, 4);
    background-position: $sprite-offset-x  $sprite-offset-y;
}
```

改为：

```scss
@mixin sprite-position($sprite,$img_height) {
    $sprite-total-height: nth($sprite, 8);
    $sprite-total-width: nth($sprite, 7);
    $sprite-y: nth($sprite, 2) / ($sprite-total-height - $img_height) * 100%;
    background-position: 0  $sprite-y;
}
```

现在我们再来看一下效果:

![sprite合成图片-5](http://oev2d4dz7.bkt.clouddn.com/sprite_8.png)

引入子图img_1：[点击查看代码效果](http://codepen.io/no1024/pen/YGOwXZ)

![sprite合成图片-6](http://oev2d4dz7.bkt.clouddn.com/sprite_7.jpg)

引入子图img_2：[点击查看代码效果](http://codepen.io/no1024/pen/KgxVpP)

自此我们实现了雪碧图按照浏览器高度来实现自适应，宽度的自适应是同样的思考和计算方式.

我写了一个[gist代码片段](https://gist.github.com/shengxihu/d00e38a0d17c71a11fdead6da45b93e4)，使用的时候将webpack-spritesmith输出的SCSS文件中对应的部分替换就可以实现.

备注：实现请参考[项目地址](https://github.com/Muxi-Studio/freshman-h5)。



