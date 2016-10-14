做移动开发的时候，有时会遇到使用很多图片的需求，然而过多图片会导致增加的http请求，带来更差的用户体验，为了解决这个问题，我们目前解决的办法是将图片合并成雪碧图，然后设置雪碧图为容器背景图。

备注：实现请参考[项目地址](https://github.com/Muxi-Studio/freshman-h5)。

## 制作雪碧图

首先需要使用雪碧图制作工具来生成雪碧图，gulp、grunt、webpack都有相关工具。关键点是要得到每个子图片在整张雪碧图中的高度、宽度、偏移量以及整张雪碧图的宽度和高度，然后再对这些值做计算转换使其可以适应页面。

下面我将以`webpack`+`webpack-spritesmith`为例来讲解：

[webpack-spritesmith](https://github.com/mixtur/webpack-spritesmith)是一个在[spritesmith](https://github.com/Ensighten/spritesmith)的基础上制作的一个工具，关于如何配合webpack使用请自行阅读相关文档，也可以参考[webpack自动雪碧图生成](http://kyon-df.com/2016/03/16/webpack_auto_sprites/)这篇文章的相关部分。

做好相关配置后，运行`webpack-spritesmith`会生成一份`_sprite.scss`文件(文件名与配置有关)。我们可以看到在这份文件中自动生成了合并之前的每个单份图片的尺寸、偏移量、名称、路径。同时还贴心的给我们写好了`mixin`。代码示例如下：

<script async src="//jsfiddle.net/shengxihu/odtw2fhe/1/embed/css/dark/"></script>

现在我们试着将雪碧图引入：

<iframe height='223' scrolling='no' src='//codepen.io/no1024/embed/mAKvrA/?height=223&theme-id=0&default-tab=css,result&embed-version=2' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>See the Pen <a href='http://codepen.io/no1024/pen/mAKvrA/'>mAKvrA</a> by shengxihu (<a href='http://codepen.io/no1024'>@no1024</a>) on <a href='http://codepen.io'>CodePen</a>.
</iframe>

不妨调整浏览器窗口大小，我们发现图片宽高并不能自适应屏幕的宽高.显然我们要将雪碧图的容器宽或高设置成百分比,然后让雪碧图的相关部分填满容器，为了实现这一点，我们需要给背景图片设置一个相对于容器的比例。那么这个比例是多少呢？我们不妨假设是100%；我们需要将

```
@mixin sprite-width($sprite) {
  width: nth($sprite, 5);
}

@mixin sprite-height($sprite) {
  height: nth($sprite, 6);
}
```

改为：

```
@mixin sprite-width($sprite) {
  width: 100%;
}

@mixin sprite-height($sprite) {
  width: 100%;
}
```

另外我们需要添加：

```
@mixin sprite-size($sprite) {
  background-size: 100%  100%;
}
```

此时，既然图片的大小是变化的，那么每一部分图片在整张雪碧图中的偏移量也应该是以一个比例来变化的，我们先讨论background-size该如何取值，把下面关于`background-position`部分先去掉：

```
@mixin sprite-position($sprite) {
  $sprite-offset-x: nth($sprite, 3);
  $sprite-offset-y: nth($sprite, 4);
  background-position: $sprite-offset-x  $sprite-offset-y;
}
```

结果为（具体改动情况参见代码相关部分）:

<iframe height='223' scrolling='no' src='//codepen.io/no1024/embed/mAKvrA/?height=223&theme-id=0&default-tab=css,result&embed-version=2' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>See the Pen <a href='http://codepen.io/no1024/pen/mAKvrA/'>mAKvrA</a> by shengxihu (<a href='http://codepen.io/no1024'>@no1024</a>) on <a href='http://codepen.io'>CodePen</a>.
</iframe>

我们看到通过为背景图片设置`backgroun-size:100% 100%;`后图片刚好填满了容器，这是因为backgroun-size的值为百分比时，它的实际值是相对于父元素的宽高来分别计算的.那么我们该如何来确定这个值应该设置为多少呢？不妨以我们合成的雪碧图中的第一个子图hsx\_1.jpg为例,从雪碧图的样式代码中`$hsx-2-width: 1440px;$hsx-2-height: 900px;` ，我们可以看出它的宽高分别为:1440px、900px。在实际开发中这个图我们是从设计稿上切下来的，那么这张图的父容器的宽高也应该分别是1440px、900px，在要求宽度自适应的情况下，我们会根据页面的总宽度计算出父容器宽度的百分比值。假设页面总高度就是900px，我们看宽度的计算方式。在前面我们已经知道了`backgroun-size:100% 100%;`图片刚好填满容器，我们要的效果是子图hsx\_1.jpg刚好填满容器,这说明整张雪碧图被缩小了，那么我们应该使用`backgroun-size`放大以实现还原，放大的倍数和缩小的倍数相等，要把宽度为x1的图片放入宽度为x2的容器中缩小的倍数应该是x1/x2.在这个例子中就是图片被缩小为原来的`容器的宽度/雪碧图的宽度`，所以我们应该讲雪碧图放大`雪碧图的宽度/容器的宽度`倍才能实现子图hsx\_1.jpg刚好填满容器.所以我们将:

```
@mixin sprite-size($sprite) {
  background-size: 100%  100%;
}

@mixin sprite($sprite) {
  @include sprite-size($sprite);
  @include sprite-image($sprite);
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

html,body {
  width:100%;
  height:900px;
  background:red;
}
.box {
  @include sprite($hsx-3);
}
```

改为：

```
@mixin sprite-size($sprite,$box_height) {
  $sprite-total-height: nth($sprite, 8);
  background-size: auto  ($sprite-total-height / $box_height) * 100%;
}

@mixin sprite($sprite,$box_height) {
  @include sprite-size($sprite,$box_height);
  @include sprite-image($sprite);
  @include sprite-width($sprite);
  @include sprite-height($sprite);
}

/*
The `sprites` mixin generates identical output to the CSS template
  but can be overridden inside of SCSS

@include sprites($spritesheet-sprites);
*/
@mixin sprites($sprites,$box_height) {
  @each $sprite in $sprites {
    $sprite-name: nth($sprite, 10);
    .#{$sprite-name} {
      @include sprite($sprite,$box_height);
    }
  }
}

html,body {
  width:100%;
  height:900px;
  background:red;
}
.box {
  @include sprite($hsx-3,900px);
}
```

运行结果为：
<iframe height='223' scrolling='no' src='//codepen.io/no1024/embed/mAKvrA/?height=223&theme-id=0&default-tab=css,result&embed-version=2' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>See the Pen <a href='http://codepen.io/no1024/pen/mAKvrA/'>mAKvrA</a> by shengxihu (<a href='http://codepen.io/no1024'>@no1024</a>) on <a href='http://codepen.io'>CodePen</a>.
</iframe>

不妨调节浏览器的高度，你会发现图片的宽度自适应了.

现在我们想引入hsx_2,很快遇到一个问题，针对不同的屏幕高度，在上面的讨论中我们将雪碧图放大到了不同的倍数，所以我们现在要重新的计算雪碧图的偏移量`background-position`值.既然雪碧图的大小是一个比例，那么background-position这个值也应该是一个比例，这个比例怎么计算呢？我们要先明确当其值为百分比时它的实际值的计算方式：

```
background-postion:x y;

x：{容器(container)的宽度—背景图片的宽度}*x百分比，超出的部分隐藏。
y：{容器(container)的高度—背景图片的高度}*y百分比，超出的部分隐藏。
```

也是就是说我们需要设置的这个百分比的计算方式应该是：

```
background-postion:x y;

x百分比：background-postion-x实际值/{容器(container)的宽度—背景图片的宽度}*100%。
y百分比：background-postion-y实际值/{容器(container)的高度—背景图片的高度}*100%。
```

所以求`background-position`的偏移量的代码块为：

```
@mixin sprite_position($sprite,$box_height) {
  $sprite-total-height: nth($sprite, 8);
  $sprite-total-width: nth($sprite, 7);
  $sprite-y: nth($sprite, 2) / ($sprite-total-height - $box_height) * 100%;
  background-position: 0  $sprite-y;
}
```

现在我们引入hsx_2，结果如下:

<iframe height='223' scrolling='no' src='//codepen.io/no1024/embed/mAKvrA/?height=223&theme-id=0&default-tab=css,result&embed-version=2' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>See the Pen <a href='http://codepen.io/no1024/pen/mAKvrA/'>mAKvrA</a> by shengxihu (<a href='http://codepen.io/no1024'>@no1024</a>) on <a href='http://codepen.io'>CodePen</a>.
</iframe>

自此我们实现了雪碧图高度的自适应，宽度的自适应是同样的思考和计算方式.



