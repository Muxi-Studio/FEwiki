做移动开发的时候，有时会遇到使用很多图片的需求，然而**过多图片会导致很多的http请求**，带来更差的用户体验，为了解决这个问题，我们目前解决的办法是将图片合并成雪碧图，然后设置雪碧图为容器背景图。

## 关于雪碧图

参考：[雪碧图](https://developer.mozilla.org/zh-CN/docs/Web/Guide/CSS/CSS_Image_Sprites)

CSS雪碧图即CSS Sprite，也有人叫它CSS精灵，是一种CSS图像合并技术，该方法是将小图标和背景图像合并到一张图片上，然后利用css的背景定位`backgroun-size`和`background-position`来控制显示需要显示的图片部分。为了**减少http请求数量，加速网页内容显示**，很多网站的导航栏图标、登录框图片等，使用的并不是<image>标签，而是CSS Sprite雪碧图。

![sprite合成图片](http://oev2d4dz7.bkt.clouddn.com/sprite_2.png)

## 制作雪碧图

首先需要使用雪碧图制作工具来生成雪碧图，gulp、grunt、webpack都有相关工具。关键是要得到每个子图片在整张雪碧图中的**高度、宽度、偏移量以及整张雪碧图的宽度和高度**，然后再对这些值做计算**转换**使其可以适应页面。

下面我将以`webpack`+`webpack-spritesmith`为例来讲解如何制作雪碧图：

[webpack-spritesmith](https://github.com/mixtur/webpack-spritesmith)是一个在[spritesmith](https://github.com/Ensighten/spritesmith)的基础上制作的一个工具，关于如何配合webpack使用可以阅读相关文档，也可以参考[webpack自动雪碧图生成](http://kyon-df.com/2016/03/16/webpack_auto_sprites/)。我在这里给出一种配置方式:

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

做好相关配置后，运行`webpack-spritesmith`会生成一份`_sprite.scss`文件(文件名与配置有关)。我们可以看到在这份文件中自动生成了合并之前的每个单份图片的**尺寸、偏移量、名称、路径**。同时还贴心的给我们写好了`mixin`,我们在使用的时候只需要引入这份scss文件，然后引入mixin就可以引入图片了，这份代码中也**讲解了如何引入雪碧图**，读者可以查看注释部分。代码示例如下：

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

$hsx-1-name: 'hsx_1';    /*子图片名（去掉了后缀名），在引入子图时作为参数传入mixin*/
$hsx-1-x: 0px;    /*子图片hsx-1，在x轴上的坐标*/
$hsx-1-y: 0px;    /*子图片hsx-1，在y轴上的坐标*/
/*在引入子图片hsx-1，在x轴上的偏移量，将用于background-position*/
$hsx-1-offset-x: 0px;    
/*在引入子图片hsx-1，在y轴上的偏移量，将用于background-position*/
$hsx-1-offset-y: 0px;    
$hsx-1-width: 1444px;    /*子图片hsx-1的宽度*/
$hsx-1-height: 893px;    /*子图片hsx-1的高度*/
$hsx-1-total-width: 1920px;    /*整张雪碧图的宽度*/
$hsx-1-total-height: 3009px;    /*整张雪碧图的高度*/
/*雪碧图的路径，用于background-image的url*/
$hsx-1-image: '../assets/sprite.png';
/*定义一个数据地图（可以理解为数组），方便mixin中对上面的各个值引用*/ 
$hsx-1: (0px, 0px, 0px, 0px, 1444px, 893px, 1920px, 3009px, '../assets/sprite.png', 'hsx_1', );
$hsx-2-name: 'hsx_2';
$hsx-2-x: 0px;
$hsx-2-y: 893px;
$hsx-2-offset-x: 0px;
$hsx-2-offset-y: -893px;
$hsx-2-width: 1719px;
$hsx-2-height: 1037px;
$hsx-2-total-width: 1920px;
$hsx-2-total-height: 3009px;
$hsx-2-image: '../assets/sprite.png';
$hsx-2: (0px, 893px, 0px, -893px, 1719px, 1037px, 1920px, 3009px, '../assets/sprite.png', 'hsx_2', );
$hsx-3-name: 'hsx_3';
$hsx-3-x: 0px;
$hsx-3-y: 1930px;
$hsx-3-offset-x: 0px;
$hsx-3-offset-y: -1930px;
$hsx-3-width: 1920px;
$hsx-3-height: 1079px;
$hsx-3-total-width: 1920px;
$hsx-3-total-height: 3009px;
$hsx-3-image: '../assets/sprite.png';
$hsx-3: (0px, 1930px, 0px, -1930px, 1920px, 1079px, 1920px, 3009px, '../assets/sprite.png', 'hsx_3', );
$spritesheet-width: 1920px;
$spritesheet-height: 3009px;
$spritesheet-image: '../assets/sprite.png';
$spritesheet-sprites: ($hsx-1, $hsx-2, $hsx-3, );
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
@mixin sprite-width($sprite) {
    width: nth($sprite, 5);
}

@mixin sprite-height($sprite) {
    height: nth($sprite, 6);
}

@mixin sprite-position($sprite) {
    $sprite-offset-x: nth($sprite, 3);
    $sprite-offset-y: nth($sprite, 4);
    background-position: $sprite-offset-x  $sprite-offset-y;
}

@mixin sprite-image($sprite) {
    $sprite-image: nth($sprite, 9);
    background-image: url(#{$sprite-image});
}

@mixin sprite($sprite) {
    @include sprite-image($sprite);
    @include sprite-position($sprite);
    @include sprite-width($sprite);
    @include sprite-height($sprite);
}

/*
The `sprites` mixin generates identical output to the CSS template
but can be overridden inside of SCSS

@include sprites($spritesheet-sprites);
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

## 无自适应效果的雪碧图

现在我们**不做任何修改**，将雪碧图引入：

引入子图hsx_1：[点击查看代码效果](http://codepen.io/no1024/pen/mAKvrA)

引入子图hsx_2：[点击查看代码效果](http://codepen.io/no1024/pen/jrpAaR/)

调整浏览器窗口大小，我们发现图片宽高并不能自适应屏幕的宽高.显然我们要**将雪碧图的容器宽或高设置成百分比**,然后让雪碧图的相关部分填满容器。下面我们修改一下代码：

## 容器大小自适应

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
/*把容器的宽高改成百分比，假设改为90%*/
@mixin sprite-width($sprite) {
    width: 90%;
}

@mixin sprite-height($sprite) {
    height: 90%;
}
```

引入子图hsx_1：[点击查看代码效果](http://codepen.io/no1024/pen/BLOoqP)

引入子图hsx_2：[点击查看代码效果](http://codepen.io/no1024/pen/ORoyaw)

现在调整屏幕宽高，我们发现容器的大小自适应屏幕的宽高了，但是图片还没有自适应。

## 图片大小自适应

我们需要使用`background-size`给图片设置一个**相对于容器大小的百分比**来实现图片的自适应。但是这个百分比是多少呢？我们要先明确`background-size`的计算方式，当`background-size`的值是**百分比**时，`background-size`的计算值（浏览器渲染是使用的值）是`百分比*容器的计算值`，也就是：

```css
background-size的设置值：
background-size: x%  y%;

background-size的计算值：
background-size: x%*父容器的宽度  y%*父容器的高度;
```

所以背景图片的展示的时候会被调整为以`width:x%*父容器的宽度`,`height:y%*父容器的高度`的图片;

下面我们添加一个`background-size`的mixin,不妨先设置`background-size`值为100%，我们看会发生什么:

```scss
@mixin sprite-size($sprite) {
    background-size: 100%  100%;
}
```

引入子图hsx_1：[点击查看效果](http://codepen.io/no1024/pen/mAGZgx)

我们看到通过为背景图片设置`backgroun-size:100% 100%;`后**整张雪碧图**刚好填满了容器，调整浏览器窗口图片也**能自适应**了，这是符合上面提到的计算方式的。但是我们要的只是**其中的某一个子图来填充容器**，例如子图hsx\_1,下面我们讨论`backgroun-size`的y值,x值是同样的计算方式。

在前面我们已经知道了`backgroun-size:100% 100%;`整张雪碧图刚好填满容器，我们要的效果是子图hsx\_1刚好填满容器,这说明整张雪碧图被**缩小**了，那么我们应该使用`backgroun-size`**放大以实现还原**，放大的倍数和缩小的倍数相等，要把高度为y1的图片放入高度为y2的容器中缩小的倍数应该是`y1/y2`.在这个例子中就是图片被缩小为原来的`容器的高度/雪碧图的高度`，所以我们应该讲雪碧图放大`雪碧图的高度/容器的高度`倍才能实现子图hsx\_1刚好填满容器.`backgroun-size`的计算方式应该为：

```css
background-size: (雪碧图的宽度/容器的宽度)*100% (雪碧图的高度/容器的高度)*100%;
```

所以我们将:

```scss
@mixin sprite-size($sprite) {
    background-size: 100%  100%;
}
```

改为：

```scss
@mixin sprite-size($sprite,$box_height) {
    $sprite-total-height: nth($sprite, 8);
    background-size: auto  ($sprite-total-height / $box_height) * 100%;
}
```

对应的我们也要修改引入mixin的方式：

```scss
@mixin sprite($sprite,$box_height) {
    //增加参数$box_height
    @include sprite-size($sprite,$box_height);
    @include sprite-image($sprite);
    @include sprite-width($sprite);
    @include sprite-height($sprite);
}
.box {
    //增加参数$box_height
    @include sprite($hsx-1,893px);
}
```

引入子图hsx_1：[点击查看代码效果](http://codepen.io/no1024/pen/xEakxq)

引入子图hsx_2：[点击查看代码效果](http://codepen.io/no1024/pen/bwxEGk)

不妨调节浏览器的高度，你会发现子图hsx_1的高度完美自适应了.

## 图片偏移量自适应（background-position）

在引入hsx_2,我们看到效果很糟糕，这是因为针对不同的屏幕高度，在上面的讨论中我们的得到到的代码会将**雪碧图放大到了不同的倍数**，然而我们代码中的`background-position`依然是一个**定值**,所以我们现在要重新的计算雪碧图的偏移量`background-position`值.既然雪碧图的大小是一个比例，那么**background-position这个值也应该是一个比例**，这个比例怎么计算呢？我们要先明确当其值为百分比时它的实际值的计算方式：

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
@mixin sprite-position($sprite,$box_height) {
    $sprite-total-height: nth($sprite, 8);
    $sprite-total-width: nth($sprite, 7);
    $sprite-y: nth($sprite, 2) / ($sprite-total-height - $box_height) * 100%;
    background-position: 0  $sprite-y;
}
```

现在我们再来看一下效果:

引入子图hsx_1：[点击查看代码效果](http://codepen.io/no1024/pen/YGOwXZ)

引入子图hsx_2：[点击查看代码效果](http://codepen.io/no1024/pen/KgxVpP)

自此我们实现了雪碧图高度的自适应，宽度的自适应是同样的思考和计算方式.

我写了一个[gist代码片段](https://gist.github.com/shengxihu/d00e38a0d17c71a11fdead6da45b93e4)，使用的时候将webpack-spritesmith输出的SCSS文件中对应的部分替换就可以实现.

备注：实现请参考[项目地址](https://github.com/Muxi-Studio/freshman-h5)。



