# background-clip的应用

## 目录

* [前言](#前言)
* [怎样实现](#怎样实现)
* [注意](#注意)

## 前言

关于background-clip这个css3的属性，其实早在2011年，国内最早接触和研究CSS3的大漠前辈已经介绍得很详细了（[传送门](http://www.w3cplus.com/content/css3-background-clip/)）这里就不多做介绍了，不过当时只讲解了`border-box`,`padding-box`,`content-box`。今天我要讲的是background-clip的另外一个属性`text`，这个属性非常有意思，不过能把它运用到什么样的程度，完全取决个人的创意和能力。比如下面这个效果（请用高级浏览器打开）。

点击进入 <a href="/demo/background-clip/index.html" target="_blank">屌炸天效果</a>
效果是不是很赞呢？那到底是怎么实现的呢？

## 怎样实现

直接上代码
```html
<h1>前端小栈</h1>
```
```css
body {
  background-color: #000;
}

h1 {
  text-align: center;
  font-size: 240px;
  background: url("../images/repeat-white.png") center center repeat;
  background-size: 80px 60px;
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  -webkit-animation: ani-h1 0.253s linear forwards infinite;
  animation: ani-h1 0.253s linear forwards infinite;
}

@-webkit-keyframes ani-h1 {
  0% {
    background-position: 0 0;
  }
  25% {
    background-position: 0 0;
  }
  26% {
    background-position: 20px -20px;
  }
  50% {
    background-position: 20px -20px;
  }
  51% {
    background-position: 40px -40px;
  }
  75% {
    background-position: 40px -40px;
  }
  76% {
    background-position: 60px -60px;
  }
  99% {
    background-position: 60px -60px;
  }
  100% {
    background-position: 0 0;
  }
}
```
从代码可以看出，我们先设置了一张自定义背景repeat-white.png。是的，这里出现了一个比较陌生的属性：`-webkit-background-clip`,即指定对象的背景图像向外裁剪的区域，那么`-webkit-background-clip: text`就是按文字的区域裁剪。这样我们就得到文字的区域了，然后把文字设置成透明的，我这里用`-webkit-text-fill-color: transparent`，其实用`color: transparent`也可以，最后我们看到文字就是由图片填充的。如果我们想给文字加点动画什么的，可以利用动画animation设置背景的position。

## 注意

* 注意`-webkit-background-clip:text`兼容性，目前网上的说法参差不齐，`caniuse`也没有给出兼容性说明。
* `background`一定要写在`-webkit-background-clip`前面，否则无效。