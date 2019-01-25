# 用compass生成雪碧图

## 目录

* [前言](#前言)
* [初始化compass](#初始化compass)
* [移动端的icon适配](#移动端的icon适配)

## 前言

本文着重介绍compass怎么样生成雪碧图，至于安装compass环境本文不做介绍，网上有很详细的介绍，移步[阮一峰](http://www.ruanyifeng.com/blog/2012/11/compass.html)的教程。

## 初始化compass

在使用compass之前，我们需要创建一个compass的目录。在一个目录下，打开终端，运行下面命令。

```
compass create myProject
```
先在img目录下新建icons文件夹，里面存放各种icon文件，然后在`sass`目录下创建`icons.scss`，工程示例如下：

```
--sass
  |--icons.scss
--css
--img
  |--icons
     |--car.png
     |--share.png
--config.rb
```

然后我们在icon.scss编写以下代码：

```scss
$icons-sprite-dimensions: true;//输出icon的高宽度
$icons-layout:horizontal; //雪碧图的排列的方向，vertical(默认纵向)/horizontal(横线)/diagonal(对角线)/smart(智能，紧密的);
$icons-spacing:5px;//配置所有的icon的间距为5px
$icons-car-spacing:20px;//配置单个的icon间距为20px
$disable-magic-sprite-selectors:false;////默认情况下compass是开启这个功能的，也就是说compass默认会将以_hover, _active等名字结尾的图片自动输出对应的:hover, :active等伪类样式。如果不需要这个功能，设置为false即可
@import "compass/utilities/sprites"; //引入sprites模块
@import "icons/*.png";//图片源
//@include all-icons-sprites;//输出所有的icon的css，建议用此配置
.car{
  @include icons-sprite(car);  
}
.share{
  @include icons-sprite(share);
}
```

常用的配置就这些了，如果需要了解其他配置，请查阅[Compass的文档](http://compass-style.org/help/tutorials/spriting/)。

## 移动端的icon适配

在移动端开发中，用compass制作出来的雪碧图，在普通屏显示是没有问题的，但是iphone4开始，屏幕开始采用retina显示屏，对图片的质量比较高。一般情况下，我们都是采用图片折半的方案，图片才能高清的显示出来。那怎么办呢？？  
这时候，我们就需要用css3的新特性background-size了，background-size有两个属性，我们只能这两个属性分别设置原icon高宽度的一半就可以了。而background-size成一半，那么background-position也必须调整。  
这里我写了一个sass的工具函数：

```scss
$icons-sprite-dimensions: true;
@import "compass/css3";
@import "compass/utilities/sprites";
@import "icons/*.png";
@mixin resize-sprite($map, $sprite, $percent) {
  $spritePath:    sprite-path($map);
  $spriteWidth:   image-width($spritePath);
  $spriteHeight:  image-height($spritePath);
  $width: image-width(sprite-file($map, $sprite));
  $height: image-height(sprite-file($map, $sprite));
 
  @include background-size(ceil($spriteWidth * ($percent/100)) ceil($spriteHeight * ($percent/100)));
  width: ceil($width*($percent/100));
  height: ceil($height*($percent/100));
  background-position: 0 floor(nth(sprite-position($map, $sprite), 2)  * ($percent/100) );
}

.icon-car {
    $spriteName: car;
    $percent:50;
    @include resize-sprite($icons-sprites, $spriteName,$percent);
}
.icon-share{
    $spriteName: share;
    $percent:50;
    @include resize-sprite($icons-sprites, $spriteName,$percent);
}
```

下载示例代码：[compass-sprite](https://github.com/linjinying/compass-sprite)