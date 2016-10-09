做移动开发的时候，有时会遇到使用很多图片的需求，然而过多图片会导致增加的http请求，带来更差的用户体验，为了解决这个问题，我们目前解决的办法是将图片合并成雪碧图，然后设置雪碧图为容器背景图。

备注：实现请参考[项目地址](https://github.com/Muxi-Studio/freshman-h5)。

## 制作雪碧图

首先需要使用雪碧图制作工具来生成雪碧图，gulp、grunt、webpack都有相关工具。关键点是要得到每个子图片在整张雪碧图中的高度、宽度、偏移量以及整张雪碧图的宽度和高度，然后再对这些值做计算转换使其可以适应页面。

下面我将以`webpack`+`webpack-spritesmith`为例来讲解：

[webpack-spritesmith](https://github.com/mixtur/webpack-spritesmith)是一个在[spritesmith](https://github.com/Ensighten/spritesmith)的基础上制作的一个工具，关于如何配合webpack使用请自行阅读相关文档，也可以参考[webpack自动雪碧图生成](http://kyon-df.com/2016/03/16/webpack_auto_sprites/)这篇文章的相关部分。

做好相关配置后，运行webpack会生成一份`_sprite.scss`文件(文件名与配置有关)。我们可以看到在这份文件中自动生成了合并之前的每个单份图片的尺寸、偏移量、名称、路径。同时还贴心的给我们写好了`mixin`。代码示例如下：

```
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
$loading-name: 'loading';
$loading-x: 0px;
$loading-y: 0px;
$loading-offset-x: 0px;
$loading-offset-y: 0px;
$loading-width: 750px;
$loading-height: 31px;
$loading-total-width: 750px;
$loading-total-height: 19939px;
$loading-image: '../sprite/sprite.png';

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

我们可以看到图片的相关尺寸都是固定值，显然它只适用于图片尺寸在页面中是固定大小的情况。如果要实现图片大小在页面中可以自适应，我们就要对这份SCSS文件做一些更改。



